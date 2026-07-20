## The day in brief

Sunday, July 19, 2026 brought a light but focused day on the Git mailing list—28 emails across 12 threads, with no single controversy dominating. The most consequential development: **Junio C Hamano approved Phillip Wood’s 9-patch rebase-dropped-commit fix for `next`**, resolving a long-standing note-corruption bug. Meanwhile, **Harald Nordgren’s `git branch --delete-merged` series remains blocked** by a correctness issue in its stacked-branch protection logic, though Phillip Wood’s proposed refactor offers a path forward. Documentation and completion-script improvements rounded out the day’s work.

---

## Notable threads

### `git branch --delete-merged` blocked by order-dependency bug

Harald Nordgren’s 7-patch series adding `--delete-merged` to `git branch` hit a **blocking correctness issue** in its stacked-branch protection logic. Junio C Hamano identified that the algorithm’s behavior depends on the **alphabetical order** of branch names rather than their logical dependency chain, violating the principle that identical branch relationships should produce identical results. Harald proposed a minimal fix (checking both `deletable` and `spared` strsets in `spare_stacked_base()`), but Junio remains skeptical it covers all edge cases. Phillip Wood then proposed a **structural refactor**: collect all spared branches in a temporary `strset` first, then remove them from `data->deletable` in a single batch operation after traversal completes. This would eliminate the order dependency by decoupling spared-branch identification from deletion-set mutation. The series remains blocked pending Junio’s approval of the fix, but Phillip’s proposal offers a concrete path forward.

---

### Rebase-dropped-commit fix approved for `next`

Phillip Wood’s 9-patch series fixing the sequencer’s handling of dropped commits during rebase received **final approval from Junio C Hamano** and is now queued for the `next` branch. The series addresses a long-standing bug where notes from dropped commits (e.g., due to empty changes or fixup sequences) were incorrectly copied to the current HEAD commit, corrupting the repository’s note history. The fix is **technically complete**, with Uwe Kleine-König’s Tested-by and no outstanding substantive concerns. A **known regression**—the fix does not handle user-initiated drops in interactive rebase (e.g., `git rebase --skip`)—will be addressed in a follow-up. The series is a well-crafted, tested-level bugfix that resolves a persistent pain point in the rebase machinery.

---

### `paint_down_to_common()` optimization queued, awaiting review

Tian Yuchen and Kristofer Karlsson’s 10-patch series optimizing `paint_down_to_common()` for one-sided histories is **queued in Junio’s tree** and procedurally unblocked, but Junio explicitly requests **additional reviewers** before advancing it further. The series achieves **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts or shallow histories) by terminating early when one side’s commit queue is exhausted. Derrick Stolee and SZEDER Gábor have already approved the changes, which include Trace2 instrumentation, clock-skew test coverage, and the removal of a commit-date fallback. The only remaining action is review; no code changes or resubmissions are needed. This is a high-impact performance optimization with no behavior changes, making it a strong candidate for `next` once review bandwidth is secured.

---

### ODB loose-object refactoring unblocked

Patrick Steinhardt’s 9-patch series refactoring loose-object writing logic into `odb/source-loose.c` was **unblocked** after Junio C Hamano resolved a build-breaking merge conflict. The series moves all loose-object writing machinery to the loose ODB backend, making it fully self-contained as part of the ongoing ODB abstraction effort. SZEDER Gábor identified the issue (a missing evil merge resolution), and Junio applied the fix, allowing substantive review to proceed. The series is now ready for technical feedback, with no prior versions or outstanding concerns. This is a foundational refactoring for the ODB abstraction project, touching 12 files and introducing no behavior changes.

---

## In brief

**Rust hash leak fixes** -- brian m. carlson fixed memory leaks and uninitialized memory risks in Git’s Rust hash bindings, addressing issues at the Rust/C boundary when non-default block algorithms (e.g., OpenSSL) are used. Jeff King (Peff) reviewed and cleared the series for merging.

**Branch deletion error messages** -- René Scharfe’s patch to clarify `git branch -d` error messages when deleting the current branch sparked a usability discussion. Phillip Wood raised a concern about confusion when `git status` shows a different branch than the error message, proposing either enhancing `git status` to show all branches being rewritten or crafting a more detailed error message. The thread remains open, with no consensus on the `BRANCH_CHECKOUT_KIND_UPDATE_REF` message design.

**Completion script enhancement** -- Yury Norov added filesystem path completion to `git send-email`, allowing users to tab-complete patch filenames alongside commit hashes. Junio C Hamano provided surface-level feedback on test robustness and naming style, but no substantive concerns were raised.

**Documentation synopsis-style conversion** -- Jean-Noël Avila posted a 4-patch series converting the man pages for `git imap-send`, `git format-patch`, `git send-email`, and `git request-pull` to the project’s new AsciiDoc synopsis style. Junio noted minor inconsistencies in backtick-quoting and AsciiDoc code block delimiters, but the changes are purely presentational and part of the ongoing documentation effort.

**Linked files proposal** -- sporteka proposed a "linked files" mechanism for Git, allowing multiple repository paths to mirror the same source file byte-for-byte. The pitch argues this belongs in Git’s tooling to leverage its content-addressed storage, with a minimal first step being to teach `git archive` to preserve hardlinks between identical files. The proposal is early-stage and lacks implementation details, but it addresses a real pain point for projects maintaining shared files across locations.

---

## On the radar

**`the_repository` removal in lockfile/tempfile APIs** -- René Scharfe’s 5-patch series refactoring tempfile and lockfile APIs to avoid `the_repository` remains queued in `seen`. Junio Hamano weighed in on the unresolved compile-time guard question, stating he is "indifferent" about the immediate decision but expects the guard to be implemented in the medium term. The series is incremental and aligned with the broader `the_repository` removal effort.

**Interactive rebase regression in rebase-dropped-commit fix** -- Phillip Wood’s approved rebase-dropped-commit fix does not handle user-initiated drops in interactive rebase (e.g., `git rebase --skip`). Phillip is actively designing a follow-up to address this, which will likely involve revising the `--empty=ask` behavior for consistency. The regression is not a blocker for the current series but will need resolution before the feature is considered complete.