# The Git Project Daily Digest - 2026/07/10

## The day in brief

Friday, July 10, 2026 brought **151 emails across 31 active threads**, making it a **heavy but productive day** for Git development. The standout themes were **ODB abstraction progress**, **reftable backend hardening**, and **subsystem refactoring**—with several long-running series reaching key milestones. The most consequential news: **Patrick Steinhardt’s ODB transaction series for `git receive-pack` (11 patches) is now queued for `next`**, marking a major step toward pluggable object databases. **Kristofer Karlsson’s reftable tombstone performance fix (2 patches) also cleared final review**, resolving a real-world regression observed in production repositories. On the feature front, **Harald Nordgren’s `git branch --delete-merged` series (v18) is implementation-complete and ready for merging**, offering safe automated branch cleanup with stacked-branch protection and per-branch opt-out. **Two security-related discussions**—one about SSH signature inspection, the other about lazy-fetching controls—highlighted ongoing tensions between usability and threat modeling.

---

## Notable threads

### ODB abstraction: `git receive-pack` now uses ODB transactions (11 patches)
**Topic:** `jt/receive-pack-use-odb-transactions`
**Author:** Justin Tobler
**Status:** **Queued for `next`** (all 11 patches merged; Junio’s final review resolved)

The series replaces `tmp_objdir` usage in `git receive-pack` with the `odb_transaction` subsystem, advancing the ODB abstraction effort. The final patch (11/11) converts the quarantine flow to use transactions, preserving existing behavior while enabling future backend-agnostic storage. Junio’s last review raised a design question about explicit `odb_transaction_abort()` calls in error paths, but Justin deferred this to follow-up work, noting the series is otherwise ready. The change is purely internal, with no user-visible impact, and carries Patrick Steinhardt’s Reviewed-by on all patches.

**Why it matters:** This is a foundational step toward pluggable object databases, enabling Git to support alternative storage backends (e.g., cloud-native, embedded) without rewriting core commands.

---

### Reftable backend: tombstone performance fix (2 patches)
**Topic:** `kk/reftable-tombstone-quadratic-fix`
**Author:** Kristofer Karlsson
**Status:** **Final review complete; ready for `next`**

The series fixes a quadratic performance regression in the reftable backend when re-creating refs after deletion (tombstoning). The issue occurs in ref-reading operations (e.g., `refs_verify_refnames_available()`) where the merged iterator’s `suppress_deletions` flag forces a full scan of all tombstone records before bounds checks can terminate. The fix moves tombstone suppression logic to call sites, enabling early termination. Performance improves from ~14s to ~0.2s in the perf test’s 8000-ref scenarios. The v3 iteration addressed Patrick Steinhardt’s feedback by making `suppress_deletions` configurable via `struct reftable_stack_options`, preserving libgit2’s current behavior while future-proofing the API.

**Why it matters:** This resolves a real-world performance issue observed by brian m. carlson in production-scale repositories, ensuring the reftable backend scales efficiently even with heavy tombstone churn.

---

### `git branch --delete-merged` (v18, 7 patches)
**Topic:** Harald Nordgren
**Status:** **Implementation-complete; ready for merging**

The series introduces a safe, automated local-branch cleanup command with **refined stacked-branch protection (abort-and-clear)**, per-branch opt-out via `branch.<name>.deleteMerged=false`, `--dry-run` preview, and the `--forked` filter. The v18 iteration addresses all prior feedback, including Phillip Wood’s test improvements and a mechanical flags-handling issue in patches 2–4. The stacked-branch protection now keeps only branches needed by unmerged branches and clears stale upstream configs for kept (merged) branches, avoiding broken dependency chains.

**Why it matters:** This addresses a long-standing workflow pain point—safe automated branch cleanup—with careful attention to edge cases (e.g., chains of dependencies, self-referential upstreams). The per-branch opt-out mechanism is particularly useful for iterative topic-branch workflows.

---

### `git replay --linearize` (v7, 3 patches)
**Topic:** `tc/replay-linearize`
**Status:** **Queued in `next`; one unresolved design question**

The series adds a `--linearize` option to `git replay` to flatten merge commits, producing a linear history. The v7 iteration fixed a regression in single-branch replay and removed mutual exclusivity between `--linearize` and `--revert`. However, **Elijah Newren identified a principle-of-least-astonishment violation**: when multiple positive refs are provided (e.g., `git replay --linearize --onto main topic1 topic2`), the current implementation flattens all commits into a single linear history, concatenating branches in an opaque order. Elijah proposes either disallowing multiple positive refs with `--linearize` (aligning with `--advance`/`--revert`) or tracking a `last_commit` per branch to preserve branch independence.

**Why it matters:** The feature offers a simpler alternative to Johannes Schindelin’s earlier merge-replay implementation, but the multi-branch ambiguity risks surprising users. The ball is in Toon Claes’s court to address Elijah’s feedback.

---

### `git history squash` (v8, 5 patches)
**Topic:** Harald Nordgren
**Status:** **Functionally complete; ready for integration**

The series introduces a `git history squash` subcommand to fold a commit range into its oldest commit while preserving descendant history. The v8 iteration aligns the `--reedit-message` template with `git rebase -i --autosquash`, addresses all prior feedback, and includes comprehensive test coverage. The only remaining discussion point is whether `--reedit-message` (or `--edit`) should be the default, but this is a minor usability tweak that does not block merging.

**Why it matters:** This provides a conflict-free alternative to `git rebase -i` for collapsing ranges, with robust safety checks (e.g., rejecting ranges with multiple bases, preserving refs pointing to interior commits).

---

### Security discussions

#### SSH signature inspection without `allowedSignersFile`
**Topic:** Grayson Tinker
**Status:** **Design debate unresolved**

The patch relaxes SSH signature verification to show basic signature details (key type, fingerprint, “Good”/“Bad” status) even when `gpg.ssh.allowedSignersFile` is unset. Junio raised a usability concern: the unconditional advice message (“Configure gpg.ssh.allowedSignersFile...”) creates noise in bulk operations (e.g., `git log --show-signature -100`). Grayson acknowledged the trade-off but argued the current one-line message is preferable to the previous error. The discussion remains open on whether to make the advice suppressible or rate-limited.

**Why it matters:** This highlights a tension between usability (inspecting signatures without configuration) and noise (repetitive warnings). The outcome may set a precedent for how Git handles optional-but-recommended configuration.

#### Lazy-fetching controls (`GIT_NO_LAZY_FETCH=fromAccepted`)
**Topic:** Christian Couder
**Status:** **Blocked by security objection**

The series introduces a `fromAccepted` mode for `GIT_NO_LAZY_FETCH`, allowing lazy fetching only from promisor remotes negotiated via the "promisor-remote" capability. **brian m. carlson raised a categorical objection**, arguing that no additional lazy-fetching flexibility should be introduced while Git’s network code remains in C, as this expands the attack surface for untrusted repositories. The objection is principled and may block the series unless the author or other participants can address the security implications.

**Why it matters:** This reflects broader concerns about Git’s threat model and the risks of exposing more network-facing C code to untrusted input. The discussion may influence future decisions about lazy-fetching and partial-clone security.

---

## In brief

### ODB abstraction
- **`ps/odb-for-each-object-filter` (8 patches):** Taylor Blau identified a performance inefficiency in the packed ODB backend’s object lookup logic (unnecessary `match_hash()` calls for MIDX objects). The series is otherwise ready for `next`.
- **`ps/odb-pluggable-housekeeping` (11 patches):** Refactored `git gc`/`maintenance` ODB housekeeping to be backend-agnostic. Cooking in `next`.

### Reftable
- **`ps/reftable-hardening` (12 patches):** Hardened reftable parsing against corrupted tables (OOB reads/writes, NULL derefs). Added fuzzer and Meson build support. Cooking in `next`.

### Performance
- **`hf/unpack-trees-quadratic-scan` (1 commit):** Fixed quadratic behavior in `next_cache_entry()` when diffing with pathspecs matching early index entries. Cooking in `next`.
- **`kk/prio-queue-cascade-sift` (3 commits):** Optimized `prio_queue_get()` with cascade-down sifting, halving comparisons per extract-min operation. **Withdrawn due to hardware-specific regression on Apple silicon.**

### New features
- **`ps/history-drop` (11 patches):** Taught `git history drop` to remove a commit and replay descendants onto its parent. Cooking in `next`.
- **`ps/cat-file-remote-object-info` (13 patches):** Added `remote-object-info` to `git cat-file --batch-command` for fetching object metadata (e.g., size) from remotes via protocol v2. **Ready for merging; all issues resolved.**

### Bugfixes
- **`js/coverity-fixes-null-safety` (12 patches):** Hardened code against NULL derefs (e.g., `shallow.c`, `bisect.c`, `pack-bitmap.c`). **Merged to `next`.**
- **`kk/commit-graph-topo-levels-fix` (2 patches):** Fixed incremental commit-graph writes by propagating `topo_levels` slab to all chain layers. **Merged to `next`.**
- **`mm/lib-httpd-cgi-safe` (3 patches):** Made test CGI helpers atomic. **Needs reroll for race condition fixes.**

### Documentation
- **`kh/doc-trailers` (10 patches):** Documentation overhaul for `git interpret-trailers`. **Stalled due to terminology bikeshedding.**

### Platform compatibility
- **Cygwin IPv6 SCP URL fix:** Ramsay Jones fixed a five-year-old regression where IPv6-style SCP URLs (e.g., `[::1]:repo`) were misinterpreted as invalid paths. **Ready for review.**

---

## On the radar

- **`ps/setup-split-discovery-and-setup` (16 patches):** Splits repository discovery and configuration phases to enable clean repository unification. **RFC; long-term effort.**
- **`sn/osxkeychain-rust-universal` (3 patches):** Enables universal macOS builds for Rust. **Blocked on Meson/Rust integration concerns.**
- **`tb/repack-geometric-cruft` (11 patches):** Combines `--geometric` and `--cruft` repacking. **Needs reroll for MIDX retention logic.**

---

## Editorial note

Today’s traffic underscored the **maturity of Git’s ODB abstraction effort**, with two major series (`jt/receive-pack-use-odb-transactions` and `ps/odb-for-each-object-filter`) clearing final review. The **reftable backend’s performance regression fix** is particularly noteworthy, as it resolves a real-world issue observed in production-scale repositories. Meanwhile, the **security discussions**—especially brian m. carlson’s objection to lazy-fetching controls—highlight the project’s ongoing struggle to balance usability with threat modeling in a codebase still largely written in C.

The **`git branch --delete-merged` series** stands out as a well-crafted feature that addresses a common workflow pain point with careful attention to edge cases. Its stacked-branch protection and per-branch opt-out mechanisms demonstrate how Git’s design continues to evolve to support complex, real-world workflows.