# Contributing to Git Profile Age

Thanks for helping. The project is intentionally small: one static `index.html`, vanilla JavaScript, no build step, no dependencies, no backend. Please keep it that way.

## Ground rules

- Vanilla HTML, CSS and JavaScript only. No frameworks, bundlers, package managers or third-party scripts.
- No analytics, tracking, cookies or external requests other than the Git service APIs.
- No API keys or secrets. Everything must work from an anonymous browser session.
- Never show raw technical errors to the user; map them to the existing `LookupError` codes.

## Adding a Git hosting service

1. **Verify the API from a browser, not just with curl.** The endpoint must return the account creation date without authentication, send `Access-Control-Allow-Origin`, and not answer browser requests with an anti-bot challenge or a sign-in redirect. A quick first check:

   ```bash
   curl -s -D - -o body.json \
     -H "Origin: https://example.org" \
     -A "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36" \
     https://host/api/...
   ```

   Then open `index.html?url=https://host/someuser` in a real browser and confirm the result. Several self-hosted GitLab and Gitea instances pass curl and fail in browsers.

2. **Register an adapter** in `index.html` next to the existing ones:
   - `defineService({...})` for a new kind of service, or
   - `gitlabService({...})` / `forgejoService({...})` for another GitLab or Gitea/Forgejo instance.

   Fill in `hosts`, `parse` (reject the service's reserved top-level paths such as `/explore`), `profileUrl`, `manualUrl`, `apiUrl` + `parseResponse` or a custom `lookup`, `support` (`auto`, `partial`, `manual`) and `summary`.

3. **If the API is not usable from a browser**, register the service with `support: 'manual'` and a `manualReason`. Users still get detection and a link to the profile page. Do not ship a lookup that works only sometimes.

4. **Test** at least: a profile URL, a repository URL, a non-existent profile and a reserved path. Add the service to the table in `README.md`. The in-page table is generated from the adapters.

## Testing

There is no test runner. Serve the directory and use the browser:

```bash
python3 -m http.server 8000
```

`window.GitProfileAge` exposes `check(url)`, `analyzeInput(url)`, `formatAge()`, `formatCreated()` and `services` in the console.

## Security

If you find a vulnerability, please do not open a public issue. Use [GitHub private vulnerability reporting](https://github.com/doutivity/git-profile-age/security/advisories/new) or contact the author via [LinkedIn](https://www.linkedin.com/in/yaroslav-podorvanov/). The site has no backend and stores nothing server-side, so most findings will concern the handling of user-supplied URLs or API responses in the browser.

## Pull requests

Keep changes focused. Describe which services you tested and how. Update `README.md` when behaviour changes.
