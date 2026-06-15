# Git Mailing List Digest - 2026/06/14 (Sunday)

**The day in brief.** A moderately busy Sunday with 34 emails across 13 threads, featuring the conclusion of Elijah Newren's merge-ort hardening series, ongoing discussion about `git replay --linearize`, and a new RFC for external diff.<driver>.process support. Performance optimizations and documentation fixes rounded out the day's traffic.

## Notable threads

### Merge-ort hardening series concludes

Elijah Newren's five-patch series to harden merge-ort against corrupt trees and improve directory/file conflict detection has completed its review cycle with maintainer approval. The final patch implements binary search in verify_cache() to catch non-adjacent D/F conflicts (like "path" vs "path-internal/file"), maintaining O(n log n) performance while expanding detection coverage. Junio Hamano's thorough review confirmed the correctness of edge case handling and array bounds checks. This concludes a series that began with error propagation fixes and now provides robust protection against tree corruption scenarios in merge operations.

### Replay linearization design discussion continues

Toon Claes' `git replay --linearize` series received substantive review from Elijah Newren, who noted several documentation and test coverage gaps while approving the technical approach. Key suggestions include adding `--linearize` to SYNOPSIS/usage strings, clarifying "drop" vs "ignore" terminology, documenting option incompatibilities, and expanding test cases to verify patch preservation. The review maintains focus on improving maintainability rather than challenging core functionality, with Elijah's merge expertise lending weight to his test case recommendations. The series now awaits responses to these points before finalization.

### External diff processor RFC v4

Michael Montalbo posted the fourth revision of an RFC introducing `diff.<driver>.process`, enabling external tools to inject diff hunks into Git's pipelines while preserving features like word-diff. Notable changes include replacing Python tests with a C-based helper (per Johannes Schindelin's recommendation) and enhanced hunk validation warnings. The 6-patch series now handles empty files consistently and includes 708 lines of test coverage. Blame integration demonstrates skipping reformat-only commits, though per-commit roundtrips currently add ~17% overhead. With Windows compatibility resolved and comprehensive testing, the series appears technically complete pending final maintainer sign-off.

## In brief

**Commit-graph future date fix** -- Elijah Newren corrected generation number calculations for commits with parent timestamps beyond 2106-02-07 by using 64-bit arithmetic, preventing truncation in repositories with extremely future-dated commits.

**Memory leak in git history reword** -- A one-line fix addresses ASAN-reported leak in `git history reword` by properly releasing commit message buffers retrieved via `repo_logmsg_reencode()`.

**cat-file batch performance** -- René Scharfe optimized `git cat-file --batch-check` formatting by replacing `strbuf_addf()` with direct methods, yielding 3-7% speedups in bulk object queries.

**Silent build improvements** -- A 2-patch series makes translation catalog generation respect `make -s` in gitk and git-gui builds, suppressing "Generating catalog" messages during silent operation.

**Historical release note typo** -- D. Ben Knoble proposed fixing a 2017 typo ("along"->"alone") in 2.14.0 release notes, though Junio questioned the value of polishing old documentation absent factual errors.

## On the radar

**Paint-down optimization coordination** -- Kristofer Karlsson and Elijah Newren are aligning implementations of a commit reachability optimization that shows 300-1000x speedups in some topologies, with GitHub already running a limited deployment.

**Clone depth environment variable debate** -- Discussion continues on Hadrien Loge's proposal for `GIT_CLONE_DEPTH`, with brian m. carlson raising concerns about unintended shallow clones and server load versus packaging workflow benefits.