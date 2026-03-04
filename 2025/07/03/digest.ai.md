# Git Mailing List Digest - 2025/07/03

**The day in brief.** A busy Thursday with 69 emails across 21 threads, dominated by technical refinements to major ongoing efforts - reftable as default ref storage, `git repo-info` command implementation, and Meson build system improvements. Notable highlights include Jeff King's fix for a `--diff-filter` regression and Patrick Steinhardt's v2 series for the reftable transition.

## Notable threads

### Reftable as default ref storage advances

Patrick Steinhardt's v2 series to make reftable the default ref storage format in Git 3.0 saw significant discussion today. The updated proposal now includes:

1. A versioned default macro (`REF_STORAGE_FORMAT_DEFAULT`) in repository.h
2. Integration with `git version --build-options` to report ref format
3. Experimental flag support (`feature.experimental=true`) to enable early testing

The architectural discussion centered on where to set the default format, with consensus forming around `repository_format_configure()` in setup.c rather than the `REPOSITORY_FORMAT_INIT` macro. Documentation improvements emphasize platform-specific benefits like case-insensitivity handling and Unicode normalization on macOS.

### `git repo-info` command under review

Lucas Seiki Oshiro's new `git repo-info` command for repository metadata inspection received detailed feedback rounds from Patrick Steinhardt. Reviews covered:

- Command naming (considering `git repo info` namespace)
- JSON output format implementation
- Test organization and coverage
- `the_repository` global variable usage in bare repo detection

The thread shows careful attention to code quality and interface design, with particular focus on making the command's field reporting system maintainable through array-based approaches rather than nested switches.

### `--diff-filter` regression fixed

Jeff King identified and fixed a regression where `git log --diff-filter=d` (excluding deletions) failed with minimal output formats like `%H`. The issue traced to a 2022 change that split negative filter bits into a separate field without updating revision setup code. The one-line fix in `revision.c` restores the expected behavior where exclusion filters work regardless of output format.

## In brief

**FreeBSD version support** -- Renato Botelho confirmed FreeBSD 13.5 (EOL April 2026) as the oldest supported version, finalizing the thread's decision to drop pre-6.0 support.

**Build system roadmap** -- Patrick Steinhardt clarified Meson remains experimental but targeted for completion this cycle, with autotools likely to be removed after Meson reaches feature parity in future releases.

**Blame ignoreRevsFile fix** -- Francesco Nicoletta Puzzillo proposed making `git blame` gracefully handle missing ignoreRevsFile when configured globally, rather than erroring out.

**SHA-256 git-gui support** -- Version 2 of a patch adding dynamic hash length handling to git-gui components, replacing hardcoded SHA-1 assumptions.

**X-Change-ID headers** -- Initial implementation for preserving change IDs through email workflows, with git-am support planned for v2 after positive Jujutsu compatibility feedback.

**Meson build improvements** -- Patrick Steinhardt's 8-patch series addressed Python detection, summary output, shell path handling, and CI test slicing.

## On the radar

**Remote name ambiguity** -- A new bug report highlights how slashes in remote names can create refname collisions, potentially requiring validation changes in `valid_remote_name()`.

**Submodule path handling** -- K Jayatheerth's June 7th series on renamed submodule handling remains in review limbo, with a polite nudge sent today.