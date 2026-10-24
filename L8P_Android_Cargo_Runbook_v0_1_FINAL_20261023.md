# L8P Rust/Cargo Setup from Android — Runbook v0.1

**Purpose:** execute the H-08 candidate in a real Rust environment without a Windows, Mac, or Linux computer.  
**Current release state:** `RC_NOT_SEALED`.  
**Constitutional principle:** **Not the Judge — The Witness.**

This uses GitHub Codespaces, which runs the development environment in the cloud and can be opened in a browser. Official instructions: https://docs.github.com/en/codespaces/developing-in-a-codespace/creating-a-codespace-for-a-repository and https://docs.github.com/en/codespaces/quickstart.

## Part A — Put the workspace on a temporary branch

1. On Android, download `L8P_H08_Cargo_Workspace_v0_3.zip` from the ChatGPT conversation to your Downloads folder.
2. In Chrome, open the repository: https://github.com/knappchristophercort-source/solid-doodle.
3. Open the branch selector currently labeled `main`. Create a new branch named `l8p-h08-cargo-verify-v0-3` from `main`.
4. Confirm the new branch is selected. Choose **Add file → Upload files** and upload `L8P_H08_Cargo_Workspace_v0_3.zip` to the repository root.
5. Commit that upload to `l8p-h08-cargo-verify-v0-3`, **not `main`**. This is only a transport step so the cloud workspace can access the bundle; it is not an L8P implementation commit.

## Part B — Create the cloud Rust environment

6. While viewing the temporary branch, tap **Code → Codespaces → Create codespace on `l8p-h08-cargo-verify-v0-3`**.
7. If GitHub shows a billing/quota prompt, review it before continuing. Do not add billing details or raise a spending limit unless you independently decide to.
8. Wait for the browser-based VS Code environment to open. Open **Terminal → New Terminal**.
9. Extract the bundle at the repository root. Paste these commands one at a time:

```bash
unzip -t L8P_H08_Cargo_Workspace_v0_3.zip
unzip -o L8P_H08_Cargo_Workspace_v0_3.zip -d .
rm L8P_H08_Cargo_Workspace_v0_3.zip
bash scripts/verify_hashes.sh
```

The ZIP is flat-root: extraction places `.devcontainer/devcontainer.json`, the Cargo workspace, the adapter, and the unchanged frozen core at their expected paths.

10. Because the devcontainer configuration was extracted after the Codespace was first created, rebuild the container: open the **Command Palette** from the VS Code browser menu, search for **Codespaces: Rebuild Container**, and select it. Wait for the rebuild and reconnect to finish. This selects the Rust devcontainer image defined in `.devcontainer/devcontainer.json`.

## Part C — Run and preserve evidence

11. Open a new terminal and run:

```bash
bash scripts/verify_hashes.sh
bash scripts/run_gates.sh
```

12. The gate script runs Rust version checks, adapter formatting check, adapter Clippy, H-08 tests, frozen-core default tests, and the frozen-core `reference-signer` test run. It records each command's combined output and exit status under `evidence/runs/<UTC timestamp>/`. It continues through later gates even if an earlier gate fails.
13. If `bash scripts/run_gates.sh` exits nonzero, do not try to make the release look green. Open the newest files in `evidence/runs/`, preserve the logs, and report the failing command and its `.exit` file. A failure is evidence to investigate, not permission to change the frozen core.
14. Do not run `cargo fmt` without `--check`, do not rewrite anything under `l8p_src/l8p_core/`, and do not merge this candidate to `main`. Only the H-08 adapter is the candidate workstream.
15. After execution, `evidence/runs/` is new evidence. It is not covered by the static `SHA256SUMS` manifest; preserve it with the command/status records and record the branch HEAD. Do not mark H-08 complete or change `RC_NOT_SEALED` merely because the commands ran.

## What this does not prove

- Passing tests in Codespaces do not certify machine power-loss behavior on every deployment filesystem.
- Directory creation durability still needs an explicit platform decision.
- H-08 remains a failure-capture persistence component, not the production admission gate. `CompleteAndDurable` is not `Admitted`.
- The production admission entry point and policy adapter remain to be identified and wired.
