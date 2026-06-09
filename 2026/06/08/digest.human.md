# Git Mailing List Digest - 2026/06/08 (Monday)

**The day in brief.** A busy Monday with 130 emails across 34 threads, featuring significant progress on several fronts. Key highlights include Patrick Steinhardt's `git history drop` series reaching v3, performance optimizations for `git ls-files` and `git describe`, and substantive discussions about submodule mirroring and config include handling. Junio Hamano provided maintainer guidance across multiple threads as several features near completion.

## Notable threads

**`git history drop` reaches v3** -- Patrick Steinhardt's 9-patch series adding a "drop" subcommand to the experimental `git history` command is now in its third iteration. The feature allows removing a commit and replaying its descendants on top of its parent, with extensive test coverage (537 lines across 4 scripts). The series refactors core reset machinery to support dry-run operations and skip reference updates when needed. Some API design questions around opt-in vs opt-out HEAD updates remain open for follow-up work.

**Performance optimization for `git ls-files`** -- Tamir Duberstein's patch to filter pathspecs before expensive `lstat` operations shows dramatic speedups (60.7s->1.06s) when most entries don't match. Jeff King confirmed the worst-case regression (~800ms for 10k files when all match) is acceptable given the optimization's benefits. The change is now ready for maintainer consideration after thorough performance analysis.

**Submodule mirroring architecture discussion** -- Jeff King weighed in on the ongoing discussion about submodule mirroring infrastructure, clarifying the tension between centralized mirroring needs and Git's decentralized nature. The thread is converging toward enhancing URL rewriting capabilities while leaving the mapping distribution problem to higher-level tools. Simon Richter's input about separating identity from location added conceptual clarity to the design challenges.

**Config include disablement proposal** -- Derrick Stolee's series adding `GIT_CONFIG_INCLUDES=0` and `--no-includes` options sparked discussion about security/workflow tradeoffs. Jeff King raised concerns that the binary switch might break legitimate workflows (like work/personal email separation) while only partially addressing sandboxing needs. The series appears useful for strict sandboxing but may need extension for more nuanced control.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Compiler version check modernization** -- Dominik Loidolt submits v3 patches simplifying GCC and Clang version checks in `compat/posix.h`, replacing legacy bit-shift comparisons with readable major/minor number checks.

**Worktree metadata tracking** -- Norbert Kiesel's v2 patch adds both creation timestamps and free-form descriptions to worktrees, with sorting options and comprehensive test coverage.

**Priority queue optimization finalized** -- Kristofer Karlsson's v4 series generalizing the lazy_queue pattern shows 1.7-2.7% speedups in traversal-heavy operations after addressing all review feedback.

## On the radar

**Rustification effort** -- Ezekiel Newren's work to introduce Rust code into Git remains paused pending resolution of platform support concerns raised by Randall Becker. The discussion has been quiet since the last exchange about NonStop compatibility.

**ODB abstraction** -- Patrick Steinhardt's 16-part series converting packed object storage to use `struct odb_source` has received positive reviews from Karthik Nayak and appears ready for maintainer consideration, representing a major step in the multi-year effort to enable pluggable storage backends.