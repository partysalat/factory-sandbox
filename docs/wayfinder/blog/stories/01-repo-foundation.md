# Blog: repo foundation

Needs merged first: nothing (prerequisites below).

## Goal

Create the monorepo skeleton so every later story has a working `make ci`.

## In scope

- Directories `web/`, `api/` (Cargo workspace with crates `core`, `public`, `admin`), `infra/`. Each holds a minimal hello project that builds and tests.
- Root `pnpm-workspace.yaml`, `package.json` with `packageManager`, `.nvmrc`, `rust-toolchain.toml`, Biome config.
- `Makefile` with `.PHONY`, `help`, and the targets from the layout ticket. `synth`, `deploy`, `destroy`, `secret`, `smoke` may be stubs that print a clear message.
- Real `ci` target. Remove the `fixed.txt` condition.
- GitHub `ci` job keeps its name. Add setup steps: Node, `corepack`, Rust toolchain, Cargo cache. Actions pinned by commit SHA.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/06-monorepo-layout-make-targets-and-ci.md`
- `docs/wayfinder/blog/tickets/08-test-strategy.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- `make ci` passes locally and in GitHub Actions.
- `make help` lists all targets.
