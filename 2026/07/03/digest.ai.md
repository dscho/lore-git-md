Here’s the daily digest for **July 3, 2026**, covering the Git mailing list traffic:

---

### **The day in brief**
A busy but focused day on the Git mailing list, with **106 emails across 26 threads**. The standout developments:
1. **`git history drop` merged** – Patrick Steinhardt’s 11-patch series adding commit-dropping to the experimental `git history` command is now fully integrated after Junio fixed a minor whitespace nit.
2. **`git refs` subcommands ready for `next`** – Patrick’s 5-patch series consolidating reference manipulation under `git refs` (adding `create`, `delete`, `update`, `rename`) is approved for the next release cycle.
3. **Reftable hardening series nears completion** – Patrick’s 12-patch security hardening for the reftable backend is technically complete, with only minor test refinements remaining.
4. **`USE_NSEC` debate intensifies** – A long-running discussion about timestamp precision in Git’s build system took a turn toward runtime configuration, with Patrick Steinhardt proposing to make nanosecond support always available but opt-in via `core.useNsec`.

The day’s tone was **pragmatic and incremental**, with most threads converging on technical consensus rather than controversy. The **`git history` and `git refs` merges** mark significant progress in Git’s history-editing and reference-handling tooling, while the **reftable hardening** and **`USE_NSEC`** discussions reflect ongoing efforts to modernize Git’s internals.

---

### **Notable threads**

#### **`git history drop` merged**
**Headline**: Patrick Steinhardt’s 11-patch series adding `git history drop` (to remove a commit and replay its descendants) is now fully merged after Junio C Hamano addressed a minor whitespace nit in `reset.c`.
**What it does**: The series modernizes the reset API, advances `the_repository` removal, and adds 561 lines of test coverage for edge cases like detached HEAD and bare repositories. The `drop` subcommand includes conflict detection, bare repository support, and dry-run mode.
**Why it matters**: This expands Git’s history-editing toolkit alongside `git rebase` and `git commit --amend`, offering a more intuitive way to surgically remove commits. The series also includes a critical fix for ref resolution in detached-HEAD states, ensuring correct HEAD movement detection.
**Status**: Merged. Junio’s final sign-off closes the thread, with no open questions.

---

#### **`git refs` subcommands ready for `next`**
**Headline**: Patrick Steinhardt’s 5-patch series adding `create`, `delete`, `update`, and `rename` subcommands to `git refs` is now approved for `next`.
**What it does**: Consolidates reference manipulation (previously scattered across `git-update-ref`, `git-symbolic-ref`, and other commands) under a unified interface. The series includes 680 lines of test coverage and resolves a long-standing usability gap in Git’s reference-handling commands.
**Key design decisions**:
- `git refs update` retains `git update-ref`’s implicit creation semantics for backward compatibility.
- `git refs create` provides an explicit atomic alternative with "must not exist" semantics.
- **Symref renaming is not supported** due to backend limitations (a gap not yet documented in the man page).
**Why it matters**: Improves discoverability and reduces fragmentation in Git’s reference-handling commands. The series is part of Patrick’s broader effort to modernize Git’s reference backends (reftable) and remove global state (`the_repository`).
**Status**: Approved for `next`. Junio’s "What’s cooking" update confirms the series is ready for integration.

---

#### **Reftable hardening series nears completion**
**Headline**: Patrick Steinhardt’s 12-patch series hardening the reftable backend against maliciously corrupted files is now technically complete, with only minor test refinements remaining.
**What it does**: Fixes a range of vulnerabilities (out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and `abort()` calls) discovered via fuzzing. The series also introduces new fuzzing infrastructure (libFuzzer + Meson support) and a test helper (`cl_reftable_write_block`) to reduce boilerplate in unit tests.
**Key fixes**:
- Patch 6/12: Heap-buffer-overflow in the log block parser (zlib decompression).
- Patch 7/12: Out-of-bounds read with bogus block sizes.
- Patch 12/12: Heap-buffer-overflow in `reftable_table_new()` with truncated files.
**Why it matters**: While reftable files are not stored remotely, hardening the parser prevents local exploitation (e.g., via malicious repositories). The fuzzing infrastructure is a reusable framework for future security work.
**Status**: Technically complete. Junio has approved several patches as "obviously correct," and the series is poised for merging once the final test refinements are addressed.

---

#### **`USE_NSEC` debate: Runtime configuration proposed**
**Headline**: The long-running discussion about Git’s `USE_NSEC` build-time option for nanosecond timestamp precision took a turn toward runtime configuration, with Patrick Steinhardt proposing to make nanosecond support always available but opt-in via `core.useNsec`.
**What’s at stake**: `USE_NSEC` enables sub-second timestamp tracking, which can improve performance by reducing "racy Git" scenarios where second-granularity timestamps force full content checks. However, the option is currently build-time only, and its reliability varies across filesystems.
**Key proposals**:
1. **Brian M. Carlson**: Flip the default to `true` for most users, with a config knob to override.
2. **Patrick Steinhardt**: Always compile nanosecond support into Git (when the platform allows it) but retain the current build-time default (`USE_NSEC` disabled). Users would opt into the feature via `core.useNsec`.
3. **D. Ben Knoble**: Retain the build-time option for parity with Autotools/Makefile, arguing it’s useful for source distributions like Gentoo.
**Why it matters**: Nanosecond precision can significantly improve performance in worktree diff operations (e.g., `git diff` in secondary worktrees), but interoperability risks (e.g., mixing `USE_NSEC` and non-`USE_NSEC` implementations like Git and JGit) and edge cases (e.g., kernel inode cache eviction) complicate the decision.
**Status**: No consensus yet. The discussion remains open, with Patrick’s runtime-configuration proposal gaining traction as a middle ground. The original Meson parity patch (adding a `nanosec` option) is still queued for merging, but its long-term viability is now in question.

---

### **In brief**
- **`git replay --linearize`**: Junio raised a procedural nit about authorship attribution for Toon Claes’s v6 series, but the series remains technically complete and ready for review.
- **`git rm` pathspec behavior**: A thread about `git rm -n *.json` unexpectedly recursing into subdirectories concluded that the behavior is correct but under-documented. The discussion highlighted the need for better cross-referencing between `git-rm(1)` and `gitglossary(7)`.
- **Memory leaks in hash implementations**: Jeff King’s 9-patch series plugging leaks in non-default hash backends (OpenSSL, libgcrypt) saw substantive discussion about making `git_hash_discard()` idempotent. Patrick Steinhardt and Brian M. Carlson endorsed the idea, citing benefits for Rust integration.
- **`GIT_TEST_LONG` tests**: Patrick Steinhardt’s 9-patch series making `GIT_TEST_LONG` tests reliable and efficient for CI is now complete, with all feedback addressed. The series enables `GIT_TEST_LONG` in GitLab CI for integration branches and optimizes several expensive tests.
- **`git history` signing**: Souma’s 3-patch series teaching `git history` to sign rewritten commits (`fixup`, `reword`, `split`) was submitted. The series adds regression tests and updates documentation, but no reviews have been posted yet.
- **`git repo info` prefix queries**: A GSoC contributor’s patch adding category-based prefix queries to `git repo info` received a design critique from Junio, who suggested supporting glob patterns (e.g., `layout.*`) instead of simple prefix matching.

---

### **On the radar**
- **`git rev-list --exclude-first-parent-only`**: Junio submitted a fix for a bug where the option incorrectly excludes commits when additional commits are explicitly specified. The patch is under review.
- **`git diff --index`**: A proposal to replace `--cached` with `--index` in `git diff` received surface-level support but no substantive review. The change aims to improve clarity but risks breaking backward compatibility.
- **`includeIf.worktree`**: Toon Claes’s v6 series adding worktree-based conditional config inclusion identified a behavioral inconsistency with symlink handling. The series is otherwise ready for merging.

---

### **Editorial note**
Today’s traffic reflects Git’s **steady march toward modernization**, with significant progress in history-editing tooling (`git history drop`), reference manipulation (`git refs`), and security hardening (reftable). The `USE_NSEC` debate stands out as a rare case where a technical decision has broader implications for performance, interoperability, and platform compatibility—highlighting the challenges of evolving a tool as widely used as Git. The day’s discussions were **pragmatic and incremental**, with most threads converging on technical consensus rather than controversy.