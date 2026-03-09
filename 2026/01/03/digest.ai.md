# Git Mailing List Digest - 2026/01/03

## The day in brief  

A moderately busy day with 19 emails across 6 threads, dominated by the conclusion of a long-running `git status` push tracking feature (now at v14) and several test/bugfix discussions. The push tracking series appears ready for merging after addressing final refspec edge cases, while Windows-specific test flakiness and `git status -z` path formatting quirks prompted deeper technical discussions.

## Notable threads  

### `git status` push tracking reaches final form  

Harald Nordgren's long-running effort to show push tracking branch divergence in `git status` output has reached its final iteration (v14) after extensive review. The series now properly handles custom refspec transformations through a full resolution chain that maps local branches through push refspecs to determine destinations, then back through fetch refspecs to find tracking branches. Phillip Wood's thorough review identified and helped resolve this key technical gap. The implementation shows both upstream and push tracking comparisons when they differ, with output format matching Git conventions. Junio Hamano had previously signed off on the technical direction, and with 210 lines of new test coverage, this appears ready for merging pending Phillip's final sign-off on the refspec implementation.

### Windows test flakiness reveals deeper stat behavior issues  

A racy test in the difftool suite (`t7800-difftool.sh`) prompted discussion about Windows filesystem behavior when detecting same-size file modifications. The immediate fix (changing test file sizes from 12 to 17 bytes) was straightforward and already queued, but Phillip Wood and Paul Tarjan's follow-up analysis revealed deeper technical nuances about how Git's stat-based change tracking interacts with Windows' lack of inodes. While the test-specific workaround is uncontroversial, the discussion suggests there may be broader implications for scripted use cases where file sizes match - an area that might warrant future investigation.

### `git status -z` path formatting inconsistencies  

Artur Pyrogovskyi reported and Jeff King analyzed an inconsistency where `git status --porcelain=2 -z` shows repository-root-relative paths while the non-NUL-terminated version respects `status.relativePaths`. Peff identified two distinct bugs: `--short -z` ignoring the prefix entirely, and porcelain v2 with `-z` unexpectedly switching to absolute paths. The discussion now awaits input from status.c experts to clarify intended behavior, as the issues span historical quirks versus documented behavior, with implications for both implementation and documentation.

## In brief  

**Reftable iterator fix** -- Pushkar Singh reviewed Tsahi Elkayam's fix for undefined behavior in the indexed table reference iterator, properly validating `value_type` before accessing union members.

**Format-patch merge commit warnings** -- A refined RFC now warns about skipped merge commits only when no patches are generated, addressing Junio Hamano and Jeff King's feedback about noise reduction and NULL-deref safety.

**Gitignore encoding improvements** -- A new patch addresses encoding issues in `.gitignore` handling, particularly improving support for UTF-16LE files with BOMs commonly created by Windows PowerShell, with comprehensive test coverage.