# Skillkit Feature Plan

## Product Goal

Skillkit should be a practical package manager for Agent Skills: a tool that
can add a skill from a local path, Git repo, URL, or registry into the right
location for the agent harness the user wants to use.

The key design constraint is portability. Skillkit should support standard
skills without requiring authors to rewrite their content for every agent
harness.

## Package Model

- A package may contain one or more skills.
- Each skill keeps the standard structure: `SKILL.md` plus optional `scripts/`,
  `references/`, and `assets/`.
- Package metadata lives in `skillkit.toml` at the package root.
- A package can declare harness compatibility, required tools, MCP dependencies,
  license, authors, repository URL, and included skill paths.
- Package identity should be stable: `publisher/name`.
- Requested versions can be arbitrary strings interpreted by the source that
  resolves them.

## Manifest Sketch

```toml
[package]
name = "git-pr-workflow"
publisher = "cmilatinov"
version = "0.1.0"
description = "Reusable Git and pull request workflow skill"
license = "MIT"

[[skills]]
path = "skills/git-pr-workflow"

[compat]
generic = true
codex = true
claude = true
opencode = true

[[registries]]
name = "community"
source = "git+https://github.com/skillkit/registry.git"
```

## Harness Targets

- Generic repo target: `.agents/skills`.
- Generic user target: platform-appropriate user skills directory.
- Codex target: repo or user skill locations supported by Codex.
- Claude target: `.claude/skills` or user-level Claude skills.
- OpenCode target: `.opencode/skills` or `~/.config/opencode/skills`.
- Future harness adapters can be added without changing the package format.

OpenCode also discovers compatible skills from `.agents/skills`,
`~/.agents/skills`, `.claude/skills`, and `~/.claude/skills`. Skillkit should
prefer OpenCode-native paths when the harness is explicitly OpenCode, while
keeping `.agents/skills` as the generic portable target.

## Core Commands

- `skillkit init`: create `skillkit.toml` and an optional starter skill.
- `skillkit new <name>`: create a new skill package from a template.
- `skillkit add <source>`: add from local path, GitHub path, Git URL, or
  registry package.
- `skillkit list`: show added packages and skills for a target.
- `skillkit remove <name>`: remove an added package safely.
- `skillkit update`: update packages according to source-specific rules.
- `skillkit search <query>`: search configured registries.
- `skillkit registry add <name> <source>`: add a registry source to the
  repo-local `skillkit.toml`.
- `skillkit registry remove <name>`: remove a registry source from the
  repo-local `skillkit.toml`.

## Lockfile

Skillkit should create `skillkit.lock` for repo-level adds.

The lockfile should record:

- package identity
- requested version or selector
- resolved source URL
- source-specific resolved identity
- added skill paths
- target harness and destination directory

This makes team setups reproducible and gives CI a way to verify added skills.

## Version Strategy

Skillkit should not silently track the latest version of a skill. `skillkit add`
should resolve the requested version or selector and record a source-specific
identity in `skillkit.lock`. `skillkit update` should be the explicit command
that moves an already-added package forward according to the source's rules.

Different sources can use different lock fields:

- Git registry or Git source: lock the resolved commit hash.
- OpenAI plugin source: lock the plugin version and marketplace identity.
- Static URL source: lock the URL plus content digest.
- Local path source: lock the path and, later, an optional content digest.

The manifest can use arbitrary strings such as `latest`, `main`, `v1`,
`0.1.0`, or a commit hash. The lockfile records what that string resolved to
for the concrete source implementation.

This keeps agent behavior reproducible. A skill update can change instructions,
tool use, file-editing behavior, harness compatibility, or dependency
requirements, so teams need a reviewable change when a skill version moves.

## Registry Strategy

MVP registry support should be simple:

- A Git-backed index repository stores package metadata.
- Registry sources are named entries in the repo-local `skillkit.toml`.
- `skillkit registry add` and `skillkit registry remove` only edit local project
  config; they do not modify the remote registry.
- Search can work from a locally cached index.

Later registry work can add an HTTP API, package ownership, download metrics,
and moderation.

## Safety

- Never execute package scripts while adding a package.
- Reject package contents with path traversal or absolute paths.
- Warn when skills declare external tool or MCP dependencies.
- Support signatures before encouraging broad third-party package use.
- Keep adds reversible by tracking owned files.

## Rust Architecture

Start as one crate with clear modules, then split only when needed:

- `cli`: clap command definitions and output formatting.
- `manifest`: `skillkit.toml` parsing.
- `skill`: `SKILL.md` discovery and frontmatter parsing.
- `source`: local path, Git, GitHub, URL, plugin, and registry source
  resolution.
- `target`: target directories, file ownership, lockfile writes.
- `harness`: harness adapters for generic, Codex, Claude, and OpenCode targets.
- `registry`: project registry config, index cache, search, and package
  metadata workflows.
- `error`: shared error types and diagnostics.

Likely dependencies:

- `clap` for CLI parsing.
- `serde`, `toml`, and `serde_yaml` for manifests and frontmatter.
- `camino` for UTF-8 paths.
- `tempfile` for safe extraction.
- `thiserror` or `miette` for diagnostics.

## MVP Milestones

1. Add a local package into `.agents/skills`.
2. Track added files and remove packages cleanly.
3. Add from a GitHub repo path.
4. Add `skillkit.lock` for reproducible repo-level adds.
5. Add `skillkit registry add` and `skillkit registry remove`.
6. Add Git-backed registry search and registry-backed adds.
7. Add harness adapters for Codex, Claude, and OpenCode.
8. Add signature verification.

## Open Decisions

- Should packages be single-skill by default with multi-skill support, or
  multi-skill from the start?
- Should the default add target be repo-local or user-local?
- Should Skillkit manage harness-specific metadata files or only copy them
  through?
- Should `skillkit.toml` live outside standard skill folders in every case?
