# The Git Mailing List Daily Digest for 2026/07/27

## The day in brief

A moderately busy Monday with 52 emails across 19 threads. The day was marked by **significant design discussions** around the `git replay --linearize` feature and ODB performance regressions, **bugfix progress** in `git cat-file` and pack-bitmap subsystems, and **test infrastructure improvements**. Several long-running threads reached resolution points, while others faced fundamental architectural questions. The "What's cooking" report provided a comprehensive snapshot of integration status, highlighting both progress and stalled efforts.

---

## Notable threads

### **`git replay --linearize` design reaches crossroads**
**Toon Claes** engaged with **Elijah Newren**'s critique of the multi-branch ambiguity in the `--linearize` option, proposing a new `--ref` syntax to explicitly specify which ref to update. The current implementation flattens all input branches into a single linear history, which Elijah argues violates the principle of least astonishment. Toon's proposal—`git replay --linearize --onto <target> <branches> --ref <ref>`—would preserve flexibility while eliminating emergent reachability, though it requires tracking `last_commit` per branch. The discussion now centers on three competing proposals: **status quo** (Junio's position), **single-ref restriction** (Elijah's minimal fix), or **`--ref` syntax** (Toon's compromise). The thread remains in **design exploration phase**, with no clear consensus yet.

**Key detail**: The `--ref` syntax would require significant implementation work to track replayed commits per branch, but Toon argues it offers the best balance of flexibility and predictability.

---

### **ODB performance regression sparks architectural redesign**
**Patrick Steinhardt** proposed a **fundamental shift** in how alternates are handled, moving their management into the "files" backend to treat them as an implementation detail rather than a cross-backend concern. This would enable pack-first-loose-second lookups across all alternates while simplifying other subsystems (commit-graph, bitmap, maintenance). The proposal gained **explicit endorsements from Peff and Justin Tobler**, who dismissed concerns about lost flexibility as unlikely to be a practical issue. Justin also outlined a parallel plan to **decouple transaction handling from the ODB source list**, further simplifying the ODB layer. The discussion has moved from a targeted fix to a **broader architectural cleanup**, with Patrick planning to implement the redesign in late July.

**Key detail**: The proposal directly addresses the root cause of the 12% performance regression in `git receive-pack` connectivity checks, which was introduced by the ODB abstraction rework.

---

### **`git stash reword` faces fundamental correctness issue**
**Emin Özata**'s v2 patch for `git stash reword` (renamed from `rename`) addressed atomicity concerns but introduced a **critical data-loss risk**. The implementation slurps the entire reflog into memory, deletes it on-disk, and rewrites it with a single entry modified. **Junio C Hamano** pointed out that if the process is killed after `refs_delete_reflog()` but before the transaction commits, the reflog is permanently lost. Junio proposed extending the reflog API to support atomic replacement of individual entries (`refs_reflog_replace()` or `refs_reflog_edit_in_bulk()`), which would eliminate the risk. The thread has shifted from "does this feature make sense?" to **"how do we implement it safely?"**, with the current patch facing a **major redesign**.

**Key detail**: The data-loss scenario is unacceptable for a user-facing feature, making the API extension a blocker for merging.

---

### **Pack-bitmap off-by-one bugfix reviewed**
**David Lin** submitted a minimal fix for an off-by-one edge case in `find_objects()` where objects at position zero in the base bitmap were incorrectly excluded. **Taylor Blau** confirmed the fix's correctness and suggested expanding test coverage to include non-pseudo-merge scenarios. The patch is **low-risk and well-motivated**, with the only open action being the test addition. The fix aligns the code with the intended behavior described in the commit message and is unlikely to be controversial.

**Key detail**: The fix changes `pos > 0` to `pos >= 0` in `find_objects()`, eliminating redundant work without altering the resulting object set.

---

### **UTF-8 string-width refactoring withdrawn**
**Hardik Kumar**'s effort to change the return type of `utf8_strwidth()` and `utf8_strnwidth()` from `int` to `size_t` was **withdrawn after sustained reviewer opposition**. The patch reintroduced explicit casts at call sites, undermining its goal of type consistency, and faced **high-weight critiques from Junio C Hamano, Pablo Sabater, and Phillip Wood**. The consensus was that the practical benefits were minimal for functions measuring display widths of strings that fit on a single line, while the risks (regressions, complexity) outweighed the benefits. The resolution was to **document the existing design's rationale** in `utf8.c`, replacing the TODO comment that originally motivated the refactoring.

**Key detail**: The new documentation explains why the functions return `int` (alignment with `printf()`-style APIs) and why `cast_size_t_to_int()` is necessary to prevent overflows.

---

## In brief

**`git cat-file --batch-command` assertion failure** -- **Alan Stokes** reported a crash when omitting `%(objecttype)` from the format string. **Jeff King** and **Pablo Sabater** collaborated on a targeted fix (save/restore `data->info.typep` around the `contents` command), which Pablo tested and confirmed. The next step is a formal patch with test coverage in `t1006-cat-file.sh`.

**`git repo info` path keys** -- **K Jayatheerth**'s series adding seven new path-related keys (e.g., `path.toplevel`, `path.hooks`) remains under review. **Junio C Hamano** identified a forward-compatibility issue in patch 1/7 (use of `startup_info->prefix`, a struct member removed from `master`), blocking progress until the series is rebased.

**`git stash` avoids sparse-index expansion** -- **Teng Long**'s optimization for `git stash push` (avoiding sparse-index expansion for in-cone paths) was merged to `next`. The patch reduces unnecessary work in monorepos and is part of a broader effort to improve sparse-checkout performance.

**Test infrastructure: `commit_body()` helper** -- **Shlok Kulshreshtha**'s v2 series introduced a reusable shell helper to extract commit message bodies, replacing ~61 duplicated instances of `git cat-file commit <rev> | sed -e '1,/^$/d'`. The series is **mechanically complete**, with all known call sites converted, and is ready for review.

**`writev(3p)` wrapper revival** -- **Patrick Steinhardt**'s effort to reintroduce a `writev(3p)`-like wrapper gained clarity on API naming. **Junio C Hamano** endorsed **Johannes Sixt**'s suggestion to use `git_write_gather()` (or similar) to avoid implying POSIX semantics, proposing a two-level abstraction for platform compatibility. The discussion now focuses on whether the performance gains justify the complexity.

**`git mv` symlink handling** -- **Lucas Zamboni Orioli**'s v4 patch for `git mv` improved error messages and added an early existence check for the destination directory, explicitly rejecting tracked symlinks. **Junio C Hamano** approved the core logic but requested a minor refactoring to deduplicate a small logic fragment. The series is **ready for integration** once the refactoring is addressed.

**`git diff --relative --cached` segfault fix** -- **Jeff King**'s bugfix and follow-up refactorings for a segfault in `git diff --relative --cached` with unmerged paths were approved by Junio and marked for `next`. The series is **technically complete**, with no outstanding objections.

**Rust hash cleanups** -- **brian m. carlson**'s memory fixes for Rust-C interop in the hash subsystem were merged to `next`. The patch is part of the ongoing Rustification effort and addresses issues in the `git_hash_algo` API.

---

## On the radar

**`git repack --geometric --cruft`** -- **Taylor Blau**'s RFC series combining `--geometric` and `--cruft` modes remains stalled, with unresolved feedback on the two-phase traversal logic in `--stdin-packs=follow-reachable`. Junio's follow-up email asked whether the series is considered complete or if further polishing is planned.

**Alternates-in-files-backend redesign** -- **Patrick Steinhardt**'s proposal to move alternates handling into the "files" backend is gaining traction, with endorsements from Peff and Justin Tobler. The redesign would resolve the ODB performance regression and simplify other subsystems, but implementation is delayed until late July.

**`git stash reword` API extension** -- The need for `refs_reflog_replace()` or `refs_reflog_edit_in_bulk()` to safely implement `git stash reword` may prompt broader reflog API improvements. The discussion highlights a gap in Git's reflog infrastructure for atomic single-entry edits.