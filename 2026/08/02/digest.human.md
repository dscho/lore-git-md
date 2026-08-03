## The day in brief

Sunday, August 2, 2026 brought a moderate volume of traffic (40 emails in 13 threads) with a mix of routine progress and lingering design debates. **The two threads that stood out**: Harald Nordgren’s `--reset-when-found` feature for `git bisect` reached a polished v6 after Junio’s final clean-ups, and a promisor-pack performance patch from Arijit Banerjee became the focal point of a policy debate about AI-assisted contributions. Elsewhere, Kristoffer Haugsbakk’s trailer-parsing fix and Siddharth Shrimali’s `--drop-filtered` RFC for partial clones saw incremental refinement, while brian m. carlson’s lowercase-hex proposal continued to spark philosophical discussion about the Robustness Principle.

---

## Notable threads

### Bisect’s `--reset-when-found` lands in polished form
Harald Nordgren posted **v6** of the `--reset-when-found[=<where>]` option for `git bisect`, incorporating Junio’s final internal clean-ups: reading `refs/bisect/bad` directly before reset (eliminating the static `first_bad_oid`) and using a sentinel value (`RESET_WHEN_FOUND_NONE`) to remove the separate boolean flag. The series is now a clean two-patch set—plumbing refactor plus feature implementation—with thorough test coverage and no open technical concerns. Junio’s post-acceptance feedback was advisory, not blocking, and the series looks ready for `next`. The feature remains a monolithic operation (staging only when *all* conflicts are resolved), but Michael Montalbo’s incremental-staging proposal (`--skip-conflict-markers`) has not gained traction.

---

### Promisor-pack performance patch hits policy snag
Arijit Banerjee’s patch to speed up `git index-pack` for promisor packs by 15–26% (avoiding a per-object mutex via thread-local oidmaps) became the day’s most contentious thread—not for technical reasons, but because the commit message disclosed AI assistance. **Brian m. carlson** objected on policy grounds, citing `SubmittingPatches`’ requirement that authors certify full understanding of non-trivial code. Junio clarified that Git’s stance is not an outright ban but a cautious, case-by-case approach, and suggested LLVM’s rejection of "extractive contributions" as a potential model. The thread remains stalled on whether the performance benefit justifies an exception, with no maintainer yet weighing in on the trade-off. Arijit’s follow-up brainstormed procedural ideas (karma systems, experimental release trains) but did not advance the core question.

---

### Trailer-parsing fix nears completion
Kristoffer Haugsbakk’s patch to stop Git from misinterpreting URLs as trailers (e.g., `https://example.com` → `https: //example.com`) received Junio’s blessing after a narrow review focused on the `://` heuristic’s robustness and pointer-arithmetic safety. The implementation—modifying `find_separator()` in `trailer.c` to reject lines containing `://`—is minimal, well-tested, and uncontroversial. With no open objections, this looks likely to graduate to `next` soon.

---

### `--drop-filtered` for partial clones refines safety guards
Siddharth Shrimali’s RFC series adding `--drop-filtered` to `git repack` (to cull oversized promisor blobs) saw a design pivot: Junio proposed replacing the current merge/rebase/cherry-pick checks with a **time-based heuristic** (avoid culling objects fetched within the last 20 minutes). Siddharth endorsed the idea, noting that promisor objects are always stored in packs (never loose), so per-object timestamps would require extending the promisor object log or adopting a cruft-pack-style `.mtimes` file. The discussion now centers on whether to implement this heuristic in the next revision or defer it to a follow-up. The core functionality (enumeration, repacking, implied `-d`) remains unchanged.

---

### Lowercase-hex proposal sharpens philosophical divide
Brian m. carlson’s RFC to restrict hex object IDs to lowercase in Git 3.0 continued to spark debate. **Jeff King** flagged an edge case: uppercase hex in object payloads (e.g., commits) creates parallel histories, as Git does not normalize case when hashing contents. Brian acknowledged this as a real issue complicating hash-algorithm compatibility (such objects cannot be round-tripped through Git’s interoperability code). Meanwhile, **Junio** invoked the Robustness Principle to argue that Git should remain liberal in what it accepts, even in a breaking-change release. Brian countered by citing modern security practices (TLS, HTTP request smuggling) to argue that lenient parsing is now a security anti-pattern. The thread remains focused on the philosophical divide, with no maintainer decision yet.

---

## In brief

**HTTP daemon test-helper race fix** -- Michael Montalbo pinged his v2 series fixing race conditions in `t/lib-httpd/` CGI scripts (atomic `mv`/`mkdir` for shared state). The fixes address real CI flakes and are ready for integration.

**Worktree add bugfix** -- Matthias Aßhauer’s series fixing an out-of-bounds read in `git worktree add` saw progress: René Scharfe clarified that the validation patch (rejecting empty strings with a reference) is necessary for the `"" HEAD` edge case, though its placement may need adjustment. The pointer-underflow fix in `worktree_basename()` remains the more urgent change.

**`cat-file --batch-command` type support** -- Pablo Sabater’s GSoC series adding `%(objecttype)` to remote-object-info queries converged on a hybrid design: a single array of lightweight structs (caller-controlled field population) to bypass `struct object_info`. Junio and Peff confirmed this avoids sentinel-value awkwardness, and the refactoring will land in the next revision.

**`git add --resolved` design debate** -- Michael Montalbo refined his critique of Junio’s `--resolved` option, clarifying that his concern was not the name but the *monolithic* nature of the operation. He proposed `--skip-conflict-markers` to allow incremental staging of resolved files, but Junio remains unconvinced. The thread now centers on whether the command should enforce all-or-nothing resolution or support partial staging.

**`git interactive` feature request** -- A new `git interactive` command (terminal menu for common operations) drew skepticism from Michael Montalbo, who questioned its necessity given existing tools (`git-gui`, `gitk`). The author has not yet addressed the overlap or provided a stronger rationale.

**Monorepo remote helper** -- Zhonghua Zhu proposed `blackgit`, a proof-of-concept remote helper to optimize monorepo workflows (Perforce-style narrow clone). The idea is exploratory, with no patches or benchmarks yet.

**Sparse-checkout performance fix** -- Sahitya Chandra’s patch to avoid index expansion in `repo_index_has_changes()` for unborn HEAD (using the empty tree) is a targeted win for sparse-checkout/sparse-index. The change is self-contained and well-tested.

---

## On the radar

- **`git add --resolved`**: The incremental-staging debate (`--skip-conflict-markers`) remains unresolved, with Junio and Michael Montalbo at an impasse.
- **Lowercase-hex RFC**: Brian and Junio’s philosophical divide (security strictness vs. Robustness Principle) will likely require PLC input.
- **Promisor-pack policy**: Arijit’s performance patch awaits maintainer guidance on whether the 15–26% speedup justifies an exception to the AI-assisted contribution policy.