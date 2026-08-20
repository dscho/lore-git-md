# Git mailing list daily digest for 2026/08/19

## The day in brief
The Git mailing list saw significant progress on several fronts today. The zsh completion bugfix for global options (`-C`, `--git-dir`, etc.) reached a stable v2, while the ODB abstraction for `git receive-pack` advanced with a comprehensive v4 series addressing all prior feedback. The `git history` bash completion series received final endorsements and is poised for integration. Meanwhile, design discussions continued around the new `report` hook for `git receive-pack` and reftable backend optimizations.

## Notable threads

### zsh completion: fix for global options
**What changed**: Lutz Lengemann posted v2 of a bugfix patch that aligns zsh completion with bash's handling of global Git options like `-C <path>`, `--git-dir`, and `-p`. The patch expands scope beyond just `-C` to cover all global options, addressing the core issue where the zsh wrapper hard-coded `__git_cmd_idx=1`.

**Problem/goal**: The zsh completion script failed to complete commands or arguments when global options were used (e.g., `git -C <path> <command> <TAB>`), due to incorrect command-index calculation.

**Subsystem**: Shell completion (`contrib/completion/git-completion.zsh`)

**Impact**: This is a long-standing usability bug affecting zsh users who rely on global options. The fix is low-risk (25 lines added) and mirrors the robust bash implementation.

**Today's developments**: The v2 patch addresses all prior feedback from Junio C Hamano and D. Ben Knoble, expanding coverage to all global options and documenting remaining limitations (e.g., chained `-C` directory completion). The patch is effectively unblocked and ready for merging.

---

### ODB abstraction for `git receive-pack`
**What changed**: Justin Tobler posted v4 of a nine-patch series completing the ODB abstraction for `git receive-pack` by introducing the generic `odb_transaction_write_pack()` interface. This iteration addresses all v3 feedback, including fixes for `.keep` file placement, context-aware unpack limit resolution, and transaction lifecycle management.

**Problem/goal**: Replace `git receive-pack`'s hardcoded use of `git index-pack` and `git unpack-objects` (which assume the "files" backend) with a backend-agnostic transaction interface, enabling alternative ODB backends (e.g., reftable, cloud storage).

**Subsystem**: Object database (ODB) abstraction

**Impact**: This is a critical step toward making Git's object storage layer pluggable. The series is technically complete, with all design flaws addressed and the interface refined based on reviewer feedback.

**Today's developments**:
- Patch 1/9 fixes a regression in `.keep` file cleanup by updating `index_pack_lockfile()` to accept an explicit `struct odb_source *`.
- Patch 2/9 splits `odb_transaction_commit()` into `commit()` and `finalize()` phases to enable deferred cleanup (e.g., `.keep` file removal).
- Patches 3–5/9 remove global state (shallow file, unpack limit, fsck settings) by passing configuration explicitly.
- Patches 6–7/9 refactor error reporting and file descriptor passing to align with ODB transaction patterns.
- Patch 8/9 adds explicit ODB source tracking to avoid fragile source list ordering.
- Patch 9/9 introduces the `odb_transaction_write_pack()` interface, moving packfile ingestion logic into the "files" backend.

The series is now ready for integration, pending any final surface-level feedback.

---

### Bash completion for `git history`
**What changed**: Vincent Mailhol's four-patch v3 series adding bash completion for `git history` received final endorsements from Patrick Steinhardt. Junio C Hamano confirmed plans to handle the trivial merge conflict with the in-flight `squash` subcommand during integration.

**Problem/goal**: Add comprehensive bash completion support for `git history`'s subcommands (`drop`, `fixup`, `reword`, `split`, and soon `squash`), including their options, option values, and positional arguments.

**Subsystem**: Shell completion (`contrib/completion/git-completion.bash`)

**Impact**: This series fills a usability gap for users of the `git history` command, with minimal risk (no on-disk format changes) and thorough test coverage.

**Today's developments**:
- Patrick Steinhardt explicitly endorsed the v3 series as "a good base to build on," signaling readiness for integration.
- Junio confirmed he will handle the trivial merge conflict with the `squash` subcommand as an "evil merge" during integration into `seen`.
- The series is now unblocked and ready for `next`.

---

### `report` hook for `git receive-pack`
**What changed**: Karthik Nayak's patch adding a new `report` hook for `git receive-pack` saw continued design discussion, with agreement on treating non-zero exit status as a "nuclear option" that discards stdout.

**Problem/goal**: Allow server administrators to intercept or modify the status report sent to clients after ref updates are committed, enabling use cases like GitLab's MVCC (multi-version concurrency control).

**Subsystem**: Hooks (`git-receive-pack`)

**Impact**: This hook fills a timing gap not covered by existing hooks (`pre-receive`, `update`, etc.), but its design (exit status vs. stdout semantics) remains under discussion.

**Today's developments**:
- Patrick Steinhardt and Karthik Nayak agreed that a non-zero exit status should be treated as a "nuclear option" for exceptional cases, discarding stdout and causing `receive-pack` to die.
- Documentation updates are planned to clarify this behavior in `githooks.adoc` and `git-receive-pack(1)`.
- The unresolved question of where to document the `pkt-line` format remains open.

---

### Reftable backend optimizations
**What changed**: Karthik Nayak posted a three-patch series optimizing reftable stack reloads during transactions, reducing `fstat()` calls from O(n) to ~55 constant. Junio C Hamano raised a safety concern about the lock relocation in patch 2/3.

**Problem/goal**: Eliminate redundant stack reloads in the reftable backend during reference transactions, addressing a performance bottleneck identified by Jeff King.

**Subsystem**: Reftable backend (`refs/reftable-backend.c`)

**Impact**: The optimization shows consistent 1–2% clock-time improvements in `git-update-ref` benchmarks, with more significant reductions in `newfstatat()` syscalls.

**Today's developments**:
- Junio raised a safety concern about patch 2/3: if multiple `reftable_addition` instances operate on the same stack, the shared lock could be prematurely released when the second instance closes.
- Justin Tobler endorsed the lock relocation as conceptually sound but deferred to Junio's safety concern.
- The series remains under review pending resolution of this issue.

---

### ODB corruption handling refactoring
**What changed**: Patrick Steinhardt posted v2 of a five-patch series refactoring how the ODB handles corrupt objects when `OBJECT_INFO_DIE_IF_CORRUPT` is used. Junio C Hamano identified a latent correctness issue in the error-reporting mechanism.

**Problem/goal**: Make handling of corrupt objects consistent and backend-agnostic, ensuring error messages are generated by the backend that encountered the failure.

**Subsystem**: Object database (ODB)

**Impact**: This refactoring improves error reporting consistency and removes abstraction-breaking code, but a latent issue with duplicate objects in corrupt packs remains unresolved.

**Today's developments**:
- Junio identified a latent correctness issue: if a corrupt pack contains duplicate objects, the error message cannot distinguish which copy was corrupt.
- The series is otherwise technically complete, with all prior feedback addressed.

---

## In brief
- **GSoC 2026 – Partial clones**: Siddharth Shrimali's progress report detailed finalized design for the `gc-promisor` maintenance task, including per-remote `.keep`-file safety mechanisms and `--dry-run` support.
- **`git worktree add` regression fix**: Yoichi Nakayama's bugfix for DWIM behavior with `-b/-B` was merged to `master`.
- **`http.sslVerifyStatus`**: Patrick Steinhardt provided a complete OCSP stapling test harness, addressing the last major test-coverage gap for graysongordon-gl's patch.
- **`core.useNanosec`**: D. Ben Knoble accepted all of Patrick Steinhardt's suggestions, including adding a platform/filesystem warning and unconditionally initializing the `use_nanosec` field.
- **Ambiguous remote branch names**: Yoichi Nakayama posted v5 of a series improving error messages for `git checkout`, `git switch`, and `git worktree add`, unifying the advice logic via a shared helper.
- **Geometric repacking threshold**: Patrick Steinhardt explained the historical rationale for the 6700 threshold in `git gc --auto`, reinforcing its stability and pragmatism.
- **Refname character tests**: Nikolaus Schuetz's patch adding coverage for documented but untested refname characters received endorsements for a refactoring to loop over forbidden characters.
- **`git symbolic-ref` exit codes**: Patrick Steinhardt questioned the design rationale for differing exit codes (128 vs. 1 with `--quiet`) and proposed additional test coverage.
- **Bash completion performance**: Matthew Hughes proposed a refined workaround for slow completion in large repositories, selectively disabling index-based completion for "slow" options.
- **`git stash` non-atomic failure**: Yuri reported that `git stash push` fails non-atomically for large (2GB) untracked files, leaving invalid stash entries and unhelpful error messages.
- **Trace2 for `git pack-objects`**: friel@openai.com posted v2 of a patch adding a Trace2 event for total bytes written, simplifying the byte-accumulation logic.