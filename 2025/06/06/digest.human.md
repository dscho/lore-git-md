# Git Mailing List Digest - 2025/06/06

**The day in brief.** A moderately busy Friday with 66 emails across 19 threads saw continued refinement of several major patch series, including OAuth2.0 support for `imap-send` and batched reference updates. Notable developments include the resolution of a critical build system regression and productive discussions around Change-ID standardization and submodule remote handling.

## Notable threads

**OAuth2.0 support for imap-send reaches v14** -- Aditya Garg's long-running series to restore broken functionality and add OAuth2.0 authentication to `imap-send` reached its 14th iteration. The latest version introduces a `try_auth_method()` helper to reduce code duplication in authentication handling and adds clarifying comments about libcurl's password handling for OAuth methods. The series has grown to include new user-facing features like command-line folder specification (`--folder`) and folder listing (`--list`), while also fixing a critical configuration parsing bug that made `imap-send` unusable since Git 2.46.0. Junio Hamano pushed for structural improvements to make the authentication code more maintainable, leading to productive discussions about code organization principles.

**Batched reference updates refined** -- Karthik Nayak's series addressing edge cases in batched reference updates progressed to v3, with two patches that fix production issues found when running Git's `next` branch in GitLab's Gitaly service. The first patch fixes a segfault in the files backend by properly skipping failed updates during batched operations, while the second restructures `git receive-pack` to handle reference deletions separately from other updates to avoid filesystem directory/file conflicts. The implementation now uses an enum for clearer phase separation and only creates transactions when needed, addressing the author's own review feedback from v2.

**Change-ID determinism debated** -- The ongoing discussion about standardizing Change-IDs in commit footers explored the tension between deterministic generation and distributed system constraints. Toon Claes and Junio Hamano discussed whether enforcing global Change-ID uniqueness is feasible given Git's distributed nature, with Junio noting that determinism might be more valuable for newly created commits than for maintaining uniqueness across distributed histories. The thread also saw agreement on adding a config option to disable Change-ID generation for privacy-conscious users, balancing utility with user control over metadata.

**Submodule remote handling challenges** -- Jacob Keller's investigation into submodule behavior with non-standard remote names uncovered deeper architectural questions about making deprecated functions in `remote.c` properly repository-aware before their planned removal in Git 3.0. Junio Hamano endorsed the direction of modifying `read_remotes_file` and `read_branches_file` to take repository pointers, noting this aligns with both immediate submodule functionality needs and broader architectural evolution. The discussion highlighted how fixing these code paths would serve both current use cases and future removal plans.

**Critical build system regression fixed** -- Johannes Sixt provided the definitive fix for a critical regression where `generate-tclindex.sh` was incorrectly deleting source TCL files during NO_TCLTK builds. The root cause was a misinterpretation of `$@` in the shell script context introduced in commit 2cc5b0facfa4. Randall S. Becker confirmed successful builds on NonStop systems after applying the fix, which changes the problematic `rm` command to explicitly target the correct file rather than expanding all arguments. This resolves an issue that had made git-gui unusable in certain build configurations since Git 2.46.0.

## In brief

**Stash option parsing improvements** -- Phillip Wood's series fixing `git stash -p` behavior and allowing more flexible option ordering received positive reviews from Martin Ågren, who suggested enhancing test coverage to better verify the interactive patch selection functionality.

**Memory leak in show --merge** -- Lidong Yan proposed an improved test approach for the recently fixed memory leak in `prepare_show_merge()`, creating actual merge conflicts through branch divergence rather than artificial index manipulation as Junio had initially suggested.

**cURL type safety fixes completed** -- Johannes Schindelin's 4-part series addressing type mismatch warnings in cURL` calls was finalized, combining Jeff King's initial Debian-focused fixes with comprehensive coverage across platforms. The changes ensure all 23 `curl_easy_setopt()` calls in the codebase properly pass `long` values where expected.

**Documentation rendering fix** -- A minor but important documentation patch fixed Asciidoctor rendering in the git-diff manpage by removing spurious backticks around ".." in a "mode" example that were causing malformed output.

**Byte-order handling refactored** -- Sebastian Siewior reorganized macro definitions in `compat/bswap.h` to properly handle byte-order conversions on big-endian systems, fixing test failures on s390x that occurred after commit 6547d1c9cb added built-in bswap support.

## On the radar

**Stash import/export feature** -- Brian Carlson confirmed an upcoming v8 of their patch series introducing new `git stash` subcommands for importing and exporting stash entries via refs, which is expected to be the final iteration before potential inclusion.

**Rebase conflict resolution tool** -- An RFC proposal for `rebase--`, a new tool that aims to improve conflict resolution during rebases by leveraging information from original merge commits, was introduced as an external Python implementation that shows promise for certain conflict scenarios.