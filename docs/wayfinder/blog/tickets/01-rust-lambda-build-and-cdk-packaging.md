---
type: research
status: closed
blocked-by: []
claimed-by:
---
## Question

How do we build and package Rust Lambdas with a TypeScript CDK app?
Check `cargo-lambda` plus the `cargo-lambda-cdk` construct against the current `aws-cdk-lib`. Target arm64 and `provided.al2023`.
List the required tools (`cargo-lambda`, `zig`, rustup targets) on macOS and on GitHub Actions `ubuntu-latest`.
Name the alternatives if the construct is weak. Give the `make` flow from source to `cdk deploy`.

## Resolution

Use `cargo-lambda` v1.9.2 with `cargo-lambda-cdk` v0.0.36 in a TypeScript CDK app. Set `architecture: ARM_64` on `RustFunction`. Runtime is `provided.al2023`.
Full note: [research/01-rust-lambda-build-and-cdk-packaging.md](../research/01-rust-lambda-build-and-cdk-packaging.md). The branch push failed (repo rule GH013), so the note stays local.
Open checks for the build stories: no install or synth was run, and the `architecture` prop was not confirmed.
Fallback: `cargo lambda build --release --arm64`, then `Code.fromAsset('target/lambda/<bin>')` with handler `bootstrap`.
