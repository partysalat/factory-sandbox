# Rust Lambda build and CDK packaging

Question: how do we build and package Rust Lambdas with a TypeScript CDK app? Target: arm64, `provided.al2023`.

Research date: 2026-11-26. All facts come from primary sources listed in "Sources". Items that I could not verify are marked "Unverified".

## Answer

Use `cargo-lambda` (CLI) and `cargo-lambda-cdk` (`RustFunction` construct). The construct is maintained and fits a TypeScript CDK app. It runs `cargo lambda build` during `cdk synth`. Pass `architecture: Architecture.ARM_64` and the construct adds `--arm64`. The default runtime is already `provided.al2023`.

Caveat: `cargo-lambda-cdk` is at 0.0.x. Pin it in `package.json`.

## Versions checked

|Component|Version|Date|Source|
|-|-|-|-|
|cargo-lambda|v1.9.2|2026-08-21|[1]|
|cargo-lambda-cdk|v0.0.36 (latest release)|2025-12-08|[2]|
|cargo-lambda-cdk main|last commit "parse TOML 1.1 manifests" (01fd35e)|2026-11-19|[3]|
|aws-cdk-lib|v2.278.0 (latest release)|2026-11-20|[4]|
|cargo-lambda-cdk peer deps|`aws-cdk-lib ^2.231.0`, `constructs ^10.0.5`|main branch|[5]|

The peer range `^2.231.0` accepts 2.278.0. I did not run an install to prove it. Unverified: runtime behavior against 2.278.0.

## cargo-lambda-cdk

Install in the CDK app: `npm i cargo-lambda-cdk` [6].

```ts
import { RustFunction } from 'cargo-lambda-cdk';
import { Architecture } from 'aws-cdk-lib/aws-lambda';

new RustFunction(this, 'Api', {
  manifestPath: '../lambdas/api', // directory with Cargo.toml, or path to the file
  binaryName: 'api',              // needed for workspaces or multi-bin packages
  architecture: Architecture.ARM_64,
  // runtime defaults to provided.al2023
});
```

Behavior (from `src/function.ts` and `src/bundling.ts` [7][8]):

- Runtime defaults to `provided.al2023`. `provided.al2` is the only other value. Handler is always `bootstrap`.
- Props include `manifestPath`, `binaryName`, `architecture`, `bundling` (`environment`, `profile`, `cargoLambdaFlags`, `dockerImage`, `dockerOptions`, `forcedDockerBundling`, `commandHooks`), `gitRemote`, `gitReference`, `gitForceClone`.
- Local build command: `cargo lambda build --lambda-dir <out> --release --arm64 --bin <name> --flatten <name>`. The profile defaults to `release`.
- ARM_64 adds `--arm64`. Any other architecture adds `--x86-64`. If you set no architecture, the construct adds no flag, so cargo-lambda builds x86-64 [8][9]. Always set `architecture`.
- A workspace manifest with no `binaryName` throws an error. A package with several `[[bin]]` entries and no `binaryName` throws an error.
- Local detection runs `cargo lambda --version`. It checks the exit status only, not the version number. If it fails, the construct falls back to Docker (`ghcr.io/cargo-lambda/cargo-lambda`). README says cargo-lambda 0.12.0 or later [6].
- Unverified: whether the `architecture` prop on `RustFunction` is read from the top-level prop or from `bundling.architecture`. The README shows it only in the `RustExtension` example, and `function.ts` reads `bundling.architecture`. Test with `cdk synth` and check the `Architectures` value in the template.

Docker fallback: do not rely on it. Local build is faster. cargo-lambda uses Zig, so Docker is not needed on macOS or Linux.

## cargo-lambda build facts

From the build command docs [9]:

- `--arm64` equals `--target aarch64-unknown-linux-gnu`.
- Default compiler is `cargo-zigbuild`. It uses Zig to cross-compile from the host OS. Alternatives: `--compiler cargo` (Linux host, no Zig) and `--compiler cross` (needs `cross` and Docker).
- Output: `target/lambda/<bin>/bootstrap`. `--output-format zip` gives a zip.
- Default profile is debug. The construct passes `--release`.

AWS documents the same flow (`cargo lambda build --release --arm64`, runtime `provided.al2023`) [10]. AWS calls cargo-lambda "a third-party open-source extension" [11]. Lambda has no managed Rust runtime. It uses the OS-only runtime [11].

## Required tools

### Common

|Tool|Why|
|-|-|
|Rust via rustup (stable)|Compiler. The Lambda runtime crate needs Rust 1.84.0 or later [12].|
|Rust target `aarch64-unknown-linux-gnu`|Target for `--arm64`.|
|cargo-lambda 1.9.x|Build CLI [1].|
|Zig|Linker for cross-compile. Some install methods add it with cargo-lambda.|
|Node.js and npm|CDK app (TypeScript).|
|AWS credentials|`cdk deploy`.|

Rust target: cargo-lambda source (`toolchain.rs`) runs `rustup target add <target>` when the target is missing [13]. I read only that file. I did not verify non-interactive behavior. Add the target explicitly in CI and in `make setup`. This is cheap and safe.

### macOS

```sh
curl https://sh.rustup.rs -sSf | sh
rustup target add aarch64-unknown-linux-gnu
brew install cargo-lambda/tap/cargo-lambda   # also installs Zig if missing [14]
```

Check: `cargo lambda --version` and `zig version`. The curl script, PyPI and Nix methods also install Zig. A plain binary download or `cargo install` does not [14]. In that case run `cargo lambda system --install-zig` or `brew install zig`.

### GitHub Actions ubuntu-latest

The AWS SAM docs give a workflow that builds for arm64 [15]. It uses `dtolnay/rust-toolchain@stable` with `targets: aarch64-unknown-linux-gnu`, `actions/cache@v4` and `pip install cargo-lambda`. The cargo-lambda docs also show `pip install cargo-lambda` [16]. The PyPI method installs Zig [14], so no separate Zig step is needed.

```yaml
- uses: actions/checkout@v5
- uses: dtolnay/rust-toolchain@stable
  with:
    targets: aarch64-unknown-linux-gnu
- uses: actions/cache@v4
  with:
    path: |
      ~/.cargo/registry/index
      ~/.cargo/registry/cache
      ~/.cargo/git/db
      target
    key: cargo-arm64-${{ hashFiles('Cargo.lock', 'Cargo.toml') }}
    restore-keys: cargo-arm64-
- run: pip install cargo-lambda
- uses: actions/setup-node@v4
  with: { node-version: 22 }
- run: make deploy
```

Notes:

- The cargo-lambda "Automating deployments" page uses `actions-rs/toolchain@v1` [16]. Do not copy it. Use `dtolnay/rust-toolchain` as in [15]. Unverified: whether `actions-rs/toolchain` is archived. I did not check its repo.
- For an explicit Zig step: `goto-bus-stop/setup-zig` says it is unmaintained and points to `mlugg/setup-zig` [17]. The mlugg README points to Codeberg [18]. Unverified: its current version tag and inputs. Prefer `pip install cargo-lambda`.
- Pin cargo-lambda in CI, for example `pip install cargo-lambda==1.9.2`. Unverified: that PyPI uses the same version number as the GitHub release.
- `ubuntu-latest` is x86-64. The arm64 binary is cross-compiled, not run. Do not run the arm64 binary on that runner.
- AWS credentials for deploy (for example OIDC) are not covered in this note.

## Alternatives if the construct is weak

The construct looks adequate. Risks: 0.0.x version, Docker fallback only, jsii wrapper. If it fails, use one of these:

1. Build outside CDK and package with `lambda.Code.fromAsset('target/lambda/<bin>')`. Run `cargo lambda build --release --arm64` first (make target). Set `runtime: Runtime.PROVIDED_AL2023`, `handler: 'bootstrap'`, `architecture: Architecture.ARM_64`. Full control, no extra dependency. Cost: the make step must run before `cdk synth`. Output layout is `target/lambda/<bin>/bootstrap` [9].
2. AWS SAM with `BuildMethod: rust-cargolambda`, `Runtime: provided.al2023`, `Handler: bootstrap` [15]. SAM needs Docker for build and test [15]. This replaces CDK, so it fits only if we drop CDK.
3. `cargo lambda deploy` [10]. It creates a role and function outside CloudFormation. Not suitable for a CDK-managed stack.

Option 1 is the fallback. It uses only documented CLI behavior.

## make flow, source to cdk deploy

The repo `Makefile` has only a `ci` target today. This is a proposal. Paths are examples.

```make
CARGO_TARGET := aarch64-unknown-linux-gnu

setup:
	rustup target add $(CARGO_TARGET)
	cd infra && npm ci

lambda-test:
	cargo test

lambda-build:
	cargo lambda build --release --arm64 --output-format binary

synth: lambda-build
	cd infra && npx cdk synth

deploy: lambda-build
	cd infra && npx cdk deploy --require-approval never
```

Flow: `make setup` (once), `make lambda-test`, `make synth`, `make deploy`.

With `RustFunction`, `lambda-build` is optional. The construct builds during synth. Keep the explicit target for a CI cache and a fast failure before synth. The construct then rebuilds with the same flags. If double build time matters, use alternative 1.

Run `cdk bootstrap` once per account and region before the first deploy.

## Open points for the plan

- Decide: construct (default) or `Code.fromAsset` (fallback).
- Decide: Cargo workspace (one package per Lambda, shared crate) or one multi-bin package. SAM docs recommend workspaces [15]. With the construct, both need `binaryName` [8].
- Run a spike: `cdk synth` with `RustFunction`. Check `Architectures: [arm64]` and `Runtime: provided.al2023` in the template.

## Sources

1. https://api.github.com/repos/cargo-lambda/cargo-lambda/releases/latest (v1.9.2)
2. https://api.github.com/repos/cargo-lambda/cargo-lambda-cdk/releases/latest (v0.0.36)
3. https://api.github.com/repos/cargo-lambda/cargo-lambda-cdk/commits
4. https://api.github.com/repos/aws/aws-cdk/releases/latest (v2.278.0)
5. https://raw.githubusercontent.com/cargo-lambda/cargo-lambda-cdk/main/package.json
6. https://github.com/cargo-lambda/cargo-lambda-cdk (README)
7. https://raw.githubusercontent.com/cargo-lambda/cargo-lambda-cdk/main/src/function.ts
8. https://raw.githubusercontent.com/cargo-lambda/cargo-lambda-cdk/main/src/bundling.ts
9. https://www.cargo-lambda.info/commands/build.html
10. https://docs.aws.amazon.com/lambda/latest/dg/rust-package.html
11. https://docs.aws.amazon.com/lambda/latest/dg/lambda-rust.html
12. https://github.com/awslabs/aws-lambda-rust-runtime (README, Rust 1.84.0 minimum)
13. https://raw.githubusercontent.com/cargo-lambda/cargo-lambda/main/crates/cargo-lambda-build/src/toolchain.rs
14. https://www.cargo-lambda.info/guide/installation.html
15. https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/building-rust.html
16. https://www.cargo-lambda.info/guide/automating-deployments.html
17. https://github.com/goto-bus-stop/setup-zig
18. https://github.com/mlugg/setup-zig
