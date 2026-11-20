# Known issues

These were found while writing this wiki from the code, as of `master` at `f95145d`. Items marked ✓ were checked by opening the exact line.

**No secrets are committed** in this repo.

## Supply chain

| # | Issue | Where | Impact |
|---|---|---|---|
| 1 | ✓ **The production build installs a personal fork** | `wuzzuf-platform/yarn.lock:72-90`, `wuzzuf-platform/scripts/Dockerfile:6-8` | `yarn install` follows `yarn.lock`, which points to `SherifMedhat/apidoc` and `SherifMedhat/apidoc-core`, not the BasharSoft repos. Code outside the company's control runs in the image build |
| 2 | ✓ **`git://` URLs** | `wuzzuf-platform/yarn.lock:72-85` | GitHub turned off the unauthenticated `git://` protocol in 2022, so a clean `yarn install` is expected to fail unless a cache or mirror is used (inferred) |
| 3 | ✓ **Branch, not tag, in `package.json`** | `apidoc/package.json:37` (`apidoc-core` from a git URL without commit) | A fresh `npm install` without the lock file takes whatever is on the default branch |
| 4 | ✓ **Lock files disagree** | `wuzzuf-platform/package-lock.json:110-132` vs `yarn.lock` | npm installs this repo (`f95145d`), yarn installs the personal fork (`2ef6647`) |

## Maintenance

- **Old stack:** Node `>= 0.10` engines, Travis on Node 6, `nomnom` (deprecated), `winston` 2, `markdown-it` 8; the template ships **jQuery 2.2.1** and Bootstrap 3-era assets (`template/vendor/`).
- **Superseded:** `wuzzuf-platform` also builds a Swagger/OpenAPI page (`gulp api-doc`, `public/api-docs`); the gulp task for this tool is literally named `old-api-doc`.
- **Generated docs are public-facing** if `public/docs` is served on the API host (inferred). The template includes a "send sample request" form that calls the API from the browser.
- **No fork diff is visible.** Without history it is not possible to tell from code what (if anything) was changed besides the `apidoc-core` source. Upstream apiDoc could likely replace it.
