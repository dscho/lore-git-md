# Git Mailing List Digest - 2025/06/16 -- 2025/06/22

**The week in brief.** A busy week with 276 emails across 88 threads, marked by the Git v2.50.0 release and significant progress on several fronts. Key developments include major performance optimizations for `git fetch --prune` and MIDX/cruft pack handling, the SHA-256 default transition nearing completion, and the long-awaited `imap-send` improvements finally reaching merge readiness. The week also saw active discussions around submodule remote handling, Windows terminal fixes, and the introduction of a new `git repo-info` command.

## Key developments

### Git v2.50.0 released

Junio Hamano announced Git v2.50.0, a major feature release with 621 commits from 98 contributors (35 new). User-facing highlights include new `--combine-cruft-below-size` for `git repack`, machine-parsable `rev-list` output, and improved `send-email` Outlook compatibility. The Windows port (Git for Windows 2.50.0) followed shortly after with updated dependencies and ReFS drive compatibility fixes. This release continues reducing `the_repository` usage and optimizing the reftable backend, setting the stage for upcoming architectural changes.

### SHA-256 default transition progresses

brian m. carlson's series to make SHA-256 the default hash algorithm when built with `WITH_BREAKING_CHANGES` received thorough review from Junio Hamano. The patches introduce new constants (`GIT_HASH_DEFAULT` and `GIT_HASH_ORIGINAL`) to distinguish between legacy SHA-1 requirements and the new default, while maintaining backward compatibility. After resolving naming questions and verifying comprehensive test coverage, this production-validated change appears ready for merging, representing a major step toward Git 3.0's planned SHA-256 transition.

### Performance optimizations land

Two significant performance improvements reached maturity this week. Phil Hord's `git fetch --prune` optimization restructures the logic to use an O(N*logN) approach, reducing runtime from 470 seconds to under 1 second for repositories with 174,000 refs. Taylor Blau's MIDX and cruft pack interaction optimization shows 5-20% speed improvements in GitHub's production environment, though a late test failure was discovered that may delay final merging. Both changes demonstrate Git's continued focus on scaling to large repositories efficiently.

### `imap-send` improvements complete

Aditya Garg's 19-iteration series overhauling `git imap-send` received Junio's approval with all major review feedback addressed. The changes combine critical bugfixes (configuration parsing regressions and memory leaks) with significant new features (OAuth2.0 authentication support and enhanced folder management). This long-running effort makes `imap-send` production-ready while adding modern authentication methods and better UX, marking the culmination of months of development and review.

### New `git repo-info` command takes shape

Lucas Seiki Oshiro's GSoC project introducing `git repo-info` progressed to v2, implementing repository metadata reporting in both JSON and plaintext formats. The series now reports reference storage format (files/reftable), bare repository status, and shallow status, with tests covering both backends. While some architectural questions remain (like `the_repository` usage for `is_bare_repository()`), these are explicitly tracked as future work, positioning this as a promising new command for programmatic repository inspection.

## In brief

**Submodule remote lookup** -- Jacob Keller's series to make submodule remote handling more robust reached its third iteration, addressing memory management and introducing URL-based remote lookup as the primary method.

**Windows terminal handling** -- James Duley submitted a critical fix for Windows terminal handling that addresses an assertion failure when toggling DUPLEX mode during interactive operations like `git add -p`.

**Platform compatibility** -- A regression in `diff --no-index` on AIX and NonStop systems was resolved by combining `d_type` checks with an `lstat()` fallback for platforms without `d_type`.

**Documentation standardization** -- Jean-Noël Avila completed conversion of the `git-log` man page to the new synopsis format style, marking significant progress in documentation modernization.

**Test modernization** -- Rodrigo Michelassi and Isabella Caselli's patch to update `t2400-worktree-add.sh` with modern test helpers was accepted after four iterations.

**Reftable block size** -- Haylin Moore identified performance bottleneck in network clones due to small 4KB reftable writes, with Brian M. Carlson confirming `reftable.blockSize` is configurable.

**Signature handling** -- Christian Couder's work to enhance signature formats in fast-export/import encountered design questions about whether the proposed syntax is necessary versus simply preserving ASCII armor lines.

**Clang-format debate** -- A lively discussion emerged about Git's `.clang-format` configuration, highlighting the tension between mechanical consistency and human judgment in code style.

**Symbolic port handling** -- Maxim Cournoyer implemented `getservbyname` resolution for service names in git-credential-netrc, despite credential protocol documentation specifying numeric ports only.

## Looking ahead

**SHA-256 transition** -- With brian m. carlson's series nearing completion, attention will turn to testing and validation as Git prepares to make SHA-256 the default hash algorithm in version 3.0.

**MIDX/cruft pack optimization** -- Taylor Blau's performance improvements need resolution of a late-discovered test failure before final merging, with the fix likely to dominate early next week.

**Global state reduction** -- Ongoing work to remove `the_repository` dependencies continues, with `is_bare_repository()` specifically mentioned as a target for future efforts.

**GSoC projects** -- Both Lucas Seiki Oshiro's `git repo-info` and Meet Soni's ref consolidation work will see active development through the summer, with regular updates expected.