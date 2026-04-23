Here's the daily digest for April 22, 2026:

**The day in brief.** A moderately busy Wednesday with 76 emails across 18 threads, featuring several notable technical discussions and patch series nearing completion. Key highlights include Patrick Steinhardt's new `git history fixup` command, Derrick Stolee's negotiation control series reaching v3, and a critical bugfix for mksh test suite hangs.

**Notable threads**

**`git history fixup` subcommand introduced** -- Patrick Steinhardt proposes a new subcommand for his experimental `git history` tool that combines `git commit --fixup` with automatic rebasing of dependent branches. The 2-patch series includes extensive tests and documentation, though early reviews from Tian Yuchen and D. Ben Knoble identified a file deletion handling bug and documentation clarity issues. The command currently aborts on merge conflicts, with plans to improve this when Git gains better conflict resolution capabilities.

**Negotiation control for fetch/push finalized** -- Derrick Stolee's v3 series introducing `--negotiation-include` and `--negotiation-restrict` options is now technically complete after addressing all feedback. The changes optimize fetch/push negotiation in monorepos by allowing control over which ref tips are prioritized. The implementation spans multiple components with thorough test coverage, and appears ready for integration pending maintainer review.

**mksh test suite hang regression identified** -- A serious regression causing test suite hangs with mksh was bisected to commit dd3693eb08 (transport-helper cleanup). Jeff King analyzed the issue and proposed reverting the change since it fails to prevent zombies while introducing hangs. The root cause involves mksh's different SIGTERM handling compared to bash/dash. This will need resolution before the next release.

**In brief**

**Subtree modernization progress** -- Johannes Schindelin endorsed Ian Jackson's Rust-based `git-subtree` rewrite for Windows performance benefits, confirming the dual-path approach (Colin Stagner's shell patches for in-tree maintenance plus Ian's external Rust tool) has community acceptance.

**Subcommand autocorrection ready** -- Jiamu Sun's parse-options autocorrection series reached v5 with all feedback addressed, including unified thresholds with main command behavior and comprehensive test coverage. The feature provides consistent mistyped-subcommand handling across Git.

**Ref backend consolidation** -- Karthik Nayak's 8-patch series to standardize object validation across ref implementations received thorough review from Patrick Steinhardt, with all technical feedback now addressed. The changes move validation logic from individual backends to the shared layer.

**Pseudo-merge bitmap fixes complete** -- Taylor Blau's bugfix series for pseudo-merge bitmaps is ready after addressing Elijah Newren's final test script feedback. The changes fix lookup sorting, commit date parsing, and memory leaks in this critical optimization.

**Build system fixes** -- Toon Claes resolved Ninja build compatibility issues for older versions (pre-1.10.0) with a solution that unconditionally collapses config-list.h dependencies to a single line, validated across implementations.

**On the radar**

**Commit signing regression** -- brian m. carlson's fix for non-UTF-8 message signing since v2.45.0 is in but Elijah Newren noted potential duplicate warning messages that may warrant follow-up.

**Source tree reorganization** -- Patrick Steinhardt's proposal to move libgit.a components to a "lib/" directory appears to have reached consensus, with implementation planning now underway.