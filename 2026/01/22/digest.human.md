Here's the Git mailing list digest for January 22, 2026:

---

### The day in brief
January 22, 2026 saw high-volume technical activity across multiple fronts, with 98 emails spanning 21 active threads. The day was dominated by security debates around sideband sanitization defaults, major refactoring efforts nearing completion (ODB abstraction, hook subsystem), and several long-running series reaching final approval. Key developments include Junio's decisive stance on sideband security defaults and the completion of Harald Nordgren's configurable branch comparison feature after 28 iterations.

---

### Notable threads

**Security debate over sideband control character defaults**
The long-running discussion about ANSI escape sequence injection (CVE-2024-32002, CVE-2024-52005) reached a critical juncture as maintainer Junio Hamano overruled Johannes Schindelin's security-conscious defaults, advocating instead for opt-in sanitization. Schindelin had argued for strict defaults based on production validation in Git for Windows and Red Hat systems, while brian m. carlson raised backward compatibility concerns. Junio's intervention prioritizes existing workflows over security hardening, marking a significant policy decision that may influence future security discussions.

**Hook subsystem modernization completes**
Adrian Ratiu's multi-year effort to refactor Git's hook infrastructure reached its final stages with v8 patches addressing I/O performance in parallel execution. The series standardizes the hook API with `feed_pipe_fn`/`consume_output_fn` callbacks and introduces parallel execution via `struct parallel_child`. Today's patches focused on bidirectional polling optimizations to eliminate 100ms latency issues identified by Kristoffer Haugsbakk, with only minor documentation cleanups remaining before integration.

**Ref backend selection series progresses**
Karthik Nayak confirmed continued work on enabling zero-downtime migrations between ref storage backends (files<->reftable), despite Junio temporarily removing the series from 'seen' for housekeeping. The implementation has settled on config-based URI syntax via `extension.refStorage` but faces its final technical hurdle: proper worktree reference handling during repository initialization. With architectural decisions resolved and comprehensive test coverage in place, the series awaits Karthik's return from vacation to address this last challenge.

**PID file debugging merges**
Paulo Casaretto's PID file feature for lock debugging received final approval after six iterations, adding `core.lockfilePid` configuration to create companion `~pid.lock` files. The implementation handles Windows compatibility via mingw_kill() and includes a 139-line test suite (t0031) verifying behavior across platforms. Jeff King, Patrick Steinhardt, and Taylor Blau contributed to the robust solution that addresses a long-standing debugging pain point while carefully documenting PID reuse limitations.

**Configurable branch comparison finalized**
Harald Nordgren's 28-iteration effort to make `git status` branch comparisons configurable via `status.compareBranches` was merged after addressing Jeff King's optimization to use `strset` for deduplication. The feature allows comparing against multiple branches (including `@{upstream}` and `@{push}`) with context-aware advice messages. A post-merge discussion emerged about expanding refname resolution rules, with Junio advocating for immediate adoption of DWIM rules rather than limiting comparisons to remote-tracking branches.

---

### In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Submodule remote handling** -- Fixes 7-year-old NEEDSWORK by adding `get-default-remote` helper to properly handle submodules with non-"origin" remotes.

**Batched ref update errors** -- Karthik Nayak and Jeff King complete their series restoring detailed error messages in batched reference operations.

**UTF-8 diffstat alignment** -- Lorenzo Pegorari's fix for UTF-8 filename width calculation in `git diff --stat` is confirmed ready for merging.

**Mailmap consolidation** -- Jean-Noël Avila's contributor identity mappings are finalized in `.mailmap` following a release note attribution error.

**String list utilities** -- Amisha Chhajed proposes `string_list_sort_u()` to combine common sort+dedupe operations, awaiting tests per Junio's feedback.

---

### On the radar

**ODB abstraction series** -- Patrick Steinhardt's object storage refactoring faces final merge conflicts in `packfile.c` between prerequisite series, with architectural questions about path-based access resolved.

**Rev-list frontier commits** -- Derrick Stolee's `--maximal-only` option for `git rev-list` receives conceptual clarification from Junio regarding its relationship to `merge-base --independent`.

**Repository config migration** -- Olamide Bello's `the_repository` removal effort progresses with config variable migration, though Phillip Wood identified a critical gap in non-primary repository handling.

**GSoC 2026 planning** -- Christian Couder and Kaartic Sivaraam assemble mentor team with GitLab support, seeking additional project ideas and mentors before the January 19.

--- 

The day showcased Git's development at both technical and process levels - from deep architectural discussions to the careful polishing of long-running features. With multiple major efforts reaching completion and new RFCs emerging, the project maintains its characteristic balance between stability and innovation.