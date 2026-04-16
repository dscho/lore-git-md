# Git Mailing List Digest - 2026/04/15 (Wednesday)

**The day in brief.** A busy Wednesday with 98 emails across 22 threads, featuring significant progress on multiple fronts. Key highlights include the completion of cross-platform fsmonitor support after 15 iterations, finalization of subcommand autocorrection, and resolution of a critical MIDX compatibility issue in the upcoming v2.54.0 release. The day also saw continued refinement of test infrastructure and several feature proposals.

## Notable threads

### **Linux fsmonitor implementation finalized**

After 15 iterations, Paul Tarjan's cross-platform filesystem monitoring (fsmonitor) series has been approved by Junio Hamano. The implementation now supports Linux (via inotify), Windows, and macOS with comprehensive process isolation, cookie-based rename tracking, and proper daemon shutdown handling. The series was temporarily reverted to the `seen` branch for additional Gentoo testing but remains technically sound. Key improvements in v15 include proper lockfile handling, memory leak fixes, and test suite hardening. This marks a major milestone in Git's filesystem monitoring capabilities.

### **Subcommand autocorrection threshold unification**

Jiamu Sun's subcommand autocorrection series reached its final decision point, with Junio Hamano recommending unified thresholds between main commands and subcommands. The maintainer suggested keeping any threshold improvements as a separate follow-up series, ensuring consistency in Git's correction behavior. The feature, which adds `PARSE_OPT_SUBCOMMAND_AUTOCORR` flag support, is now ready for merging with all technical and documentation concerns addressed.

### **MIDX version compatibility crisis**

A critical backward compatibility issue emerged in Git v2.54.0-rc2 where the new MIDX version 2 format breaks repositories for users of Git v2.53.0. Johannes Schindelin reported the problem affects downgrade scenarios and mixed-version environments, contradicting the commit's claim of graceful degradation. Junio engaged Taylor Blau to provide recovery instructions (deleting `.midx` files) while the team's response strategy is formulated. This unexpected compatibility break may delay the release until resolved.

### **Test suite hardening with `set -e`**

Patrick Steinhardt's long-running effort to enable `set -e` in Git's test suite reached consensus on limiting the change to Bash 5+ due to inconsistent shell implementations. The series systematically addresses signal handling, daemon management, and cleanup routines to work reliably under strict error checking. Junio endorsed the pragmatic approach, prioritizing robust error detection in modern CI environments over universal shell compatibility. The final patch activates `set -e` with version guards after extensive preparatory work.

### **Negotiation controls for fetch/push**

Derrick Stolee's 7-patch series introducing `--negotiation-require` and `--negotiation-restrict` options advanced with detailed review feedback. The changes allow specifying critical refs that must be included in negotiation while restricting scope for performance. Junio raised edge cases around config inheritance and missing ref notification, but the core functionality appears sound. The series addresses monorepo scaling challenges where important branches were being truncated from negotiation.

## In brief

**Lisp dialect userdiff support** -- Scott L. Burson's series extending Git's diff highlighting for Scheme, Common Lisp, Emacs Lisp and Clojure received Johannes Sixt's final Acked-by, ready for integration.

**Autostash refinements** -- Harald Nordgren continued post-merge polishing of the `checkout -m` autostash feature, addressing error handling and test organization based on Phillip Wood's review.

**`the_repository` removal progress** -- Olamide Caleb Bello's configuration migration series advanced with consensus on eager parsing requirements for `trust_ctime` during stat comparison.

**Dynamic mailmap toggling** -- Siddharth Asthana's `git cat-file --batch-command` mailmap feature was approved with final documentation focusing on user-visible behavior over implementation details.

**Promisor-remote config fix** -- Trieu Huynh's patch correcting repository context for `promisor.quiet` in submodules was validated after initial test fragility concerns were resolved.

**`git backfill` progress reporting** -- Trieu Huynh's progress meter addition received test refinement suggestions from Tian Yuchen to better handle interaction with underlying fetch output.

## On the radar

**Subtree recursion limits** -- Colin Stagner's patches to fix shell recursion limits in `git subtree split` remain stalled due to lack of subtree user review, highlighting maintenance challenges for niche components.

**Worktree submodule support** -- JAM's RFC for `git worktree add --recurse-submodules` faces technical hurdles around submodule state management across worktrees, with Phillip Wood and Junio identifying key isolation requirements.

**Partial clone recursive fetch** -- Paul Tarjan's promisor-remote fix for recursive lazy fetching was dropped from integration branches pending resolution of filter persistence questions, though the core solution remains technically validated.