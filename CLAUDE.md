# Git Profile Age

## Task

Build and finish the Git Profile Age web tool.

Repository:

https://github.com/doutivity/git-profile-age

The goal is to create a privacy-friendly, recruiter-oriented static website that checks when a developer's Git profile was created.

The user enters a Git profile URL. The application automatically detects the Git hosting service, queries its public API when possible, and displays the profile creation date and the profile's age.

The application must work entirely in the browser.

## Important constraints

* Vanilla JavaScript only.
* Plain HTML and CSS.
* No frontend frameworks.
* No backend.
* No database.
* No authentication.
* No API keys or secrets.
* No analytics.
* No tracking.
* No advertising trackers.
* No unnecessary external dependencies.
* Prefer a single `index.html` containing HTML, CSS and JavaScript.
* Do not introduce a build system unless there is a compelling technical reason.
* Do not add unnecessary dependencies.

## Primary requirements

### Git services

Automatically detect the Git hosting service from the submitted URL.

Do not limit support to GitHub, GitLab and Bitbucket.

Research major Git hosting services and their public APIs and support as many services as can be reliably implemented with a frontend-only architecture.

The architecture must make adding another service easy.

Use a service-adapter approach with each service defining:

* hostname detection
* profile URL parsing
* URL normalization
* API endpoint
* API response parsing
* creation date extraction
* manual fallback URL
* service name

If a service cannot reliably be supported from a pure frontend, do not create a fake or unreliable implementation.

### URL input

Support common valid forms such as:

* `https://github.com/user`
* `github.com/user`
* `https://github.com/user/`

Automatically normalize URLs.

Handle unnecessary trailing slashes, query parameters and fragments appropriately.

If a repository URL can safely be converted to the owner's profile URL, do so automatically.

Reject URLs that cannot be reliably interpreted as Git profiles.

### Unsupported services

If the service is not supported, clearly tell the user that it is not currently supported.

Provide a contact link:

https://www.linkedin.com/in/yaroslav-podorvanov/

The purpose is to allow users to request support for additional services.

### API

Use public APIs without authentication whenever possible.

Handle:

* success
* 404
* 403
* 429
* rate limits
* network errors
* timeouts
* malformed responses
* unexpected API responses
* CORS limitations

Never expose raw technical errors in the main UI.

If automatic lookup is impossible, provide a useful manual fallback link where possible.

### Result

Show:

* Git service
* normalized profile URL
* original creation date/time
* human-readable profile age

For example:

Profile created

January 15, 2018 at 14:32 UTC

8 years, 7 months ago

The exact timestamp returned by the API must remain visible.

The relative age must be calculated dynamically.

### Sharing

Use:

`?url=...`

for the profile URL.

When the page is opened with a `url` query parameter:

1. read it
2. decode it
3. validate it
4. normalize it
5. automatically perform the lookup

Do not require the user to press Submit.

After a normal search, update the browser URL without reloading the page.

Implement sharing using:

1. Web Share API
2. Clipboard API fallback
3. another reasonable fallback if both are unavailable

### Caching

Cache results locally using the normalized profile URL as the cache key.

Try available browser storage mechanisms and gracefully fall back if a mechanism is unavailable.

Prefer persistent storage when possible.

A cache failure must never break the application.

Use a sensible TTL.

Do not store unnecessary information.

### Privacy

The application must not collect user data.

There must be:

* no analytics
* no tracking
* no third-party analytics scripts
* no tracking pixels
* no user accounts
* no project backend
* no project-side logging
* no cookies for analytics/tracking

The profile URL should only be processed locally and sent to the corresponding Git service API when required.

The website should explicitly state that no personal data is collected and no analytics or tracking are used.

Do not claim that external Git services do not log requests.

## UX

The target audience is recruiters and hiring managers.

The UI should be:

* minimal
* professional
* fast
* easy to understand
* mobile-friendly
* responsive

The main interface should focus on the profile URL input and result.

Support:

* loading state
* success state
* invalid URL
* unsupported service
* profile not found
* API error
* rate limit
* network error
* manual fallback

Use clear human-readable messages.

## Accessibility

Use:

* semantic HTML
* proper labels
* keyboard navigation
* Enter-to-submit
* visible focus states
* accessible buttons
* ARIA only where useful
* `aria-live` for dynamic results/errors
* sufficient contrast

Do not communicate important information using color alone.

## SEO

Optimize for Ukrainian and English-speaking recruiters.

Include natural SEO content around:

* GitHub profile creation date
* GitHub account age
* GitLab profile creation date
* Bitbucket account age
* Git profile checker
* developer profile verification
* developer experience verification
* дата створення GitHub профілю
* вік GitHub акаунта
* перевірка профілю розробника

Do not keyword-stuff.

Include:

* title
* meta description
* semantic headings
* Open Graph metadata

## Project files

Prefer:

```text
index.html
README.md
CLAUDE.md
favicon.svg
```

Keep the project minimal.

Create a custom favicon related to Git/profile/date/age.

The website must contain a subtle link to:

https://github.com/doutivity/git-profile-age

Prefer placing it in the footer.

## README

Generate a proper `README.md` after implementing the application.

It should document:

* what Git Profile Age does
* how to use it
* supported services
* API architecture
* caching
* sharing
* privacy
* no analytics/tracking
* frontend-only architecture
* local development
* deployment
* adding new Git services
* limitations

The README must describe the actual implementation rather than planned features.

## Naming

Before implementation, propose at least 15 possible product names.

Select the best three and then choose the strongest final name.

The current working project/repository name is:

`git-profile-age`

Do not rename the repository unless explicitly requested.

## Implementation workflow

Do not immediately start coding blindly.

First:

1. Inspect the repository.
2. Inspect existing files.
3. Research public APIs for Git hosting services.
4. Determine which services can actually work from a frontend-only application.
5. Determine API/CORS limitations.
6. Propose possible product names.
7. Design the service-adapter architecture.
8. Implement the application.
9. Test URL normalization.
10. Test API success/error states.
11. Test caching.
12. Test query-parameter auto-search.
13. Test sharing.
14. Test responsive behavior.
15. Review accessibility.
16. Review SEO.
17. Review privacy.
18. Generate/update README.
19. Verify favicon.
20. Review the final repository for unnecessary dependencies or files.

## Definition of Done

The task is complete only when:

* the application is functional
* Git service detection works
* supported public APIs work
* unsupported services have a useful fallback
* URL normalization works
* repository URLs are handled where safely possible
* `?url=` sharing works
* automatic lookup from `?url=` works
* sharing works
* caching works
* storage fallback works
* loading and error states work
* the UI is responsive
* accessibility requirements are addressed
* SEO metadata exists
* favicon exists
* GitHub repository link exists
* privacy statement exists
* there is no analytics/tracking
* there are no secrets
* README accurately documents the implementation
* the project remains frontend-only
* the project remains Vanilla JS/HTML/CSS
* the repository is clean and ready for continued development with Claude Code

Do not consider the task complete merely because the page renders. Verify the actual behavior and review the implementation against every requirement above.CLAUDE.md 
