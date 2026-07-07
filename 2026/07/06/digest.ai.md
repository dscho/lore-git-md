Here is the digest for **2026/07/06 (Monday)**:

---

### The day in brief
A busy Monday on the Git mailing list, with **109 emails across 35 threads**. The day was dominated by **refactoring and infrastructure work**, with notable progress on the `git log --graph` cascading indentation series (now at v7 and ready for final review), the `git refs` subcommand consolidation (fully merged), and a systemic fix for rebase-dropped commits (queued for integration). **Performance optimizations** also featured prominently, with a merged patch resolving quadratic-time behavior in the reftable backend. **Usability improvements** included a fix for `git blame -b`’s hash abbreviation and a safeguard for `git subtree` to prevent mixed-tool corruption. The day’s tone was **collaborative and forward-looking**, with several threads converging on design resolutions and others advancing toward integration.

---

### Notable threads

#### `git log --graph` cascading indentation (v7)
**Pablo Sabater**’s seven-month effort to implement cascading indentation in `git log --graph` reached a milestone: the **v7 series is now complete and ready for final review**. The architectural crossroads (peek functions vs. lookahead buffer) has been resolved in favor of Kristofer Karlsson’s lookahead buffer redesign, which Junio Hamano approved. All three patches are now technically sound, with only minor cosmetic and defensive-programming feedback remaining (e.g., `ARRAY_SIZE()` usage, assertions). The series is **on track for inclusion** once Pablo incorporates the last suggestions. **Key files**: `graph.c`, `revision.c`, `t4218-log-graph-indentation.sh`.

#### `git refs` subcommand consolidation (merged)
Patrick Steinhardt’s **five-patch series adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` is now fully merged**. The series consolidates reference manipulation under a unified interface, improving discoverability and reducing fragmentation. Junio Hamano queued v4 after minor test additions clarified symbolic reference handling. **Usability questions** remain (e.g., `--no-deref`’s counterintuitive behavior in `git refs create`), but these are now documented in tests and error messages. **Key files**: `builtin/refs.c`, `t1464-refs-delete.sh` to `t1467-refs-rename.sh`.

#### Rebase-dropped commits (queued)
Phillip Wood’s **11-patch systemic overhaul** to prevent notes from being copied from dropped commits during rebase is now **queued in Junio’s tree**. The series addresses a long-standing correctness issue where the sequencer incorrectly records dropped commits as rewritten, corrupting note history. All known edge cases (empty changes, fixup sequences, external merge failures) are now handled, and the series carries Uwe Kleine-König’s `Tested-by`. **Key files**: `sequencer.c`, `t3400-rebase.sh`, `t5407-post-rewrite-hook.sh`.

#### Reftable backend performance (merged)
Patrick Steinhardt’s **two-patch fix for quadratic-time behavior in the reftable backend** is now merged. The patch removes the `suppress_deletions` flag from the merged iterator and pushes tombstone handling to call sites, enabling early termination and restoring O(1) behavior for `refs_verify_refnames_available()`. Performance improved from **~14s to ~0.2s** in the 8,000-ref perf test. **Key files**: `refs/reftable-backend.c`, `reftable/merged.c`.

#### `git history squash` (v7)
Harald Nordgren’s **`git history squash`** feature (folding commit ranges into a single commit) advanced to v7, addressing all prior feedback. The series now adopts `git rebase -i`’s squash-message template and rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`. Junio Hamano marked it **"Will replace"**, signaling intent to queue it. **Design questions** remain (e.g., whether `--reedit-message` should be the default), but the series is **functionally complete**. **Key files**: `builtin/history.c`, `t3455-history-squash.sh`.

#### Rustification build system (v6)
Shardul Natu and Koji Nakamaru’s **v6 series for macOS Universal Binary support in Rust components** resolved the blocking `main()` symbol conflict. The series now avoids `$(GITLIBS)` (which includes `common-main.o`) and instead depends on `$(LIB_FILE)` and `$(RUST_LIB)`, fixing the duplicate symbol issue. A new patch adds CI verification for `git-credential-osxkeychain`. The series is **unblocked and ready for integration**. **Key files**: `Makefile`, `contrib/Makefile`.

---

### In brief

**`git blame -b` hash abbreviation** -- René Scharfe’s patch to align `git blame -b`’s hash length with `core.abbrev` is ready for integration. The fix removes an extra hex digit reserved for an unused caret marker, resolving a usability friction in manual workflows. **Files**: `blame.c`, `t8002-blame.sh`.

**`greplint.pl` test linter (v4)** -- Michael Montalbo’s **6-patch series introducing `greplint.pl`** (a linter to convert bare `grep` to `test_grep`) is approved for merging. The series fixes 10+ pre-existing bugs, extracts a shared shell parser, and adds comprehensive test coverage. **Files**: `greplint.pl`, `t/greplint/`, `t/README`.

**`USE_NSEC` build option** -- D. Ben Knoble’s patch to add a `nanosec` Meson option for `USE_NSEC` sparked a **design discussion** about the knob’s viability. Jeff King’s testing showed modern Linux filesystems preserve nanosecond timestamps, but interoperability risks (e.g., JGit) remain. Patrick Steinhardt proposed making the feature runtime-configurable (e.g., `core.useNsec`). **Files**: `meson_options.txt`, `meson.build`.

**`git subtree` safeguard** -- Ian Jackson’s **2-patch series** adds a safeguard to the shell-script `git subtree` to detect and reject repositories modified by a forthcoming Rust rewrite. The patch prevents mixed-tool corruption by checking for a Rust-specific config file (`.git-subtree/config`). **Files**: `contrib/subtree/git-subtree.sh`, `t7900-subtree.sh`.

**Sparse-index segfault** -- Derrick Stolee’s **patch** fixes a segfault in the sparse-index subsystem when collapsing a full index with an intent-to-add entry outside the cone. The fix skips invalidated cache-tree nodes, preventing an out-of-bounds access. **Files**: `sparse-index.c`, `t3705-add-sparse-checkout.sh`.

**Memory leaks in hash implementations** -- Jeff King’s **9-patch series** plugs leaks in non-default hash backends (OpenSSL, libgcrypt). The series introduces `git_hash_discard()` and systematically applies it to leaky subsystems. **Design discussion** about idempotency (e.g., `git_hash_release()`) is ongoing. **Files**: `hash.c`, `csum-file.c`, `patch-id.c`.

**`GIT_TEST_LONG` tests** -- Patrick Steinhardt’s **9-patch v3 series** makes `GIT_TEST_LONG` tests reliable and efficient for CI. The series skips broken tests on 32-bit platforms, replaces slow `dd` pipelines, and enables `GIT_TEST_LONG` in GitLab CI. **Files**: `t0021-conversion.sh`, `t4141-apply-too-large.sh`, `.gitlab-ci.yml`.

---

### On the radar
- **`includeIf.worktree` symlink handling**: Chen Linxuan proposed a technical solution to align `worktree` and `gitdir` behavior by storing a non-realpath worktree path in `struct repository`. The discussion is ongoing.
- **`git rebase -i -x`**: Trevor Gross’s patch series faces **high-weight objections** from Junio Hamano, Jeff King, and Phillip Wood, who argue the feature misaligns with `rebase`’s core semantics. A pivot to `cherry-pick -i` is likely.
- **`git history squash` default behavior**: Whether `--reedit-message` should be the default remains unresolved, with Phillip Wood advocating for it as a commit hygiene measure.

---

### Editorial note
The day’s traffic reflected a **maturing phase** in several long-running efforts (e.g., `git log --graph`, `git refs`, reftable backend). The **collaborative tone** was notable, with reviewers and authors converging on design resolutions (e.g., rebase-dropped commits, `USE_NSEC`). **Performance work** (reftable backend, `GIT_TEST_LONG`) and **usability improvements** (`git blame -b`, `git subtree` safeguard) demonstrated the project’s balance between correctness, efficiency, and user experience. The **Rustification effort** continues to advance, with build system patches now unblocked.