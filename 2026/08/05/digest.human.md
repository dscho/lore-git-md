Here is the digest for **2026/08/05**, covering the Git mailing list traffic in a concise, front-page-style overview.

---

# The day in brief

**2026/08/05** was a **heavy but routine** day on the Git mailing list, with **152 emails** across **28 active threads**. The standout event was the **final v25 of Harald Nordgren’s `--delete-merged` series**, now **merged to `next`** after a 25-iteration journey. Other notable threads included **Johannes Schindelin’s 12-patch MinGW build updates** (now in `seen`), **Patrick Steinhardt’s ODB streaming unification** (v2 posted), and **Junio C Hamano’s "What’s cooking" report**, which confirmed the graduation of seven topics to `master` and flagged several stalled efforts. The day also saw **bug reports** (e.g., `git rebase -x` losing notes, `git push --porcelain` inconsistencies) and **design discussions** (e.g., `git history` notes preservation, `git replay`/`git history` message handling).

---

# Notable threads

## `git branch --delete-merged` lands in `next` after 25 iterations
**Thread**: Final v25 of Harald Nordgren’s `--delete-merged` series
**Author**: Harald Nordgren
**Status**: **Merged to `next`**

Harald Nordgren’s **25th iteration** of the `--delete-merged` series has been **merged to `next`**, capping a **six-month effort** to add safe, automated local branch cleanup to `git branch`. The series introduces:
- **`--delete-merged <pattern>`**: Deletes local branches merged into any of the supplied upstream patterns (exact refnames, remote shorthand, or shell-style globs). The option can be repeated to widen the match (e.g., `--delete-merged origin/main --delete-merged origin/next`).
- **Order-independent stacked-branch protection**: If a branch is used as an upstream for an unmerged branch, it is spared and its stale upstream config is cleared. The v24 update simplified this logic to protect only the *direct* upstream of a surviving branch.
- **Per-branch opt-out**: `branch.<name>.deleteMerged=false` exempts specific branches from automated cleanup while still allowing explicit deletion via `git branch -d`.
- **`--dry-run`**: Prints "Would delete" or "Would skip" messages without modifying refs or config.
- **`--forked` filter**: A composable `--list` mode filter that underpins `--delete-merged` and can be combined with existing filters (`--merged`, `--no-merged`).

The series **resolves two blocking correctness issues** from v23:
1. **Push-detection logic**: The `branch_pushes_to_upstream()` function now correctly applies the *push* refspec to the local branch, applies the *fetch* refspec in reverse, and compares the result to the upstream branch, ensuring accurate detection regardless of `remote.pushDefault` configuration.
2. **Stacked-branch protection**: The logic now clears upstream configs for kept (merged) branches whose own upstream is being deleted, aligning the implementation with the design goal.

Junio C Hamano and Phillip Wood **confirmed the series is ready for merging** after the v24 fixes. The v25 changes are purely mechanical: **wrapping long test command lines in patch 1/7 for readability**. A follow-up to extend stacked-branch protection to `git branch -d` is planned but not a blocker.

**Key files**: `builtin/branch.c`, `ref-filter.c`, `Documentation/git-branch.adoc`, `t/t3200-branch.sh`.
**Why it matters**: This is a **long-awaited safety feature** for iterative topic branch workflows, addressing a common pain point (local branch clutter) while preserving critical safety checks (checked-out branches, missing upstreams, push divergence, per-branch opt-out, and stacked-branch protection).

---

## MinGW build updates upstreamed to mainline
**Thread**: `[PATCH 0/12] MinGW/Windows build and runtime adjustments`
**Author**: Johannes Schindelin
**Status**: **In `seen`; waiting for response**

Johannes Schindelin (Dscho) posted a **12-patch series** upstreaming Git for Windows-specific build and runtime adjustments into the main Git codebase. The series addresses hard-coded assumptions, MSYS2 integration, linking behavior, deprecated build artifacts, a locale-handling regression in the MSYS2 runtime, and direct `git.exe` invocation. Key changes:
- **Patch 1**: Enables Python buildability by removing `NO_PYTHON` for MINGW in `config.mak.uname`.
- **Patch 3**: Removes the `-D_USE_32BIT_TIME_T` compiler flag, which is incompatible with the UCRT64 runtime and was only needed for the original MinGW runtime (deprecated since 2015).
- **Patch 4**: Refines the `-Wl,--large-address-aware` linker flag to apply only to 32-bit builds (`MINGW32`), omitting it for 64-bit builds where it is irrelevant or unsupported.
- **Patch 9**: Disables generation of hard-linked `git-<command>` executables (e.g., `git-commit`) in `libexec/git-core/`, eliminating a 30-second build-time overhead and aligning with the long-standing deprecation of the dashed form (since Git 1.5.4, February 2008).
- **Patch 11**: Fixes a locale-handling regression in the MSYS2 runtime (introduced between versions 2.4.1 and 2.6.0) that caused Git to mishandle command-line arguments containing non-ASCII characters when shelling out to subprocesses. The fix sets `LC_CTYPE=C.UTF-8` as a fallback in `compat/mingw.c:setup_windows_environment()`.
- **Patch 12**: Allows `git.exe` to be called directly instead of through the "Git wrapper" (`C:\Program Files\Git\cmd\git.exe`), eliminating an extra process hop. The patch adds logic in `compat/mingw.c` to detect when `MSYSTEM` is unset and initializes it, along with `PLINK_PROTOCOL` and a carefully constructed `PATH`.

Junio C Hamano **flagged a cross-build-system inconsistency** in Patch 8: the Makefile strips the leading slash from `MINGW_PREFIX` before passing it to the compiler, but the Meson build retains the leading slash. This mismatch could cause subtle path-handling issues on Windows and should be corrected by aligning Meson’s behavior with the Makefile’s. Junio also **suggested removing dashed-form executables entirely at the Git 3.0 boundary**, though this is a forward-looking proposal and not a request for changes to the current series.

**Key files**: `compat/mingw.c`, `config.mak.uname`, CMake/Meson build files, `t/t0060-path-utils.sh`.
**Why it matters**: These patches **reduce Windows-specific build and runtime overhead**, improve MSYS2 integration, and address real-world compatibility issues (e.g., locale handling, credential leaks). The series is a **routine but important** step toward making the mainline Git codebase more compatible with Windows without affecting other platforms.

---

## ODB streaming API unification (v2)
**Thread**: `[PATCH 0/7] odb: unify streaming API`
**Author**: Patrick Steinhardt
**Status**: **In review; v2 posted**

Patrick Steinhardt posted **v2 of his 8-patch series** unifying the object database (ODB) streaming API by consolidating `odb_write_stream` and `odb_read_stream` into a single `odb_stream` structure. The series is part of the ongoing ODB abstraction effort to enable pluggable backends. Key changes in v2:
- **Patch 3/8**: Fixes a **critical correctness issue** in the in-memory ODB backend by propagating `stream->type` to `hash_object_file()`, preventing silent corruption or hash collisions for non-blob objects.
- **Patch 5/8**: Merges the two structures into a single `odb_stream` and introduces wrapper structures (`struct odb_stream_write` and `struct odb_stream_read`) for the two special cases in `builtin/unpack-objects.c` and `odb/streaming.c`.
- **Patch 6/8**: Renames `struct read_object_fd_data` to `struct fd_stream` to better reflect its expanded role.
- **Patch 7/8**: Renames `struct input_zstream_data` to `struct zlib_stream` to better reflect its role as a stream wrapper.
- **Patch 8/8**: Standardizes function names (`odb_stream_from_object()` and `odb_stream_from_fd()`) and documents file descriptor ownership semantics.

The series **resolves all prior review feedback**, including the critical bug in Patch 3/8 and cosmetic renames in Patches 6/8 and 7/8. The unified `odb_stream` structure supports bidirectional streaming via a direction-agnostic interface, enabling use cases like reading an object and writing it elsewhere without intermediate buffering. The changes are **mechanical and low-risk**, with no intended behavior changes.

**Key files**: `odb/streaming.c`, `odb/streaming.h`, `builtin/unpack-objects.c`, `archive-tar.c`, `archive-zip.c`, `t/unit-tests/u-odb-inmemory.c`.
**Why it matters**: This refactoring **simplifies the ODB streaming API**, reduces redundancy, and prepares the codebase for pluggable ODB backends. It is a **foundational step** in the broader ODB abstraction effort.

---

## `git history` and `git replay` silently drop notes
**Thread**: `git-replay` and `git-history` silently drop notes during history rewriting
**Author**: D. Ben Knoble
**Status**: **Design discussion; no patch yet**

D. Ben Knoble reported that **`git replay --onto=...` and `git history fixup` silently drop Git notes** when rewriting commits, even when `notes.rewriteref` is configured to include the relevant notes ref (`refs/notes/benknoble/commits`). The issue is specific to the interaction between `-x` and `--amend` in `git rebase` (which `git history` and `git replay` build upon): the rebase machinery records the new commit OID *before* the external command runs, so `git commit --amend` creates a new OID that the notes-rewriting step does not map correctly.

Phillip Wood **proposed two fix strategies**:
1. **Post-amend update**: Update the rebase’s internal `rewritten` file after `git commit --amend` (prior work in his `wip/rebase-update-rewritten` branch). This worked for simple `exec` amend cases but stalled on interactive editing robustness.
2. **Pre-exec copy**: Copy notes *before* stopping for an `exec` or `edit` command, allowing `git commit --amend` to handle note copying directly to the amended commit. This avoids OID-mapping issues but may complicate conflict handling during interactive editing.

The discussion has **converged on the pre-exec copy approach** for its simplicity, though no patch has been posted yet. The thread also touched on whether `git replay` (a plumbing command) should have a `--copy-notes` flag (Phillip’s preference) or rely on config (rejected as inappropriate for plumbing).

**Key subsystems**: rebase, notes.
**Why it matters**: Notes are a **critical part of many workflows** (e.g., code review annotations, deployment metadata), and silently dropping them during history rewriting is a **serious usability and data integrity issue**. The proposed fix would ensure notes are preserved by default in `git history` and via an opt-in flag in `git replay`.

---

## `git rebase -x` with `git commit --amend` loses notes
**Thread**: `git rebase -x` with `git commit --amend` loses notes
**Author**: D. Ben Knoble
**Status**: **Root cause identified; no patch yet**

D. Ben Knoble reported that **`git rebase -x` combined with `git commit --amend` fails to rewrite notes** attached to the tip commit, even when `notes.rewriteref` is configured. Phillip Wood **diagnosed the root cause**: the rebase machinery records the new commit OID *before* the external command runs, so `git commit --amend` creates a new OID that the notes-rewriting step (post-rebase) does not map correctly. The notes are copied from the old OID to the pre-amend OID, not the post-amend one, making them appear lost.

Phillip proposed **copying notes before stopping for an `exec` or `edit` command**, allowing `git commit --amend` to handle note copying directly to the amended commit. This approach avoids the OID-mapping issue but may complicate conflict handling during interactive editing. No patch has been posted yet.

**Key subsystems**: rebase, notes.
**Why it matters**: This is a **regression or edge case** in `git rebase -x` that affects users who rely on notes for workflow metadata (e.g., code review annotations, deployment tracking). The proposed fix would ensure notes are preserved during amend operations in rebase.

---

## `git push --porcelain` inconsistency
**Thread**: `git push --delete --porcelain` outputs human-readable errors for non-existent refs
**Author**: Xavier Morel
**Status**: **Bug confirmed; no patch yet**

Xavier Morel reported that **`git push --delete --porcelain` outputs human-readable error messages** (e.g., "remote ref does not exist") instead of the documented machine-readable porcelain format when attempting to delete a ref that no longer exists on the remote. This breaks the expected output consistency for `--porcelain`, which is intended for script consumption.

Junio C Hamano **acknowledged the issue as an oversight** and invited a patch to fix it. The proposed solution is to use `!` as an error prefix for machine-readable output in failure cases (e.g., non-existent refs). No patch has been posted yet.

**Key files**: `push.c`, `transport.c`, or `remote.c`.
**Why it matters**: The `--porcelain` flag is **critical for automation**, and inconsistent output prevents scripts from reliably detecting deletion failures for non-existent refs. This is a **well-scoped, uncontroversial bugfix** with clear value for script authors.

---

# In brief

- **`git worktree add` regression fix**: Yoichi Nakayama’s patch (v3) to **disable DWIM when `-b`/`-B` is given** was **merged to `master`**. The fix ensures explicit branch creation takes precedence over DWIM behavior, addressing a regression introduced in Git 2.42.0.
- **`git history` completion**: Vincent Mailhol’s patch adding bash completion for `git history` received **positive reviews** from Patrick Steinhardt and D. Ben Knoble. Junio C Hamano suggested a future refactoring to reduce boilerplate, but the patch is otherwise ready.
- **`gitk` custom commands**: Tim Wiederhake’s patch adding user-configurable custom commands to `gitk`’s context menus was **blocked by Johannes Sixt** due to missing motivation in the commit message and AI assistance disclosure. The patch will not proceed without a revised commit message and confirmation of code ownership.
- **`git add --interactive` removal**: A patch to remove `--interactive` (`-i`) mode from `git add` was **rejected by Junio C Hamano** for lack of justification. The maintainer emphasized that breaking changes require **strong evidence of harm or large-scale disuse**, neither of which was provided.
- **`git maintenance` geometric repack with promisor packs**: Taylor Blau’s **2-patch bugfix series** for `git maintenance`’s geometric repack task with promisor packs was posted. The series ensures the maintenance task properly evaluates both ordinary and promisor pack progressions before deciding whether to fall back to all-into-one repack, and adjusts the `--auto` trigger to consider promisor pack geometry.
- **`git replay` and `git history` message handling**: The ongoing design discussion for `git history squash`’s message-handling behavior has **converged on a simpler approach**: the editor will open by default during squash operations, eliminating the need for a dynamic default. The fate of `--reedit-message` and the appropriateness of the rebase squash template remain unresolved.
- **`git config` editor failure**: Johannes Schindelin’s **11-patch series** fixing unchecked return values (e.g., `curl_easy_duphandle`, `deflateInit`, `repo_parse_commit`) is **complete and ready for final review**. The series adds missing error checks to harden Git against silent failures, with no behavior changes.
- **`git fsmonitor` macOS cookie flush**: Tamir Duberstein’s patch fixing a macOS-specific performance regression in Git’s fsmonitor daemon was **reviewed by Patrick Steinhardt**, who asked for clarification on FSEvents queue draining guarantees. The patch is narrowly scoped to the fsmonitor daemon’s cookie wait mechanism.
- **`git regex` memory leak on macOS**: Chungmin Lee’s patch fixing a memory leak in Git’s macOS builds when the system regex engine (TRE) processes invalid UTF-8 sequences was **reviewed by Patrick Steinhardt**, who questioned the long-term maintainability of the current workaround. The discussion remains open.
- **`fetch_if_missing` in `struct repository`**: Tian Yuchen’s patch moving `fetch_if_missing` into `struct repository` remains **blocked on design questions** about repository-less invocations and a null pointer dereference risk in `builtin/index-pack.c`. The author has agreed to require `git index-pack` to always run inside a repository, eliminating the need for the conditional check.
- **`writev(3p)` wrapper revival**: Patrick Steinhardt’s **6-patch series** reintroducing a `writev(3p)`-like wrapper for performance was **reviewed by Junio C Hamano and Johannes Sixt**. The architectural question about I/O abstraction is resolved: Git’s pktline and cat-blob use cases do not require atomicity, allowing the API to be implemented as a simple loop of `write(2)` calls. The series is now technically complete, with no immediate reroll expected.
- **`git last-modified` Bloom filter optimization**: Toon Claes’s **4-patch series** optimizing `git last-modified` with Bloom filters remains **stalled on a maintainability concern** about `bloom_filter_settings` management in patch 4/4. The `--show-trees` correctness fix (using `match_pathspec()`) is the sole remaining technical blocker.
- **`git maintenance` geometric repack with promisor packs**: Taylor Blau’s **2-patch bugfix series** for `git maintenance`’s geometric repack task with promisor packs was posted. The series ensures the maintenance task properly evaluates both ordinary and promisor pack progressions before deciding whether to fall back to all-into-one repack, and adjusts the `--auto` trigger to consider promisor pack geometry.
- **`git diff` bash completion**: Junio C Hamano’s **3-patch series** fixing bash completion for `git diff` was posted. The series makes `git diff` path completion behave consistently with `git status` and `git add`, especially when invoked with `git -C <directory>`, and adds untracked path completion as a fallback.
- **`git repo config` refactoring**: Tian Yuchen’s **3-patch series** refactoring repository config handling in `environment.c` received **mixed reviews** from Junio C Hamano. Patch 1/3 (removal of redundant NULL checks) was approved, but Patches 2/3 and 3/3 (unified documentation and field reordering) were questioned for scope, accuracy, and maintainability benefit.
- **`.mailmap` update**: D. Ben Knoble’s patch updating his `.mailmap` entry to map `ben.knoble+github@gmail.com` to `ben.knoble@gmail.com` was posted. The change is a routine housekeeping update.
- **Credential helper protocol regression**: Adien Akhmad reported a regression in the credential helper protocol during HTTP 301 redirects, where the `wwwauth[]` attribute (containing the WWW-Authenticate header) is stripped from credential helper input. The issue affects helpers like `git-credential-oauth` that rely on this header to detect authentication schemes. No patch or root-cause analysis has been posted yet.

---

# On the radar

- **`hn/history-squash`**: The `squash` subcommand for `git history` remains **in `seen`**, with **no recent activity**. The topic was ejected from `next` due to fundamental flaws in reachability logic and UX inconsistencies. Phillip Wood is developing fixups, but no timeline has been set.
- **`tb/repack-geometric-cruft`**: Taylor Blau’s series teaching `git repack` to combine `--geometric` and `--cruft` remains **stalled for over a month** due to unresolved test failures.
- **`za/completion-hide-dotfiles`**: A patch updating path completion to hide dotfiles by default remains **stalled for over a month** with no recent activity.
- **`ps/odb-streams`**: Patrick Steinhardt’s ODB streaming API unification series is **waiting for review**.
- **`bc/restrict-hex-to-lowercase`**: brian m. carlson’s series preparing for Git 3.0 by restricting hex object IDs to lowercase is **waiting for review**.
- **`ps/writev`**: Patrick Steinhardt’s `writev(3p)` wrapper revival series is **waiting for response**.
- **`tc/last-modified-bloom`**: Toon Claes’s `git last-modified` Bloom filter optimization series is **expecting a reroll** to address the `--show-trees` correctness fix and the maintainability concern about `bloom_filter_settings` management.
- **`kk/merge-base-exhaustion`**: Karthik K’s merge-base optimization series is **expecting a reroll**.
- **`ij/subtree-reject-v2-config`**: Ian Johnson’s patch preventing the shell `git subtree` from running if the Rust version’s config is present is **expecting a reroll**.

---

# Closing notes

Today’s traffic was **heavy but routine**, with a mix of **long-running series landing**, **bug reports**, and **design discussions**. The **standout event** was the **final v25 of Harald Nordgren’s `--delete-merged` series**, which has been **merged to `next`** after a 25-iteration journey. Other notable threads included **Johannes Schindelin’s MinGW build updates**, **Patrick Steinhardt’s ODB streaming unification**, and **ongoing discussions** about `git history`/`git replay` notes preservation and `git rebase -x` losing notes.

The **next few days** are likely to see **rerolls for stalled topics** (e.g., `tc/last-modified-bloom`, `kk/merge-base-exhaustion`) and **new patches** for confirmed bugs (e.g., `git push --porcelain` inconsistency, `git rebase -x` notes loss). The **Git 3.0 preparation** (`bc/restrict-hex-to-lowercase`) and **pluggable ODB effort** (`ps/odb-streams`, `ps/odb-make-creation-pluggable`) remain key long-term themes.