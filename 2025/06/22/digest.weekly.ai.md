# Git Mailing List Weekly Digest - 2025/06/16 -- 2025/06/22

**The week in brief.** A busy week with 276 emails across 87 threads, featuring the Git v2.50.0 release and significant progress on several major initiatives. Key developments include performance optimizations for MIDX/cruft packs and `git fetch --prune`, finalization of the long-running `imap-send` improvements, and SHA-256 transition patches nearing completion. The week also saw active discussions around submodule remote handling, Windows terminal fixes, and the new `git repo-info` command taking shape.

## Key developments

### Git v2.50.0 release

Junio Hamano announced Git v2.50.0, a major feature release with 621 commits from 98 contributors (35 new). User-facing highlights include new `--combine-cruft-below-size` for `git repack`, machine-parsable `rev-list` output, and improved `send-email` Outlook compatibility. Johannes Schindelin followed with Git for Windows 2.50.0, featuring updated dependencies and fixes for ReFS drive compatibility. The release continues reducing `the_repository` usage and optimizing the reftable backend, marking steady progress toward Git 3.0.

### MIDX and cruft pack optimizations

Taylor Blau's 9-part series optimizing MIDX and cruft pack interaction during repacking reached its fifth iteration, showing 5-20% speed improvements in GitHub's production environment. The series introduces configurable control over whether cruft packs should be included in MIDX files via `repack.midxMustContainCruft`. A late test failure discovered by Jeff King when running with specific test environment variables may require additional investigation before merging, but the production-validated optimization appears fundamentally sound.

### SHA-256 default transition progresses

brian m. carlson's 10-part series to make SHA-256 the default hash algorithm when built with `WITH_BREAKING_CHANGES` received thorough review from Junio Hamano. The patches introduce new constants (`GIT_HASH_DEFAULT` and `GIT_HASH_ORIGINAL`) to distinguish between legacy SHA-1 requirements and the new default while maintaining backward compatibility. With comprehensive test coverage verified and naming questions resolved (settling on `GIT_HASH_LEGACY_SHA1`), this represents a major step toward Git 3.0's planned SHA-256 transition.

### IMAP-send improvements finalized

Aditya Garg's 19-iteration series overhauling `git imap-send` received Junio's approval with all major review feedback addressed. The changes combine critical bugfixes (configuration parsing regressions and memory leaks) with significant new features (OAuth2.0 authentication support and enhanced folder management). The final version improves string handling safety by using `xstrfmt()` and `strbuf` for credential encoding, making `imap-send` production-ready while adding modern authentication methods.

### Submodule remote handling refactored

Jacob Keller and Glen Choo's series to make submodule remote handling more robust reached its third iteration, with Junio Hamano indicating he will apply it. The series addresses several pain points in submodule configuration handling, including memory management improvements in branch and remote struct teardown, replacement of `repo_get_default_remote()` with a more robust `repo_default_remote()`, and addition of URL-based remote lookup as the primary method. The v3 version notably removes the `branch->merge_name` field entirely.

## In brief

**Performance boost for `git fetch --prune`** -- Phil Hord's optimization reduces runtime from 470 seconds to under 1 second when pruning 15,000 refs from a 174,000-ref repository by restructuring the logic to use an O(N*logN) approach.

**Platform compatibility fixes** -- A regression in `diff --no-index` on AIX and NonStop systems was resolved by combining direct `d_type` checks with an `lstat()` fallback for platforms without `d_type`.

**New `git repo-info` command** -- Lucas Seiki Oshiro's GSoC project introduces repository metadata reporting in both JSON and plaintext formats, now implementing key=value output (e.g., `layout.bare=true`) after review feedback.

**Windows terminal handling** -- James Duley's critical Windows terminal fix addresses assertion failures during interactive operations by changing `restore_term()` to check handle validity directly.

**Documentation standardization** -- Jean-Noël Avila completed conversion of the `git-log` man page to the new synopsis format style, with Junio praising the clearer enumerated list format for `--decorate` options.

**Reftable block size discussion** -- Haylin Moore identified performance bottlenecks in network clones due to small 4KB reftable writes, with Brian M. Carlson noting `reftable.blockSize` is configurable but cautioning about read performance tradeoffs.

**Batched reference updates** -- Karthik Nayak clarified the transactional limitations in his merged series, confirming the current solution maintains pre-batched failure semantics while tracking true transactional behavior as future work.

**Clang-format debate** -- Junio Hamano and Christian Couder discussed style enforcement strategies, revealing philosophical differences about whether to prioritize mechanical consistency or human judgment in code formatting.

**Symbolic port handling** -- Maxim Cournoyer's v2 series implements `getservbyname` resolution for service names in git-credential-netrc, despite credential protocol documentation specifying numeric ports only.

## Looking ahead

**SHA-256 transition** -- With brian m. carlson's series nearing completion, attention will turn to testing and documentation as Git moves toward making SHA-256 the default in version 3.0.

**MIDX/cruft pack optimizations** -- Taylor Blau's series appears ready pending resolution of the test failure discovered late in the week, which may require additional iteration.

**Global state reduction** -- Ongoing work to remove `the_repository` dependencies continues, with `is_bare_repository()` specifically mentioned as a target for future efforts.

**Rustification effort** -- While quiet this week, Ezekiel Newren's work to introduce Rust code into Git remains a contentious topic, particularly regarding platform support concerns raised by Randall S. Becker.