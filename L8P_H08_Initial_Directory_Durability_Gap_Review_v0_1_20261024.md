# L8P H-08 Initial Directory Durability — Gap Review v0.1

**Date:** 2026-10-24  
**Status:** Review artifact only; no source changes made.  
**Release:** `RC_NOT_SEALED`  
**Constitutional principle:** **Not the Judge — The Witness.**

## Scope

Reviewed the persisted H-08 Cargo workspace v0.3 source and runbook. This review isolates the explicitly open initial-directory-creation durability question. It does not replace or modify frozen L8P core code, H-08 candidate code, or normative decisions.

## Observed source behavior

`FileBackend::open(root)` currently performs:

1. `create_dir_all(root)`
2. `create_dir_all(root/blobs)`
3. `create_dir_all(root/metadata)`
4. `create_dir_all(root/commits)`
5. returns `FileBackend`

It does not explicitly sync the parent directory that contains `root`. It also does not explicitly sync `root` after creating the three child directories before returning. Later persistence operations sync the data files and some relevant directories, but those later operations do not prove that the initial directory tree was durable at the time `open()` returned.

## Risk statement

A process can observe a successfully created directory tree without that tree's directory entries necessarily being durable across sudden power loss on every filesystem. The exact guarantee is platform/filesystem dependent. This is a gap in the durability contract, not evidence that data has been lost.

## Required design decision before implementation

Define one explicit contract:

- **Strict durable initialization:** `open()` may report initialization success only after syncing newly created directory entries and the required ancestor directory entries, using a supported platform primitive; or
- **Non-durable initialization:** `open()` may create the layout but must not imply that the layout itself is crash-durable until a separate initialization barrier succeeds.

Do not silently claim the first behavior on platforms where equivalent directory synchronization is unavailable. Preserve fail-closed behavior unless a tested platform-specific implementation is approved.

## Regression-test requirements

1. Existing-root case: opening an already initialized root must not change frozen core or wire/signature behavior.
2. New-root case: test the chosen initialization contract and assert the exact returned status/error.
3. Missing nested directories case: verify `blobs`, `metadata`, and `commits` creation and the required sync ordering.
4. Sync-failure injection: failure while syncing a child directory, root, or parent must not be reported as durable initialization success.
5. Reopen/recovery case: after simulated interruption at each initialization stage, reopening must classify the resulting state without upgrading unknown state to complete.
6. Platform contract: Unix-specific directory sync tests run only where supported; unsupported platforms must fail closed and be explicitly reported.

A simulated process interruption is not equivalent to physical machine power-loss validation. Real target-filesystem tests remain a separate release gate.

## Safe next action

First run the existing workspace in a Cargo-enabled Codespace and preserve all logs/exit statuses. Reconcile those results before implementing the initialization change. Then add the smallest H-08-only patch and targeted tests on a disposable branch; do not edit `l8p_src/l8p_core/` or merge to `main` during this work.

## Evidence boundary

This is a static source review only. Rust formatting, compilation, Clippy, tests, target filesystem tests, and machine power-loss tests were not run in this review.

**Not the Judge — The Witness.** This report records a gap in the source contract; it does not infer a failure event or certify production readiness.
