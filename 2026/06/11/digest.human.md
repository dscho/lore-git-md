# Here's the Git mailing list digest for June 11, 2026:

### The day in brief
A busy Friday with 98 emails across 30 threads, featuring significant progress on several fronts. Key highlights include Junio Hamano's Git v2.55.0-rc0 release announcement, Patrick Steinhardt's `git history drop` series nearing completion, and a security update accelerating the timeline for ANSI escape sequence sanitization. Performance optimizations and documentation improvements dominated much of the day's discussion.

### Notable threads

### Security hardening timeline accelerated

Junio Hamano announced that the sideband sanitization series addressing CVE-2024-32002 and CVE-2024-52005 will now ship in Git 2.55 rather than waiting for Git 3.0. The change removes the transitional compatibility layer after successful production validation in Git for Windows and Red Hat environments. The implementation prevents terminal state corruption and input buffer injection attacks while preserving legitimate ANSI color sequences used by pre-receive hooks.

**`git history drop` implementation**

Patrick Steinhardt's 10-patch series introducing a `drop` subcommand to the experimental `git history` command reached v5 with all technical feedback addressed. The implementation builds on refactored reset machinery and includes extensive test coverage (537 lines) for edge cases. Junio identified one remaining issue regarding dry-run behavior with reference updates, prompting further discussion about the interaction between dry-run mode and the new `RESET_WORKING_TREE_UPDATE_HEAD` flag.

**Performance optimizations converge**

Multiple performance threads saw significant progress:
- Arijit Banerjee's index-pack delta resolution optimization concluded with Junio opting for the simpler v2 implementation after Jeff King's recommendation
- Tamir Duberstein's memoized commit traversal for `--contains` queries received thorough review from Peff, who validated the approach while suggesting test improvements
- The `git ls-files --deleted/--modified` optimization was approved by Junio following discussion about pathspec handling tradeoffs

**Documentation standardization continues**

Jean-Noël Avila and Tuomas Ahola's documentation fixes for AsciiDoc markup reached v3, resolving octothorpe escaping issues through backtick syntax after extensive discussion of rendering quirks. Kristoffer Haugsbakk's SubmittingPatches trailer documentation series also progressed, with Junio providing feedback on DCO requirements for the `Based-on-patch-by` trailer.

### In brief

**Reftable backend refactoring** -- Patrick Steinhardt's series modernizing reference backend infrastructure introduced absolute path handling while Jeff King noted historical performance optimizations using relative paths that might warrant consideration.

**Test hygiene fix** -- Junio fixed lingering FIFO files in t1400-update-ref.sh that could cause test hangs, with Patrick Steinhardt acknowledging the improvement.

**Email etiquette documentation** -- Weijie Yuan proposed adding quoted text trimming advice to MyFirstContribution.adoc, which Junio approved as clear and well-placed.

**HTTP authentication test failure** -- A reported test failure in t5563.18 revealed behavioral differences between curl versions in Fedora 44 vs 45, requiring investigation into whether test expectations need updating.

**Reachability function consolidation** -- Kristofer Karlsson's effort to unify `get_reachable_subset()` and `tips_reachable_from_bases()` prompted discussion about project norms for discussing major approach changes before implementation.

### On the radar

**Revision walking optimizations** -- Mirko Faina requested coordination between the `--max-count-oldest` feature and their own related patch series, leading Junio to hold the change from merging to master pending alignment.

**Config include sandboxing** -- The discussion around Derrick Stolee's `--no-includes` proposal is evolving toward a more comprehensive security boundary solution after Jeff King raised concerns about partial coverage of config vectors.

**Timestamp precision improvements** -- brian m. carlson proposed runtime configuration for nanosecond support (USE_NSEC) as work continues to address worktree diff performance issues related to second-granularity timestamps.