# Usage in Wuzzuf

## Where it runs

`wuzzuf-platform/scripts/Dockerfile:1-11` (builder stage on a `node10` image):

```dockerfile
RUN yarn install
COPY . .
RUN yarn run gulp api-doc && \
  yarn run gulp old-api-doc
```

`old-api-doc` (`wuzzuf-platform/gulpfile.js:15-28`) runs, from `apiDoc/config/`:

```
node_modules/.bin/apidoc \
  -i app/Http/Controllers/ -i app/Representations/ -i apiDoc/EntitiesDocs/ \
  -o public/docs \
  -f ".*Controller.php" -f ".*Schema.php" -f ".*.js"
```

The same command can be run as `php artisan apidoc` (`wuzzuf-platform/app/Console/Commands/ApiDoc.php`), with paths from `config/console.php` / env `APIDOC_INPUT_PATH`, `APIDOC_OUTPUT_PATH`, `APIDOC_CONFIG_PATH`, `APIDOC_FILE_FILTER`.

Project settings come from `wuzzuf-platform/package.json` (`"apidoc": {"title": "Wuzzuf API documentation", "url": "https://api.wuzzuf.net/v1"}`) and `apiDoc/config/apidoc.json`, `_apidoc.js`, `header.md`, `footer.md`.

The output lands in `public/docs`, inside the web root of the API, so it is served with the app (inferred; the welcome route in `routes/web.php:14` mentions `/docs` in a comment).

## Which version is installed

| File in `wuzzuf-platform` | Resolves `apidoc` to | Resolves `apidoc-core` to |
|---|---|---|
| `package.json` | `https://github.com/BasharSoft/apidoc.git` (no commit) | via apidoc's `package.json` |
| `package-lock.json` | `BasharSoft/apidoc.git#f95145d` (this snapshot) | `BasharSoft/apidoc-core.git#27a8561` (the `apidoc-core` snapshot) |
| `yarn.lock` (used by the Dockerfile) | `git://github.com/SherifMedhat/apidoc.git#2ef6647` | `git://github.com/SherifMedhat/apidoc-core.git#27a8561` |

So the production build installs from the personal fork in `yarn.lock`, not from this repo. `apidoc-core` is the same commit either way.

## How the tool works

| Step | Code |
|---|---|
| Parse CLI options (`-i`, `-o`, `-f`, `-e`, `-t`, `-c`, `--private`, `--parse-*` plugins, ...) | `bin/apidoc` (uses `nomnom`) |
| Read `apidoc.json` or the `apidoc` key in `package.json`, header/footer markdown | `lib/package_info.js` |
| Call `apidoc-core` to find files, parse comments, run workers, and return `data` + `project` JSON | `lib/index.js` |
| Copy `template/` to the output and write `api_data.js(on)`, `api_project.js(on)` | `lib/index.js` |
| Render in the browser with Handlebars, show versions/diffs, optional "send sample request" form | `template/main.js`, `template/utils/send_sample_request.js` |

Tests (`test/`, mocha + should) compare generated output to fixtures. No CI runs them for this fork (`.travis.yml` targets Node 6 and upstream).

`Dockerfile` in this repo installs the **upstream npm** `apidoc` globally on `node:6-alpine`; it does not build this fork.
