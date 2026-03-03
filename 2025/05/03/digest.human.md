Here's the daily digest for May 3, 2025:

## The day in brief

A moderately busy day with 38 emails across 12 threads, featuring progress on several long-running efforts including `whatchanged` deprecation, BSD platform compatibility fixes, and documentation standardization. The most notable developments include a finalized deprecation plan for `git whatchanged` and a proposed solution for BSD errno quirks in symlink handling.

## Notable threads

### `git whatchanged` deprecation finalized

Junio C Hamano posted a comprehensive 6-patch series that completes the deprecation process for `git whatchanged`. The series formalizes Git's deprecation infrastructure by generalizing the warning mechanism from `git pack-redundant`, systematically removes `whatchanged` references from documentation and tests, and introduces a `--i-still-use-this` opt-in requirement. The final patches add conditional compilation via `WITH_BREAKING_CHANGES` and document the planned removal in BreakingChanges.adoc. This establishes a template for future command deprecations while taking a cautious approach - first gathering usage data via forced opt-in before full removal.

### BSD errno handling in symlink operations

An extensive discussion developed around platform-specific errno behavior when handling symlinks with O_NOFOLLOW. Collin Funk initially provided patches to handle NetBSD's EFTYPE and FreeBSD's EMLINK errno values, but Jeff King proposed a more portable solution using `lstat()` in `packed-backend.c` instead of accommodating platform errno differences. The thread reached consensus on this approach, which avoids platform-specific code while maintaining the same functionality. A secondary discussion emerged about performance optimizations for large packed-refs files during fsck operations, with Jeff noting mmap could be more efficient than strbuf for repositories with gigabyte-sized packed-refs files.

### `git apply --intent-to-add` index corruption

Raymond Pasco provided detailed analysis of a longstanding bug where `git apply -N` incorrectly clobbers the index. While confirming Johannes Altmanninger's 2021 patch had correct code despite its confusing message, Pasco identified an additional edge case where the command incorrectly attempts to apply entire diffs when `ita_only` is enabled. The thread now has both a reproduction case and precise technical understanding but needs an updated patch addressing both the original index corruption and this newly found partial-diff issue.

## In brief

**Wildcard pathspec handling fix** -- A revised patch addressing edge cases in `git add` and `git commit` when handling literal filenames containing wildcard characters, now with better test structure following maintainer feedback.

**Windows Meson CI improvements** -- Johannes Schindeler's patch to force Release mode rather than Debug mode builds in GitHub Actions Windows CI, avoiding modal assertion failures that hang test jobs.

**Documentation standardization** -- Junio C Hamano continued the synopsis-style conversion effort with patches updating `verify-commit`, `verify-tag`, `verify-pack`, `git var`, `git write-tree`, and `git-daemon` documentation to use consistent backquoting and formatting.

**Remote group management proposal** -- A first-time contributor proposed adding a `git remote group` subcommand for listing configured remote groups, with initial implementation and bash completion support but open questions about rename behavior in local configs.

**Character escapes in .gitignore** -- A feature request proposing to add escape sequence support (like `\r`) in `.gitignore` patterns to better handle non-printable characters in filenames, motivated by macOS Icon files with carriage returns.

**Merge commit visualization** -- Britton Kerin explored adding 3-pane Meld view support for reviewing already-committed merges, similar to conflict resolution tools but for post-merge review.

## On the radar

**Platform compatibility** -- The BSD errno discussion may spawn follow-up patches as the lstat()-based solution is implemented, and the packed-refs performance optimization for large repositories remains an open question.

**Documentation standardization** -- Jean-Noël Avila's ongoing effort to convert all man pages to consistent synopsis-style formatting continues producing steady documentation patches.