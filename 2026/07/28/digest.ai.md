Here’s the daily digest for **2026/07/28**, covering the Git mailing list traffic:

---

### **The day in brief**
A **busy but focused** day on the Git mailing list, with **78 emails across 27 threads**. The standout developments:
1. **`git replay --linearize` v8** landed, resolving the "multi-branch ambiguity" that had blocked the series in `next`.
2. **`git stash reword`** gained clarity on its reflog API requirements, with Junio detailing a `refs_reflog_edit_in_bulk()` design.
3. **Memory leaks and security fixes** dominated the bugfix landscape, including a long-standing leak in `git merge` and a Windows credential exfiltration patch.
4. **ODB abstraction** saw active review, with Patrick Steinhardt’s series making progress toward pluggable backends.

The day’s tone was **collaborative and forward-looking**, with reviewers digging into design details (e.g., `--allow-duplicate-objects` vs. `--strict`) and maintainers queuing patches for integration.

---

### **Notable threads**

#### **`git replay --linearize` v8 resolves multi-branch ambiguity**
**Headline**: The `--linearize` option for `git replay` is now restricted to single-branch usage, addressing Elijah Newren’s "minimal fix" proposal and unblocking the series for `next`.
**What happened**: Toon Claes posted v8 of the series, which disallows `--linearize` with multiple revision ranges or `--contained`. The change eliminates emergent reachability issues while preserving the option’s core functionality. Junio removed the prior version from `next` and will queue v8 for review, signaling readiness for integration.
**Why it matters**: This series supersedes Johannes Schindelin’s earlier merge-replay implementation (pull.2106) and provides a predictable, all-or-nothing flattening behavior. The restriction to single branches aligns with Git’s principle of least astonishment and paves the way for future per-branch linearization (e.g., via `--ref` syntax).
**Key details**:
- **Files touched**: `replay.c`, `replay.h`, `builtin/replay.c`, `Documentation/git-replay.adoc`, `t/t3650-replay-basics.sh`.
- **New restrictions**: `--linearize` cannot be combined with multiple positive revisions or `--contained`.
- **Test coverage**: New tests verify the multi-range and `--contained` restrictions, alongside existing edge cases (e.g., replaying to root, `--advance` compatibility).
**Loose ends**: Whether `--linearize` should eventually support per-branch linearization (deferred to future work).

---

#### **`git stash reword` API design takes shape**
**Headline**: Junio C Hamano detailed a `refs_reflog_edit_in_bulk()` API to fix the data-loss risk in `git stash reword`, shifting the discussion from "should we do this?" to "how?".
**What happened**: Junio’s follow-up to Emin Özata’s v2 patch outlined how the new API would handle edge cases like index drift during insert/delete operations, multi-line messages, and backend consistency (files vs. reftable). The design includes pre-scanning the edit array, stable sorting, and validation of impossible sequences (e.g., deleting an entry before replacing it).
**Why it matters**: The current implementation’s data-loss risk (reflog exists only in memory during rewrite) is a blocker. The proposed API would make `stash reword` truly atomic and could unify other reflog operations (e.g., `stash drop`, `reflog expire`).
**Key details**:
- **API name**: `refs_reflog_edit_in_bulk()`.
- **Edge cases**: Index drift, multi-line messages (squashed to single line), and validation of edit sequences.
- **Backend mechanics**: Files backend would use a temporary file; reftable would handle edits more efficiently.
**Loose ends**: Implementation work remains; no patch yet.

---

#### **Memory leak in `git merge` plugged**
**Headline**: Toon Claes fixed a long-standing memory leak in `git merge` when `merge.defaultToUpstream` is enabled, with Junio queuing the patch for `seen`.
**What happened**: The leak occurred when `setup_with_upstream()` allocated an `argv` array that was never freed. Toon’s patch tracks the array with a new `argv_to_free` variable and frees it at the end of `cmd_merge()`. Jeff King (Peff) confirmed the leak and the fix, though he noted the surrounding code’s awkwardness (e.g., overwriting `argv[0]`).
**Why it matters**: While the leak is small, it’s been present since 2011 and affects a common workflow (rebasing with upstream). The fix is minimal and well-tested, with a new test in `t/t7600-merge.sh` exercising the default-to-upstream path.
**Key details**:
- **Files touched**: `builtin/merge.c`, `t/t7600-merge.sh`.
- **New variable**: `argv_to_free` in `cmd_merge()`.
- **Test coverage**: New test confirms the leak and fix under leak sanitizer.

---

#### **Windows credential exfiltration patched**
**Headline**: Johannes Schindelin’s patch to prevent credential exfiltration via symlink auto-detection in Git for Windows was queued for integration.
**What happened**: The patch closes a vulnerability (CVE-2026-32631) where a crafted repository with symlinks enabled could leak the user’s NTLMv2 hash via an SMB connection triggered by symlink type auto-detection. The fix skips auto-detection for targets starting with a backslash (UNC paths or drive-less absolute paths), defaulting to file symlinks and warning users to set `symlink=dir` in gitattributes if needed.
**Why it matters**: This is a **high-priority security fix** for Windows users, already released in Git for Windows v2.53.0(3). The patch aligns with prior mitigations for similar vulnerabilities (e.g., CVE-2025-66413 in MinTTY).
**Key details**:
- **Files touched**: `compat/mingw.c`.
- **Function modified**: `process_phantom_symlink()`.
- **Behavior**: Skips auto-detection for backslash-prefixed targets; emits a warning.

---

#### **ODB abstraction series sees active review**
**Headline**: Patrick Steinhardt’s ODB abstraction series (making on-disk structures pluggable) received substantive feedback from Justin Tobler, focusing on initialization sequence design and multi-backend consistency.
**What happened**: Justin’s reviews of patches 1–3 raised questions about:
1. **Initialization sequence**: Whether the `APPLY_REPOSITORY_FORMAT_SKIP_ODB_CREATION` flag is the cleanest way to defer ODB creation (suggesting an explicit `odb_new()` call instead).
2. **Multi-backend consistency**: Whether `should_use_loose_object_map()` might behave inconsistently across multiple backends, leading to redundant checks.
3. **Downcast safety**: The robustness of unconditional downcasts in loops iterating over ODB sources.
**Why it matters**: This series is foundational for future ODB backends (e.g., cloud-based object stores). The feedback highlights design tensions between flexibility and consistency, with Justin’s questions probing the long-term maintainability of the approach.
**Key details**:
- **Files touched**: `loose.c`, `odb/source-loose.c`, `odb/source-files.c`, `setup.c`, `repository.c`.
- **New symbols**: `create_on_disk` callback, `loose_object_map_load()`.
- **Edge cases**: Multi-backend scenarios, initialization order, and downcast safety.

---

### **In brief**
- **`git cat-file --batch-command` assertion failure**: Jeff King’s patch to fix a NULL dereference when `%(objecttype)` is omitted was queued for `next`. The fix saves/restores `data->info.typep` around the `contents` command.
- **Pack-bitmap off-by-one fix**: David Lin’s patch to correct an edge case in `find_objects()` (objects at position zero in the base bitmap) was accepted for integration. The fix changes `pos > 0` to `pos >= 0` in `pack-bitmap.c`.
- **`git repo info` path-keys series**: K Jayatheerth’s v3 series (adding path-related keys like `path.objects` and `path.hooks`) saw substantive feedback from Justin Tobler on ODB abstraction compatibility and `/dev/null` handling. The author will address a forward-compatibility issue in patch 1/7 (use of `startup_info->prefix`).
- **`git add --resolved`**: Junio posted a new feature series introducing `--resolved` to stage only conflict-resolved paths. The series consolidates conflict-marker detection logic and adds a helper for index removal with flags.
- **`WITH_BREAKING_CHANGES` test failures**: Jeff King’s two-patch series fixed `t0014-alias.sh` failures by dynamically generating the list of deprecated commands. The `HAVE_DEPRECATED` prereq skips tests if no deprecated commands are available.
- **`git history fixup` worktree corruption**: Toon Claes and Phillip Wood discussed implementation choices for fixing index corruption in other worktrees. Options include refusing to rewrite branches checked out elsewhere (safer) or attempting a "best-effort" update (more user-friendly).
- **UTF-8 string-width documentation**: Hardik Kumar’s v5 patch to document the rationale for `utf8_strwidth()` returning `int` (alignment with `printf()`-style APIs) is ready for final revision after addressing feedback from Junio and Phillip Wood.

---

### **On the radar**
- **`git stash reword` API implementation**: The `refs_reflog_edit_in_bulk()` design is now well-defined; expect a patch soon.
- **ODB abstraction follow-ups**: Justin Tobler’s questions about initialization sequence and multi-backend consistency may prompt revisions to Patrick Steinhardt’s series.
- **`fetch.bundleURI` performance**: Ryszard Knop’s inquiry about slow bundle-based clones of the Linux kernel repository remains unanswered. The bottleneck (`git index-pack` single-threaded) suggests potential for optimization.
- **`git rebase --update-refs` symref bug**: Son Luong Ngoc’s v4 is expected to address the design tension between robustness and simplicity in handling symrefs to tags/other namespaces.