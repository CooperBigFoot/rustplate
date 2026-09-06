# Disk-efficient development debug information

## Outcome

Reduce debug-information disk usage for rustplate and projects created from its template. Configure the workspace root `Cargo.toml` with:

```toml
[profile.dev]
# Reduce debug-information disk usage while retaining file and line information for backtraces.
debug = "line-tables-only"
incremental = true
```

Test builds must inherit these settings from the dev profile without a separate `[profile.test]`. Keep incremental compilation enabled. Release builds must remain unchanged.

## Accepted trade-off

The user accepts reduced source-level debugger visibility: line tables retain file and line information for backtraces, but omit variable and type debug information. Savings depend on the project; no fixed reduction is promised. Incremental caches still consume disk space. This change does not clean existing build artifacts.

## Repository context

At discovery, the root `Cargo.toml` has no profile overrides. It defines the workspace, root binary package, dependencies, and shared Clippy lints. The library manifest is `crates/core/Cargo.toml`; profile configuration belongs in the workspace root.

`init.sh` renames packages in the manifests and otherwise preserves their content. No script change is expected for this profile setting.

`tests/init_template.rs` exercises the actual initialization script in an isolated template tree. Its `ROOT_MANIFEST` constant asserts the exact generated root manifest and must be updated to include the new profile and comment. Preserve the existing initialization guarantees.

## Evidence of success

- The root dev profile uses exactly `debug = "line-tables-only"` and explicitly retains `incremental = true`.
- An adjacent comment explains the disk-saving purpose and retained line information.
- Normal test builds inherit the dev settings without a redundant test profile.
- Initialization preserves the profile, as demonstrated through the real initialization test rather than only an unrelated fixture check.
- Relevant tests and the workspace formatting, Clippy, and test checks pass. Follow the repository's regression-proof-before-fix rule where applicable; establish the expected manifest behavior before applying the configuration change.
- No release profile or unrelated behavior changes.

## Scope boundaries

This is a repository-template configuration change with associated test maintenance. Do not change user-wide Cargo configuration, introduce cargo-sweep, schedule launchd jobs, clean build artifacts, disable incremental compilation, or alter existing downstream repositories. A disk-space benchmark is not required, and measured savings are not an acceptance threshold.

## Handoff status

This document is a local draft. It has not been verified on the target branch. Before substantive implementation, `implement-vision` must publish and verify this new standalone draft according to its target-branch durability gate.
