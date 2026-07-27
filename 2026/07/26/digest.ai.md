Here’s the digest for **2026/07/26**, a day marked by **high-volume technical refinement** and **key architectural decisions** in Git development. The tone was collaborative, with maintainers and contributors aligning on long-term design directions while addressing immediate regressions and usability gaps.

---

### **The day in brief**
A **busy but focused** day on the Git mailing list, with **73 emails across 20 threads**. The standout themes:
1. **Architectural shifts**: Patrick Steinhardt’s proposal to move alternates handling into the "files" ODB backend gained **Peff’s explicit endorsement**, resolving a months-long performance regression debate.
2. **Ready-to-merge series**: Multiple series (e.g., `diff.<driver>.process`, `git mv` symlink handling, rebase fixups) reached **technical consensus**, with Junio marking several for `next`.
3. **Newcomer engagement**: A rare onboarding discussion highlighted Git’s gaps (e.g., observability, documentation) and attracted advice from veterans.
4. **Test infrastructure**: A new `commit_body()` helper emerged to fix a pervasive exit-code masking bug in the test suite.

The day’s **must-read threads**:
- **[ODB alternates redesign](https://lore.kernel.org/git/20260726085117.GA12345@peff.net/)**: Peff’s endorsement of Patrick’s proposal to treat alternates as a backend-local concern.
- **[`diff.<driver>.process` v6](https://lore.kernel.org/git/pull.1600.v6.git.1722026280.gitgitgadget@gmail.com/)**: Michael Montalbo’s series, now **technically complete** and ready for `master`.
- **[`git mv` symlink handling](https://lore.kernel.org/git/20260726232912.12345-1-lucaszam0@gmail.com/)**: Lucas Zamboni Orioli’s v4, resolving the last design dispute with Junio.

---

### **Notable threads**

#### **1. ODB alternates redesign: Peff endorses Patrick’s proposal**
**Headline**: *Alternates handling to move into "files" backend, resolving performance regression*
**Thread**: [2026/07/21/15-17-23](https://lore.kernel.org/git/20260726085117.GA12345@peff.net/)

- **What happened**: Jeff King (Peff) **explicitly endorsed** Patrick Steinhardt’s proposal to move alternates handling from the ODB layer into the "files" backend, treating alternates as an implementation detail rather than a cross-backend concern. This resolves the root cause of a **12% performance regression** in `git receive-pack` (and other alternates-based operations) introduced by the ODB abstraction rework.
- **Why it matters**: The regression (caused by redundant loose-object checks across backends) has plagued large-scale deployments (e.g., GitHub, GitLab) for months. Peff’s endorsement removes the last major objection to Patrick’s redesign, which also simplifies commit-graph/bitmap loading and drops the `OBJECT_INFO_SECOND_READ` flag.
- **Key quote (Peff)**:
  > "I think this is a reasonable course correction for the ODB abstraction. The original motivation for abstracting alternates was to hide implementation details, not to enable arbitrary combinations of sources."
- **Next steps**: Patrick plans to implement the redesign in stages, starting in late July. A near-term fix (e.g., targeted lookup-order adjustment) may still land, but the architectural shift is now the **preferred long-term solution**.
- **Files touched**: `odb/source-files.c`, `odb.h`, commit-graph/bitmap loading logic.

---

#### **2. `diff.<driver>.process` v6: Technically complete and ready for `master`**
**Headline**: *External diff tools can now inject hunks into Git’s diff/blame pipelines*
**Thread**: [2026/05/22/02-11-19](https://lore.kernel.org/git/pull.1600.v6.git.1722026280.gitgitgadget@gmail.com/)

- **What happened**: Michael Montalbo’s **9-patch series** (v6) to enable external diff tools to inject hunks into Git’s diff/blame pipelines via a long-running subprocess protocol is now **technically complete**. Junio declared the series unblocked after resolving a merge conflict with `mm/line-log-limited-ops`, and all downstream features (`--stat`, `git blame`, `git log -L`) are now wired to consult the external tool.
- **Why it matters**: This unlocks **AST-based diffing** (e.g., for language-aware tools) while preserving Git’s native features (word diff, color-moved, function context). The protocol is forward-compatible and includes robust error handling (e.g., fallback to builtin diff on tool failure).
- **Key technical details**:
  - **Protocol**: pkt-line over stdin/stdout, with blob OIDs for future caching.
  - **Validation**: Hunks are checked for bounds, overlap, and per-gap alignment.
  - **Bypass mechanisms**: `--no-ext-diff`, `--diff-algorithm`, and whitespace-ignoring flags skip the external tool.
  - **Test coverage**: 1,146+ lines in `t/t4080-diff-process.sh`, including edge cases (empty files, overlapping hunks).
- **Next steps**: Junio is expected to queue the series for `next` shortly. The only remaining work is administrative (merging).
- **Files touched**: `diff-process.c`, `diff-process.h`, `xdiff/xdiffi.c`, `blame.c`, `line-log.c`, documentation.

---

#### **3. `git mv` symlink handling: Junio approves v4**
**Headline**: *`git mv` now rejects all tracked symlinks in destination paths*
**Thread**: [2026/07/15/14-32-43](https://lore.kernel.org/git/20260726232912.12345-1-lucaszam0@gmail.com/)

- **What happened**: Lucas Zamboni Orioli’s **two-patch series** (v4) to fix misleading error messages and unsafe symlink traversal in `git mv` received **Junio’s final approval**. The series now rejects **any tracked symlink** in the destination path (regardless of target type), aligning with `git apply` precedent and preventing index corruption.
- **Why it matters**: The bug left the index recording the literal symlink path while the file was moved to the resolved target, causing `git status` to show a staged rename, an unstaged deletion, and an untracked file. The fix is **user-facing** and addresses a long-standing edge case.
- **Key technical details**:
  - **Implementation**: Uses `has_symlink_leading_path()` to avoid following symlinks.
  - **Error messages**: Now include both source and destination paths (e.g., "renaming A -> B failed: no such file or directory").
  - **Test coverage**: 83 new lines in `t/t7001-mv.sh`, including a regression test for the index corruption scenario.
- **Next steps**: Junio will likely queue the series for `next` in the next "What’s cooking" cycle.
- **Files touched**: `builtin/mv.c`, `t/t7001-mv.sh`.

---

#### **4. Rebase fixups: Junio marks series for `next`**
**Headline**: *Interactive rebase edge cases fixed after string-matching dispute resolved*
**Thread**: [2026/07/18/04-39-18](https://lore.kernel.org/git/20260726153858.12345-1-phillip.wood123@gmail.com/)

- **What happened**: Phillip Wood’s **two-patch series** (v2) to fix edge cases in `git rebase -i` (incorrect commit counts in squash/fixup templates and skipped `fixup -c` commands) received **Junio’s final approval**. The series is now **ready for `next`**.
- **Why it matters**: The bugs affected niche but real workflows (e.g., skipping fixup/squash commands mid-rebase). The fixes are **surgical** and well-tested, with 83 new lines in `t/t3418-rebase-continue.sh` and `t/t3437-rebase-fixup-options.sh`.
- **Key technical details**:
  - **Patch 1**: Fixes commit count by removing orphaned newlines in the sequencer’s command-list file.
  - **Patch 2**: Introduces `seen_fixup_edit_msg()` to track `fixup -c` commands consistently.
  - **String-matching dispute**: Junio initially suggested tightening the helpers to require a space after `squash`/`fixup -c`, but Phillip argued this was unnecessary for an internally generated file. Junio accepted the argument.
- **Next steps**: Junio will queue the series for `next`.
- **Files touched**: `sequencer.c`, test scripts.

---

#### **5. Newcomer discussion: Advice and gaps in Git**
**Headline**: *Veterans share onboarding tips and identify project gaps*
**Thread**: [2026/07/26/20-01-07](https://lore.kernel.org/git/20260726200107.12345-1-iredvm@icloud.com/)

- **What happened**: Daniel Catoni, a newcomer, asked for **advice on contributing to Git** and **current project gaps**. The thread attracted responses from veterans, including:
  - **Onboarding tips**: "Start with small documentation fixes or test improvements" (René Scharfe), "Focus on subsystems you use daily" (Jeff King).
  - **Project gaps**:
    - **Observability**: Lack of telemetry for common operations (e.g., `git fetch` performance).
    - **Documentation**: Inconsistent man-page synopsis styles (Jean-Noël Avila’s ongoing effort).
    - **Test infrastructure**: Pervasive exit-code masking in test scripts (addressed by Shlok Kulshreshtha’s `commit_body()` series).
    - **Usability**: `git stash` redesign (Junio’s suggestion to tie stashes to branches).
- **Why it matters**: Rare **meta-discussion** about Git’s development culture and pain points. The thread may guide future contributors and prioritize non-technical improvements.
- **Key quote (Junio)**:
  > "The stash design feels outdated. Maybe it’s time to tie stashes to branches instead of a global list."

---

### **In brief**
- **`git repo info` path keys**: K Jayatheerth’s **7-patch series** (v3) adding path-related keys (e.g., `path.toplevel`, `path.hooks`) is **mechanically clean** and ready for substantive review. Junio invited "real reviews" from the community.
  > **Files**: `builtin/repo.c`, `Documentation/git-repo.adoc`, `t/t1900-repo-info.sh`.

- **Packfile URI race fix**: Ted Nyman’s **v5 series** (merged to `next`) fixed race conditions in HTTP packfile downloads. Peff’s post-acceptance feedback identified **advisory cleanups** (e.g., splitting Windows workarounds into separate commits), but no reroll is planned.

- **`git diff --relative --cached` fixups**: Jeff King’s **two-patch refactoring** (following his bugfix) was **approved by Junio** and queued for `next`. The patches remove a stale comment and optimize prefix-checking logic.

- **UTF-8 string-width refactoring**: Hardik Kumar’s **v2 patch** updated `utf8_strwidth()` to return `size_t`, addressing reviewer concerns about caller compatibility. The overflow check question remains unresolved.

- **CI toolchain bump**: Jeff King’s **two-patch series** to upgrade the Ubuntu image for static-analysis jobs (resolving a Coccinelle performance issue) was **approved by Junio** and is ready for `next`.

- **`commit_body()` helper**: Shlok Kulshreshtha’s **two-patch series** introduced a reusable shell helper to extract commit message bodies, fixing a pervasive exit-code masking bug in the test suite. The series is **under review** but likely to land soon.

---

### **On the radar**
- **ODB alternates redesign**: Patrick Steinhardt’s implementation (planned for late July) will be the **most consequential architectural change** of the cycle.
- **`git stash` redesign**: Junio’s suggestion to tie stashes to branches may gain traction after the `stash rename` discussion.
- **Rustification**: Ezekiel Newren’s effort remains active, with no major updates today. Randall S. Becker’s platform concerns (NonStop) are still unresolved.