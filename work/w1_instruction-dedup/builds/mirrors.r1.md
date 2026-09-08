# Beads Viewer instruction mirror build

## Verdict

- Manifest application: PASS.
- Required UBS gate: FAIL. UBS 5.4.0 selected zero files because the installed module directory has no `contract.json` and contains only JavaScript and Python modules. The repo has 361 Go files. This is an external tool installation defect, not a pending user check.
- Worker outcome: implementation complete and committed; UBS failure preserved and diagnosed.

## Scope

- Primary prefix: `/home/ben/projects/tools/beads_viewer`
- Worker root: `/home/ben/worktrees/beads_viewer/w1_instruction-dedup`
- Approved package root: `/home/ben/worktrees/shared-docs/w903_instruction-dedup/work/w903_instruction-dedup`
- Selected approved-manifest entries: 2
- Selected mirror-manifest entries: 1
- Preserve obligations: 0
- Trash-before-symlink obligations: 1
- No subagents, other repo writes, primary repo writes, worktree creation, merge, push, live configuration changes, hook changes, application changes, application tests, or runtime proof reruns were performed.

## Applied outputs

1. Verified the existing worker `AGENTS.md` and primary source both had SHA-256 `c86fd60b8793e61c2ab36e6493bf4de5636205f7368a441cfe4cc562daf205d7`.
2. Verified root `CLAUDE.md` was absent.
3. Verified the named proposal had SHA-256 `41eb3ec376a8d78338ee45f10effdfbe8019c5bb0189550c9a0a25b6a2e05430`.
4. Copied the proposal byte-for-byte to root `CLAUDE.md` before displacing the old source.
5. Rechecked the old `AGENTS.md` hash immediately before trashing it.
6. Trashed the regular `AGENTS.md`, then created `AGENTS.md -> CLAUDE.md` with the literal relative target.
7. Verified both instruction paths resolve to SHA-256 `41eb3ec376a8d78338ee45f10effdfbe8019c5bb0189550c9a0a25b6a2e05430`.

Trash batch: `/home/ben/.local/share/agent-trash/beads_viewer/20260908T052026Z-3437527`

The trash receipt records the reversible restore source. The replacement symlink must be moved aside before any restore because restore never overwrites.

## Checks

| Check | Result |
|---|---|
| Approved source and proposal hashes before copy | PASS |
| Expected-new `CLAUDE.md` absent before copy | PASS |
| Copied bytes equal named proposal | PASS |
| Old `AGENTS.md` hash immediately before trash | PASS |
| `AGENTS.md` is a symlink with target `CLAUDE.md` | PASS |
| Symlink resolves inside this worker root | PASS |
| Scoped `verify_migration.py text` with explicit candidate mapping | `PASS approved-text` |
| Scoped `verify_migration.py mirrors` with explicit candidate mapping | `PASS mirrors` |
| Primary checkout source remains unchanged | PASS |
| Existing `.worktree-check` | Not present, so no repo gate ran |
| `git diff --cached --check` before implementation commit | PASS |
| UBS required gate | FAIL: zero files selected; evidence in `mirrors.r1.ubs_v1.txt` through `mirrors.r1.ubs_v5.txt` |

## Implementation commit

`aef0e90e568742fdd25be233b5318b12dad9a53d` (`docs: reconcile local instructions`)

## Exact committed paths and SHA-256

`AGENTS.md` is mode `120000`, target `CLAUDE.md`. Its resolved content hash is listed below.

| Path | SHA-256 |
|---|---|
| `AGENTS.md` | `41eb3ec376a8d78338ee45f10effdfbe8019c5bb0189550c9a0a25b6a2e05430` |
| `CLAUDE.md` | `41eb3ec376a8d78338ee45f10effdfbe8019c5bb0189550c9a0a25b6a2e05430` |
| `work/w1_instruction-dedup/builds/mirrors.r1.checks_v1.txt` | `8e7985bae4cdf65a132147383d0edcb90b1dc0792bd36cd3c4bc154c13fa5ec2` |
| `work/w1_instruction-dedup/builds/mirrors.r1.corrections_v1.txt` | `c800e7ca5be3439ac6f2483589a41e10f75eb4f0498a6da1fc43a3852c9dca37` |
| `work/w1_instruction-dedup/builds/mirrors.r1.hashes_v1.txt` | `3abaa81d28ef7ef9b46538909481d06487284af3a8c330974fa8b33dd9496937` |
| `work/w1_instruction-dedup/builds/mirrors.r1.hashes_v2.txt` | `aea62ef97bc26d6721c2a0cbd7fa2c2705755823b35ae3aac16a4cd01a7c6d7f` |
| `work/w1_instruction-dedup/builds/mirrors.r1.mirror_v1.txt` | `42dbef4514de07896d7daf28ec4426ab9b6c85be6595bf43e82d8fc00ad9645c` |
| `work/w1_instruction-dedup/builds/mirrors.r1.preflight_v1.txt` | `20faea30ab7a04c91bbf53956a00338ec645a49dbad9bba7a39acf37461c7fff` |
| `work/w1_instruction-dedup/builds/mirrors.r1.tracking_v1.txt` | `26541f5126ad86c0b15f364625fe4b8e96c474ce9f3b091f0633bc0b6c664ab1` |
| `work/w1_instruction-dedup/builds/mirrors.r1.ubs_staged_v1.txt` | `47b59aa39f9f2ac2e20dbb7f957b0d192aba6294c6ef2da0b95001f15a823042` |
| `work/w1_instruction-dedup/builds/mirrors.r1.ubs_v1.txt` | `73eb426c12fcc1ba5faaef1a7ae6e9da24655ac445c7dd6f0cb0e83bbeb21735` |
| `work/w1_instruction-dedup/builds/mirrors.r1.ubs_v2.txt` | `ab63a3783fad852d4a07906b527dfd7cf2010e9c399eda0592bc2f49e2472022` |
| `work/w1_instruction-dedup/builds/mirrors.r1.ubs_v3.txt` | `242d83145ef34ddbd698597a45008fbf9c983de5a1611544344269d93a11e70d` |
| `work/w1_instruction-dedup/builds/mirrors.r1.ubs_v4.txt` | `2f080579ca243d1c6a90493f3990e83632763db507a987d124246d20f5aa5177` |
| `work/w1_instruction-dedup/builds/mirrors.r1.ubs_v5.txt` | `701e66bb7c55687747982e863ca2d978f712df5c46dc728f0e9f9e9fb25e063d` |

## UBS failure diagnosis

1. Installed UBS 5.0.0 rejected the repo instruction's multiple-file command at argument parsing.
2. The supported `ubs --diff` retry unexpectedly auto-updated the external executable to 5.4.0.
3. UBS 5.4.0 reported zero selected files for changed, staged, full-tree, and explicit Go scans.
4. Read-only inspection found 361 Go files, an `.ubsignore` containing only `beads_reference/`, and an installed UBS module directory containing only `ubs-js.sh` and `ubs-python.sh` with no `contract.json`.
5. UBS 5.4.0 derives language extensions from that missing contract. Its file-list code therefore assigns no `.go` files to the Go language and exits 3 with “nothing scanned.”
6. No external tool repair was attempted because this worker owns only this repo's manifest outputs and evidence.

## Observed mistakes and corrections

1. `mirrors.r1.preflight_v1.txt` contains a stray Unicode character in one operation label. The hashes, command receipt, and batch are valid. `mirrors.r1.corrections_v1.txt` records the correction without rewriting evidence.
2. One symlink command call used a mistyped workdir. Process creation returned ENOENT, so no command ran. The corrected call passed.
3. The first UBS call used a syntax unsupported by installed UBS 5.0.0. The failure was preserved before using the supported interface.
4. The supported UBS retry auto-updated the external executable. This unintended side effect is recorded; no manual tool or configuration mutation followed.
5. UBS's no-scan exit was initially retried with its documented one-shot allowance. Because its own output still said nothing was checked, this was not claimed as a pass. Full and Go-specific runs established the concrete failure.
6. One diagnostic invoked the unavailable `file` utility. That subcommand failed without changing state; other diagnostics completed.
7. One tool call used the invalid argument key `cmdECH`. The tool rejected it before shell launch.
8. One hash-evidence command mistyped `ubs_v4.txt` as `ubse_v4.txt`. The partial `hashes_v1.txt` was preserved and a complete `hashes_v2.txt` was created.

## Unrelated state preserved

The pre-existing untracked dispatcher files under `work/w1_instruction-dedup/builds/` and `work/w1_instruction-dedup/supervisor/lock` were not staged or modified by the implementation commit. They include the `.log`, `.meta`, `.launch`, `.lock`, `.prompt`, `.input.md`, and `.telemetry` artifacts shown by `git status`.

## Resource cleanup

- No server, background job, subprocess session, scratch worktree, or live hook was created.
- All foreground commands exited.
- UBS managed its own temporary scan workspace; no owned process remains.
- The trash batch remains intentionally available for rollback and is not cleanup debris.
