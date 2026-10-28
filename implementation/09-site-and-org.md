# Site and organization

## `synodlang.github.io`

The organization site. It is a static site (no server), built by a Go
standard-library program and deployed with GitHub Actions to GitHub Pages for
the organization at https://synodlang.github.io until a domain exists
(decision 106).

### Build

- `deps.json` pins, per docs version (one entry per MINOR), the `dsl`, `std`,
  `examples` and (from v0.3) `looks` tags with their commits, and the compiler
  and player release tags with the SHA-256 of each asset used, the player's
  fonts asset included (decision 106). The v0.3 entry is `/docs/v0.3/` and
  `latest` is `v0.3`; `/docs/v0.2/` stays as built.
- `go run ./tools/build fetch` downloads the assets, checks each against the
  release `SHA256SUMS` and the pin, extracts the native `synod` (the host's
  own: linux-amd64 in CI, darwin-universal locally), checks that `synod
  version` equals the pin, and checks the repositories out byte-exact at the
  pinned commits with the git settings of decision 95. `build` writes
  `_site/`, `check` verifies it (links, anchors, page counts, hashes, no
  recordings) and `serve` serves it on 127.0.0.1. `-local` uses sibling
  checkouts and marks pages as a local build.
- Every `synod` block, example and step is checked or run natively during the
  build; an unexpected exit code fails it.
- A workflow builds, checks and deploys with GitHub Actions on every push to
  `main`; the pull-request workflow also runs the browser tests (Playwright,
  used for tests only). The owner switches Pages from the legacy branch build
  to Actions once.
- The site self-hosts the four families of the font set (decision 118):
  Inter, JetBrains Mono, Source Serif 4 and Shantell Sans, each SIL OFL 1.1,
  as variable woff2 files with their `OFL-<family>.txt`, in `web/fonts/`. The
  build copies them from the verified fonts asset of the pinned `player`
  release (`synod-player-fonts-<tag>.tar.gz`), so the page text and the player
  draw the same glyphs; the site loads no font service. Every player on the
  site is mounted with `fontBase` set to that folder.
- Docs live at `/docs/v<MAJOR>.<MINOR>/` and `/docs/` leads to the latest
  (decision 107). The site highlights code from `synod-check/1` tokens only
  (decision 108), and a Run calls `synod.wasm` in a Web Worker (decision 109).

- **Home:** the icon, one sentence ("A small language for watching
  distributed systems think"), a looping Raft election played live by
  `player` (no recording): the build makes its scene with the native pinned
  `synod run` of `examples/raft`, so the first page loads no wasm (decision 110), three buttons: Start the tutorial · Install · See examples.
- **Install:** `/install/`, one self-contained page: the visitor's OS
  first, then macOS, Linux, Windows and Go. Each shows the package-manager
  command, the `install.sh` command (macOS, Linux) and direct download
  links filled from the latest GitHub release, with a link to Releases as
  fallback. It notes the unsigned binaries and `synod version`. The script
  itself is served at the site root: `/install.sh`.
- **Docs:** the tutorial, the language reference, the library, guides
  and error pages, built from `dsl` and `std` at build time. Structure,
  sources and the tutorial plan are in `11-documentation.md`. From M11 the
  build also downloads the native `synod` of the pinned compiler release and
  runs `synod doc --json` and `synod explain --json` for the Library and Errors
  pages; the wasm API is unchanged.
- **Examples:** one page per example with editable source and a Run
  button. It calls `synod.wasm` (check, run) and plays the result
  with `player`; diagnostics are shown inline. No recordings. The site
  build downloads the wasm pair of a pinned compiler release, verifies it
  against `SHA256SUMS` and serves it from the site; the module runs in a
  Web Worker. It likewise downloads the assets of a pinned `player` release
  (ESM bundle, IIFE bundle, `.d.ts`), verifies them against `SHA256SUMS` and
  serves them itself, with the fonts asset above; the home loop plays with
  `loop: true` and starts after `player.fontsReady()` resolves. An example page passes the examples module as `files` with
  `program` set to the example's folder; a snippet that imports the
  examples carries them under `vendor/`.
- **Looks & Themes (M12):** `/docs/v0.3/looks/`, linked from the Library. One
  section per package of `github.com/synodlang/looks` (looks `lumen` and
  `doodle`; themes `solarized/light`, `solarized/dark`, `terminal`, `sketch`
  and `chalk`) and per std package `std/look/basic`, `std/theme` and
  `std/theme/default`, each with its source, its doc comments and an
  editable `demo.synod` with a Run button. A **gallery** shows the matrix of
  every look × every theme: the build makes each cell's scene with the native
  pinned `synod run` of the look's `demo.synod` under that theme (the demo
  main file imports the look and the theme), so the gallery loads no wasm,
  and every cell plays with `player` (`loop: true`). A Run on a demo passes
  the looks module as `files` with `program` set to the demo's folder, and a
  snippet that imports `looks` carries it under
  `vendor/github.com/synodlang/looks@<ver>/` (decision 71). `synod.wasm`
  embeds only std, so the page never fetches modules. The pairings it names
  are `lumen` + default, `doodle` + `sketch` and `lumen` + `chalk`.
- **Tribute:** a short page on Leslie Lamport's "The Part-Time Parliament"
  and why the language is named after the Synod protocol. It credits Manim
  (MIT) and Solarized (MIT) by name as inspiration only; no look, theme or font
  uses another project's name or marks.

Design: the brand from `assets/icon.svg`: navy ground `#151A2E`, pale
`#EDE9E1`, amber accent `#F4A340`, JetBrains Mono for the wordmark and code
(the colors are the values of `std/theme/default`).

## `.github`

- `profile/README.md`: what Synod is, links to every repository. It lists `player`, `looks` (looks and themes beyond the defaults, decision 122) and the site (follow-up in that repo).
- Shared workflows: `go.yml` (vet, test, golden tests), `synod.yml`
  (install `synod`, run `synod fmt --check` and `synod check ./...`). The
  `agent-skills` sync workflow lives in `agent-skills` itself, not here
  (decision 97).
- `CONTRIBUTING.md`: changes to the language start as a pull request to
  `dsl` that updates the spec and `decisions.md`, together with matching
  pull requests to `examples` and the tutorial.
- Issue templates: language proposal, bug, example request.

## Licences

Code under Apache-2.0; documentation and examples under CC BY 4.0.
