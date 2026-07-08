# Skillkit

Skillkit is a Rust package manager for reusable Agent Skills.

The goal is to make skills easy to create, validate, install, share, pin, and
update across agent clients without locking the skill author into one host.

## Early Direction

- Use the existing Agent Skills folder shape as the portable base:
  `SKILL.md`, optional `scripts/`, `references/`, and `assets/`.
- Add package-manager metadata outside the skill when possible.
- Support repo-local installs first, then user-level installs.
- Keep install behavior transparent: no install-time script execution.
- Treat registries, signatures, and lockfiles as first-class features before
  broad publishing.

## Planned CLI

```text
skillkit init
skillkit validate
skillkit pack
skillkit install
skillkit list
skillkit update
skillkit uninstall
skillkit search
skillkit publish
skillkit doctor
```

See [docs/FEATURE_PLAN.md](docs/FEATURE_PLAN.md) for the working feature plan.
