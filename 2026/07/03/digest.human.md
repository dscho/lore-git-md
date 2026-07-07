# The Git Project Mailing List Daily Digest

**2026/07/03**

## The day in brief

A busy day on the list, with 106 emails across 26 threads. The standout developments: Patrick Steinhardt’s `git history drop` series was **merged** after a final whitespace nit was resolved, and Jeff King’s memory-leak fixes for non-default hash implementations gained **traction** with a consensus forming around idempotent cleanup. Two long-running series—Toon Claes’s `git replay --linearize` and Patrick’s reftable hardening—**advanced toward resolution**, while a new feature request for `git repo info` **sparked a design debate** about glob patterns versus prefix matching.

---

## Notable threads

### `git history drop` series merged
**What happened**: Patrick Steinhardt’s 11-patch series adding the `git history drop` subcommand was **merged** after Junio C Hamano fixed a final whitespace nit locally. The series modernizes the reset API, advances `the_repository` removal, and includes 561 lines of test coverage for edge cases like detached HEAD and bare repositories.

**Why it matters**: This is a significant milestone for Git’s history-editing toolkit, offering a more intuitive way to remove a commit and replay its descendants. The series also lays groundwork for future ODB and ref backend improvements.

**What’s next**: The feature will ship in Git 2.56. Follow-up work on root/merge commit support and test helper consolidation remains non-blocking.

---

### Memory-leak fixes for non-default hash implementations
**What happened**: Jeff King’s 9-patch series to plug memory leaks in OpenSSL and libgcrypt hash backends gained **substantive review** from Patrick Steinhardt and Brian M. Carlson. The discussion converged on making `git_hash_discard()` **idempotent**—a change that would simplify error paths and Rust integration.

**Why it matters**: The leaks are invisible with Git’s default hash backends but become critical under `SANITIZE=leak` with `OPENSSL_SHA256=1` or `GCRYPT_SHA256=1`. The proposed idempotency would align with Git’s existing resource-cleanup patterns (e.g., `strbuf_release()`).

**What’s next**: Jeff King will likely submit a follow-up patch to implement idempotency, adding an `active` flag to `struct git_hash_ctx` and enforcing strictness with `BUG()` on misuse.

---

### `git replay --linearize` nears resolution
**What happened**: Toon Claes’s v6 series introducing `--linearize` to `git replay` received **maintainer feedback** from Junio C Hamano. The series is technically complete, but Junio raised a **procedural nit** about authorship attribution (whether Johannes Schindelin should remain the primary author given the extensive v6 changes).

**Why it matters**: The `--linearize` option flattens merge commits, offering a simpler alternative to Johannes’s earlier merge-replay implementation. The series has been contentious due to its divergence from `git rebase`’s `--rebase-merges=<mode>` syntax, but the current design is now accepted as intentional.

**What’s next**: Junio will likely merge the series once the authorship question is clarified. The interface design debate is effectively closed.

---

### Reftable hardening series advances
**What happened**: Patrick Steinhardt’s 12-patch series hardening the reftable backend against corrupted files **addressed all prior feedback** in v3. The series includes fuzzing infrastructure, a test helper to reduce boilerplate, and fixes for out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage.

**Why it matters**: While reftable files aren’t stored remotely, the hardening is comprehensive and includes new fuzzing infrastructure to prevent regressions. The series is now **ready for merging**, with Junio having approved several patches as "obviously correct."

**What’s next**: Expect this to graduate to `next` shortly, as no technical blockers remain.

---

### `git repo info` prefix querying sparks design debate
**What happened**: A GSoC contributor proposed adding **category-based prefix querying** to `git repo info` (e.g., `git repo info layout` returns all `layout.*` keys). Junio C Hamano **critiqued the design**, arguing for glob patterns (e.g., `layout.*`) instead of simple prefix matching.

**Why it matters**: The feature addresses usability friction as the number of supported metadata keys grows. Junio’s feedback highlights a tension between simplicity (prefix matching) and expressiveness (globs), with a hybrid approach now favored.

**What’s next**: The contributor will likely revise the patch to support glob patterns, preserving backward compatibility with exact matches.

---

## In brief

**`git refs` subcommands**: Patrick Steinhardt’s series adding `create`, `delete`, `update`, and `rename` to `git refs` was **marked ready for `next`** after minor clarifications. The series consolidates reference manipulation under a unified interface.

**`GIT_TEST_LONG` tests**: Patrick Steinhardt’s 9-patch series making `GIT_TEST_LONG` tests reliable and efficient for CI **addressed all feedback** in v3. The series enables `GIT_TEST_LONG` in GitLab CI for integration branches and optimizes disk usage in expensive tests.

**`USE_NSEC` discussion**: The debate over whether to flip the default of `USE_NSEC` to `true` or make it runtime-configurable **remains unresolved**. D. Ben Knoble confirmed XFS preserves nanosecond timestamps, bolstering the case for a default flip, but interoperability risks (e.g., Git/JGit mixing) persist.

**`git rm` pathspec behavior**: A thread about `git rm -n *.json` unexpectedly recursing into subdirectories **converged on a documentation gap**. The behavior is correct (pathspecs match `/` by default), but `git-rm(1)` should cross-reference `gitglossary(7)` and mention the `:(glob)` modifier.

**`git rev-list --exclude-first-parent-only`**: Junio C Hamano submitted a **bugfix patch** for a misbehavior when additional commits are explicitly specified. The fix ensures the option correctly handles commits not part of the first-parent ancestry path.

**`greplint.pl`**: Michael Montalbo’s 6-patch series introducing a linter to convert bare `grep` assertions to `test_grep` **addressed all feedback** in v3. The series fixes pre-existing test bugs and includes shared shell parser infrastructure for future linters.

**`git history` signing**: Souma’s 3-patch series teaching `git history` to sign rewritten commits **added test coverage** for `reword`, `split`, and `fixup` subcommands. The series respects `commit.gpgsign` and command-line options like `-S/--gpg-sign`.

**macOS Unicode filename crash**: Ihar Hrachyshka’s patch fixing a crash in `precompose_utf8` with long UTF-8 filenames **addressed all feedback** in v2. The test now avoids Perl, and the fix converts `d_name` to a flexible array member.

---

## On the radar

**`includeIf.worktree` symlink inconsistency**: Patrick Steinhardt identified a **behavioral discrepancy** between `includeIf.worktree` and `includeIf.gitdir`—the former resolves symlinks to the real path, while the latter matches both symlinked and real paths. This must be resolved before the series graduates to `next`.

**`paint_down_to_common()` optimization**: Kristoffer Karlsson’s series optimizing `paint_down_to_common()` for one-sided histories **awaits a procedural rebase** on `kk/commit-reach-find-all-fix`. The series is otherwise ready for merging.

**CI consolidation**: The proposal to merge `linux-reftables-leaks` and `linux-TEST-vars` into `linux-TEST-vars-leaks` **gained maintainer endorsement**. Implementation is pending.