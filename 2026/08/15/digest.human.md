# Git mailing list daily digest for 2026/08/15

## The day in brief

Junio C Hamano confirmed the `fetch_if_missing` libification series is ready for queuing, while a new bugfix patch for worktree repair surfaced, addressing dead code in `.git` file path resolution. The MinGW/Windows build series saw real-world validation from Johannes Sixt, and the stalled RFC on filesystem-level copy-on-write for `git worktree add` received a critical correctness warning about ReFS.

## Notable threads

### `fetch_if_missing` libification ready for queuing
**What changed?**
Junio C Hamano [2026/08/15/15-58-49] confirmed Tian Yuchen’s v6 patch series moving the global `fetch_if_missing` variable into `struct repository` is mechanically complete and ready for queuing. The series eliminates all direct uses of `the_repository` for this flag, preserving existing behavior (default value remains `1`, enabled).

### Why it matters

This continues the project-wide effort to eliminate global state in favor of repository-specific configuration, a prerequisite for Git’s libification goals. The patch touches 15 files, including critical paths like `index-pack.c` and `revision.c`, and resolves a design inelegance in `option_parse_missing_action()` by using the `value` field in `OPT_CALLBACK_F()` to pass the repository address directly.

### Open question

Junio questioned whether running `git index-pack` outside a repository is a legitimate use case or a user error, but explicitly stated the established behavior should not be altered in this patch. The broader architectural question of whether `fetch_if_missing` belongs in `struct repository` remains unresolved.

---

### Worktree repair bugfix: dead code in `.git` file path resolution
**What changed?**
Yoichi NAKAYAMA [2026/08/15/13-11-19] posted a bugfix patch correcting how Git detects and handles relative versus absolute paths in `.git` files during worktree repair. The patch modifies `read_gitfile_gently()` to optionally report whether the path in the `.git` file was originally absolute or relative, eliminating dead code that could never execute because the function always returned an absolute path.

### Why it matters

The worktree repair logic previously contained dead branches that could never trigger due to `read_gitfile_gently()`’s behavior. The patch introduces an `absolute` output parameter to preserve the original path style, enabling the repair logic to detect mismatches between configured and actual path styles. The test suite is expanded with four new cases verifying transitions between absolute and relative paths in both main and side worktrees.

### Files touched

`builtin/init-db.c`, `setup.c`, `setup.h`, `worktree.c`, and `t/t2406-worktree-repair.sh`.

---

### MinGW/Windows build series gains real-world validation
**What changed?**
Johannes Sixt [2026/08/15/14-10-23] reported running a subset of Johannes Schindelin’s 12-patch MinGW/Windows build series (patches 2–6, 8, and either 9 or 12) in personal Windows builds since November 2025 without issues. The full series also passes GitHub CI on Windows.

### Why it matters

The series upstream Git for Windows-specific adjustments, including Python buildability, UCRT64 compatibility, and direct `git.exe` invocation. Sixt’s validation provides real-world confirmation that the core changes are stable, though two open suggestions remain: guarding against an empty `MSYSTEM` export in Patch 12 and reordering CMake/Meson hunks in Patch 8.

### Key detail

Patch 12 allows `git.exe` to be called directly instead of through the "Git wrapper," eliminating an extra process hop by initializing `MSYSTEM`, `PLINK_PROTOCOL`, and a carefully constructed `PATH`.

---

### ReFS corruption risk blocks copy-on-write RFC
**What changed?**
Brian M. Carlson [2026/08/15/14-33-09] reported that ReFS’s block cloning implementation is unreliable when data has been written but not flushed to disk, risking corruption. He cited a recent Git LFS issue as evidence and argued Git should not add ReFS support until only fixed versions of Windows are publicly available.

### Why it matters

The RFC proposed filesystem-level copy-on-write (CoW) or block cloning optimizations for `git worktree add` to reduce storage overhead, with Windows (ReFS) as a key target platform. Carlson’s report introduces a concrete, platform-specific blocker that undermines the assumption that ReFS block cloning is safe to use. Peter Morris [2026/08/15/18-09-00] later clarified that the Windows `CopyFile` API’s locking behavior prevents the race condition described, but the underlying ReFS bug remains unresolved.

### Architectural challenge

The checkout machinery’s lack of visibility into sibling worktrees remains the primary blocker, requiring a complex refactoring of `checkout_entry()`.

---

### Bash completion slowdown: `feature.manyFiles` ruled out
**What changed?**
Matthew Hughes [2026/08/15/19-52-46] ruled out `feature.manyFiles` (fsmonitor) as a mitigation for the bash completion slowdown in large repositories. The issue stems from `git ls-files --exclude-standard --others --directory` with wildcard patterns, which forces Git to scan every directory and generate hundreds of thousands of system calls.

### Why it matters

The slowdown (1–2 s hang) occurs in repositories with hundreds of thousands of tracked files, making completion unusable. Hughes tested `feature.manyFiles` in a fresh repository and observed no measurable improvement, confirming the optimization does not address the root cause. The workaround (`GIT_COMPLETION_NO_COMPLETE_INDEX`) sacrifices accuracy for speed, but no deeper fix has been proposed.

### Key detail

The completion script’s use of `git ls-files` bypasses fsmonitor and the untracked cache, leaving the wildcard-driven scans unoptimized.

---

## In brief
- **`git repo info` path keys**: Lucas Seiki Oshiro [2026/08/15/23-06-45] asked why the test for the first patch does not use the `test_repo_info_path` helper, a surface-level question about test hygiene.
- **`git format-rev` options**: Junio C Hamano [2026/08/15/02-17-34] identified a defensive programming issue in the `date_cb()` callback, where the `BUG_ON_OPT_NEG(unset)` assertion was placed after dereferencing `option->value`.
- **Submodule config bugfix**: tilak-raaz [2026/08/15/07-18-29] posted v4 of a patch fixing a crash when `submodule.active` is set without a value, replacing `repo_config_get_string_multi()` with `repo_config_get_value_multi()` to safely handle valueless entries.
- **`chdir_notify` refactoring**: Jeff King [2026/08/15/05-12-13] confirmed the v2 commit message for removing the unused `name` parameter from the `chdir_notify` API now fully documents the historical context.
- **What’s cooking**: Junio’s [2026/08/15/00-33-49] report listed `kh/format-rev-more-options`, `ps/odb-pluggable-pack-generation`, `ty/repository-fetch-if-missing`, and `hn/history-squash` as new topics under review.