# Git Mailing List Digest — 2025/02/10

**The day in brief.** A moderately busy Monday with 38 emails across 19 threads, featuring steady progress on several fronts: the `the_repository` removal effort saw multiple patches advance, Junio shared a comprehensive "What's cooking" status update, and several new features for `git clean` and `git rebase` were proposed. The most notable technical discussion centered on cross-compilation build system issues and a bug report about NFSv4 packfile handling.

## Notable threads

### Cross-compilation build system fixes

Patrick Steinhardt and Peter Seiderer continued their discussion about fixing Git's Meson build system for cross-compilation scenarios where host and target shell paths differ. The proposed solution involves using Meson's `native:` parameter to properly separate build-time and target-time tool paths. Junio Hamano raised the broader question of whether similar fixes should be made to the make-based build system, though the discussion leaned toward focusing on Meson given the lack of historical demand for make-based cross-compilation support. The thread highlights the technical challenges of maintaining multiple build systems while addressing real-world platform compatibility issues.

### NFSv4 packfile permissions issue

Bryan Maloney reported a bug where Git's temporary packfile handling causes EACCES errors during NFSv4 server failovers, due to creating files with O_RDWR flags but 0444 permissions. Brian Carlson responded that this behavior is actually POSIX-compliant and suggested the issue may need to be addressed on the NFSv4 side rather than in Git. The discussion reveals an interesting edge case in distributed filesystem behavior that affects Git's assumptions about file permissions during operations like cloning.

### `git clean` exclusion enhancements

A three-patch series from D. Ben Knoble introduced new configuration and command-line options for `git clean` to better handle excluded files. The changes add a `clean.exclude` config option (similar to the existing `-e` flag) and a `--remove-excluded` flag to override all exclusions when needed. The series includes preparatory refactoring of pattern matching logic and careful documentation of the new features' interaction with existing options. This provides more granular control over which files are protected from deletion, particularly useful for developer-specific files like IDE configurations.

### Rebase's `--update-refs` gets interactive mode

Another feature enhancement came from Kristoffer Haugsbakk, who proposed an `interactive` mode for `git rebase --update-refs`. This allows the feature to be automatically enabled for interactive rebases while remaining disabled for non-interactive ones, addressing a common pain point where config settings caused unexpected ref updates in automated contexts. The patch includes thorough documentation updates and careful handling of option precedence between config and command-line settings. Reviewers suggested minor documentation polish and noted the need for test coverage before finalizing the change.

### MIDX verification progress reporting

Ayush Chandekar proposed adding progress reporting for the QSORT operation during multi-pack-index verification, addressing a TODO in the code. However, Junio Hamano raised significant concerns about the technical approach, noting that tracking progress via qsort() comparisons makes unreliable assumptions about sort order. This may require a redesign of the progress tracking mechanism to provide accurate feedback during this potentially slow operation.

## In brief

The `the_repository` removal effort saw progress with Usman Akinyemi's patch converting `builtin/update-server-info.c`, now awaiting final commit message polish per Junio's feedback. Patrick Steinhardt's reftable decoupling series gained Windows build system improvements, with Johannes Sixt noting the need for Git-for-Windows maintainer coordination. A subtree bugfix addressed confusing error messages for invalid HEAD cases, though Junio suggested refinements to the handling of unborn HEAD states. Tomáš Trnka defended his promisor pack handling patch against technical concerns with real-world performance data. The hash algorithm refactoring series received positive reviews from Taylor Blau and appears ready for merging. A Thunderbird patch processing script fix was finalized with added POSIX compliance.

## On the radar

Junio's "What's cooking" email provides a comprehensive snapshot of the project's current state, with several significant features in development including Rust language bindings, bulk blob downloading via `git backfill`, and improvements to shallow cloning. The ongoing GSoC 2025 planning thread confirmed mentor assignments, with project selection pending student applications. The editor error message discussion during rebase operations continues to explore the tension between generic error handling and command-specific needs.