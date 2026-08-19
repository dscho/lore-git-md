# Git mailing list daily digest for 2026/08/18

## The day in brief
The Git mailing list saw active discussion on several fronts today. A **security feature** adding OCSP staple validation (`http.sslVerifyStatus`) reached its final documentation tweak before merging. **ODB refactoring** continued with Junio raising design concerns about error signaling conventions. **Geometric repacking** received a critical bugfix for a race condition causing missing-object errors. **Zsh completion** and `git repo info` patches advanced toward readiness. A new **`report` hook** for `git-receive-pack` sparked design questions about exit status handling.

## Notable threads

### ODB corruption handling refactoring
**What changed**: Junio C Hamano raised substantive design concerns about Patrick Steinhardt's seven-patch series refactoring how the object database handles corrupt objects. The series introduces a tri-state return convention for `odb_source_read_object_info()` (zero for success, positive for missing, negative for corrupt), but Junio questions whether positive return values overload the error signaling mechanism. He suggests using negative error codes (e.g., `-ENOENT`) for all error conditions, including missing objects, and proposes an enum for clarity. In patch 3/7, Junio also questioned whether marking an entire pack as bad upon encountering a single corrupt object is too coarse for multi-pack scenarios.

**Why it matters**: This refactoring is part of the ongoing ODB abstraction effort, which aims to make Git's object storage layer more modular and backend-agnostic. The return-value convention affects how all ODB backends signal errors, making it a critical design decision. The `bad_pack` logic impacts performance in multi-pack scenarios, where a single corrupt object in one pack shouldn't necessarily invalidate the whole pack for unrelated objects.

**Current status**: The series remains under review, with two unresolved design concerns. Junio's feedback is narrowly scoped to the return-value semantics and the `bad_pack` logic, not the series' overall motivation or correctness. The author has not yet responded to these concerns.

---

### OCSP staple validation (`http.sslVerifyStatus`)
**What changed**: The `http.sslVerifyStatus` security feature reached its final documentation tweak in v6. Grayson Gordon addressed Junio C Hamano's request to rephrase the documentation to focus on the user-facing outcome ("prevents connections to servers with revoked certificates") rather than technical jargon ("OCSP stapling"). The v6 iteration is now technically complete, with all prior feedback addressed, including Patrick Steinhardt's requests for backend parity documentation, libcurl error strings, and test coverage.

**Why it matters**: This feature closes a security gap where Git ignores stapled OCSP responses, leaving OpenSSL-linked binaries (common in FIPS-compliant deployments) vulnerable to accepting revoked certificates. The default-off design preserves compatibility, while the fail-closed behavior when enabled ensures revoked certificates are rejected. The feature is critical for government and enterprise users who mandate OCSP stapling.

**Current status**: The patch is ready for `next` and eventual graduation to `master`. The only deferred item is a positive test case for valid OCSP staple validation, which the author prototyped but left out due to complexity. Junio and Patrick have not insisted on its inclusion.

---

### Geometric repacking race condition fix
**What changed**: Elijah Newren introduced a two-patch bugfix series addressing a race condition in Git's geometric repacking mechanism. The issue arises when a multi-pack-index (MIDX) references a packfile removed by a concurrent geometric repack, causing processes using the stale MIDX to fail to locate objects. Patch 1/2 fixes a SIGSEGV in `git replay` by adding a missing check for unreadable objects. Patch 2/2 modifies the packed-object backend to recover from MIDX-packfile mismatches by scanning all packs covered by the MIDX when the normal lookup fails.

**Why it matters**: This race condition affects multiple server-side operations, including `git merge-tree`, `git diff`, `git rev-list`, and object resolution commands like `rev-parse` and `cat-file`. The bug can cause silent data corruption or crashes, making it a high-priority fix for production environments. The series is well-motivated by real-world evidence and includes thorough test coverage.

**Current status**: The series is under review and uncontroversial. No objections or requests for changes have been raised. The patches are small, focused, and include tests for both the crash scenario and MIDX recovery logic.

---

### Zsh completion bugfix
**What changed**: The zsh completion bugfix thread wrapped up with D. Ben Knoble confirming the v2 patch works as intended and accepting the deferral of remaining gaps (chained `-C` completion, `git -<tab>` bug). Lutz Lengemann confirmed the patch is ready and has been updated in the GitGitGadget PR.

**Why it matters**: The patch fixes a long-standing issue where zsh completion fails for commands or arguments when global options (`-C`, `--git-dir`, etc.) are used. The fix aligns zsh completion with the bash implementation, ensuring consistent behavior for constructs like `git -C <path> <command> <TAB>`. The remaining gaps are documented and left for follow-ups.

**Current status**: The patch is effectively unblocked and waiting for final review or merge. All substantive feedback has been addressed, and the remaining limitations are documented in the commit message.

---

### `git repo info` path keys
**What changed**: Junio C Hamano intervened in the design debate over `path.git-prefix`, rejecting Lucas Seiki Oshiro's objection that the key represents "current directory information" rather than "repository information." Junio analogized `path.git-prefix` to `git rev-parse --show-cdup` and `path.commondir.relative`, arguing both are inherently repository-centric. He proposed adding `path.cdup` for symmetry, framing it as a logical extension of the existing design.

**Why it matters**: The `git repo info` command exposes filesystem locations of repository components in a user-facing, scriptable format. The `path.git-prefix` key provides the relative path from the top-level working tree to the current directory, which is useful for scripting. Junio's proposal to add `path.cdup` (the inverse of `path.git-prefix`) completes the set of relative-path keys, making the design more intuitive and consistent.

**Current status**: The design question is now resolved in favor of including `path.git-prefix`, contingent on the addition of `path.cdup`. Future discussion will likely focus on implementation details for the new key.

---

### `report` hook for `git-receive-pack`
**What changed**: Karthik Nayak introduced a new `report` hook for `git-receive-pack`, allowing server administrators to intercept and modify the status report sent back to the client. Junio C Hamano raised design questions about the interaction between the hook's exit status and its stdout, noting the current implementation discards stdout on non-zero exit.

**Why it matters**: The hook addresses a gap in Git's server-side extensibility, enabling use cases like GitLab's multi-version concurrency control (MVCC) system. The design question—whether the exit status should override the rewritten report or whether the two signals should be orthogonal—affects how administrators can use the hook to enforce policies or modify responses.

**Current status**: The patch is under review, with Junio's feedback highlighting a potential ambiguity in the hook's behavior. The author has not yet responded to these concerns.

---

### `git pull --hard`
**What changed**: Artur Bieniek introduced `git pull --hard`, a new mode that resets the current branch, index, and working tree to the fetched branch using `git reset --hard`. Junio C Hamano objected to adding destructive operations to `git pull`, preferring the explicit `git fetch && git reset --hard` sequence.

**Why it matters**: The feature targets users who want to discard local state and sync exactly to a remote branch without merge or rebase steps. Junio's objection frames the decision as a design philosophy question: whether `git pull` should remain focused on history integration or also include destructive operations. The discussion highlights the tension between convenience and the risk of accidental data loss.

**Current status**: The patch is under review, with Junio's feedback indicating it is unlikely to be accepted in its current form. The author may choose to defend the feature, propose an alternative command, or abandon the effort.

---

### Scalable connectivity check
**What changed**: Kristofer Karlsson proposed a two-part optimization for `check_connected()` to make it proportional to the incoming set rather than the entire repository size. The proposal includes "opportunistic trusted-tree discovery" (lazy verification of changed entries) and "finding the boundary" (bounded walk seeded with likely-useful refs).

**Why it matters**: The current `check_connected()` implementation scales with the entire repository size, creating unnecessary overhead for frequent small pushes. The optimization targets large repositories with high concurrency, where the check's cost can become a bottleneck. Preliminary numbers show a 30–150× speed-up for small pushes.

**Current status**: The thread is an RFC seeking feedback on whether the problem is worth solving and whether the proposed direction is sound. No patches have been posted yet.

---

### `git config -e` exit code propagation
**What changed**: The discussion about whether `git config -e` should propagate the editor's exit code continued, with Karthik Nayak and brian m. carlson supporting the change, while Junio C Hamano framed it as a trade-off between conceptual correctness and script breakage. Karthik Nayak also suggested minor test-script cleanups (using `test_env` for `GIT_EDITOR` setup).

**Why it matters**: Propagating the editor's exit code aligns `git config -e` with commands like `git commit` and prevents scripts from silently ignoring editor failures. The feature was introduced mid-2024, limiting the scope of potential breakage. The discussion centers on whether the benefit of consistency outweighs the risk of breaking existing scripts.

**Current status**: The thread remains unresolved, with maintainer direction still pending. The patch is technically sound and includes test coverage, but the conceptual question is still open.

---

### Trace2 data point for pack bytes written
**What changed**: Junio C Hamano suggested simplifying the byte-accumulation logic in a Trace2 instrumentation patch for `git pack-objects`. The patch adds a new Trace2 event (`write_pack_file/wrote_bytes`) to record the cumulative size of all pack files generated during a single invocation. Junio questioned the clarity of the two-variable approach (`pack_bytes` and `bytes_written`) and proposed directly accumulating `bytes_written` in the common code path.

**Why it matters**: The patch fills an observability gap for measuring how compression settings affect push performance. Trace2 already records object counts but not byte counts, which are needed for performance analysis. Junio's feedback focuses on code clarity, not the patch's motivation or design.

**Current status**: The patch is under review, with Junio's feedback targeting a single aspect of the implementation. The author has not yet responded to the suggestion.

---

### Line-wise file indexing
**What changed**: Ernst Reissner requested line-wise file indexing in Git, citing its usefulness in Gittyup (a Git GUI client). The request lacks technical details about how the feature would work or what problem it would solve.

**Why it matters**: Line-wise indexing could enable new workflows or performance optimizations, but the request is too vague to evaluate. The author did not check whether this has been requested before or provide implementation specifics.

**Current status**: The thread is an expression of interest rather than a concrete proposal. No technical discussion or patches have been posted.

---

## In brief
- **[PATCH v4] repo info path keys**: K Jayatheerth explained why the `test_repo_info_path` helper is inapplicable to `path.toplevel` (returns `$ROOT` and `../`, not `$ROOT/$expected_dir`).
- **[PATCH v3] USE_NSEC to runtime config**: Junio C Hamano identified a correctness issue in the initialization logic for `use_nanosec`, which is unconditionally defined but only initialized when `NO_NSEC` is not defined, risking uninitialized reads in `NO_NSEC` builds.
- **[PATCH] odb: refactor corrupt objects**: Junio C Hamano pointed out a typo in patch 5/7's commit message subject line ("signal mark objects" → "signal missing objects").
- **[PATCH 0/7] odb: refactor corrupt objects**: Junio C Hamano published a "What's cooking in git.git" report for August 2026 (#08), summarizing the state of all in-flight topics, including critical bugfixes, ODB pluggability, libification, performance improvements, new features, security updates, and breaking-changes preparation for Git 3.0.
- **[PATCH] format-rev: add --abbrev, --color, --date**: Kristoffer Haugsbakk acknowledged a missing cleanup step in the `--date` callback, where `parse_date_format()` may allocate a custom format string that requires later release via `date_mode_release()`.
- **[PATCH] config: propagate editor failure exit code**: Karthik Nayak suggested minor test-script cleanups for the `git config -e` exit code patch, using `test_env` to set `GIT_EDITOR` inline rather than manually exporting and unsetting it.