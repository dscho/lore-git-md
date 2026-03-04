# Git Development Digest - 2025/07/22

## The day in brief

A busy day with 135 emails across 30 threads, dominated by several major discussions: final polish on the `git last-modified` feature, Rust integration debates, reflog migration fixes, and stabilization of `git-switch`/`git-restore`. Notable technical progress includes Patrick Steinhardt's reftable reflog fixes nearing completion and Leon Michalak's interactive patch context configuration series resolving its final test issues.

## Notable threads

### `git last-modified` feature finalization

Toon Claes and Taylor Blau concluded their review of the `git last-modified` series (formerly called `blame-tree`), addressing remaining architectural questions about callback structures and performance optimization details. The Bloom filter integration shows promising speedups (50.6% for top-level checks, 36.4% for subdirectories), though some edge cases around subdirectory test methodology were identified. The series appears ready for merging after addressing these final performance test refinements.

### Rust integration debates intensify

The ongoing discussion about Rust adoption saw significant platform compatibility concerns raised by Gentoo maintainers, particularly regarding architectures without Rust support (HPPA/Alpha/m68k) and 32-bit systems. Patrick Steinhardt and Elijah Newren advocated for full Rust adoption, while Junio Hamano noted architectural challenges in Git's current code organization that complicate partial integration. The thread revealed concrete deployment obstacles that may influence Git's Rust strategy, with supply chain risks and dependency management emerging as additional concerns.

### Reflog migration fixes

Patrick Steinhardt's 8-part series addressing reflog migration issues between storage formats reached its final review stages. The patches fix critical bugs where migrated reflog entries were losing identity information and incorrectly zeroing old object IDs. Junio Hamano provided the final review feedback, suggesting a minor shell scripting improvement to the test infrastructure. The comprehensive series includes both core fixes and extensive test coverage for scenarios that were untested since the feature's introduction in 2008.

### `git-switch` and `git-restore` stabilization

Justin Tobler's proposal to remove experimental markers from these commands sparked debate about interface design decisions from their 2019 introduction. While consensus emerged that the experimental status should be removed after six years of stable use, Junio Hamano objected to removing the `-m` short flag, citing his own workflow dependencies. The documentation updates appear ready to proceed, but the interface changes may need revision to preserve existing short flag behavior.

## In brief

**Interactive patch context configuration** -- Leon Michalak's series adding `-U/--unified` and `--inter-hunk-context` options to interactive patch modes (`add -p` etc.) resolved its final test issues and is ready for merging.

**IMAP sent-folder archiving** -- Aditya Garg's v3 patch for `git send-email` adopted the "imap-sent-folder" naming while maintaining flexibility to archive to any folder, though debate continues about the feature's fundamental scope.

**GPG path expansion** -- A two-patch series fixed tilde expansion for `gpg.program` configuration, with tests verifying home directory and relative path handling.

**File descriptor leaks** -- Hoyoung Lee's resource leak fixes for test helpers (`test-truncate`, `test-delta`) were refined through review, though debate continues about cleanup necessity in termination paths.

**`git blame` optimization** -- Han Young proposed skipping full commit parsing when only author info is needed, though reviewers requested performance measurements before merging.

**`git imap-send` documentation** -- Aditya Garg improved docs for the `--list` option's IMAP folder discovery, though Junio questioned whether the output format should be made more user-friendly.

## On the radar

**VCS metadata protection** -- A new thread explores whether Git should treat directories like `.jj` (Jujutsu) as precious files to prevent accidental deletion, with discussion leaning toward a general mechanism rather than special cases.

**Partial clone edge cases** -- Jeff King identified a race condition between commit graph lookups and object repacking in treeless clones, with follow-up discussion about submodule behavior in such repositories.

**`git repo info` subcommand** -- Lucas Seiki Oshiro's GSoC project added repository metadata fields while debating output format semantics, with Junio questioning the NUL-delimited format's field separator choice.