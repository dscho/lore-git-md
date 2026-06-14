Here's the daily digest for June 13, 2026:

## The day in brief

A moderately busy Saturday with 45 emails across 16 threads, featuring security updates, performance optimizations, and ongoing refactoring work. The most notable developments include a security release for tig, significant performance improvements in merge-base calculations, and continued progress on the `the_repository` removal effort. Junio Hamano weighed in on several threads, providing key maintainer guidance.

## Notable threads

**Tig 2.6.1 security release** -- Thomas Koutcher announced a security and maintenance release for the tig Git text-mode interface, addressing a command injection vulnerability (#1432) introduced in the major 2.6.0 release. The update includes 15 bugfixes and 4 improvements across 119 files, with fixes for notes view handling, grep behavior, Cygwin builds, and status operation performance. The release maintains cross-platform support while hardening security and functionality.

**Merge-base optimization proposal** -- Kristofer Karlsson provided concrete performance data supporting his optimization for `paint_down_to_common` in commit-reach.c. Measurements show 300-1000x speedups (4.85s -> 6ms) in monorepo scenarios where one branch's commits are exhausted early. The thread includes a detailed correctness proof and addresses edge cases like imported repositories with generation 0 roots. Derrick Stolee's review has helped refine the approach, which now appears technically sound pending octopus merge testing.

**`the_repository` removal progress** -- Tian Yuchen's refactoring to move `protect_hfs` and `protect_ntfs` into repository-specific storage was merged into `next` after thorough review. The v3 changes were purely administrative (commit subject tweaks), following substantive technical work in v2 that introduced helper functions while maintaining Windows filesystem validation behavior. This marks another successful step in eliminating global variables from Git's codebase.

**Shallow clone configuration debate** -- Two parallel threads (from Matt Hunter and Hadrien Loge) proposed using environment variables for shallow clone configuration, prompting a maintainer response. Junio Hamano objected to the environment variable approach, preferring `clone.*` configuration instead, citing concerns about precedent and maintainability. The discussion highlights tensions between workflow convenience and configuration philosophy.

## In brief

**`git log --graph` indentation improvements** -- Pablo Sabater sent v5 of his series improving visualization of commits with excluded parents, fixing test failures by unsetting COMMIT_GRAPH in the test environment. The cascading indentation approach has been well-reviewed, with one known edge case documented.

**Compiler compatibility updates** -- Dominik Loidolt's series modernizing version check macros in `compat/posix.h` was acked by Junio for integration into `next`. The changes improve Clang's handling of the UNUSED attribute while maintaining cross-platform compatibility.

**Global fetch.followRemoteHEAD** -- Matt Hunter addressed review feedback on his implementation, agreeing to use case-sensitive matching and non-fatal error handling for invalid values. The series is technically complete pending final review.

**Test grep standardization** -- Michael Montalbo's series introducing a lint tool to convert bare `grep` to `test_grep` saw maintainer pushback on the automated conversion approach despite uncovering 10 test bugs. The discussion continues about methodology versus demonstrated value.

**Documentation on review etiquette** -- Weijie Yuan's RFC proposing guidelines for patch reroll timing and feedback discussion received positive feedback from Junio, who emphasized how slower cadences improve review quality and global participation.

## On the radar

**Rust compatibility on NonStop** -- Randall S. Becker confirmed `NO_RUST=yes` works as a build workaround, but the long-term compatibility question remains unresolved as Git moves toward mandatory Rust in 3.0.

**Secure hook execution proposal** -- Jamison Phillips' RFC for verifying hooks in unzipped repositories sparked a security model discussion with brian m. carlson, who raised concerns about creating false security assurances while other attack vectors remain.