# Git mailing list daily digest for 2026/08/15

## The day in brief
Junio C Hamano confirmed the `fetch_if_missing` libification series is ready for queuing, while a MinGW/Windows build series received real-world validation from Johannes Sixt. A stalled RFC on filesystem-level copy-on-write for `git worktree add` faced a critical correctness issue on Windows (ReFS), and a new bugfix patch addressed path resolution in worktree repair.

## Notable threads

### `fetch_if_missing` libification ready for queuing
**What changed**: Junio C Hamano confirmed the v6 patch series moving the global `fetch_if_missing` variable into `struct repository` is mechanically complete and ready for queuing. The series eliminates all direct uses of `the_repository` for this flag, continuing the project-wide effort to remove global state.

**Problem/goal**: The global `fetch_if_missing` variable controls whether missing objects trigger lazy fetches from promisor remotes. Moving it into `struct repository` enables per-repository control, a key step in Git’s libification effort.

**Subsystem**: Repository initialization, object database, partial clone.

**Impact**: The change preserves existing behavior (default value remains `1`, enabled) while enabling future per-repository configuration. Special handling in `fsck.c` and `index-pack.c` ensures the flag is only accessed after parameter parsing to avoid null pointer dereferences.

**Today’s development**: Junio questioned whether running `git index-pack` outside a repository is a legitimate use case or a user error, but explicitly stated the established behavior should not be altered in this patch. The broader architectural question of whether `fetch_if_missing` belongs in `struct repository` remains unresolved, but the series is now ready for queuing unless new substantive concerns arise.

---

### MinGW/Windows build adjustments validated
**What changed**: Johannes Sixt reported running a subset of Johannes Schindelin’s 12-patch MinGW/Windows build and runtime adjustment series in personal builds since November 2025 without issues. The full series also passes GitHub CI on Windows.

**Problem/goal**: The series upstream Git for Windows-specific adjustments, including MSYS2 integration, compiler flexibility, and direct `git.exe` invocation, without affecting non-Windows platforms.

**Subsystem**: Build system (`config.mak.uname`, CMake, Meson), runtime compatibility (`compat/mingw.c`).

**Impact**: The patches enable Python buildability, remove deprecated compiler flags, and allow direct `git.exe` invocation (eliminating an extra process hop). Patches 10 (simplifying `ETC_GITCONFIG`/`ETC_GITATTRIBUTES` path logic) and 11 (locale-handling regression fix) remain uncontroversial.

**Today’s development**:
- Johannes Sixt proposed moving CMake and Meson hunks introducing compiler definitions (`ENSURE_MSYSTEM_IS_SET` and `MINGW_PREFIX`) from Patch 8 to Patch 12, where the definitions are consumed by the C code, to improve logical ordering.
- Sixt also reported real-world validation of the series, running patches 2–6, 8, and either 9 or 12 in personal Windows builds since November 2025 without issues. The full series passes GitHub CI on Windows, providing confidence in its stability.

---

### Filesystem-level copy-on-write for `git worktree add` stalled
**What changed**: Brian M. Carlson reported a critical correctness issue with ReFS, the Windows filesystem proposed for filesystem-level copy-on-write (CoW) optimizations in `git worktree add`. Carlson cited a recent Git LFS issue where ReFS’s block cloning implementation corrupted data if writes were not flushed to disk.

**Problem/goal**: The RFC proposed leveraging CoW or block cloning (e.g., ReFS, Btrfs, APFS) to avoid physically copying files during `git worktree add`, reducing SSD wear and speeding up worktree creation for large repositories.

**Subsystem**: Worktree management, checkout machinery, platform compatibility.

**Impact**: The optimization would benefit scenarios like concurrent AI coding agents or large repositories with static assets (e.g., 80GB of images), but the ReFS issue undermines its viability on Windows.

**Today’s development**:
- Carlson reported that ReFS’s block cloning is unreliable when data has been written but not flushed to disk, risking corruption. He cited a Git LFS issue as evidence and argued Git should not add ReFS support until only fixed versions of Windows are publicly available.
- Peter Morris clarified that the Windows `CopyFile` API’s locking behavior (shared-read, deny-write) prevents the race condition described by Carlson, but did not address the underlying ReFS bug. The thread remains stalled pending resolution of the ReFS issue or a new contributor to tackle the architectural challenges.

---

### Worktree repair bugfix for `.git` file path resolution
**What changed**: Yoichi NAKAYAMA posted a bugfix patch correcting how Git detects and handles relative versus absolute paths in `.git` files during worktree repair. The patch introduces an optional parameter to `read_gitfile_gently()` to report the original path style and removes dead code that could never execute.

**Problem/goal**: The existing code contained dead branches because `read_gitfile_gently()` always returned an absolute path, making it impossible to detect mismatches between the configured path style (absolute or relative) and the actual `.git` file contents.

**Subsystem**: Worktree management, path resolution, `.git` file parsing.

**Impact**: The patch ensures the worktree repair logic correctly handles transitions between absolute and relative paths in both main and side worktrees, including emitting appropriate error messages for mismatches.

**Today’s development**: The patch modifies `read_gitfile_gently()` to optionally report whether the path in the `.git` file was originally absolute or relative. Four new test cases in `t/t2406-worktree-repair.sh` verify the repair logic handles path style transitions correctly. The changes are mechanical and focused, addressing a real correctness issue in a subsystem prone to subtle path-handling bugs.

---

### `git format-rev` gains formatting options
**What changed**: Junio C Hamano identified a defensive programming issue in the `date_cb()` callback of Kristoffer Haugsbakk’s patch series adding `--abbrev`, `--color`, and `--date` options to `git format-rev`. The `BUG_ON_OPT_NEG(unset)` assertion was placed after dereferencing `option->value`, which could lead to a segfault if the callback were ever invoked with `unset==1`.

**Problem/goal**: The series extends `git format-rev` with formatting options to bring it closer to parity with `git log`'s flexibility. The preparatory patches refactor option handling and documentation, while the final patch reuses existing `git log` logic.

**Subsystem**: Revision walking, formatting, CLI options.

**Impact**: The new options (`--abbrev`, `--color`, `--date`) enable users to customize the output of `git format-rev` without altering its core functionality. The series is uncontroversial and reuses well-tested logic from `git log`.

**Today’s development**: Junio suggested reordering the code to ensure the assertion triggers before any dereference of `option->value`. The fix is trivial (reordering two lines) and will likely be incorporated into the next revision. The rest of the patch remains uncontroversial.

---

### Bash completion performance regression
**What changed**: Matthew Hughes ruled out `feature.manyFiles` (fsmonitor) as a mitigation for the bash completion slowdown in large repositories. The slowdown occurs because the completion script invokes `git ls-files --exclude-standard --others --directory` with a wildcard pattern, forcing Git to scan every directory in the repository.

**Problem/goal**: Git’s bash completion script becomes unusably slow (1–2 s hang) when completing paths for commands like `git add` in repositories with hundreds of thousands of tracked files. The root cause is the completion logic’s wildcard-driven `git ls-files` invocations.

**Subsystem**: Bash completion (`contrib/completion/git-completion.bash`), path completion.

**Impact**: The issue affects usability in large repositories, particularly those with many directories (e.g., 25,000 directories × 12 files). The workaround (`GIT_COMPLETION_NO_COMPLETE_INDEX`) sacrifices completion accuracy for speed.

**Today’s development**: Hughes noted that `feature.manyFiles` is ineffective for this use case and that his test repository already disables it via `git-scalar`. The discussion now focuses on whether the completion script can be optimized or made smarter about large repositories.

---

### Submodule configuration bugfix
**What changed**: tilak-raaz posted v4 of a bugfix patch addressing a crash when parsing a valueless `submodule.active` configuration entry (e.g., `[submodule] active` with no pathspec). The patch replaces `repo_config_get_string_multi()` with `repo_config_get_value_multi()` to safely handle such entries and emits a warning via `config_error_nonbool()`.

**Problem/goal**: Git crashes when encountering a valueless `submodule.active` entry. The patch ensures malformed entries are skipped after warning, preserving existing caller semantics.

**Subsystem**: Submodule configuration parsing.

**Impact**: The fix resolves a documented `NEEDSWORK` comment in `submodule.c` and includes test coverage in `t/t7400-submodule-basic.sh`. The change is narrowly scoped and does not alter the behavior for valid entries.

**Today’s development**: The v4 iteration fixes commit message tense and terminology ("valueless true") and corrects whitespace in the test script. The patch is ready for review.

---

## In brief
- **[2026/07/16/01-21-31]** Lucas Seiki Oshiro asked why the test for the `git repo info` path-related keys series does not use the `test_repo_info_path` helper, which would reduce boilerplate and align with project testing conventions.
- **[2026/08/14/19-38-49]** Jeff King confirmed the v2 commit message for the `chdir_notify` refactoring patch satisfactorily documents the historical context behind the unused `name` parameter’s removal.
- **[2026/08/15/00-33-49]** Junio’s "What’s cooking" report listed new topics under review, including `kh/format-rev-more-options`, `ps/odb-pluggable-pack-generation`, `ty/repository-fetch-if-missing`, and `hn/history-squash`. Stalled topics include `bl/t7412-use-test-path-helpers` and `tb/repack-geometric-cruft`.