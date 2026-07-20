# The Git Project Mailing List Digest - 2026/07/19

## The day in brief

A Sunday with moderate traffic (28 emails across 12 threads) saw **blocking correctness issues surface in two major series**, while a long-running branch cleanup feature inched closer to resolution. The most urgent item: **Junio C Hamano identified an order-dependency bug in Harald Nordgren’s `git branch --delete-merged` series** that threatens its core safety guarantees. Meanwhile, **Phillip Wood’s rebase-dropped-commits fix received final maintainer approval** and is now queued for `next`. Documentation and completion script improvements rounded out the day’s work.

---

## Notable threads

### `git branch --delete-merged` blocked by order-dependency bug

The **v19 iteration of Harald Nordgren’s stacked-branch protection**—a safety mechanism for the new `--delete-merged` command—hit a **blocking correctness issue** when Junio C Hamano demonstrated that the algorithm’s behavior depends on the **alphabetical order** of branch names rather than their logical dependency chain. In two equivalent dependency graphs (`a_tip→b_mid→c_lower` vs. `tip→mid→lower`), the outcome (which branches are spared or have their upstream cleared) differed solely due to naming, violating the principle that identical relationships should produce identical results.

Harald proposed a **minimal fix** (checking both `deletable` and `spared` strsets in `spare_stacked_base()`), but Junio remains skeptical, speculating that a **multi-pass algorithm** may be needed to fully eliminate order dependency. Phillip Wood then proposed a **structural refactor**: collect all spared branches in a temporary `strset` during traversal, then remove them from `deletable` in a single batch operation afterward. This would decouple spared-branch identification from deletion-set mutation, eliminating the sequential, order-dependent behavior.

The series remains **blocked pending Junio’s satisfaction with the correctness fix**, though Phillip’s proposal offers a concrete path forward. The earlier agreement to extend this protection to `git branch -d` as a follow-up is unaffected.

---

### Rebase dropped-commits fix approved for `next`

Phillip Wood’s **9-patch v3 series fixing the sequencer’s handling of dropped commits during rebase** received **final maintainer approval** from Junio C Hamano. The series, which prevents notes from being incorrectly copied to HEAD when commits are dropped (e.g., due to empty changes or fixup sequences), is now **queued for `next`** pending the squashing of minor typofixes.

A **known regression**—the fix does not cover user-initiated drops in interactive rebase (e.g., `git rebase --skip` or `--continue` without committing)—was acknowledged but **not a blocker for integration**. Phillip is designing a follow-up to address this gap, which stems from the interactive backend’s `--empty=ask` behavior (drops on `--continue`, keeps on `--skip`). The series also includes fixes for **external merge strategy failures** (e.g., merge tool crashes) and **command execution failures** (e.g., `git commit` failing to run), along with a systemic refactoring of the sequencer’s control flow using `enum pick_result`.

---

### `paint_down_to_common()` optimization queued, awaiting review

Tian Yuchen and Kristofer Karlsson’s **10-patch v6 series optimizing `paint_down_to_common()`**—which yields **100-1000x speedups** for asymmetric merge-base queries—is now **procedurally unblocked and queued in Junio’s tree**. The series terminates early when one side of a query exhausts its commit queue, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts or shallow histories).

Junio confirmed that Kristofer’s synthetic base approach (merging only `kk/commit-reach-find-all-fix` into `master`) is correct, but the series remains in **holding pattern awaiting additional reviewers**. No code changes are needed; the bottleneck is now **review bandwidth**. The optimization touches `commit-reach.c`, `commit-graph.c`, and test scripts, and includes Trace2 instrumentation to measure step counts.

---

### Loose object backend refactoring unblocked

Patrick Steinhardt’s **9-patch v1 series refactoring the loose object backend**—moving loose-object writing logic into `odb/source-loose.c` as part of the ODB abstraction effort—was **unblocked after a build-breaking merge resolution error** was fixed. The series now compiles, clearing the way for substantive review.

The refactoring centralizes object hash computation, compatibility ID handling, object-existence checks, and mtime support in the generic ODB layer, eliminating duplication across backends. A memory leak in `force_object_loose()` was also fixed. The series depends on `jt/receive-pack-use-odb-transaction` and `jk/git-hash-cleanups`, and is now **ready for technical feedback**.

---

## In brief

**`git send-email` completion** -- Yury Norov (NVIDIA) extended bash completion to include patch files alongside revision identifiers. Junio C Hamano praised the fix for addressing a long-standing annoyance but requested a **test robustness improvement** (moving `test_when_finished` earlier in the setup).

**Rust hash leak fixes** -- brian m. carlson fixed memory leaks and uninitialized memory risks in the Rust bindings for Git’s hash functions. Jeff King (Peff) reviewed the patches and **cleared them for merging**, noting they address pre-existing issues in the Rust/C boundary layer.

**Branch deletion error messages** -- René Scharfe’s patch to clarify `git branch -d` error messages when deleting the current branch sparked a **usability discussion**. Phillip Wood raised concerns about confusion when `git status` shows a different branch than the error message, proposing either enhancing `git status` to show all branches being rewritten or crafting a more detailed error message for the `UPDATE_REF` case (e.g., "because it is being updated by a rebase running in '../feature'"). The `int` vs. `enum` debate for the checkout kind variable was resolved in favor of `enum` for debuggability.

**Documentation synopsis conversion** -- Jean-Noël Avila posted a **4-patch v1 series** converting `git imap-send`, `git format-patch`, `git send-email`, and `git request-pull` to the new AsciiDoc synopsis style. Junio C Hamano provided **surface-level feedback** on backtick-quoting consistency and AsciiDoc code block delimiters, but no substantive concerns were raised.

**`the_repository` removal** -- René Scharfe’s **5-patch series introducing repository-aware tempfile/lockfile APIs** remains queued in `seen`. Junio Hamano weighed in on the unresolved compile-time guard question, stating he is "indifferent" about the immediate decision (whether to keep or drop patch 5/5) so long as the project agrees on the longer-term direction. The guard remains a **medium-term requirement**, but its implementation is decoupled from this series.

---

## On the radar

**Linked files proposal** -- sporteka proposed a "linked files" mechanism to allow multiple repository paths to mirror the same source file byte-for-byte, addressing the pain point of maintaining identical copies across locations. The idea is novel for Git and could spark debate about whether such guarantees belong in core tooling or remain workflow-specific. No implementation or patch series exists yet.

**`git last-modified` Bloom filter optimization** -- Taylor Blau’s `tc/last-modified-bloom` series received feedback from Jeff King about Bloom filter implementation details and is **awaiting a reroll**. The series aims to optimize `git last-modified` using Bloom filters but is not yet in `seen`.