# Site and organization

## `synodlang.github.io`

The organization site. It is a static site
(Astro or plain HTML; no server), deployed with GitHub Pages for the
organization at https://synodlang.github.io until a domain exists.

- **Home:** the icon, one sentence ("A small language for watching
  distributed systems think"), a looping Raft election played live by
  `player` (no recording), three buttons: Start the tutorial · Get the app · See examples.
- **Docs:** the tutorial, the language reference, the library, guides
  and error pages, built from `dsl` and `std` at build time. Structure,
  sources and the tutorial plan are in `11-documentation.md`.
- **Examples:** one page per example with editable source and a Run
  button. It calls `synod.wasm` (check, run, scene) and plays the result
  with `player`; diagnostics are shown inline. No recordings.
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
  `dsl` that updates the spec, `decisions.md`, and examples together.
- Issue templates: language proposal, bug, example request.

## Licences

Code under Apache-2.0; documentation and examples under CC BY 4.0.
