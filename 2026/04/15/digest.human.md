# Here's the daily digest for April 15, 2026:

**The day in brief.** A busy Wednesday with 98 emails across 22 threads, dominated by major feature work nearing completion. Key highlights include the Linux fsmonitor implementation being approved after 15 iterations, finalization of subcommand autocorrection thresholds, and a critical MIDX compatibility issue surfacing in the v2.54.0-rc2 release candidate. The test suite modernization effort also reached a milestone with `set -e` activation for Bash 5+.

## Notable threads

### Linux fsmonitor implementation approved

After 15 iterations, Paul Tarjan's cross-platform fsmonitor series received maintainer approval from Junio Hamano. The final version (v15) includes comprehensive Linux inotify support alongside existing Windows and macOS implementations, with careful handling of directory watches, rename tracking, and remote filesystem detection. The series was temporarily reverted to `seen` for Gentoo build testing but remains technically sound. Ben Knoble confirmed the Gentoo issues stem from local patch conflicts rather than problems in the core implementation.

### Subcommand autocorrection thresholds unified

Jiamu Sun's subcommand autocorrection series reached its final decision point as Junio Hamano directed that thresholds should be unified between main commands and subcommands. The maintainer suggested any future threshold improvements should update both simultaneously, maintaining consistency across Git's interface. This resolves the last open question in a series that has already completed implementation, testing, and documentation.

### MIDX version 2 breaks backward compatibility

Johannes Schindelin reported that the MIDX version 2 format in Git v2.54.0-rc2 makes repositories unusable with Git v2.53.0, contrary to claims of graceful degradation. The issue affects downgrade scenarios, mixed-version environments, and interoperability with libgit2/JGit. Junio engaged Taylor Blau to provide recovery instructions (deleting .midx files) while the compatibility implications are assessed. This has become a critical blocker for the upcoming release.

### Test suite activates `set -e` for Bash 5+

Patrick Steinhardt's test modernization effort culminated with `set -e` being activated for Bash 5+ environments after extensive preparation across 12 patches. The version restriction addresses inconsistent shell behavior (particularly macOS's Bash 3.2.57) while providing robust error detection in modern environments. Junio endorsed the pragmatic approach, recognizing the value of strict error checking during test development despite platform limitations.

## In brief

**Lisp dialect userdiff support** -- Scott L. Burson's series extending Git's diff highlighting for Scheme, Common Lisp, Emacs Lisp and Clojure received Johannes Sixt's Acked-by, completing its review cycle.

**Promisor-remote recursive fetch fix stalled** -- Junio placed Paul Tarjan's fix for recursive lazy fetching on hold due to unresolved questions about filter configuration persistence, though the core `GIT_NO_LAZY_FETCH` solution remains technically sound.

**Mailmap toggling in cat-file batch** -- Siddharth Asthana's dynamic mailmap control for `git cat-file --batch-command` was finalized with documentation focused on user-visible behavior rather than implementation details.

**Fetch/push negotiation controls** -- Derrick Stolee's series adding `--negotiation-require` and `--negotiation-restrict` options progressed with feedback on config inheritance edge cases and error messaging consistency.

**Autostash refinements post-merge** -- Harald Nordgren continued polishing the merged `checkout -m` autostash feature, addressing error handling details and test optimizations raised by Phillip Wood.

## On the radar

**Subtree recursion limit fix** -- Colin Stagner's series to fix shell recursion limits in `git subtree split` remains stalled due to lack of review from subtree users, highlighting maintenance challenges for niche components.

**Worktree submodule support** -- JAM's RFC for `git worktree add --recurse-submodules` faces technical hurdles around submodule state isolation, with Junio and Phillip Wood emphasizing the need for proper worktree-specific state management.