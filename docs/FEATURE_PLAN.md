# Skillkit Feature Plan

## Product Goal

Skillkit should be a practical package manager for Agent Skills: a tool that
can install a skill from a local path, Git repo, URL, or registry into the
right location for the agent client the user wants to use.

The key design constraint is portability. Skillkit should support standard
skills without requiring authors to rewrite their content for every agent host.

## Package Model

- A package may contain one or more skills.
- Each skill keeps the standard structure: `SKILL.md` plus optional `scripts/`,
  `references/`, and `assets/`.
- Package metadata lives in `skillkit.toml` at the package root.
- A package can declare host compatibility, required tools, MCP dependencies,
  license, authors, repository URL, and included skill paths.
- Package identity should be stable: `publisher/name`.
- Versions should use SemVer.

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
```

## Install Targets

- Generic repo target: `.agents/skills`.
- Generic user target: platform-appropriate user skills directory.
- Codex target: repo or user skill locations supported by Codex.
- Claude target: `.claude/skills` or user-level Claude skills.
- OpenCode target: `.opencode/skills` or `~/.config/opencode/skills`.
- Future host adapters can be added without changing the package format.

OpenCode also discovers compatible skills from `.agents/skills`,
`~/.agents/skills`, `.claude/skills`, and `~/.claude/skills`. Skillkit should
prefer OpenCode-native paths when the target is explicitly OpenCode, while
keeping `.agents/skills` as the generic portable target.

## Core Commands

- `skillkit init`: create `skillkit.toml` and an optional starter skill.
- `skillkit new <name>`: create a new skill package from a template.
- `skillkit validate`: check manifest, `SKILL.md` frontmatter, paths, size, and
  portability warnings.
- `skillkit pack`: create a deterministic package archive with checksums.
- `skillkit install <source>`: install from local path, GitHub path, Git URL,
  archive URL, or registry package.
- `skillkit list`: show installed packages and skills for a target.
- `skillkit uninstall <name>`: remove an installed package safely.
- `skillkit update`: update packages according to version constraints.
- `skillkit search <query>`: search configured registries.
- `skillkit publish`: publish a package to a registry or index.
- `skillkit doctor`: inspect host paths, config, registry auth, and common
  installation problems.

## Lockfile

Skillkit should create `skillkit.lock` for repo installs.

The lockfile should record:

- package identity and version
- resolved source URL
- commit SHA or archive digest
- installed skill paths
- target host and install directory

This makes team setups reproducible and gives CI a way to verify installed
skills.

## Registry Strategy

MVP registry support should be simple:

- A Git-backed index repository stores package metadata.
- Package archives can live in GitHub releases or another static host.
- Search can work from a locally cached index.
- Publishing opens or updates index entries.

Later registry work can add an HTTP API, authenticated publishing, package
ownership, download metrics, and moderation.

## Safety

- Never execute package scripts during install.
- Reject archives with path traversal or absolute paths.
- Verify package checksums before installing.
- Warn when skills declare external tool or MCP dependencies.
- Support signatures before encouraging broad third-party package use.
- Keep installs reversible by tracking owned files.

## Rust Architecture

Start as one crate with clear modules, then split only when needed:

- `cli`: clap command definitions and output formatting.
- `manifest`: `skillkit.toml` parsing and validation.
- `skill`: `SKILL.md` discovery and frontmatter validation.
- `source`: local path, Git, GitHub, URL, and registry source resolution.
- `package`: pack/unpack, checksums, and archive safety.
- `install`: target directories, file ownership, lockfile writes.
- `host`: host adapters for generic, Codex, Claude, and OpenCode targets.
- `registry`: index cache, search, and publish workflows.
- `error`: shared error types and diagnostics.

Likely dependencies:

- `clap` for CLI parsing.
- `serde`, `toml`, and `serde_yaml` for manifests and frontmatter.
- `camino` for UTF-8 paths.
- `semver` for versions.
- `sha2` for checksums.
- `tar` and `zstd` for package archives.
- `tempfile` for safe extraction.
- `thiserror` or `miette` for diagnostics.

## MVP Milestones

1. Validate a local skill package.
2. Install a local package into `.agents/skills`.
3. Track installed files and uninstall cleanly.
4. Install from a GitHub repo path.
5. Add `skillkit.lock` for reproducible repo installs.
6. Add deterministic `pack` output and checksum verification.
7. Add Git-backed registry search and install.
8. Add publish workflow.
9. Add host adapters for Codex, Claude, and OpenCode.
10. Add signature verification.

## Open Decisions

- Should packages be single-skill by default with multi-skill support, or
  multi-skill from the start?
- Should the default install target be repo-local or user-local?
- Should Skillkit manage host-specific metadata files or only copy them through?
- Should registry publishing require signed packages from day one?
- Should `skillkit.toml` live outside standard skill folders in every case?
