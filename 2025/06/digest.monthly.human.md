# Git Mailing List Monthly Digest - June 2025

**The period in brief.** June 2025 was a highly productive month for the Git project with 1,995 emails across 576 threads. The month saw the release of Git v2.50.0 with significant performance improvements and new features, while major technical efforts reached completion including the `imap-send` modernization, stash import/export functionality, and SHA-256 default transition preparations. Platform compatibility work remained prominent with fixes for NonStop, Windows, and various Unix systems. The month's standout achievements were Patrick Steinhardt's object database refactoring and Taylor Blau's MIDX/cruft pack optimizations, both representing substantial architectural improvements.

## Key developments

### Git v2.50.0 release cycle

The month was dominated by the Git v2.50.0 release cycle, progressing through three release candidates (rc0-rc2) to final release on June 16. The version introduced several notable features including machine-parsable `rev-list` output, TCP keepalive for HTTP, new cruft pack controls, and deprecation of the merge-recursive backend in favor of "ort". Performance optimizations in ref handling and reftable operations were particularly impactful, while continued reduction of `the_repository` usage set the stage for future architectural changes. The release incorporated 621 commits from 98 contributors (35 new), demonstrating healthy project growth.

### Object database refactoring

Patrick Steinhardt's comprehensive refactoring of Git's object database subsystem reached maturity this month. The 17-patch series systematically removed `the_repository` dependencies while renaming core structures (`raw_object_store` → `object_database`) and moving implementation files (`object-store.{c,h}` → `odb.{c,h}`). This foundational change enables future introduction of pluggable ODB backends while improving code organization. The work required updating over 140 files with mechanical conversions while maintaining identical behavior, with v5 addressing final naming refinements and merge conflicts before receiving positive reviews from Derrick Stolee and others.

### `imap-send` modernization completes

After 19 iterations spanning several months, Aditya Garg's comprehensive overhaul of `git imap-send` was approved for merging. The work fixed a critical configuration parsing regression dating back to Git 2.46.0 while adding modern OAuth2.0 authentication support (both standard OAUTHBEARER and Google's XOAUTH2), implementing proper PLAIN authentication for OpenSSL, and introducing new folder management features. The series also addressed numerous memory leaks and improved user experience with better credential prompts. Junio Hamano's approval marked the conclusion of one of the longest-running patch series in recent memory, significantly enhancing Git's email-based workflow capabilities.

### MIDX and cruft pack optimizations

Taylor Blau's performance optimization series for MIDX and cruft pack interaction during repacking reached its final form in June. The v6 iteration properly handled edge cases with incremental MIDX writing where cruft packs could incorrectly appear in the MIDX, introducing a new `repack.midxMustContainCruft` config (default true) that allows excluding cruft packs from MIDX when safe. Production testing showed 5-20% speed improvements in GitHub's environment, with the series receiving thorough review from Junio Hamano and Elijah Newren. This work represents a significant step in Git's ongoing large-repository performance efforts.

### SHA-256 default transition progresses

brian m. carlson's series to make SHA-256 the default hash algorithm when built with `WITH_BREAKING_CHANGES` received thorough review and appears ready for merging. The patches introduce new constants (`GIT_HASH_DEFAULT` and `GIT_HASH_ORIGINAL`) to distinguish between legacy SHA-1 requirements and the new default while maintaining backward compatibility. This production-validated change represents a major step toward Git 3.0's planned SHA-256 transition, with comprehensive test coverage and naming questions resolved during June discussions.

## In brief

**Stash import/export functionality** -- Phillip Wood's series implementing `git stash export`/`import` completed after 8 iterations, enabling robust bidirectional transfer of stashes between repositories via `refs/stash-export/`.

**Promisor-remote protocol** -- Christian Couder extended the protocol to support configurable validation of remote attributes via `promisor.sendFields` and `promisor.checkFields`.

**`git apply --intent-to-add` fix** -- Johannes Altmanninger addressed long-standing issues with the effectively non-functional `-N` option through a focused 5-patch series.

**Performance optimizations** -- Phil Hord's `git fetch --prune` optimization reduced runtime from 470 seconds to under 1 second for large repositories by using an O(N*logN) approach.

**NonStop platform fixes** -- Carlo Marcelo Arenas Belón resolved build issues with the reftable subsystem by making `__attribute__((__unused__))` conditional on `__GNUC__`.

**Windows terminal handling** -- James Duley fixed a Windows terminal assertion failure when toggling DUPLEX mode during interactive operations like `git add -p`.

**Documentation standardization** -- Jean-Noël Avila completed conversion of the `git-log` man page to standardized AsciiDoc format, one of the largest remaining documentation efforts.

**New `git repo-info` command** -- Lucas Seiki Oshiro's GSoC project introduced repository metadata reporting in both JSON and plaintext formats.

**Bloom filter optimization** -- Lidong Yan enabled bloom filter acceleration for multiple pathspec queries in revision traversal via `struct bloom_keyvec`.

**Maintenance task locking** -- Patrick Steinhardt refactored maintenance operations to prevent lockfile races by splitting tasks into foreground and background phases.

## Looking ahead

**SHA-256 transition** -- With brian m. carlson's series nearing completion, attention will turn to broader testing and validation as Git prepares for version 3.0.

**Global state reduction** -- Ongoing work to remove `the_repository` dependencies continues, with `is_bare_repository()` specifically mentioned as a target.

**GSoC projects** -- Both Lucas Seiki Oshiro's `git repo-info` and Meet Soni's ref consolidation work will see active development through the summer.

**Reftable compaction** -- Patrick Steinhardt's reftable work was recently mentioned in "What's cooking" as needing review attention in the coming weeks.

**Git Merge 2025** -- Taylor Blau's announcement of September's conference schedule may prompt increased activity around planned features and demonstrations.