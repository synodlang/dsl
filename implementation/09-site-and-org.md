# Site and organization

## `synodlang.github.io`

The organization site. It is a static site
(Astro or plain HTML; no server), deployed with GitHub Pages for the
organization at https://synodlang.github.io until a domain exists.

- **Home:** the icon, one sentence ("A small language for watching
  distributed systems think"), a looping Raft election played live by
  `player` (no recording), three buttons: Start the tutorial · Install · See examples.
- **Install:** `/install/`, one self-contained page: the visitor's OS
  first, then macOS, Linux, Windows and Go. Each shows the package-manager
  command, the `install.sh` command (macOS, Linux) and direct download
  links filled from the latest GitHub release, with a link to Releases as
  fallback. It notes the unsigned binaries and `synod version`. The script
  itself is served at the site root: `/install.sh`.
- **Docs:** the tutorial, the language reference, the library, guides
  and error pages, built from `dsl` and `std` at build time. Structure,
  sources and the tutorial plan are in `11-documentation.md`.
- **Examples:** one page per example with editable source and a Run
  button. It calls `synod.wasm` (check, run) and plays the result
  with `player`; diagnostics are shown inline. No recordings. The site
  build downloads the wasm pair of a pinned compiler release, verifies it
  against `SHA256SUMS` and serves it from the site; the module runs in a
  Web Worker. An example page passes the examples module as `files` with
  `program` set to the example's folder; a snippet that imports the
  examples carries them under `vendor/`.
- **Tribute:** a short page on Leslie Lamport's "The Part-Time Parliament"
  and why the language is named after the Synod protocol.

Design: the brand from `assets/icon.svg`: navy ground `#151A2E`, pale
`#EDE9E1`, amber accent `#F4A340`, JetBrains Mono for the wordmark and code.

## `.github`

- `profile/README.md`: what Synod is, links to every repository. It lists `player` and the site (follow-up in that repo).
- Shared workflows: `go.yml` (vet, test, golden tests), `synod.yml`
  (install `synod`, run `synod fmt --check` and `synod check ./...`),
  `skills-sync.yml`.
- `CONTRIBUTING.md`: changes to the language start as a pull request to
  `dsl` that updates the spec and `decisions.md`, together with matching
  pull requests to `examples` and the tutorial.
- Issue templates: language proposal, bug, example request.

## Licences

Code under Apache-2.0; documentation and examples under CC BY 4.0.
