# Git Mailing List Digest - 2025/05/03

**The day in brief.** A moderately busy Saturday with 38 emails across 12 threads, featuring progress on several technical fronts. The most notable developments include a finalized deprecation plan for `git whatchanged`, platform-specific errno handling fixes for BSD systems, and a new feature proposal for remote group management. Documentation standardization work also saw multiple updates.

## Notable threads

### `git whatchanged` deprecation finalized

Junio C Hamano has posted a 6-patch series that formalizes the deprecation process for `git whatchanged`, establishing infrastructure that may serve as a template for future command removals. The series progresses through several logical phases: generalizing the deprecation warning infrastructure from `git pack-redundant`, removing `whatchanged` references from documentation and tests, requiring explicit `--i-still-use-this` opt-in, and preparing for eventual removal with conditional compilation. The approach suggests this may become Git's standard template for command removal, with `git whatchanged` serving as the pilot case. The series maintains the measured approach discussed earlier - first gathering usage data via forced opt-in before full removal.

### BSD errno handling in symlink operations

An extended discussion has developed around platform-specific errno behavior in symlink operations, particularly on BSD systems. The thread initially focused on fixing test failures in `t0602-reffiles-fsck.sh` where NetBSD returns EFTYPE and FreeBSD returns EMLINK instead of the POSIX-standard ELOOP when O_NOFOLLOW encounters a symlink. After initial patches proposed BSD-specific errno translation in `wrapper.c`, Jeff King suggested a more portable solution using `lstat()` directly in `packed-backend.c`. The thread has now settled on this approach, which avoids platform-specific code while maintaining the same functionality. A secondary discussion emerged about performance optimizations for large `packed-refs` files during fsck operations.

### New `git remote group` subcommand proposal

A first-time contributor has proposed adding a `git remote group` subcommand for managing remote groups, with an initial implementation focusing on listing functionality. The patch adds bash completion support and tests, while leaving open questions about how remote renames should interact with group memberships in local config files. The implementation appears well-structured with thorough test coverage, though the thread awaits maintainer guidance on some design questions before proceeding further. This represents the beginning of what could become a more comprehensive remote group management feature set.

## In brief

**Wildcard pathspec handling fix** -- A revised patch addressing edge cases in wildcard pathspec handling for `git add` and `git commit` now includes improved test structure and commit message clarity following maintainer feedback.

**Windows Meson CI improvements** -- Johannes Schindeler submitted a patch to force Release mode rather than Debug mode builds in GitHub Actions Windows CI, avoiding modal assertion failures that can hang test jobs.

**`git apply --intent-to-add` bug analysis** -- Raymond Pasco provided detailed technical analysis of an index corruption issue, confirming the viability of a 2021 patch while identifying additional edge cases that need addressing.

**Documentation standardization** -- Junio C Hamano continued his documentation standardization series with patches updating `verify-commit`, `verify-tag`, `verify-pack`, `git var`, `git write-tree`, and `git-daemon` man pages to consistent synopsis-style formatting.

**Minor documentation fixes** -- Kristoffer Haugsbakk submitted a two-patch series correcting minor formatting issues in `git-reflog.adoc` and `git-branch.adoc`, fixing HTML rendering problems and a typo in verbatim formatting.

**`.gitignore` escape sequences proposal** -- A feature request was made to add character escape support in `.gitignore` patterns, motivated by difficulties excluding macOS Icon files containing carriage return characters.

**Merge visualization proposal** -- Britton Kerin explored adding 3-pane Meld view support for reviewing already-committed merges, similar to what `git mergetool` offers during conflict resolution.