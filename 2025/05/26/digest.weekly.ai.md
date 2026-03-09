# Git Mailing List Digest - 2025/05/26 -- 2025/06/01

**The period in brief.** A busy week with 460 emails across 133 threads saw significant progress on multiple fronts. Key developments include the release of Git v2.50.0-rc0, finalization of the `imap-send` OAuth2.0 authentication series, and completion of stash import/export functionality. The week also featured extensive work on test infrastructure modernization, signature verification improvements, and platform-specific fixes. Notable threads converged included MIDX/cruft pack optimizations, NonStop compatibility fixes, and maintenance task locking refinements.

## Key developments

### Git v2.50.0-rc0 released

Junio Hamano announced the first release candidate for Git 2.50, containing 541 non-merge commits from 73 contributors. The release includes new cruft pack controls, TCP keepalive for HTTP, machine-parsable rev-list output, and continued work to reduce `the_repository` usage. The merge-recursive backend has been officially deprecated in favor of the ort strategy. Several documentation standardization efforts also landed in this release, including Jean-Noël Avila's `git-verify-*` updates completing another piece of the synopsis format conversion.

### IMAP authentication overhaul completed

Aditya Garg's comprehensive `imap-send` series reached v11 with all technical concerns resolved. The implementation now provides OAuth2.0 support (both OAUTHBEARER and XOAUTH2 protocols), fixes critical configuration parsing bugs, and adds user-friendly features like folder listing (`--list`) and command-line folder specification (`--folder`). The series underwent extensive review of error message formatting to match Git conventions and represents a major usability improvement for Git's email sending capabilities. The final version drops OpenSSL backend configuration support to simplify maintenance, focusing solely on libcurl.

### Stash import/export functionality finalized

Patrick Steinhardt's 4-patch series introducing `git stash export`/`import` subcommands reached its seventh iteration with all technical concerns addressed. The implementation provides robust bidirectional transfer of stashes between repositories via refs under `refs/stash-export/`, preserving topology through commit chains. Key refinements include adopting Phillip Wood's `commit_list` optimization, standardizing on reflog traversal APIs per Junio's guidance, and addressing platform-specific concerns. With 268 lines of test coverage and consensus from all major reviewers, this represents the culmination of discussions since 2022 about stash portability.

### MIDX and cruft pack optimizations

Taylor Blau's performance optimization series for MIDX and cruft pack interaction during repacking reached completion. The final version clarified when objects from cruft packs become reachable and the relationship between `--stdin-packs` and `--stdin-packs=follow` modes. The changes have been validated in GitHub's production environment and include a new `repack.midxMustContainCruft` configuration option. This work represents a significant improvement in handling large repositories with complex object storage patterns.

### Maintenance task locking refinements

Patrick Steinhardt's series addressing race conditions in maintenance operations progressed to v2 with improved error handling and task phase management. The key architectural change splits task execution into "before detach" (foreground) and "after detach" (background) phases to safely handle reference locking. The series includes 12 patches covering everything from initial refactoring to the final GC task implementation, with detailed discussion about the interaction between `git-maintenance` and `git-gc` flags crystallizing the concept of "foreground work" that should run regardless of detachment status.

### NonStop platform compatibility fixes

After thorough discussion, the project settled on a solution for NonStop platform compatibility with the reftable subsystem. Carlo Marcelo Arenas Belón's fix makes the `REFTABLE_UNUSED` attribute conditional on `__GNUC__`, with an empty fallback for other compilers. This addresses Randall S. Becker's original report of build failures in Git 2.50.0-rc0 and will be fast-tracked into the codebase before the next release candidate.

## In brief

**Signature verification improvements** -- Christian Couder's v2 series now focuses on verification workflows before tackling fast-import/export signature handling, introducing a `--summary` option for `git verify-commit` that outputs standardized verification status.

**Test runner behavior** -- Patrick Steinhardt's Meson test integration work sparked discussion about handling unexpected passes in TODO tests, with consensus forming that runners should fail on unexpected passes to maintain test expectation integrity.

**Path-walk delta compression** -- Taylor Blau and Derrick Stolee concluded review of the path-walk delta compression series with final approvals for the v3 implementation introducing path-based delta compression options.

**R language support** -- Rodrigo Carvalho's R language support for Git diffs reached completion in v2, handling both `<- function` and `= function` assignment syntax variants with comprehensive test coverage.

**Documentation standardization** -- Multiple documentation efforts reached completion including Aditya Garg's email-related formatting, Kim W.W.'s sparse-checkout list numbering, and BUG() message standardization removing translation markers from programmer-facing assertions.

**BSD platform improvements** -- Brad Smith contributed patches improving CPU and memory detection on OpenBSD/NetBSD systems, correctly handling SMT-disabled configurations and >=4GB memory reporting.

**git-cvsserver deprecation** -- Following dead code removal, Junio Hamano signaled approval for proceeding with full deprecation of the legacy CVS bridge component.

## Looking ahead

The upcoming week will likely focus on finalizing Git v2.50.0, with the rc1 release expected to incorporate the NonStop compatibility fix and possibly the completed `imap-send` series. Several major topics remain in flight:

**ODB abstraction naming** -- The thread debating `odb_alternate` vs `odb_source` terminology appears to be settling on "source" as the preferred term after Junio's analysis of glossary definitions.

**GPG signing for git-subtree** -- Patrik Weiskircher's series is working through option parsing challenges for the `-S/--gpg-sign` flag, with Junio suggesting a hybrid stuck/unstuck approach.

**Test infrastructure** -- Patrick Steinhardt's Meson/TAP integration series may see final merging after addressing remaining platform-specific edge cases.