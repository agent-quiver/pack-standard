# The Quiver pack standard

A **Quiver pack** is one git repository that holds a versioned set of agent artifacts: skills, resources
(rulebooks and references), workflows, tool contracts and agent profiles. The repository is the system of record.
GitHub or GitLab holds access, review, history and releases. Quiver reads a released tag, checks it and serves it
to agents.

- [`STANDARD.md`](STANDARD.md): the rules (version 1.0.0).
- [`schemas/pack.schema.json`](schemas/pack.schema.json): the pack manifest, `quiver-pack.yaml`.
- [`schemas/manifest.schema.json`](schemas/manifest.schema.json): the artifact manifest, `quiver.yaml`.

Published at <https://agent-quiver.github.io/spec/latest/>; the schemas are served at
`https://agent-quiver.github.io/schemas/v1/`.

## Why a standard

Skills and rules for agents are spreading as loose files. A pack gives them what code already has: one home,
review, versions, a check, and a way to pin what you depend on. The standard works without any service: a pack is
usable as plain files or as a git submodule.

## Start a pack

Use [`agent-quiver/pack-template`](https://github.com/agent-quiver/pack-template). To list a public pack, open a
pull request on [`agent-quiver/catalog`](https://github.com/agent-quiver/catalog).

## Licence

Apache-2.0 (`LICENSE`).
