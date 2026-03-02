# Git Mailing List Digest - 2025/04/21

**The day in brief.** A moderately busy Monday with 31 emails across 7 threads, dominated by build system improvements and platform compatibility work. The most notable developments include resolution of CI issues for Meson header checking, Windows/ARM64 support patches, and macOS maintenance scheduler fixes. Several refactoring efforts also progressed, with const-correctness improvements and Meson build system standardization moving forward.

## Notable threads

### Meson header checking CI issues resolved

The long-running effort to implement header checking in Meson builds reached its final stages today as contributors diagnosed and fixed a CI-specific regression. Phillip Wood identified the root cause: GitHub Actions was extracting tarballs rather than performing Git checkouts when Git wasn't installed, breaking `git ls-files` operations. Karthik Nayak followed up with a fix that explicitly installs Git before checkout, using platform-appropriate package managers. 

This was the last blocking issue for the otherwise-complete series, which has undergone extensive review to align Meson's header checking with Makefile capabilities. Junio Hamano noted the interesting divergence from Makefile behavior (which falls back to `find` when Git isn't available) but didn't object to the Meson approach. The solution maintains the design choice to rely on Git while documenting the CI-specific workaround.

### Windows/ARM64 support series

Dennis Ameling and Johannes Schindelin submitted a comprehensive 6-patch series enabling Windows/ARM64 support, particularly targeting GitHub Actions' new ARM64 runners. The changes include:

- ARM64 bswap optimizations using compiler built-ins
- Build system adjustments for clangarm64 toolchain
- Disabling nedmalloc on ARM64 due to compatibility issues
- MSVC build support
- System config file location fixes
- Stack depth safety adjustments for clang/ARM64

Junio quickly acknowledged the "cleanly structured" series, suggesting it will likely progress smoothly. The work builds on Git for Windows' existing ARM64 support and addresses real-world needs as ARM64 ecosystem support expands.

### macOS maintenance scheduler fixes

Josh Heinrichs corrected long-standing bugs in Git's macOS maintenance scheduler that caused jobs to run at incorrect frequencies. The original implementation used undocumented `Day` keys in launchctl plists, resulting in daily jobs running only the first six days of each month and weekly jobs running daily. The fix replaces these with properly documented `Weekday` keys, aligning with launchd.plist(5) specifications.

Junio reviewed and approved the change, noting some platform inconsistencies in Sunday handling (day 0 vs 7) but considering those out of scope for this macOS-specific fix. The patch is narrowly focused and addresses clear behavioral problems without introducing new complexity.

## In brief

**Const-correctness refactoring** -- Ahelenia Ziemiańska standardized usage string declarations as `const` across 15 files in builtins and test helpers, improving consistency with Git's existing patterns. Junio confirmed the technical correctness of the changes after a minor compiler error was fixed.

**Meson build system standardization** -- A 6-patch series began refactoring function availability tests in the Meson build system to use a centralized array-based approach, addressing networking dependencies, Solaris portability quirks, and Windows-specific cases. The changes reduce duplication while maintaining existing behavior.

**Sparse-checkout documentation fix** -- A minor patch standardized list numbering in sparse-checkout technical docs from mixed "(1)" and "(b)" formats to consistent numeric numbering.

## On the radar

**`git stash`/`cherry-pick` recovery UX** -- Following last week's discussion about expected behavior when these commands interact, Markus Raab suggested potential usability improvements for recovery scenarios, including new commands and better output hints. The core behavior remains accepted, but tooling enhancements may emerge.