# CLAUDE.md

BasharSoft fork of the open-source apiDoc generator (`0.17.6`, MIT). Node.js CLI that reads `@api...` comments from source files and writes a static HTML docs site (Handlebars + Bootstrap 3 + jQuery 2.2.1 template).
For Wuzzuf it builds the **old API docs of wuzzuf-platform** (`public/docs`). The only visible BasharSoft change: `apidoc-core` is pulled from `BasharSoft/apidoc-core.git` instead of npm.
Live branch: `master`. 2017-era stack (Node >= 0.10 engines, Travis on Node 6).

## Commands

```bash
npm install
npm test                 # jshint lib/ test/ + mocha test/
npm run jshint           # lint only (rules in package.json "jshintConfig")
npx mocha test/apidoc_test.js     # one test file
npm run build-example    # bin/apidoc -i example/ -o tmp/
bin/apidoc -i <src> -o <out> -f ".*Controller.php"   # run the CLI
```

`Dockerfile` installs the **upstream** npm `apidoc` on `node:6-alpine`; it does not build this fork.

## Architecture

| Path | What |
|---|---|
| `bin/apidoc` | CLI entry; parses options with `nomnom` (`-i`, `-o`, `-f`, `-e`, `-t`, `-c`, `--private`, `--parse-*`) |
| `lib/index.js` | `createDoc()`: calls `apidoc-core` to find files, parse comments and run workers; `createOutputFiles()` copies `template/` and writes `api_data.js(on)`, `api_project.js(on)` |
| `lib/package_info.js` | `PackageInfo`: reads `apidoc.json` or the `apidoc` key in `package.json`, header/footer markdown |
| `template/` | Browser site: `main.js` (render, version compare, `renderArticle`), `utils/send_sample_request.js`, `locales/`, `vendor/` |
| `test/` | mocha + should: `apidoc_test.js`, `custom_markdown_parser_test.js`, `multi_input_folder_test.js`, fixtures |
| `example/` | Sample input for `build-example` |

Flow: `bin/apidoc` -> `lib/index.js createDoc()` -> `apidoc-core` parse -> write JSON + copy `template/` -> browser renders with Handlebars.

## Conventions

- Parsing logic belongs in `apidoc-core`; this repo only does CLI, config and output.
- Template changes go in `template/`; generated output (`api_data.js`, `api_project.js`) is never committed.
- Lint with the `jshintConfig` in `package.json` (camelcase, eqeqeq, undef, unused). Tests compare generated output to `test/fixtures`.

## Gotchas

- wuzzuf-platform's `yarn.lock` (used by its Dockerfile) resolves `apidoc` to a personal fork `SherifMedhat/apidoc#2ef6647` over `git://`, not this repo. Its `package-lock.json` pins this repo at `f95145d`. Changing this repo doesn't change the production build until that lock is fixed.
- `git://` GitHub URLs no longer work, so a clean `yarn install` in wuzzuf-platform is expected to fail without a cache.
- `package.json` pulls `apidoc-core` from a git URL without a commit, so a fresh install takes the default branch.
- Deprecated deps: `nomnom`, `winston` 2, `markdown-it` 8; `template/vendor/` ships jQuery 2.2.1.
- No CI runs the tests for this fork (`.travis.yml` targets upstream).
- The template's "send sample request" form calls the API from the browser if `public/docs` is served.

## Connections

- Depends on [apidoc-core](https://github.com/BasharSoft/apidoc-core/blob/master/docs/wiki/README.md) (npm git dependency): the parser.
- Consumer: [wuzzuf-platform](https://github.com/BasharSoft/wuzzuf-platform/blob/master/docs/wiki/README.md) (npm dev dependency). Run by `gulp old-api-doc` (`gulpfile.js`) or `php artisan apidoc` on `app/Http/Controllers`, `app/Representations`, `apiDoc/EntitiesDocs`, config in `apiDoc/config/`, output `public/docs`.
- [basharjobs](https://github.com/BasharSoft/basharjobs/blob/production_wuzzuf/docs/wiki/README.md) vendors its own older apidoc `0.13.1`; it doesn't use this repo.

## Deploy

Not deployed or published to npm. Consumers install it from `https://github.com/BasharSoft/apidoc.git` at a commit pinned in their lock file; push to `master`, then update the lock in wuzzuf-platform.

## More

- Wiki: [docs/wiki/README.md](docs/wiki/README.md); [usage-in-wuzzuf.md](docs/wiki/usage-in-wuzzuf.md), [known-issues.md](docs/wiki/known-issues.md)
- Code map: [docs/wiki/code-map/index.md](docs/wiki/code-map/index.md)
- Graph: `graphify explain "createDoc" --graph ../graphify/repos/apidoc/graph.json`
