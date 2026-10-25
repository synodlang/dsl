# Site and organization

## `synodlang.github.io`

The organization site. It is a static site (no server), built by a Go
standard-library program and deployed with GitHub Actions to GitHub Pages for
the organization at https://synodlang.github.io until a domain exists
(decision 106).

### Build

- `deps.json` pins, per docs version (one entry per MINOR), the `dsl`, `std`
  and `examples` tags with their commits, and the compiler and player release
  tags with the SHA-256 of each asset used (decision 106).
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
- JetBrains Mono (Regular and Bold, OFL) is committed and self-hosted; the
  site loads no font service. The player's labels use the system sans.
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
  serves them itself. The player bundles no fonts: the site serves JetBrains
  Mono itself, and the home loop plays with `loop: true`. An example page passes the examples module as `files` with
  `program` set to the example's folder; a snippet that imports the
  examples carries them under `vendor/`.
- **Tribute:** a short page on Leslie Lamport's "The Part-Time Parliament"
  and why the language is named after the Synod protocol.

Design: the brand from `assets/icon.svg`: navy ground `#151A2E`, pale
`#EDE9E1`, amber accent `#F4A340`, JetBrains Mono for the wordmark and code.

## `.github`

- `profile/README.md`: what Synod is, links to every repository. It lists `player` and the site (follow-up in that repo).
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
