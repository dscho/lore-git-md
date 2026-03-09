# Git Mailing List Digest - 2025/11/29

## The day in brief

A moderately busy Saturday with 40 emails across 14 threads, featuring Junio Hamano's weekly "What's cooking" status update and continued progress on several significant efforts. Key developments include Johannes Schindelin's 10-part Windows symlink compatibility series, documentation improvements for `scalar` configuration, and resolution of a tricky `git-last-modified` bitmap initialization bug.

## Notable threads

### Windows test suite symlink compatibility

Johannes Schindelin submitted a comprehensive 10-patch series preparing Git's test suite for MSYS2's upcoming default enablement of symlink support on Windows. The changes address various test failures that would occur when symlinks become fully functional, including:

- Perl test path handling fixes (t9700)
- `git apply` executable bit regression (t4115)
- Windows-specific `open()` behavior with dangling symlinks
- Test expectations for symlink resolution differences
- Platform-specific handling of `/dev/null` symlinks

The series demonstrates careful attention to maintaining test semantics while accommodating Windows' path handling quirks. With MSYS2 planning to enable symlink support by default, these preemptive fixes will help ensure Git's test suite remains reliable across platforms.

### Scalar configuration documentation

Matthew Hughes and Derrick Stolee continued work on improving `scalar` configuration discoverability with a two-patch series. The first adds inline comments in `scalar.c` to clarify why specific config values are set, while the second documents the previously undocumented `gui.GCWarning` option. 

The discussion revealed parallel work on GitHub (PR #2010) had already addressed some of these concerns, prompting the authors to refocus their submission. This highlights the ongoing coordination challenges between mailing list and GitHub-based contributions.

### `git-last-modified` bitmap initialization fix

A bug in `git-last-modified`'s bitmap handling, originally reported by Anders Kaseorg on s390x, reached resolution with Junio Hamano approving Toon Claes' fix for backport to maint. The issue involved incorrect `memset()` sizing that only partially cleared bitmap memory. 

The thread discussion explored broader code hygiene considerations around type-safe memory operations but settled on the minimal fix for immediate backport. Jeff King-verified solution properly initializes the scratch bitmap by accounting for `eword_t` size in the `memset()` call.

## In brief

**`git replay` documentation fixes** -- Junio Hamano corrected inaccuracies in the git-replay manual regarding revision range syntax, clarifying it accepts only a single range like most Git commands.

**`git check-ignore` exit code discussion** -- David Goldstein provided additional context about discovering the exit code discrepancy while debugging `ripgrep` behavior, reinforcing that this is a low-priority issue given the command's debugging nature.

**CI test output visibility** -- Johannes Schindelin confirmed Junio's changes to both commit message wording and technical implementation (`chmod o+w` to `a+w`) for Docker CI test output permissions.

**`git diff --max-depth` path handling bug** -- Han Jiang reported a bug where the command fails to produce output when paths end with slashes or are just '.', with reproduction cases showing the issue is consistent across depth values.

**ARM64 submodule fetch SIGSEGV** -- Jeff King identified the commit (d70f554cdf) that fixed an ARM64-specific crash during submodule fetches with commit-graph writing enabled, closing the investigation.

## On the radar

**Hook subsystem refactoring** -- Adrian Ratiu's stdin callback mechanism for hooks (part 2/10 of the series) received sign-offs and only needs minor documentation fixes before merging, though email formatting issues were noted.

**`git replay --revert` revision ranges** -- Junio Hamano clarified that disconnected commit sets would require new syntax beyond current `prepare_revision_walk()` capabilities, setting boundaries for future enhancements.