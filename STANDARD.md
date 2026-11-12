# Quiver pack standard

A **Quiver pack** is one git repository on a forge (GitLab, GitHub) that holds a versioned set of agent
artifacts: skills, resources, workflows, tool contracts and agent profiles. The repository is the system of record:
the forge holds access, review, history and releases. Quiver only reads a released tag and serves it to agents.

## 1. One repository, one pack, one access unit

- A pack is exactly one repository. Who may read it is who may read the repository on the forge.
- Artifacts that different people may read go in different packs. There is no per-artifact audience inside a pack.
- Quiver writes nothing to the repository. Every change is a commit by a person or by that person's own agent.

## 2. Layout

```text
quiver-pack.yaml          required: the pack manifest (`schemas/pack.schema.json`)
README.md                 required: what the pack is for, how to connect, the artifact index
CHANGELOG.md              required: one entry per release tag
LICENSE                   required for a pack that is shared beyond its owner
CODEOWNERS                recommended: who reviews which directory
quiver-pack.lock          required when the pack depends on other packs
skills/<name>/            SKILL.md + quiver.yaml
resources/<name>/         the entrypoint file(s) + quiver.yaml
workflows/<name>/         the entrypoint file(s) + quiver.yaml
tools/<name>/             a tool contract + quiver.yaml; never code Quiver runs
agent_profiles/<name>/    <name>.md + quiver.yaml
evals/<artifact-id>/      representative tasks and the expected behaviour
```

- Every artifact is a directory holding its `quiver.yaml` (the manifest schema) and its entrypoint. The directory
  name is the artifact id with dots replaced by hyphens.
- `SKILL.md` keeps the standard Agent Skills frontmatter and nothing Quiver-specific, so `skills/` works as is in
  any harness that reads Agent Skills.
- Generated harness adapters (a Claude Code plugin manifest, a Codex skills directory) are build output. They are
  never the source and are not committed unless the forge serves them from the repository.

### Skill references

A skill reference is `/<name>` written as its own code span (`` `/orient` ``, optionally followed by arguments,
`` `/adr new` ``), or at the start of a line, after any indentation, inside a fenced code block. It names a skill
in the same pack and must resolve to one: `<name>` is that skill's directory.

Anything else that begins with a slash is written so that it is not a lone `/<name>`: a URL path with its method or
host (`` `GET /healthz` ``), a file path with its directory (`` `./scripts/check.py` ``). A skill in another pack is
named in prose with its pack id. Version 1.x has no cross-pack reference syntax.

## 3. The pack manifest (`quiver-pack.yaml`)

- `id`: `<forge namespace>/<pack>`, matching the repository path, lower case.
- `version`: semver. It equals the release tag without the `v`.
- `title`, `card` (one line, ≤ 40 tokens: when to use the pack), `description`.
- `maintainers`, `license`.
- `requires`: connectors and capabilities the pack's artifacts need, for example a mail connector. Never a grant.
- `dependencies`: other packs by id and semver range, resolved in `quiver-pack.lock` to a commit and sha256.
- `compatibility.quiver`: the pack standard version range the pack follows.

## 4. Versions and releases

- A release is a protected tag `vX.Y.Z` on the default branch. Nothing else is a release. A tag is never moved
  or deleted. Quiver records each tag by commit and sha256 and keeps serving what it recorded.
- The pack version follows semver over the pack as a whole:
  - **major:** an artifact removed or renamed, or a breaking contract change;
  - **minor:** an artifact added or changed compatibly;
  - **patch:** fixes.
- Each artifact keeps its own semver in its `quiver.yaml`. Agents cite `id@version`. A published artifact version
  is immutable: a change bumps it.

## 5. The gate

- Every merge request runs `quiver check` in the forge's CI, using the CI component or Action that Quiver
  publishes. It covers the manifests, budgets, ids, frontmatter, the lock file and a secret scan.
- Quiver runs the same check again when it reads a tag, and serves only tags that pass its own run.
- The maintainer role on the forge is the only way to merge or tag. CODEOWNERS names reviewers per directory.

## 6. Consuming a pack

- **In an agent (any surface):** connect to the pack's Quiver URL. Quiver serves the latest release, or the
  version pinned in the URL (`…/p/<id>@<version>/mcp`).
- **In a code repository:** pin the pack in the project. A git submodule at a release tag is supported and gives a
  local copy, or the project's `quiver-pack.lock` pins the version Quiver serves to agents working there. Moving to
  a new release is a reviewed change to the pin.

## 7. Transitional (manifest schema 1)

Artifact manifests imported from the single-registry era still carry `owner` and `audience`. Inside a pack Quiver
ignores both: the pack's `maintainers` and the repository's access decide. The next manifest schema drops them.
