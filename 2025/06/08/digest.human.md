# Git Mailing List Digest - 2025/06/08

**The day in brief.** A moderately busy Sunday with 49 emails across 11 threads, featuring significant progress on several fronts. The `imap-send` series reached its 15th iteration with maintainer approval, a `git stash` bugfix nears completion, and Junio introduced a new `--compact-summary` option for merges. Documentation work continued with a major `git-log` man page conversion.

## Notable threads

### `imap-send` feature/bugfix series reaches v15

After 15 iterations and extensive review, Aditya Garg's comprehensive `imap-send` overhaul received maintainer approval from Junio Hamano. The series fixes a configuration parsing regression that broke `imap-send` since Git 2.46.0 while adding OAuth2.0 support (both standard OAUTHBEARER and Google's XOAUTH2), RFC-compliant PLAIN authentication, and folder management via new `--list` and `--folder` options. The finalizes the reorganization of CRAM-MD5 authentication patches as requested by Junio, with only minor documentation nits remaining on the last few patches. This represents a significant improvement to Git's IMAP capabilities after months of development.

### `git stash` branch name corruption fix finalized

K Jayatheerth's fix for `git stash` incorrectly showing submodule branch names in superproject contexts reached its polished form after incorporating feedback from Junio Hamano and René Scharfe. The solution uses `xstrdup()` to preserve the superproject's branch name through submodule operations, with proper memory management and comprehensive test coverage. Junio's final review focused on minor style tweaks (removing redundant else clauses, test verbosity) indicating this is ready for merging. The fix addresses a subtle but important edge case in submodule workflows.

### New `--compact-summary` option for merges

Junio Hamano introduced a new `--compact-summary` option for `git merge` and `git pull` that enhances the standard diffstat output by marking created/deleted files with `(new)`/`(gone)` annotations. The implementation builds on existing diff machinery while providing more context about file operations. The patch includes thorough documentation updates and tests verifying the new output format. This small but useful addition follows Git's philosophy of incremental improvements to user experience.

### Submodule configuration management improvements

K Jayatheerth's submodule configuration series (now at v8) addresses two distinct issues: preventing `.gitmodules` overwrites during submodule addition (patch 1, already approved) and avoiding redundant `.active` config entries (patch 2). The second patch now takes a focused approach within `module_add()` rather than using the suggested `is_submodule_active()` helper, after extensive discussion with Junio. The changes maintain backward compatibility while cleaning up submodule configuration, with thorough test coverage. The series appears ready for final review.

## In brief

**Solaris build configuration updates** -- Brad Smith and Collin Funk collaborated on version-specific function availability checks for Solaris, removing blanket `NO_*` declarations now that modern Solaris versions provide `strtoumax()`, `mkdtemp()`, `memmem()`, and `strcasestr()`.

**`the_repository` removal progress** -- Ayush Chandekar sent a 2-patch series migrating `repository_format_precious_objects` into `struct repository` and removing `the_repository` usage from `builtin/prune.c`, continuing the long-running effort to eliminate globals.

**Submodule memory leak fix** -- A small but important fix addresses a `remote_name` leak in `update_submodule()` when `remote_submodule_branch()` fails, ensuring proper cleanup of allocated strings.

**Bash completion customization** -- A new `completion.forRefSort` configuration option allows customizing ref sort order (e.g., by committer date) in completion contexts like `git checkout <TAB>`.

## On the radar

**Documentation standardization** -- Jean-Noël Avila's 9-part series converting the massive `git-log` man page to the new synopsis format is nearing completion, with all patches now posted for review. This represents one of the largest remaining documentation conversion efforts.