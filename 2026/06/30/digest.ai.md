# The Git Project Mailing List Daily Digest

**The day in brief.**
June 30, 2026 (UTC) was a busy day on the Git mailing list, with **94 emails across 23 threads**. The day was dominated by **refactoring and infrastructure work**, particularly Patrick Steinhardt’s ongoing ODB abstraction efforts and repository setup cleanup. Notable threads included **security hardening for the reftable backend**, **memory leak fixes**, and **usability discussions** around `git blame` and `git history`. One long-running feature (`git replay --linearize`) reached a post-merge regression debate, while another (`git history squash`) neared completion with final usability polish.

---

## Notable threads

### ODB abstraction stack advances: `ps/odb-generalize-prepare` approved
**Topic:** `[PATCH v1 0/2] odb: generalize reprepare() callback for pluggable ODBs`
**Author:** Patrick Steinhardt
**Status:** **Approved and ready for `next`**

Patrick Steinhardt’s two-patch series to generalize the `reprepare()` callback into a more flexible `prepare()` API received **final approval** from both Toon Claes and Junio C Hamano. The series enables `git grep` to work with pluggable ODB backends by decoupling cache invalidation from ODB preparation, replacing the old `reprepare()` with a new `odb_prepare()` that accepts an `ODB_PREPARE_FLUSH_CACHES` flag. Toon’s earlier concerns about downcasting and flag semantics were fully resolved, and Junio explicitly approved the series for advancement to `next`. This is a key milestone in Patrick’s broader ODB abstraction effort, which aims to make Git’s object storage layer backend-agnostic.

---

### Reftable security hardening: fuzzer finds quadratic-time behavior
**Topic:** macOS CI hang in t5551/t5559 – root cause and fix
**Author:** Jeff King (Peff)
**Status:** **Post-merge analysis reveals deeper performance issue**

After merging the fix for CI hangs in `t5551` and `t5559` (caused by Apache’s `mod_http2` bug 70131), Jeff King dug deeper into the reftable backend’s performance and uncovered a **quadratic-time behavior** during bulk ref deletion and re-creation. When deleting and re-creating 8,000+ refs, runtime grew from 1.8s to 7.1s for a 2x input size increase, suggesting inefficient iteration over tombstone entries. Peff’s analysis also quantified the impact of redundant `stat()` calls during bulk ref creation (2,000 calls consuming ~25% of runtime) and unbuffered reflog writes (potential 2x speedup with stdio). While these optimizations are not urgent, they provide a clear roadmap for future reftable improvements. The thread also saw the **acceptance of a GitLab CI badge** in `README.md`, addressing Junio’s visibility concern.

---

### `git replay --linearize` post-merge regression sparks design debate
**Topic:** `[PATCH v5 0/3] replay: introduce --linearize option`
**Author:** Toon Claes
**Status:** **Merged, but three critical issues identified**

Toon Claes’s `git replay --linearize` feature, which flattens merge commits into a linear history, was merged to `master` but immediately revealed **three critical issues**:
1. **Silent commit dropping regression** in single-branch replay (highest priority, root cause confirmed).
2. **CLI design inconsistency** with `git rebase` (now a philosophical debate about UX vs. consistency).
3. **Merge commit divergence handling** (secondary concern).

Johannes Schindelin identified the regression, where intermediate commits (including merges) were silently dropped, and argued that `--linearize` should produce a **single linear sequence regardless of input branches**. Patrick Steinhardt raised the CLI inconsistency, noting that `--linearize` deviates from `git rebase`’s `--rebase-merges` syntax. The debate evolved into a **process-level discussion** about documenting design decisions in commit messages, with Patrick conceding the UX argument but demanding explicit justification for any divergence. A follow-up patch to fix the regression is urgently needed, and the CLI design question remains unresolved.

---

### `git history squash` nears completion with final usability polish
**Topic:** `[PATCH v6 0/2] history: add squash subcommand`
**Author:** Harald Nordgren
**Status:** **Ready for Junio’s final review; minor usability questions remain**

Harald Nordgren’s `git history squash` feature, which folds a range of commits into one while replaying descendants, is **code-complete and ready for Junio’s final review**. The series addresses all prior feedback, including **merge commit handling**, **input validation**, and **`--reedit-message` template formatting**. However, **user feedback** from Matt Hunter and Phillip Wood raised **usability questions**:
- Should `--reedit-message` be the default (aligning with `git rebase -i`’s interactive flow)?
- How should `squash!` messages be treated in the template (retained vs. omitted)?
- **Recoverability concerns** if `--update-refs` moves branches unexpectedly.

Phillip Wood expanded the recoverability critique into a **systemic problem**, noting that Git lacks a way to undo multi-ref operations atomically and suggesting **reflog transaction IDs** as a potential solution. Harald deferred further template improvements to maintain consistency with `git rebase -i`, but the default behavior and recoverability questions may inspire future work. The series is otherwise uncontroversial and well-tested.

---

### Sequencer memory leaks fixed in systemic overhaul
**Topic:** `[PATCH 0/11] sequencer: avoid copying notes from dropped commits during rebase`
**Author:** Phillip Wood
**Status:** **Complete and queued for final review**

Phillip Wood’s 11-patch series **systemically overhauls how the sequencer handles dropped commits during rebase**, fixing a long-standing bug where notes from dropped commits were incorrectly copied to the current HEAD. The series also addresses **external merge strategy failures**, **command execution failures**, and **final fixup cleanup**, making the rebase machinery more robust. Junio queued the series with minor typofixes after resolving a `b4` tooling issue, and no technical objections remain. The core fix (patch 11/11) introduces a `PICK_RESULT_DROPPED` enum member and modifies `pick_one_commit()` to skip `record_in_rewritten()` for dropped commits. The series is a **tested-level contribution** that directly addresses technical debt in the rebase subsystem.

---

### Repository setup refactoring: discovery and setup phases separated
**Topic:** [13-patch series] Separate repository discovery from setup
**Author:** Patrick Steinhardt
**Status:** **Initial submission complete; Junio queuing patches**

Patrick Steinhardt’s 13-patch series **separates repository discovery from setup** in Git’s core code, introducing a new `struct repo_discovery` to hold discovery results and consolidating repository configuration logic. The series is a foundational refactor for the `the_repository` removal effort, with key changes including:
- Moving the prefix and worktree configuration into the discovery phase.
- Extracting the discovery phase into a new `repo_discover()` function.
- Making the worktree path an explicit parameter to `init_db()`.
- Removing `set_git_work_tree()` from the public API.

Junio began queuing patches, starting with patch 2/13 after fixing a minor typo. The series is well-structured, with each patch addressing a specific concern, and no behavior changes are intended. Reviewers may focus on the new `repo_discovery` API and its implications for edge cases like environment variable overrides or worktree configuration.

---

## In brief

> **Memory leak fixes in `git format-patch` and test harness** -- Jeff King fixed two small leaks: one in `prepare_bases()` (allocated `rev_info` struct) and one in the test harness (LSan output misdirected to stdout). Patrick Steinhardt proposed enabling LSan for the `linux-TEST-vars` CI job to catch similar leaks earlier.

> **`git refs` subcommands merged** -- Patrick Steinhardt’s five-patch series adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` was merged. The series consolidates reference manipulation under a unified interface, improving discoverability without altering on-disk formats.

> **`git blame -b` output formatting fix** -- René Scharfe posted a patch to fix a usability friction where `git blame -b` reserved an extra hex digit for an unused caret marker, causing commit hashes to exceed `core.abbrev` by one character. The patch refactors mark-handling logic to align hash lengths with user settings.

> **`USE_NSEC` debate continues** -- The discussion about whether to flip the default of `USE_NSEC` (sub-second file timestamp tracking) or convert it to a runtime setting stalled on **auto-detection challenges**. Jeff King’s testing showed no reliable, portable way to detect filesystem timestamp preservation, complicating Patrick Steinhardt’s proposal for runtime configuration.

> **Test modernization in `t7412-submodule-absorbgitdirs.sh`** -- Bryan B. Lima replaced raw test primitives with descriptive helpers, improving test failure messages. Junio queued the patch, calling it "perfect" for a first-time contributor.

> **Rustification build system fix** -- Jan Palus removed a spurious Makefile dependency that forced the Rust static library to rebuild whenever the C library changed, even though the Rust code doesn’t link against it.

---

## On the radar

- **`git replay --linearize` regression fix** -- A follow-up patch is urgently needed to address the silent commit dropping regression in Toon Claes’s merged feature. The CLI design debate (UX vs. consistency) may require maintainer intervention.
- **`git history squash` default behavior** -- Phillip Wood’s suggestion to make `--reedit-message` the default (for commit hygiene) remains unresolved. Junio may need to weigh in on whether `git history` should prioritize automation or interactivity.
- **Reftable performance optimizations** -- Jeff King’s analysis of quadratic-time behavior and redundant `stat()` calls in the reftable backend may inspire future work, though no immediate action is planned.
- **`USE_NSEC` runtime configuration** -- The lack of reliable auto-detection leaves the proposal for a runtime config knob (e.g., `core.useNsec`) in limbo. The project may need to choose between a conservative default or retaining the build-time knob.