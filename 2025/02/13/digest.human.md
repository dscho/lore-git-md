# Git Mailing List Digest — 2025/02/13

**The day in brief.** A busy Thursday with 58 emails across 19 threads, dominated by performance investigations in the reftable backend and merge-recursive optimizations. Key developments include a confirmed 30x slowdown in reftable mass ref operations, approval of one merge-recursive optimization while another was rejected, and ongoing discussions about protocol design and rebase behavior. Windows compatibility and documentation standardization also saw activity.

## Notable threads

### Reftable backend shows 30x slowdown in mass ref operations

Brian M. Carlson reported a severe performance regression where creating 50,000 refs took nearly a minute with reftable compared to 5.5 seconds with the traditional files backend. Patrick Steinhardt confirmed the issue stems from tombstone records preventing auto-compaction while still performing expensive conflict checks. The discussion identified multiple optimization opportunities:

1. Skipping unnecessary `repo_get_oid()` calls in `git-update-ref` (already yielding a 3x improvement)
2. Batching ref verification to avoid repeated scans (complicated by iterator limitations)
3. Suppressing ambiguous refname warnings that add overhead
4. Potential long-term changes to compaction heuristics

Jeff King and Junio Hamano both questioned whether the ambiguous ref warnings provide enough value to justify their cost, suggesting they might be better handled at ref creation time. The thread revealed fundamental challenges in the iterator infrastructure that Steinhardt is working to address, making this a multi-faceted performance investigation likely to continue.

### Merge-recursive optimizations: one approved, one rejected

Meet Soni's series optimizing string list processing in merge-recursive saw mixed results. The first patch, converting `process_renames()` from O(n²) to O(n log n) operations, received final approval from Elijah Newren after incorporating review feedback. However, the second patch attempting similar optimizations for `get_unmerged()` faced strong resistance.

Newren questioned both the practical impact (given typically small conflict counts) and strategic value of optimizing merge-recursive, which is slated for eventual replacement by merge-ort. Junio Hamano and Newren discussed theoretical O(n) approaches but agreed the effort wasn't justified for legacy code. This leaves the series with one accepted optimization while highlighting the project's shifting focus toward merge-ort.

### Protocol limitations for bundle-uri with filtered clones

Brian Carlson confirmed Brian Celenza's observation that the Git protocol's command sequence prevents servers from knowing client filter preferences before selecting bundles. The current bundle-uri command lacks parameters to convey this information, forcing servers to make suboptimal bundle choices for filtered clones. While the discussion remains at the design stage, it revealed a protocol limitation that could affect large repository workflows, with potential solutions involving capability extensions or command parameterization.

### Rebase's interactive update-refs reaches design consensus

Phillip Wood and Ivan Shapovalov finalized the design for `git rebase --update-refs=interactive`, settling on a configuration-driven approach (`rebase.updateRefs=interactive`) while preserving the simple CLI interface. The solution supports complex branch dependency graphs while leaving room for future pattern-based filtering capabilities. The thread shows all major design questions resolved, with only implementation and documentation remaining before integration.

## In brief

**Windows compatibility** Johannes Schindelin reported build failures in Git-for-Windows CI from Patrick Steinhardt's reftable decoupling series, particularly objecting to duplicate header filenames in mingw/ and msvc/ directories. **Tag fetching regression** Taylor Blau confirmed a protocol v2 bug where commit 3f763ddf28") broke tag decoration during shallow fetches, proposing to always include "refs/tags/" in requests when tag following is enabled. **Platform warnings** Zejun Zhao's series fixing `-Wsign-comparison` in `apply.c` received detailed review from Patrick Steinhardt and Karthik Nayak, with Junio Hamano weighing in on signed vs unsigned tradeoffs. **Mailmap crash** Jacob Keller reported a segfault in `git check-mailmap` when processing full mailmap entries with email-only lookups, prompting discussion about expected behavior versus implementation bugs. **Build fixes** Two separate threads addressed Makefile syntax (tab/space issues with zlib configuration) and Meson support for credential helpers (wincred and libsecret).

## On the radar

The **`diff_filepair` callback compatibility** discussion between Patrick Steinhardt and Justin Tobler confirmed the need to preserve existing callback signatures while introducing new functions, with documentation improvements planned for v3. The **file collision investigation** connected to renormalization issues expanded to include case-sensitive filesystem behavior, with Torsten Bögershausen suggesting `git ls-files` analysis to understand the anomalies. The **pre-push hook efficiency** proposal from Jayce Cao about identifying new commits during push operations drew responses highlighting security limitations and existing `--not --remotes` optimizations, leaving the core challenge unresolved.