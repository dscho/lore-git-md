# Git Mailing List Digest - December 19, 2025

**The day in brief.** A busy Friday with 65 emails across 22 threads, dominated by continued debate over the proposed `git-history` command's branch handling and a contentious discussion about a `--no-indicators` diff option. Junio's "What's cooking" report provides a comprehensive snapshot of the current development landscape, while several documentation and test fixes moved toward completion.

## Notable threads

**git-history branch rewrite behavior refined** -- Patrick Steinhardt's RFC for the new `git-history` command saw significant discussion about how it should handle dependent branches during history editing operations. Steinhardt proposed a compromise where the command would implicitly rewrite all commits between the target and HEAD, similar to `git-replay`'s behavior. However, SZEDER Gábor and Elijah Newren raised concerns about conflict resolution and consistency, with Newren strongly advocating for either rewriting all dependent branches or requiring explicit user specification. The thread reveals ongoing tension between UI simplicity and behavior consistency in Git's evolving history editing capabilities.

**Sideband security discussion continues** -- Junio Hamano responded to Johannes Schindelin's proposal for handling ANSI escape sequences in Git's sideband channel, suggesting the configuration mechanism should move to the transport layer rather than being HTTP-specific. Hamano also revisited the security model debate, challenging the assumption that remote servers don't become more trustworthy after initial interaction. The exchange highlights differing philosophies about security defaults in Git's communication protocols.

**Windows symlink support finalized** -- Johannes Schindelin and Patrick Steinhardt reached agreement on handling PATH_MAX limitations in Windows symlink support, settling on a 32,768 character practical upper bound. This resolves the last open question in the series, which prepares Git for better symlink handling on Windows. The solution balances security concerns with real-world filesystem constraints while maintaining cross-platform compatibility.

**Documentation overhaul for git reset** -- D. Ben Knoble shepherded Julia Evans' documentation improvements for `git reset` through a final iteration, incorporating Junio Hamano's technical feedback while preserving Evans' pedagogical approach. The 4-patch series reorders command forms to match common usage, clarifies mode differences, and simplifies pathspec explanations. All substantive issues from earlier reviews were addressed, making this documentation effort ready for merging.

**Maintenance config file proposal reviewed** -- Junio Hamano provided detailed feedback on Matthew Hughes' RFC for a `maintenance.configFile` feature, questioning whether the maintenance-specific approach is optimal versus a more general solution for version-controlled configs. The review identified documentation gaps, code style issues, and test improvements needed, while keeping the door open for a revised proposal. Patrick Steinhardt also suggested complementary changes to `git for-each-repo` to better support the feature.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Bundle-URI validation improved** -- Sam Bostock's v2 series adds explicit NULL URI checks and better debug output for malformed bundle configurations.

**Worktree documentation clarified** -- A v2 series standardizes terminology around "missing" worktrees for the `--expire` option, incorporating Eric Sunshine's suggestion to use "prune" rather than "expire" in help text.

**Rust build system fixed for BSD/macOS** -- D. Ben Knoble's patch replaces GNU sed's `-s` extension with portable `sed -n` to correctly extract cargo's host information on BSD-derived systems.

**Duplicate free in path cache cleanup** -- Junio Hamano confirmed a straightforward fix removing a redundant `FREE_AND_NULL()` call in `repository.c`'s path cache cleanup.

**Refs tracing debug output fixed** -- A simple patch corrects `optimize_required` debug output that was showing incorrect values due to missing pointer dereferencing.

## On the radar

**Object handling performance regression** -- Junio's minimal fix for the regression is in place while Patrick Steinhardt prepares to rebase his more comprehensive series. Discussion continues about commit message conventions for fixes to unreleased code.

**Curl 8.18.0 test compatibility** -- Jeff King confirmed curl's latest fixes resolve Git's test failures, allowing simplification of the test adjustment series. The changes handle curl's stricter HTTP header unfolding behavior.