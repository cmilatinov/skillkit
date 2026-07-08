# Skillkit

Skillkit is a Rust package manager for reusable Agent Skills.

The goal is to make skills easy to create, add, share, pin, and update across
agent clients without locking the skill author into one host.

## Early Direction

- Use the existing Agent Skills folder shape as the portable base:
  `SKILL.md`, optional `scripts/`, `references/`, and `assets/`.
- Add package-manager metadata outside the skill when possible.
- Support repo-local adds first, then user-level adds.
- Keep add behavior transparent: no add-time script execution.
- Treat registries, signatures, and lockfiles as first-class features before
  broad third-party adds.

## Planned CLI

```text
skillkit init
skillkit new
skillkit add
skillkit list
skillkit update
skillkit remove
skillkit search
```

See [docs/FEATURE_PLAN.md](docs/FEATURE_PLAN.md) for the working feature plan.
