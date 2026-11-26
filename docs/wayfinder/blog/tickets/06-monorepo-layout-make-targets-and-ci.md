---
type: grilling
status: closed
blocked-by: [01-rust-lambda-build-and-cdk-packaging]
claimed-by: Ben Lenser
---
## Question

What is the directory layout and the `make` target set?
Decide the dirs (`web`, `api`, `infra`), the pnpm workspace and Cargo workspace setup, and targets for build, test, lint, and deploy.
Decide what `make ci` runs, and how it keeps the existing `fixed.txt` factory check working.

## Resolution

- Facts: the factory runs `make ci` in the podman image `sf-worker`. The image has `make`, `git`, Node 22, `npm`, `python3`. It lacks `pnpm`, `cargo`, `cargo-lambda`, `zig`. The GitHub job must stay named `ci` (`factory doctor` checks it).
- Layout: `web/` (Vite + SolidJS + TypeScript), `api/` (Cargo workspace: `core` for model, validation, and DynamoDB access; `public`; `admin`), `infra/` (CDK in TypeScript). Repo root: `pnpm-workspace.yaml` (members `web`, `infra`), `Makefile`, `GLOSSARY.md`, `docs/`.
- `make ci` runs only checks that need no `cargo-lambda`, `zig`, or AWS: `cargo fmt --check`, `cargo clippy -D warnings`, `cargo test`; web lint, typecheck, test, build; infra lint and `tsc --noEmit`. No `cdk synth`, so a broken stack shows at `make deploy`.
- Pinning: `rust-toolchain.toml` (pinned stable, `rustfmt`, `clippy`), `packageManager` in the root `package.json` with `corepack`, `.nvmrc` for Node 22. `make setup` installs `cargo-lambda` v1.9.2 (deploys only).
- Worker image: the user adds `rustup` to `sf-worker` outside this repo before the first Rust story is refined. Until then Rust stories fail at the `make ci` check. `pnpm` needs no image change.
- GitHub workflow: the `ci` job keeps its name. New steps before `make ci`: Node, `corepack`, Rust toolchain from `rust-toolchain.toml`, Cargo cache. Actions stay pinned by commit SHA.
- `fixed.txt` gate: the real `ci` target replaces the stub and the `fixed.txt` condition goes away. Other tracer files stay. Removing them is a separate cleanup.
- Makefile: `.PHONY`, a `help` target built from `##` comments, top-level targets delegating to `web-*`, `api-*`, `infra-*`: `setup`, `fmt`, `lint`, `test`, `build`, `ci`, `dev`, `synth`, `deploy`, `destroy`, `secret`.
- Lint and format: Biome for `web/` and `infra/`. `rustfmt` and `clippy` for `api/`. Test frameworks are left to the test strategy step.
