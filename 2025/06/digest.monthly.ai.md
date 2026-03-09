# Git Mailing List Digest - June 2025

**The month in brief.** June 2025 saw significant progress across Git's development, with 1,995 emails across 576 threads. The month was bookended by major releases (v2.50.0-rc0 at the start and v2.50.0 final at mid-month) and featured several long-running efforts reaching completion. Key highlights include the stabilization of `imap-send` authentication after 19 iterations, the introduction of stash import/export functionality, and foundational work on SHA-256 as the default hash algorithm. Performance optimizations dominated technical discussions, particularly around MIDX/cruft pack handling and batched reference updates, while platform compatibility work addressed issues on NonStop, Windows, and various BSD systems.

## Key developments

### `imap-send` modernization completes

After 19 iterations spanning multiple months, Aditya Garg's comprehensive overhaul of `git imap-send` was approved for merging. The series fixes a critical configuration parsing regression dating back to Git 2.46.0 while adding modern OAuth2.0 support (both standard OAUTHBEARER and Google's XOAUTH2), proper PLAIN authentication for OpenSSL, and new folder management features. The final version includes memory leak fixes, improved credential prompts, and RFC-compliant folder listing via `--list`. This represents one of the most extensive single-command modernizations in recent Git history, making the email-based workflow significantly more robust.

### Stash import/export functionality lands

Phillip Wood's series introducing `git stash export` and `import` subcommands completed its 8-iteration journey, enabling bidirectional transfer of stashes between repositories. The implementation stores commit chains under `refs/stash-export/` while preserving topology and message structure. Key refinements included adopting `commit_list` optimizations, standardizing reflog traversal APIs, and addressing platform-specific concerns. With 369 lines of test coverage and consensus from all major reviewers, this long-requested feature (discussed since 2022) provides a robust solution for stash portability.

### SHA-256 default transition progresses

brian m. carlson's series to make SHA-256 the default hash algorithm when built with `WITH_BREAKING_CHANGES` received thorough review. The patches introduce new constants (`GIT_HASH_DEFAULT` and `GIT_HASH_ORIGINAL`) to distinguish between legacy SHA-1 requirements and the new default while maintaining backward compatibility. This production-validated change represents a major step toward Git 3.0's planned SHA-256 transition, with comprehensive test coverage and naming questions resolved. The work positions Git for its most significant cryptographic upgrade since inception.

### MIDX and cruft pack optimizations

Taylor Blau's performance optimization series for MIDX and cruft pack interaction during repacking reached completion after addressing edge cases with incremental MIDX writing. The implementation introduces `repack.midxMustContainCruft` configuration (default true) that allows excluding cruft packs from MIDX when safe, reducing bloat while maintaining reachability guarantees. Production testing shows 5-20% speed improvements, with the series benefiting from review by both Junio Hamano and Elijah Newren. This work exemplifies Git's continued focus on scaling to large repositories efficiently.

### Object database refactoring matures

Patrick Steinhardt's 17-patch series to remove `the_repository` from Git's object database subsystem progressed through five iterations, systematically converting the code to use explicit `object_database` parameters. The work renames core structures (`raw_object_store` → `object_database`), moves implementation files, and updates over 140 files while maintaining identical behavior. This foundational change enables future pluggable ODB backends while improving code organization, with Derrick Stolee among the reviewers signing off on the approach.

## In brief

**Git v2.50.0 released** -- The major feature release includes 621 commits from 98 contributors, featuring new `--combine-cruft-below-size` for `git repack`, machine-parsable `rev-list` output, and improved `send-email` Outlook compatibility.

**NonStop platform fixes** -- Build issues with reftable subsystem were resolved by making `__attribute__((__unused__))` conditional on `__GNUC__`, with an empty fallback for other compilers.

**`git apply --intent-to-add` fixed** -- Johannes Altmanninger addressed long-standing issues with the effectively non-functional `-N` option through a five-patch series adding safety checks and proper index handling.

**Promisor-remote protocol** -- Christian Couder extended the protocol to support configurable validation of remote attributes via `promisor.sendFields` and `promisor.checkFields`.

**`git fetch --prune` optimization** -- Restructured logic reduced runtime from 470 seconds to under 1 second for repositories with 174,000 refs by using an O(N*logN) approach.

**Windows terminal handling** -- James Duley submitted a critical fix for assertion failures when toggling DUPLEX mode during interactive operations like `git add -p`.

**Documentation standardization** -- Jean-Noël Avila completed conversion of the `git-log` man page to standardized AsciiDoc format, one of the largest remaining documentation efforts.

**New `git repo-info` command** -- Lucas Seiki Oshiro's GSoC project introduces repository metadata reporting in JSON and plaintext formats, covering reference storage format and bare/shallow status.

**Bloom filter optimization** -- Lidong Yan enabled bloom filter acceleration for multiple pathspec queries in revision traversal while maintaining the single-pathspec fast path.

**String-list test modernization** -- Shejialuo converted shell-based tests to C unit tests while improving the core implementation and fixing sign comparison warnings.

## Looking ahead

**SHA-256 transition** -- With brian m. carlson's series nearing completion, attention turns to testing and validation as Git prepares to make SHA-256 the default hash algorithm in version 3.0.

**Refs list command** -- The `git refs list` discussion continues, with documentation improvements being made to the existing `git-for-each-ref` as part of the review process.

**Reftable compaction** -- Patrick Steinhardt's reftable work was recently mentioned in "What's cooking" as needing review attention in the coming weeks.

**GSoC projects** -- Both Lucas Seiki Oshiro's `git repo-info` and Meet Soni's ref consolidation work will see active development through the summer, with regular updates expected.

**Git Merge 2025** -- Taylor Blau's announcement of September's conference schedule may prompt increased activity around planned features and demonstrations.