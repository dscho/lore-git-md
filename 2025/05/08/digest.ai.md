# Git Mailing List Digest - 2025/05/08

**The day in brief.** A busy Thursday with 87 emails across 25 threads saw significant progress on several fronts. Key developments included final approval for the `git send-email` authentication improvements, ongoing refinement of the `git-blame-tree` integration debate, and multiple performance optimizations landing for packed-refs and sparse index operations. Junio's "What's cooking" report provided a comprehensive snapshot of the project's active branches.

## Notable threads

**Stash import/export capability finalized** -- A v5 series from Lidong Yan introduces `git stash export` and `import` subcommands, solving the long-standing limitation that stashes couldn't be transferred between repositories. The implementation converts stashes to commit chains with two parents - one linking stashes sequentially and another preserving the original stash data. After five iterations addressing memory leaks and edge cases, the series now includes robust test coverage and documentation.

**Packed-refs memory optimizations refined** -- Jeff King and Junio Hamano discussed final polish for a series optimizing memory usage in packed-refs operations. The key insight was that the current mmap approach was making unnecessary in-memory copies during fsck operations, defeating the intended memory savings. The solution will use direct mmap/verify/unmap patterns for verification-only cases. This caps off a series that already showed dramatic improvements - from 67.6s to 18.8s for a 2.7GB promisor pack.

**`git-blame-tree` integration debate continues** -- The discussion about whether to integrate directory-level blame into `git blame` or keep it as a separate command saw extensive back-and-forth. Marc Branchaud strongly advocated integration for discoverability, while D. Ben Knoble proposed a middle ground mirroring `git diff`/`git diff-tree`. Junio clarified key constraints: `git blame <file>` must maintain its current line-by-line behavior, while directory handling could default to non-recursive with an option to recurse. The thread remains in RFC status pending resolution of these UI design questions.

**`git send-email` authentication improvements approved** -- After seven iterations refining both technical implementation and documentation, Junio queued Aditya Garg's series improving OAuth2.0 support and RFC1035 domain validation. The changes include stricter FQDN checking to prevent SMTP failures and comprehensive documentation of OAuth methods for Gmail/Outlook. The author demonstrated the workflow by sending the patches using the documented `git-credential-outlook` helper.

**Build system path handling standardized** -- Ramsay Jones' 5-part series aligned path handling across Git's build systems (Make, Meson, Autoconf). Changes included proper sysconfdir handling in Meson, GIT_EXEC_PATH consistency, and upgrading autoconf's `sysinfo()` detection to match Meson's more robust approach. The discussion revealed broader questions about autoconf's future as Eli Schwartz noted recent fixes appear motivated by completeness rather than active use.

## In brief

**Reftable memory leak fix** -- Lidong Yan addressed a theoretical leak in the REFTABLE_REALLOC_ARRAY macro, though René Scharfe noted the macro appears unused in the codebase.

**Commit-graph pack leak fix** -- A simple 2-line change properly frees the `packed_git` struct when `open_pack_index()` fails during commit-graph operations.

**Mailinfo header decoding fix** -- Lidong Yan fixed a strbuf leak in email header decoding when `convert_to_utf8()` fails, with Junio suggesting longer-term API improvements.

**Sparse-index performance gains** -- Derrick Stolee's series making `git add -p/-i` sparse-index aware showed up to 96.7% runtime reduction for sparse checkouts.

**Perl path customization finalized** -- The meson build system now properly supports custom Perl library paths like `/usr/share/perl5/vendor_perl`, matching Makefile behavior.

**Git Merge 2025 CFP extended** -- Taylor Blau announced the talk proposal deadline extended to May 20, with remote presentation options now available.

## On the radar

**`the_repository` removal continues** -- Patrick Steinhardt's 18-patch series advancing the object store abstraction work appeared in Junio's "What's cooking" report, indicating this major architectural effort remains active.

**Trailer API improvements** -- Li Chen is working on sequencer.c changes to support `git rebase --trailer` after Phillip Wood's detailed review established technical requirements.

**Bundle-based workflows** -- The backup strategy discussion uncovered timestamp vulnerabilities in `--since`-based approaches, with Jeff King proposing state-tracking alternatives.