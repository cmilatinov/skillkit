# Agent Instructions

This is a Rust CLI project.

- Keep the first implementation CLI-first and library-backed.
- Prefer small, testable modules over a large command implementation in
  `main.rs`.
- Use existing Agent Skills conventions; do not invent an incompatible skill
  format unless there is no portable alternative.
- Keep harness-specific behavior behind adapter modules.
- Run `cargo fmt`, `cargo test`, and `cargo clippy` when changing Rust code.
- Update `docs/FEATURE_PLAN.md` when product scope or command behavior changes.
