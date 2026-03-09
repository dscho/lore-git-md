# Git Mailing List Weekly Digest - 2025/05/19 -- 2025/05/25

**The week in brief.** A busy week with 522 emails across 133 threads saw significant progress on multiple fronts as Git prepares for upcoming releases. Key developments include the completion of the promisor-remote protocol enhancements, finalization of batched reference updates for performance gains, and the introduction of stash import/export functionality. The week also featured important security improvements to exec-path handling, ongoing discussions about MPTCP support, and multiple performance optimizations nearing completion. Junio Hamano's "What's cooking" report provided a comprehensive overview of the project's current state, highlighting several series ready for the next release.

## Key developments

### Promisor-remote protocol enhancements finalized

Christian Couder's 5-patch v3 series enhancing the promisor-remote protocol with configurable validation of remote attributes reached completion after extensive review. The changes introduce server-side field advertisement (`promisor.sendFields`) and client-side validation (`promisor.checkFields`) for partial clone filters and authentication tokens. Key improvements in the final version include replacing string_list with direct struct members for better type safety, comprehensive documentation updates, and strict handling of unknown fields. The series has addressed all technical feedback and awaits only Junio's final review before integration.

### Batched reference updates ready for 2.51

Karthik Nayak's performance optimization series introducing batched reference updates to `git-fetch` and `git-receive-pack` has been approved for integration targeting Git 2.51. The v3 iteration shows impressive speedups - 22x faster fetches and 18x faster receive-pack operations for the reftable backend. Junio Hamano gave final approval after all technical issues were resolved, including memory leak fixes and refined error handling. The changes build on commit 23fc8e4f61 and represent a major optimization for operations involving many references.

### Stash import/export reaches final form

The long-running effort to enable stash portability between repositories has culminated in v6 of the series, now approved for merging. The implementation introduces `git stash export` and `git stash import` commands that store stashes as commit chains under `refs/stash-export/`. The final version includes rigorous validation of imported commits, comprehensive test coverage, and addressed all major review concerns. Junio Hamano provided detailed feedback on the import validation logic and memory handling, ultimately giving the implementation his approval pending Phillip Wood's final sign-off.

### Security: Removing exec-path from child process PATH

A security-focused series from Jeff Hostetler addresses potential issues when Git's internal binaries are exposed to non-Git child processes via PATH. The 4-patch series starts with test cleanups, modernizes editor environment handling to use `strvec`, refactors `prep_childenv()` for cross-platform consistency, and finally implements the core change to remove Git's exec-path from PATH for non-Git commands. Junio raised concerns about the blanket approach potentially breaking legitimate cases where child processes need Git binaries, suggesting a more granular control via a new `drop_git_exec_path` flag in `child_process`. This discussion highlights the balance between security and functionality in Git's process spawning.

### MIDX performance optimizations refined

Multiple threads converged on performance improvements for multi-pack-index handling. Jeff King and Junio Hamano engaged in detailed discussion about edge case safety in the negative lookup cache implementation, ultimately confirming the approach correctly handles pack lookups across MIDX chains. A separate series from Phillip Wood addressing integer overflow issues in MIDX repack functionality reached v2 with improved documentation of the mtime-based tie-breaking behavior. These changes collectively improve handling of large repositories, particularly on 32-bit systems.

## In brief

**Build system standardization complete** -- Ramsay Jones's build system standardization series has cleared all validation requirements after successful testing on Linux, Cygwin, and Solaris, aligning path handling between Make and Meson builds.

**Non-standard object type removal progresses** -- Jeff King's 13-patch series removing support for non-standard object types is nearing completion, with only minor documentation questions remaining about the deprecation of `--allow-unknown-type` in `git cat-file`.

**Submodule configuration safety finalized** -- K Jayatheerth's two-patch series addressing submodule configuration safety reached its final form after seven iterations, preventing silent overwrites when reusing paths from moved submodules.

**imap-send gains OAuth2.0 support** -- A two-part series revitalized the neglected `imap-send` command, fixing a configuration parsing bug and adding modern OAuth2.0 authentication support using curl's native API.

**Packed-refs memory handling finalized** -- Jeff King confirmed approval of the final iteration in the packed-refs memory optimization series, which standardizes memory handling across runtime and fsck paths.

**Merge-tree --quiet approved** -- Elijah Newren's `--quiet` flag for merge-tree (providing efficient mergeability checking) has been approved after evolving through multiple naming iterations.

**Pathspec support for diff --no-index** -- Jacob Keller's series adding pathspec filtering to `git diff --no-index` reached advanced stages of review with comprehensive test coverage for various matching scenarios.

**Documentation standardization** -- Jean-Noël Avila's ongoing effort to convert man pages to AsciiDoc saw a follow-up patch updating references from .txt to .adoc extensions.

**Bitmap corruption tests** -- The test infrastructure for detecting memory leaks during corrupt bitmap loading is being reorganized per Jeff King's suggestion to use shell-based corruption via `dd`.

**MPTCP transport support explored** -- Matthieu Baerts outlined integration pathways for Git's various transports (git://, HTTPS, SSH) with the multipath TCP protocol.

## Looking ahead

**Rustification effort** -- Ezekiel Newren's ongoing work to introduce Rust code into Git remains a significant architectural discussion point, particularly around platform support concerns raised by Randall S. Becker.

**ODB abstraction** -- Patrick Steinhardt's object database abstraction work continues in the background, with recent activity around reftable and performance optimizations suggesting more patches may surface soon.

**Breaking changes for Git 3.0** -- Several threads reference planned breaking changes for Git 3.0, including stash behavior modifications and other backward-incompatible improvements, indicating more proposals may emerge as the release approaches.

**Test infrastructure improvements** -- Discussions emerged about making path comparisons more robust across platforms and whether to treat unexpected test passes as failures, suggesting potential test suite modernization work ahead.