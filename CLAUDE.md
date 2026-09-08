# beads_viewer

## Go Toolchain

- Use **Go 1.22+** (check `go.mod` for exact version).
- Build: `go build ./...`
- Test: `go test ./...` (add `-v` for verbose, `-race` for race detection)
- Vet: `go vet ./...` (run before commits)
- Format: `gofmt -w .` or `goimports -w .`

### Key Commands

```bash
go build ./...                    # Build all packages
go test ./...                     # Run all tests
go test ./... -race               # Run with race detector
go test ./pkg/analysis/... -v     # Verbose tests for specific package
go vet ./...                      # Static analysis
gofmt -w .                        # Format all Go files
```

### Module Management

- Lockfile: `go.sum` (auto-managed by `go mod`)
- Dependencies: `go mod tidy` to clean up unused deps
- Never manually edit `go.sum`

---

### Code Editing Discipline

- Do **not** run scripts that bulk-modify code (codemods, invented one-off scripts, giant `sed`/regex refactors).
- Large mechanical changes: break into smaller, explicit edits and review diffs.
- Subtle/complex changes: edit by hand, file-by-file, with careful reasoning.

---

### Logging & Console Output

- Use structured logging patterns; avoid raw `fmt.Println` for production logs.
- TUI output goes through lipgloss styling; don't mix raw prints with styled output.
- Robot mode (`--robot-*`) outputs JSON to stdout; human mode uses styled TUI.
- Errors should be wrapped with `fmt.Errorf("context: %w", err)` for traceability.

---

### Third-Party Libraries

When unsure of an API, look up current docs rather than guessing. Key dependencies:

- **bubbletea**: TUI framework (Elm architecture)
- **lipgloss**: Terminal styling
- **bubbles**: Reusable TUI components
- **cobra**: CLI framework
- **viper**: Configuration management

---

## Issue Tracking with bd (beads)

All issue tracking goes through **bd**. No other TODO systems.

Key invariants:

- `.beads/` is authoritative state and **must always be committed** with code changes.
- Do not edit `.beads/*.jsonl` directly; only via `bd`.

### Basics

Check ready work:

```bash
bd ready --json
```

Create issues:

```bash
bd create "Issue title" -t bug|feature|task -p 0-4 --json
bd create "Issue title" -p 1 --deps discovered-from:bv-123 --json
```

Update:

```bash
bd update bv-42 --status in_progress --json
bd update bv-42 --priority 1 --json
```

Complete:

```bash
bd close bv-42 --reason "Completed" --json
```

Types:

- `bug`, `feature`, `task`, `epic`, `chore`

Priorities:

- `0` critical (security, data loss, broken builds)
- `1` high
- `2` medium (default)
- `3` low
- `4` backlog

Agent workflow:

1. `bd ready` to find unblocked work.
2. Claim: `bd update <id> --status in_progress`.
3. Implement + test.
4. If you discover new work, create a new bead with `discovered-from:<parent-id>`.
5. Close when done.
6. Commit `.beads/` in the same commit as code changes.

Never:

- Use markdown TODO lists.
- Use other trackers.
- Duplicate tracking.

---

## Using bv as an AI Sidecar

bv is a graph-aware triage engine for Beads projects (.beads/beads.jsonl). Instead of parsing JSONL or hallucinating graph traversal, use robot flags for deterministic, dependency-aware outputs with precomputed metrics (PageRank, betweenness, critical path, cycles, HITS, eigenvector, k-core).

**Scope boundary:** bv handles *what to work on* (triage, priority, planning). It does not coordinate agents; the global rules cover that.

**⚠️ CRITICAL: Use ONLY `--robot-*` flags. Bare `bv` launches an interactive TUI that blocks your session.**

### The Workflow: Start With Triage

**`bv --robot-triage` is your single entry point.** It returns everything you need in one call:
- `quick_ref`: at-a-glance counts + top 3 picks
- `recommendations`: ranked actionable items with scores, reasons, unblock info
- `quick_wins`: low-effort high-impact items
- `blockers_to_clear`: items that unblock the most downstream work
- `project_health`: status/type/priority distributions, graph metrics
- `commands`: copy-paste shell commands for next steps

```bash
bv --robot-triage        # THE MEGA-COMMAND: start here
bv --robot-next          # Minimal: just the single top pick + claim command
```

### Other Commands

**Planning:**
| Command | Returns |
|---------|---------|
| `--robot-plan` | Parallel execution tracks with `unblocks` lists |
| `--robot-priority` | Priority misalignment detection with confidence |

**Graph Analysis:**
| Command | Returns |
|---------|---------|
| `--robot-insights` | Full metrics: PageRank, betweenness, HITS, eigenvector, critical path, cycles, k-core, articulation points, slack |
| `--robot-label-health` | Per-label health: `health_level` (healthy\|warning\|critical), `velocity_score`, `staleness`, `blocked_count` |
| `--robot-label-flow` | Cross-label dependency: `flow_matrix`, `dependencies`, `bottleneck_labels` |
| `--robot-label-attention [--attention-limit=N]` | Attention-ranked labels by: (pagerank × staleness × block_impact) / velocity |

**History & Change Tracking:**
| Command | Returns |
|---------|---------|
| `--robot-history` | Bead-to-commit correlations: `stats`, `histories` (per-bead events/commits/milestones), `commit_index` |
| `--robot-diff --diff-since <ref>` | Changes since ref: new/closed/modified issues, cycles introduced/resolved |

**Other Commands:**
| Command | Returns |
|---------|---------|
| `--robot-burndown <sprint>` | Sprint burndown, scope changes, at-risk items |
| `--robot-forecast <id\|all>` | ETA predictions with dependency-aware scheduling |
| `--robot-alerts` | Stale issues, blocking cascades, priority mismatches |
| `--robot-suggest` | Hygiene: duplicates, missing deps, label suggestions, cycle breaks |
| `--robot-graph [--graph-format=json\|dot\|mermaid]` | Dependency graph export |
| `--export-graph <file.html>` | Self-contained interactive HTML visualization |

### Scoping & Filtering

```bash
bv --robot-plan --label backend              # Scope to label's subgraph
bv --robot-insights --as-of HEAD~30          # Historical point-in-time
bv --recipe actionable --robot-plan          # Pre-filter: ready to work (no blockers)
bv --recipe high-impact --robot-triage       # Pre-filter: top PageRank scores
bv --robot-triage --robot-triage-by-track    # Group by parallel work streams
bv --robot-triage --robot-triage-by-label    # Group by domain
```

### Understanding Robot Output

**All robot JSON includes:**
- `data_hash` — Fingerprint of source beads.jsonl (verify consistency across calls)
- `status` — Per-metric state: `computed|approx|timeout|skipped` + elapsed ms
- `as_of` / `as_of_commit` — Present when using `--as-of`; contains ref and resolved SHA

**Two-phase analysis:**
- **Phase 1 (instant):** degree, topo sort, density — always available immediately
- **Phase 2 (async, 500ms timeout):** PageRank, betweenness, HITS, eigenvector, cycles — check `status` flags

**For large graphs (>500 nodes):** Some metrics may be approximated or skipped. Always check `status`.

### jq Quick Reference

```bash
bv --robot-triage | jq '.quick_ref'                        # At-a-glance summary
bv --robot-triage | jq '.recommendations[0]'               # Top recommendation
bv --robot-plan | jq '.plan.summary.highest_impact'        # Best unblock target
bv --robot-insights | jq '.status'                         # Check metric readiness
bv --robot-insights | jq '.Cycles'                         # Circular deps (must fix!)
bv --robot-label-health | jq '.results.labels[] | select(.health_level == "critical")'
```

**Performance:** Phase 1 instant, Phase 2 async (500ms timeout). Prefer `--robot-plan` over `--robot-insights` when speed matters. Results cached by data hash. Use `bv --profile-startup` for diagnostics.

Use bv instead of parsing beads.jsonl—it computes PageRank, critical paths, cycles, and parallel tracks deterministically.

---

### Hybrid Semantic Search (CLI)

`bv --search` supports hybrid ranking (text + graph metrics).

```bash
# Default (text-only)
bv --search "login oauth"

# Hybrid mode with preset
bv --search "login oauth" --search-mode hybrid --search-preset impact-first

# Hybrid with custom weights
bv --search "login oauth" --search-mode hybrid \
  --search-weights '{"text":0.4,"pagerank":0.2,"status":0.15,"impact":0.1,"priority":0.1,"recency":0.05}'

# Robot JSON output (adds mode/preset/weights + component_scores for hybrid)
bv --search "login oauth" --search-mode hybrid --robot-search
```

Env defaults:
- `BV_SEARCH_MODE` (text|hybrid)
- `BV_SEARCH_PRESET` (default|bug-hunting|sprint-planning|impact-first|text-only)
- `BV_SEARCH_WEIGHTS` (JSON string, overrides preset)

---

### Static Site Export for Stakeholder Reporting

Generate a static dashboard for non-technical stakeholders:

```bash
# Interactive wizard (recommended)
bv --pages

# Or export locally
bv --export-pages ./dashboard --pages-title "Sprint 42 Status"
```

The output is a self-contained HTML/JS bundle that:
- Shows triage recommendations (from --robot-triage)
- Visualizes dependencies
- Supports full-text search (FTS5)
- Works offline after initial load
- Requires no installation to view

**Deployment options:**
- `bv --pages` → Interactive wizard for GitHub Pages deployment
- `bv --export-pages ./dir` → Local export for custom hosting
- `bv --preview-pages ./dir` → Preview bundle locally

**For CI/CD integration:**
```bash
bv --export-pages ./bv-pages --pages-title "Nightly Build"
# Then deploy ./bv-pages to your hosting of choice
```

---

## Testing Guidelines

### Never Open Browsers

**Tests must NEVER automatically open a browser.** All browser-opening functions check `BV_NO_BROWSER` and `BV_TEST_MODE` environment variables. These are set globally via `TestMain` in:
- `tests/e2e/common_test.go`
- `pkg/export/main_test.go`
- `pkg/ui/main_test.go`

When adding new browser-opening code, always check these env vars first:
```go
if os.Getenv("BV_NO_BROWSER") != "" || os.Getenv("BV_TEST_MODE") != "" {
    return nil
}
```

### Test Commands

```bash
go test ./...                           # All tests
go test ./pkg/analysis/... -v           # Verbose for specific package
go test ./tests/e2e/... -v              # E2E tests
go test ./... -race                     # With race detector
go test ./... -cover                    # With coverage
go test -run TestSpecificName ./pkg/... # Run specific test
```

### Test Patterns

- Use table-driven tests for multiple cases
- Use `t.TempDir()` for temporary files
- Use `t.Helper()` in test helpers
- Check `testing.Short()` for long-running tests

---

## Go Best Practices

Follow all practices in `GOLANG_BEST_PRACTICES.md`. Key points:

### Error Handling

```go
// Always wrap errors with context
if err != nil {
    return fmt.Errorf("loading config: %w", err)
}

// Check errors immediately after the call
result, err := doSomething()
if err != nil {
    return err
}
```

### Division Safety

```go
// Always guard against division by zero
if len(items) > 0 {
    avg := total / float64(len(items))
}
```

### Nil Checks

```go
// Check for nil before dereferencing
if dep != nil && dep.Type.IsBlocking() {
    // safe to use dep
}
```

### Concurrency

```go
// Use sync.RWMutex for shared state
mu.RLock()
value := sharedMap[key]
mu.RUnlock()

// Capture channels before unlock to avoid races
mu.RLock()
ch := someChannel
mu.RUnlock()
for item := range ch {
    // process
}
```

---

## ast-grep vs ripgrep

**Use `ast-grep` when structure matters.** It parses code and matches AST nodes, so results ignore comments/strings, understand syntax, and can safely rewrite code.

- Refactors/codemods: rename APIs, change patterns
- Policy checks: enforce patterns across a repo

**Use `ripgrep` when text is enough.** Fastest way to grep literals/regex.

- Recon: find strings, TODOs, config values
- Pre-filter: narrow candidates before precise pass

**Go-specific examples:**

```bash
# Find all error returns without wrapping
ast-grep run -l Go -p 'return err'

# Find all fmt.Println (should use structured logging)
ast-grep run -l Go -p 'fmt.Println($$$)'

# Quick grep for a function name
rg -n 'func.*LoadConfig' -t go

# Combine: find files then match precisely
rg -l -t go 'sync.Mutex' | xargs ast-grep run -l Go -p 'mu.Lock()'
```

---

## UBS Quick Reference

UBS = "Ultimate Bug Scanner" — static analysis for catching bugs early.

**Golden Rule:** `ubs <changed-files>` before every commit. Exit 0 = safe. Exit >0 = fix & re-run.

```bash
ubs file.go file2.go                    # Specific files (< 1s)
ubs $(git diff --name-only --cached)    # Staged files
ubs --only=go pkg/                      # Go files only
ubs .                                   # Whole project
```

**Output Format:**
```
⚠️  Category (N errors)
    file.go:42:5 – Issue description
    💡 Suggested fix
Exit code: 1
```

**Fix Workflow:**
1. Read finding → understand the issue
2. Navigate `file:line:col` → view context
3. Verify real issue (not false positive)
4. Fix root cause
5. Re-run `ubs <file>` → exit 0
6. Commit

**Bug Severity (Go-specific):**
- **Critical**: nil dereference, division by zero, race conditions, resource leaks
- **Important**: error handling, type assertions without check
- **Contextual**: TODO/FIXME, unused variables
