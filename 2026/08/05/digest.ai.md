Here is the digest for **2026-08-05**, covering the Git mailing list traffic:

---

## The day in brief

**August 5, 2026** was a **heavy but routine** day on the Git mailing list, with **152 emails** across **28 active threads**. The day’s traffic was dominated by **platform compatibility fixes** (Windows, macOS, NonStop), **ODB abstraction work**, and **regression fixes**, with a notable **policy update** from Junio C Hamano about the `seen` integration branch. The most consequential development was the **finalization of Harald Nordgren’s `--delete-merged` series**, now merged to `next` after 25 iterations, while **Patrick Steinhardt’s ODB streaming unification** and **Johannes Schindelin’s Windows build updates** advanced toward readiness. A **new policy for the `seen` branch** was announced, aiming to balance reviewer productivity with quality control.

---

## Notable threads

### **`git branch --delete-merged` lands in `next` after 25 iterations**
**Thread**: Final v25 of Harald Nordgren’s `--delete-merged` series
**Author**: Harald Nordgren
**Status**: **Merged to `next`**

Harald Nordgren’s **25th iteration** of the `--delete-merged` series for `git branch` has been **merged to `next`**, marking the culmination of a **six-month effort** to provide safe, automated local branch cleanup. The series introduces a **repeatable `--delete-merged`** option that deletes local branches merged into specified upstreams (e.g., `--delete-merged origin/main --delete-merged origin/next`) while preserving branches used as upstreams for unmerged work via **order-independent stacked-branch protection**. Key features include:
- **Per-branch opt-out** (`branch.<name>.deleteMerged=false`).
- **`--dry-run` preview** (avoids modifying config during dry runs).
- **`--forked` filter** (composable with `--merged`/`--no-merged`).
- **Flags-based API** (eliminates local boolean caching risks).

The **v25 changes were purely mechanical** (wrapping long test command lines), with **no functional changes** since v24, which resolved the last two blocking issues:
1. **Push-detection logic**: Fixed `branch_pushes_to_upstream()` to correctly apply push/fetch refspecs.
2. **Stacked-branch protection**: Simplified logic to clear upstream configs for kept branches.

**Junio C Hamano and Phillip Wood** confirmed the series is ready for merging, though **Junio requested a follow-up** to extend stacked-branch protection to `git branch -d` for consistency. The series is **well-tested**, with **100+ new test cases** covering edge cases like self-referential upstreams and missing upstreams.

**Why it matters**: This is a **major usability improvement** for Git’s branch management, particularly for users juggling stacked branches or topic branches tied to multiple upstreams. The stacked-branch protection design—**order-independent and configurable**—sets a new standard for safety in automated branch cleanup.

---

### **ODB streaming API unification advances**
**Thread**: [PATCH 0/8] odb: unify streaming API
**Author**: Patrick Steinhardt
**Status**: **v2 posted, ready for review**

Patrick Steinhardt’s **eight-patch v2 series** unifies `odb_read_stream` and `odb_write_stream` into a single `odb_stream` structure, eliminating redundancy and enabling **bidirectional streaming** (e.g., reading an object and writing it elsewhere without intermediate buffering). The series is part of the **ongoing ODB abstraction effort** to enable pluggable backends.

**Key changes in v2**:
- **Critical bugfix**: Patch 3/8 now correctly propagates `stream->type` to `hash_object_file()` in the in-memory ODB backend, preventing silent corruption for non-blob objects.
- **Wrapper structures**: Patch 5/8 introduces `struct odb_stream_write` and `struct odb_stream_read` for special cases, with allocation and container-of access.
- **Cosmetic renames**: Patches 6–8/8 rename `read_object_fd_data` → `fd_stream` and `input_zstream_data` → `zlib_stream`, and standardize function names (`odb_stream_from_object()`, `odb_stream_from_fd()`).

**Review status**:
- **Justin Tobler** provided substantive reviews of v1, endorsing the approach.
- **Junio C Hamano** identified the **critical bug in Patch 3/8**, which is now fixed.
- The series is **mechanically complete** and ready for final review, though reviewers may scrutinize the **wrapper allocation logic** and **performance impact** of the additional indirection.

**Why it matters**: This refactoring is **foundational for pluggable ODB backends**, enabling future backends (e.g., cloud storage) to implement streaming without duplicating logic. The unified API also simplifies code paths in `pack-objects`, `archive`, and other streaming-heavy subsystems.

---

### **Windows build updates upstreamed**
**Thread**: [PATCH 0/12] MinGW/Windows build and runtime adjustments
**Author**: Johannes Schindelin
**Status**: **v1 posted, waiting for response**

Johannes Schindelin’s **12-patch series** upstream a batch of **Git for Windows-specific build and runtime adjustments** into mainline Git, addressing hard-coded assumptions, MSYS2 integration, and deprecated artifacts. Key changes:
- **Python support**: Removes `NO_PYTHON` for MINGW, enabling Python buildability (Patch 1).
- **Compiler flexibility**: Drops hard-coded `CC = gcc` to allow Clang (Patch 2).
- **Deprecated flags**: Removes `-D_USE_32BIT_TIME_T` (incompatible with UCRT64) and refines `-Wl,--large-address-aware` for 32-bit builds (Patches 3–4).
- **MSYS2 integration**: Replaces hard-coded paths with MSYS2 environment variables (`MINGW_PREFIX`, `MSYSTEM`) (Patches 6–8).
- **Dashed-form executables**: Disables generation of `git-<command>` hard-links (deprecated since 2008) (Patch 9).
- **Locale regression fix**: Sets `LC_CTYPE=C.UTF-8` as a fallback to fix non-ASCII argument handling in MSYS2 (Patch 11).
- **Direct `git.exe` invocation**: Allows `git.exe` to be called directly (without the Git wrapper), eliminating an extra process hop (Patch 12).

**Review status**:
- **Junio C Hamano** raised a **cross-build-system inconsistency** in Patch 8: the Makefile strips the leading slash from `MINGW_PREFIX`, but Meson retains it. This could cause path-handling issues on Windows and should be fixed before merging.
- **Junio also suggested** removing dashed-form executables entirely in Git 3.0 (Patch 9), though this is forward-looking.
- The series is **otherwise uncontroversial** and addresses **real-world Windows compatibility issues**.

**Why it matters**: These patches **reduce the maintenance burden** of Git for Windows by upstreaming stable adjustments, improving **build flexibility** (e.g., Clang support) and **runtime robustness** (e.g., locale handling). The direct `git.exe` invocation (Patch 12) is a **user-facing improvement** that simplifies Windows workflows.

---

### **`seen` branch policy updated: no promotion without review**
**Thread**: What’s cooking in git.git (Aug 2026, #01; Tue, 5)
**Author**: Junio C Hamano
**Status**: **Policy now active**

Junio C Hamano announced a **new policy for the `seen` branch**, Git’s ephemeral integration branch for all in-flight topics. The policy aims to **balance reviewer productivity with quality control**:
1. **No automatic promotion**: Topics will not graduate to `next` without **at least one substantive review** or acknowledgment (e.g., `Reviewed-by`, `Acked-by`, or a "this looks reasonable" comment).
2. **Automatic expiration**: Topics will be **dropped after four weeks** if they receive no positive feedback.
3. **No maintainer obligation**: Junio disclaims responsibility for reviewing every patch himself.

**Motivation**: The policy addresses concerns that `seen` was becoming a **dumping ground** for under-reviewed patches, increasing the risk of flawed topics reaching `next`. It also responds to **Phillip Wood’s feedback** that `seen` is useful for reviewers (eliminating manual patch application) while mitigating Junio’s worry about premature promotion.

**Impact**:
- **Contributors** must now **actively solicit reviews** to keep their patches in `seen`.
- **Reviewers** may see **fewer stale patches** in `seen`, but the policy could **discourage participation** if patches expire too quickly.
- The policy **does not address systemic review bandwidth issues**, though it may indirectly encourage deeper engagement.

**Why it matters**: This is a **significant shift in Git’s integration pipeline**, reflecting the project’s **growing scale** and the **tension between velocity and correctness**. The policy’s effectiveness will depend on **community engagement**—whether reviewers step up to provide timely feedback.

---

### **Regression fixes and edge cases**
Several threads addressed **regressions and edge cases**:
1. **`git worktree add -b` regression fixed**
   - **Thread**: [PATCH v3] worktree add: disable DWIM when -b/-B is given
   - **Author**: Yoichi Nakayama
   - **Status**: **Merged to `master`**
   - **Fix**: `git worktree add -b <branch>` now **ignores DWIM** when the branch doesn’t exist, restoring pre-v2.42.0 behavior. The regression was introduced by `128e5496b3` (2023-05-17).

2. **`git push --delete --porcelain` inconsistency**
   - **Thread**: `git push --delete --porcelain` inconsistent output
   - **Author**: Xavier Morel
   - **Status**: **Bug confirmed, fix invited**
   - **Issue**: `--porcelain` outputs human-readable errors (e.g., "remote ref does not exist") instead of machine-readable format when deleting non-existent refs. **Junio C Hamano** acknowledged the oversight and invited a patch to use `!` as an error prefix.

3. **NonStop textconv regression**
   - **Thread**: git-replay and git-history silently drop notes during history rewriting
   - **Author**: Randall S. Becker
   - **Status**: **Root cause isolated, workaround proposed**
   - **Issue**: Textconv filters receive **empty temp files** for binary files (null-byte-prefixed) on NonStop, breaking workflows. The regression is **platform-specific** (NonStop’s I/O stack behavior). **Jeff King and D. Ben Knoble** confirmed they cannot reproduce the issue on other platforms.
   - **Workaround**: Randall proposed a `%f` placeholder in textconv filters to pass the original file path, though this **does not fix the root cause** (empty temp files).

4. **`git rebase -x` with `git commit --amend` loses notes**
   - **Thread**: `git rebase -x` with `git commit --amend` loses notes
   - **Author**: D. Ben Knoble
   - **Status**: **Root cause confirmed, two fix strategies proposed**
   - **Issue**: Notes are lost when `git rebase -x` is combined with `git commit --amend` because the rebase machinery records the new commit OID **before** the external command runs.
   - **Fix strategies**:
     1. **Post-amend update**: Update the rebase’s `rewritten` file after `git commit --amend` (Phillip Wood’s stalled `wip/rebase-update-rewritten` branch).
     2. **Pre-exec note copying**: Copy notes **before** stopping for `exec`/`edit`, letting `git commit --amend` handle note copying directly (new proposal).

---

## In brief

### **Performance and correctness**
- **`git maintenance` geometric repack fix**
  - **Thread**: [PATCH 0/2] maintenance: fix geometric repack with promisor packs
  - **Author**: Taylor Blau
  - **Fix**: Ensures the geometric repack task evaluates **both ordinary and promisor pack progressions** before falling back to all-into-one repack. The `--auto` trigger now considers `geometry.promisor_split`.

- **`git last-modified` Bloom filter optimization**
  - **Thread**: [PATCH 0/4] last-modified: reuse Bloom filters for speed
  - **Author**: Toon Claes
  - **Status**: **Expecting a reroll**
  - **Issue**: The `--show-trees` correctness bug (ancestor tree entries not covered by the original pathspec) must be fixed using `match_pathspec()` before the series can proceed.

- **`git replay --linearize`**
  - **Thread**: [PATCH 0/11] tc/replay-linearize
  - **Author**: Toon Claes
  - **Status**: **Needs review**
  - **Feature**: Adds `--linearize` to `git replay` to drop merge commits, enabling linear history rewriting.

### **Documentation and usability**
- **`git-refs(1)` warning reorganization**
  - **Thread**: [PATCH v2] gitk: add user-configurable custom commands to context menus
  - **Author**: Kristoffer Haugsbakk
  - **Status**: **Queued in `seen`**
  - **Improvement**: Moves a critical ref migration warning into a `WARNING` admonition block under the `migrate` subcommand, making it harder to overlook.

- **`gitk` custom commands**
  - **Thread**: [PATCH] gitk: add user-configurable custom commands to context menus
  - **Author**: Tim Wiederhake
  - **Status**: **Blocked on process**
  - **Issue**: The maintainer (**Johannes Sixt**) will not review the implementation until the **commit message explains the feature’s motivation** and the author confirms they have **reviewed every line of the AI-assisted code**.

- **`git history` bash completion**
  - **Thread**: [PATCH] completion: add git history
  - **Author**: Vincent Mailhol
  - **Status**: **Approved, needs rebase**
  - **Improvement**: Adds completion for `git history` subcommands (`drop`, `fixup`, `reword`, `split`) and options. The patch will need a **trivial rebase** once the `squash` subcommand lands.

### **Code hygiene and refactoring**
- **Coverity unchecked return values**
  - **Thread**: [PATCH 0/11] Coverity: unchecked return values
  - **Author**: Johannes Schindelin
  - **Status**: **v2 posted, needs review**
  - **Fixes**: Adds missing error checks for `curl_easy_duphandle`, `deflateInit`, `repo_parse_commit`, `lseek`, `dup`, `strbuf_getline_lf`, and `rename`. The series is **mechanically complete** and addresses **real-world error paths**.

- **Repository config handling refactor**
  - **Thread**: [PATCH 0/3] Refactor repo config handling in environment.c
  - **Author**: Tian Yuchen
  - **Status**: **Needs response**
  - **Improvement**: Reorganizes `repo_config_values` and simplifies getter functions. **Junio C Hamano** approved Patch 1/3 but raised concerns about **documentation scope** (Patch 2) and **field reordering maintainability** (Patch 3).

- **Hex object IDs to lowercase for Git 3.0**
  - **Thread**: [PATCH 0/6] Restrict hex object IDs to lowercase
  - **Author**: brian m. carlson
  - **Status**: **Waiting for review**
  - **Breaking change**: Prepares for Git 3.0 by teaching the hex parser to reject uppercase characters in object IDs when `WITH_BREAKING_CHANGES` is enabled. **Junio C Hamano** signaled tentative acceptance but remains ambivalent about the **Robustness Principle** trade-off.

---

## On the radar

1. **`git history` notes preservation**
   - **Thread**: `git-replay` and `git-history` silently drop notes during history rewriting
   - **Status**: **Design discussion ongoing**
   - **Proposal**: Add `--[no-]copy-notes` to `git replay` (plumbing) and handle notes automatically in `git history` (user-facing). **Phillip Wood** and **Patrick Steinhardt** are driving the discussion.

2. **`git bisect --reset-when-found`**
   - **Thread**: [PATCH 0/2] bisect: add `--reset-when-found`
   - **Author**: Han-Wen Nienhuys
   - **Status**: **Waiting for response**
   - **Feature**: Automatically resets `git bisect` after finding the culprit commit, reducing manual steps.

3. **`git add --resolved`**
   - **Thread**: [PATCH 0/2] add: introduce `--resolved` to stage conflict-resolved paths
   - **Author**: Junio C Hamano
   - **Status**: **Likely to merge to `next`**
   - **Feature**: Stages conflict-resolved paths while leaving unrelated changes unstaged, improving workflows for partial conflict resolution.

---

## Closing thoughts

Today’s traffic underscored Git’s **maturity as a project**: while **new features** (e.g., `--delete-merged`, `--resolved`) continue to land, much of the effort is focused on **platform compatibility**, **regression fixes**, and **infrastructure refactoring** (e.g., ODB abstraction, streaming APIs). The **`seen` branch policy update** is a **notable shift**, reflecting the project’s need to **scale its review process** without sacrificing quality. For contributors, the message is clear: **proactive engagement** (soliciting reviews, addressing feedback promptly) is now essential to keep patches in the pipeline.

The **ODB abstraction work** (Patrick Steinhardt) and **Windows improvements** (Johannes Schindelin) are particularly **strategic**, as they lay the groundwork for **future backends** (e.g., cloud storage) and **better Windows support**. Meanwhile, **regression fixes** (e.g., `git worktree add`, `git push --porcelain`) highlight the **ongoing challenge of maintaining consistency** in a complex codebase.

**What to watch for tomorrow**:
- **ODB streaming unification** (Patrick Steinhardt) and **Windows build updates** (Johannes Schindelin) are **close to merging** and may graduate to `next`.
- **`git history` notes preservation** and **`git bisect --reset-when-found`** could see **design proposals** or patches.
- The **`seen` branch policy’s impact** will become clearer as contributors adapt to the new requirements.