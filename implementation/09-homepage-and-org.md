# Homepage and organization

## `homepage`

A static site (Astro or plain HTML; no server), deployed with GitHub Pages at
`synodlang.github.io` until a domain exists.

- **Home:** the icon, one sentence ("A small language for watching
  distributed systems think"), a looping recording of Raft electing a
  leader, three buttons: Get the app · Read the spec · See examples.
- **Docs:** rendered from `dsl` at build time (spec, std catalog), with
  each `synod` block highlighted.
- **Examples:** one page per example with its recording (exported from
  the Mac app) and source.
- **Tribute:** a short page on Leslie Lamport's "The Part-Time Parliament"
  and why the language is named after the Synod protocol.

Design: the brand from `assets/icon.svg`: navy ground `#151A2E`, pale
`#EDE9E1`, amber accent `#F4A340`, JetBrains Mono for the wordmark and code.

## `.github`

- `profile/README.md`: what Synod is, links to every repository.
- Shared workflows: `go.yml` (vet, test, golden tests), `synod.yml`
  (install `synod`, run `synod fmt --check` and `synod check ./...`),
  `skills-sync.yml`.
- `CONTRIBUTING.md`: changes to the language start as a pull request to
  `dsl` that updates the spec, `decisions.md`, and examples together.
- Issue templates: language proposal, bug, example request.

## Licences

Code under Apache-2.0; documentation and examples under CC BY 4.0.
