# Git Profile Age

Check when a GitHub, GitLab, Bitbucket, Codeberg or other Git profile was created.

Git Profile Age is a privacy-friendly, recruiter-oriented static web tool. Paste a Git profile URL, and the page detects the hosting service, queries its public API directly from your browser and shows the exact account creation timestamp together with a human-readable profile age (for example "8 years, 7 months ago").

Everything runs in the browser. There is no backend, no database, no API key, no analytics and no tracking. The site is one `index.html` file with plain HTML, CSS and vanilla JavaScript.

## How to use

1. Open the page.
2. Paste a profile link such as `https://github.com/torvalds`. Shorter forms like `github.com/torvalds`, repository links like `github.com/torvalds/linux` and SSH clone URLs like `git@github.com:torvalds/linux.git` work too; repository URLs are converted to the owner's profile.
3. Press Enter or click **Check profile**. On browsers that allow reading the clipboard, a **Paste** button inside the empty field pastes the copied link and checks it in one tap; once the field has text it turns into a **Clear** button (Escape does the same).
4. Read the result: the Git service, the normalized profile URL, the creation date in UTC, the raw timestamp returned by the API and the calculated profile age.
5. Click **Share result** to send a link that reproduces the same check.

You can also open the page with a `url` query parameter, for example `index.html?url=https%3A%2F%2Fgithub.com%2Ftorvalds`. The page reads, decodes, validates and normalizes the value and runs the lookup automatically.

## Supported services

Every service was verified end-to-end from a browser (public API, CORS headers, no bot challenge for browser requests). The `Support` column reflects what actually works without authentication.

| Service | Hosts | Support | What the API returns |
| --- | --- | --- | --- |
| GitHub | `github.com`, `www.github.com`, `gist.github.com` | Automatic | Users and organizations: exact timestamp (`created_at`). 60 anonymous requests per hour per network. |
| GitLab | `gitlab.com` | Partial | Groups: exact timestamp (`created_at`). Personal accounts: gitlab.com hides the date from anonymous requests, so the tool verifies the user exists and links to the profile, which shows "Member since". |
| Bitbucket | `bitbucket.org` | Automatic | Workspace creation date (`created_on`). Workspaces were introduced on 29 November 2018, so older accounts show that migration date. The UI says so. |
| Codeberg | `codeberg.org` | Automatic | Users and organizations: exact timestamp (`created`). |
| Gitea.com | `gitea.com` | Automatic | Users and organizations: exact timestamp (`created`). |
| Gitee | `gitee.com` | Partial | Users: exact timestamp. Organizations: verified and linked; the API does not include their creation date. |
| GitCode | `gitcode.com` | Automatic | Users and organizations: exact timestamp (`created_at`). |
| Hugging Face | `huggingface.co`, `hf.co` | Partial | Users: exact timestamp (`createdAt`). Organizations: verified and linked. Dataset and Space URLs resolve to their owner. |
| Framagit | `framagit.org` | Partial | Self-hosted GitLab, same behaviour as gitlab.com. |
| Tor Project GitLab | `gitlab.torproject.org` | Partial | Self-hosted GitLab, same behaviour as gitlab.com. |
| FSFE Git | `git.fsfe.org` | Automatic | Gitea instance, same behaviour as Gitea.com. |
| SourceForge | `sourceforge.net/u/<name>` | Manual | Detected and linked. The REST API returns the creation date, but SourceForge's bot protection answers browser requests with a challenge page. |
| Launchpad | `launchpad.net/~<name>` | Manual | Detected and linked. The Launchpad API does not send CORS headers, so browsers cannot read it. |
| SourceHut | `sr.ht`, `git.sr.ht`, `hg.sr.ht`, … | Manual | Detected and linked. The API requires authentication. |
| Azure DevOps | `dev.azure.com/<org>` | Manual | Detected and linked. No anonymous API access. |
| NotABug | `notabug.org` | Manual | Detected and linked. The Gogs API does not include the creation date. |

Any other hostname is reported as unsupported with a link to request support: <https://www.linkedin.com/in/yaroslav-podorvanov/>.

Services that were evaluated and deliberately left out because they do not work reliably from a static page: Debian Salsa, KDE Invent, GNOME GitLab, freedesktop.org GitLab, Haskell GitLab, VideoLAN, Xfce GitLab, Heptapod, Arch Linux GitLab and 0xacab (all answer browser requests with anti-bot challenges or sign-in redirects), git.disroot.org and opendev.org (Gitea without CORS), Pagure (no CORS, unstable API), Gogs instances (no creation date in the API).

## Architecture

The application lives in `index.html`; the other files in the repository are static supporting files (see Deployment). `index.html` contains:

- HTML with semantic sections (form, live result region, supported-services table, SEO content in English and Ukrainian, privacy statement, footer), Open Graph and Twitter card metadata, and JSON-LD structured data (`WebApplication` and `FAQPage`).
- CSS with system fonts, light and dark colour schemes (`prefers-color-scheme`), visible focus styles and a responsive layout down to 320 px.
- One vanilla JavaScript IIFE, ES5-compatible syntax, no dependencies.

### Service adapters

Each Git hosting service is a plain object registered with `defineService()`:

| Field | Purpose |
| --- | --- |
| `id`, `name` | Identity and display name |
| `hosts` | Lowercase hostnames that identify the service |
| `parse(segments)` | Path segments of the URL → profile handle, or `null` when the URL is not a profile or repository page (reserved paths such as `/explore` or `/settings` are rejected) |
| `profileUrl(handle)` | Normalized profile URL, also the cache key and the value written into `?url=` |
| `manualUrl(handle)` | Where the user can look manually |
| `apiUrl(handle)` + `parseResponse(json)` | Public API endpoint and the mapping to `{ createdAt, kind, precision?, note? }` |
| `lookup(handle)` | Optional custom multi-step lookup (GitLab tries users then groups, Gitee/GitCode/Hugging Face try users then organizations) |
| `support` | `auto`, `partial` or `manual` |
| `summary`, `manualReason` | Text for the supported-services table and for manual-only results |

`gitlabService()` and `forgejoService()` are factories for GitLab-based and Gitea/Forgejo-based instances.

### Request handling

`apiGet()` is the single HTTP helper. It uses `fetch` with a 10-second timeout via `AbortController`, `credentials: 'omit'` and `referrerPolicy: 'no-referrer'`, and converts every outcome into a `LookupError` code:

| Code | Trigger | UI |
| --- | --- | --- |
| `not_found` | 404 / 410 | "Profile not found" with a link to the page |
| `rate_limited` | 429, or 403 with `X-RateLimit-Remaining: 0` / "rate limit" in the body | Explains the limit and shows the reset time when the API sends one |
| `forbidden` | 401 / 403 | "Refused the request" (private, restricted or deactivated) |
| `network` | fetch failed (offline, CORS, blocked) | "Could not reach …" with a retry button |
| `timeout` | no answer in 10 s | Retry |
| `server_error` | 5xx | Retry |
| `malformed` | body is not JSON or the date is missing/invalid | Generic message with a manual link |
| `date_unavailable` | profile exists but the service hides the date | Warning with a manual link |
| `manual_only` | service has no usable public API | Warning with a manual link |

Raw technical errors are never shown to the user.

### URL normalization

`analyzeInput()` trims the input, converts SSH URLs (`git@host:owner/repo.git`, `ssh://git@host/...`) to HTTPS, adds `https://` when the scheme is missing, upgrades `http://` to `https://`, lowercases the host, drops query strings, fragments, trailing slashes and `.git` suffixes, resolves `www.` aliases, and hands the path segments to the adapter. The result is one of `invalid` (empty, not a URL, or a service page that is not a profile), `unsupported` (unknown host) or `ok` (service, handle, normalized profile URL).

## Caching

Successful results are cached with the normalized profile URL as the key (`git-profile-age:v1:<profile URL>`). Storage is chosen at start-up: `localStorage`, then `sessionStorage`, then an in-memory map. Every read and write is wrapped in try/catch; a failing cache never breaks a lookup. Entries expire after 7 days and expired entries are pruned on page load. Only the creation timestamp, precision, profile kind, the service note and the fetch time are stored. Errors and "not found" results are not cached. A cached result is labelled as such and offers **Check again**, which bypasses and refreshes the cache.

## Sharing

Every result has a **Share result** button. The shareable link is `<page URL>?url=<encoded normalized profile URL>`. The button tries, in order:

1. the Web Share API (`navigator.share`),
2. the Clipboard API (`navigator.clipboard.writeText`, with a 2-second guard in case the browser leaves the permission prompt pending),
3. a read-only text field with the link pre-selected plus a **Copy** button (`document.execCommand('copy')` where available, otherwise a hint to press Ctrl+C).

After each lookup the address bar is updated with `history.replaceState` so the current URL is always shareable.

## Privacy

- No analytics, tracking scripts, tracking pixels, cookies, accounts, backend or project-side logging.
- The profile URL is processed locally and sent only to the public API of the Git service it belongs to. Requests are made without credentials and without a Referer header.
- Results are cached in the visitor's own browser storage only.
- The page states this explicitly and does not claim that the external Git services do not log API requests; their privacy policies apply to those requests.

## Local development

No build step. Either open `index.html` directly, or serve the directory so that `?url=` links and `history.replaceState` behave as they will in production:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/
```

`window.GitProfileAge` exposes `check(url)`, `analyzeInput(url)`, `formatAge()`, `formatCreated()` and the `services` list for manual testing in the browser console.

## Deployment

The repository deploys to GitHub Pages through `.github/workflows/deploy-pages.yml` on every push to `main` (GitHub Actions source; enable **Settings → Pages → Source: GitHub Actions** once). `.nojekyll` disables Jekyll processing so `.well-known/` and other dotfiles are served as they are.

Any other static host (Netlify, Cloudflare Pages, a plain web server) works too. Files that make up the site:

| File | Purpose |
| --- | --- |
| `index.html` | The application |
| `favicon.svg` | Icon |
| `og-image.png` | 1200×630 preview for Open Graph / Twitter cards |
| `site.webmanifest` | Web app manifest (name, icon, theme colour) |
| `robots.txt`, `sitemap.xml` | Crawler hints |
| `humans.txt` | Credits and technology list |
| `.well-known/security.txt` | Security contact (RFC 9116) |
| `404.html` | Not-found page served by GitHub Pages |

If the site is served from a different address than `https://doutivity.github.io/git-profile-age/`, update the absolute URLs in the `<head>` of `index.html` (`canonical`, `og:url`, `og:image`, `twitter:image`, JSON-LD), the `Sitemap:` line in `robots.txt`, the `<loc>` in `sitemap.xml` and the `Canonical:` line in `.well-known/security.txt`.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for ground rules, the service-verification checklist and how to report security issues.

## Adding a new Git service

1. Verify from a browser that the service's public API returns the account creation date without authentication and with `Access-Control-Allow-Origin` headers, and that it does not answer browser requests with an anti-bot challenge. `curl -H "Origin: https://example.org" -A "<a Chrome user agent>" <api url>` is a good first check; a headless-browser test is the final word.
2. Add a `defineService({...})` block (or a `gitlabService()` / `forgejoService()` call for GitLab or Gitea/Forgejo instances) in `index.html`, with `hosts`, `parse`, `profileUrl`, `manualUrl`, `apiUrl` + `parseResponse` (or `lookup`), `support` and `summary`. Include the service's reserved top-level paths in `parse` so pages like `/explore` are not mistaken for profiles.
3. If the API cannot be used from a browser, register the service with `support: 'manual'` and a `manualReason` so users still get detection and a link instead of a misleading result.
4. Add the service to the table above. The in-page table is generated from the adapters automatically.

## Limitations

- Profile age is a weak signal on its own: developers work in private repositories, switch accounts, or join a service late.
- GitLab.com does not expose creation dates of personal accounts to anonymous API requests; only groups are dated.
- Bitbucket reports the workspace creation date, which for accounts older than 29 November 2018 is the migration date, not the signup date.
- Anonymous API rate limits apply per network (GitHub: 60 requests per hour). Results are cached to reduce repeated calls.
- Self-hosted instances are supported only when explicitly listed; there is no reliable way to detect an arbitrary GitLab or Gitea host from the URL alone.
- Services behind anti-bot challenges or without CORS cannot be queried from a static page and are offered as manual checks.

## License

MIT, see [LICENSE](LICENSE).
