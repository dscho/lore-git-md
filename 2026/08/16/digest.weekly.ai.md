# The Git Project -- Weekly Digest for 2026/08/10 -- 2026/08/16

## The period in brief

This week (2026/08/10--2026/08/16) saw **heavy traffic** (450+ emails across 120+ threads) with **three critical regressions**, a **security flaw in `gitk`**, and **major architectural advances** in the ODB abstraction effort. The most consequential developments were the `git rebase` submodule regression, the `git maintenance` geometric repacking concurrency bug, and Justin Tobler’s ODB transaction extension to `git receive-pack`. The week also featured **design debates** over source tree reorganization, filesystem-level copy-on-write for `git worktree`, and a proposed `git squash` command.

---

## Key developments

### Critical regressions in `git rebase` and `git maintenance`
Two high-impact regressions surfaced this week, both introduced in Git 2.54 and affecting common workflows. The `git rebase` bug (`bb5da75d61`) causes fatal errors (`invalid commit position. commit-graph is likely corrupt`) when submodules are involved, due to `lookup_commit_reference_gently()` resolving submodule OIDs against the main repository’s commit graph. Patrick Steinhardt confirmed the issue persists on `master` and requested a minimal reproducer. Meanwhile, Stefan Haller reported **two critical issues** in `git maintenance`’s geometric repacking: a **concurrency failure** where `prune_packed_objects()` removes fanout directories while concurrent writers attempt to create temporary files, and a **misconfigured auto-condition** (`geometric-repack.auto`) that triggers 256× more aggressively than documented. Patrick Steinhardt proposed raising the default threshold from 100 to 6700 to match `gc.auto`, but the underlying race condition remains unaddressed. These regressions are **blockers for the next release** and demand urgent attention.

### `gitk` security flaw: arbitrary command injection
Tim Wiederhake’s `gitk` patch adding user-configurable custom commands contained a **critical security flaw**: unquoted placeholder substitution (e.g., `%t` for commit titles) allowed arbitrary command injection via crafted commit messages. Junio C Hamano provided a concrete exploit example (`title?'; echo no'`). The author is evaluating two fixes: reverting to direct command execution (requiring wrapper scripts) or implementing proper escaping for all interpolated values. This flaw is a **blocker** for the feature, which aims to extend `gitk`’s workflow flexibility. The discussion highlights the tension between usability and security in Git’s UI tooling.

### ODB abstraction: transactions extended to `git receive-pack`
Justin Tobler’s nine-patch series (v3) extended ODB transactions to `git receive-pack`, addressing all feedback from v2. Key improvements include:
- Fixing a **critical lockfile lifecycle flaw** in the "files" backend via explicit `struct odb_source *` tracking.
- Introducing `odb_transaction_finalize()` and `odb_transaction_commit_and_finalize_or_die()` to enforce proper transaction lifecycle management.
- Removing reliance on global variables (e.g., `alt_shallow_file`, unpack limit caching) by introducing a `struct unpack_opts` and explicit file descriptor passing.
- Adding `odb_transaction_write_pack()`, the generic interface completing the ODB abstraction for `git receive-pack`.

The series is a **major milestone** in Patrick Steinhardt’s ODB abstraction effort, which aims to support alternative backends (e.g., reftable, cloud storage). Junio C Hamano’s feedback focused on transaction lifecycle and source-tracking design debt, but the series is now cooking in `seen` and poised for advancement.

### `git repack --drop-filtered` for partial clones (v4)
Siddharth Shrimali’s `git repack --drop-filtered` series (v4) enables users to reclaim disk space in partial clones by safely removing locally cached promisor blobs exceeding a size threshold. The series is **feature-complete** (7/7 patches) and addresses all prior feedback, including Junio’s mechanical fixes. The **safety guards** (merge/rebase checks, index validation) are now framed as UX optimizations, not correctness measures. Junio’s proposed time-based heuristic (avoiding recently fetched objects) remains deferred. The series is **production-ready** and proposed for `next`, marking a long-awaited feature for partial clone users.

### Source tree reorganization: data-driven counterproposal
Michael Montalbo proposed a **data-driven methodology** to identify cohesive subsystems for incremental reorganization, addressing Junio C Hamano’s critique of Patrick Steinhardt’s `lib/` directory approach. The proposal uses **dual-signal analysis** combining commit history patterns (weighted by commit size) and call-graph relationships (derived from `cscope`) to group 163 of Git’s 231 `libgit.a` source files into 14 directories (e.g., `transport/`, `index/`, `revision/`). Boundary cases (e.g., `shallow.c` grouped with `transport/` despite its `revision:` commit label) are flagged as review targets. The script’s innovation lies in its **mechanical, reproducible methodology**, avoiding the arbitrariness of the `lib/` approach while enabling incremental progress. The proposal reframes the debate around **how** to reorganize, not **whether**, and could break the deadlock if the community finds the methodology compelling.

### Filesystem-level copy-on-write for `git worktree add`
Peter Morris proposed leveraging **filesystem-level copy-on-write (CoW)** or block cloning (e.g., ReFS, Btrfs, APFS) for `git worktree add` to avoid physically duplicating data until modification. The RFC targets scenarios like concurrent AI coding agents, where multiple worktrees are created for the same repository. Junio C Hamano outlined the core architectural challenge: the checkout machinery (`checkout_entry()`) lacks visibility into sibling worktrees. He proposed intercepting `checkout_entry()` to perform CoW based on a mapping of blob objects to cleanly checked-out files across all worktrees, while flagging TOCTOU races and side effects on other commands. Brian M. Carlson later reported that ReFS’s block cloning implementation is unreliable when data has been written but not flushed to disk, risking corruption. The proposal is architecturally feasible but high-effort, with implications for correctness, performance, and cross-platform consistency.

### `git squash` command proposal
The Super Techie proposed a new `git squash <n>` command to combine the last `<n>` commits into a single new commit, equivalent to `git reset --soft HEAD~<n> && git commit`. An optional `-s` flag would add a Signed-off-by trailer. Matej Dujava noted that `git rebase --autosquash` with `fixup!`/`squash!` commits already provides a more flexible solution, and linked to a v13 `git history squash` patch series as prior discussion. The thread now encompasses two distinct approaches: the original simple `git squash <n>` and the more flexible `git history squash`. No maintainer guidance has emerged, but the proposal highlights ongoing demand for simpler commit management workflows.

---

## In brief

**`git send-email`** -- Harald Nordgren’s bugfix for missing "Subject:" headers was approved and marked for `next`.

**`git worktree add`** -- Yoichi NAKAYAMA’s v3 patch improving error messages for ambiguous remote branches addressed Junio’s feedback on commit-message clarity. A separate bugfix by Yoichi NAKAYAMA corrected dead code in `.git` file path resolution during worktree repair, eliminating branches that could never trigger due to `read_gitfile_gently()`’s behavior.

**`git maintenance` promisor packs** -- Taylor Blau’s two-patch bugfix for geometric repacking was reviewed by Patrick Steinhardt, who noted a potential edge case (now resolved).

**`git history` Bash completion** -- Vincent Mailhol’s v2 series added completion for the new `git history` command, with D. Ben Knoble advocating for a simplified approach to avoid `--option <value>` syntax.

**GSoC updates** -- Siddharth Shrimali’s Week 11 progress on partial clone disk-space recovery and `gc-promisor` maintenance task were shared.

**CI/build system** -- Jeff King’s patch to bump the Ubuntu image for static-analysis jobs was contested after SZEDER Gábor’s benchmarks showed Coccinelle 1.3.1 as **4.5× slower** than 1.1.1. Johannes Schindelin’s MinGW/Windows build series (v2) addressed cross-build-system inconsistencies, with real-world validation from Johannes Sixt.

**`gitk` color-preference dialog** -- Johannes Sixt’s v2 series overhauled the gitk color-preference dialog, consolidating color-setting logic, refactoring repetitive Tcl/Tk code, making color swatches clickable, updating UI labels to natural language, ensuring translatability, and reordering color options.

**`fetch_if_missing` libification** -- Tian Yuchen’s v6 series moving the global `fetch_if_missing` variable into `struct repository` is mechanically complete and ready for queuing, preserving existing behavior while aligning with Git’s libification goals.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder’s series (replacing `GIT_NO_LAZY_FETCH=fromAccepted`) is stalled on a submission format issue but addresses a key security objection. Junio’s reviews raised stylistic improvements, scalability concerns, API design critiques, and documentation ambiguities.

**`git rev-parse`** -- Dimitri John Ledkov introduced `repo_oid_to_algop()` to compute object names in a different hash algorithm on demand, enabling interoperability between repositories using different object formats (e.g., SHA-1 and SHA-256).

**Submodule** -- Tilak Raaz (GSoC microproject) proposed warning on valueless `submodule.active` configuration entries, with Junio questioning whether the command should fail or continue after encountering a valueless key.

**`git repo info`** -- K Jayatheerth’s seven-patch series adding path-related keys to `git repo info` faced a design objection to the `path.git-prefix` key, which exposes the relative path from the repository’s working tree root to the current directory. Lucas Seiki Oshiro argued it represents "current directory information" rather than "repository information," mismatching the command’s intended scope.

**Bash completion slowdown** -- Matthew Hughes reported a **performance regression** in bash completion for large repositories (1–2 s hang for `git add ./<tab>` in repositories with ~368,000 files), caused by `git ls-files --others --modified --directory` with a wildcard pattern. The workaround (`GIT_COMPLETION_NO_COMPLETE_INDEX`) sacrifices accuracy for speed, but no deeper fix has been proposed.

**`chdir_notify` refactoring** -- Colin Hinton’s patch removing the unused `name` parameter from the `chdir_notify` API was updated to document the historical context of the parameter’s deprecation, addressing Jeff King’s review feedback.

**Experimental bitmap configuration** -- kapicharly@icloud.com introduced documentation for experimental `bitmapPseudoMerge.<name>.*` options, clarifying the distinction between "unstable" and "stable" pseudo-merge bitmaps.

**Memory-safety fix** -- René Scharfe finalized the fix for the memory-safety flaw in `worktree_basename()`, replacing a pointer-decrement loop with an integer-index loop to eliminate underflow risk. The patch is ready for `next` and addresses a critical bug that could lead to data corruption.

---

## Looking ahead

**Regressions demand urgency** -- The `git rebase` submodule regression and `git maintenance` geometric repacking bugs are **blockers for the next release** and will dominate the coming week. Minimal reproducers and fixes are expected.

**ODB abstraction matures** -- Justin Tobler’s ODB transaction series and Patrick Steinhardt’s pluggable packfile generation patches show steady progress toward backend-agnostic Git. The next steps include addressing Junio’s feedback on transaction lifecycle and source-tracking design debt.

**Source tree reorganization** -- Michael Montalbo’s data-driven counterproposal may break the deadlock on root-directory reorganization, but the community must first evaluate the methodology’s merits.

**Filesystem-level CoW for `git worktree`** -- The proposal faces **two major blockers**: the architectural challenge of sibling worktree visibility and the ReFS corruption risk. A revised RFC addressing these concerns is unlikely in the near term.

**`git squash` command** -- The proposal will likely stall unless a maintainer weighs in on its necessity relative to existing tools like `git rebase --autosquash`.

**`git repo info` path keys** -- The `path.git-prefix` key’s design objection may delay or block the series, even if the implementation is correct. The author must address the conceptual mismatch with the command’s scope.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder’s series will see a v3 addressing Junio’s feedback on submission format, stylistic improvements, scalability, API design, and documentation. The security implications of lazy fetching remain a key concern.