# Git Mailing List Digest - 2025/12/17

**The day in brief.** A busy Wednesday with 79 emails across 16 threads, dominated by major platform work on Windows symlink support and security fixes for ANSI escape sequences. Key developments include Johannes Schindelin's v2 security series addressing CVE-2024-32002/CVE-2024-52005, the completion of Justin Tobler's object size reporting series, and the submission of comprehensive Windows symlink support patches from Karsten Blees and Johannes Schindelin.

## Notable threads

### Windows symlink Support Finalized

Johannes Schindelin and Karsten Blees submitted an 18-part series implementing comprehensive symlink support for Windows, culminating nearly a decade of development in Git for Windows. The patches provide POSIX-compatible symlink operations through direct Win32 API calls rather than CRT functions, including proper `stat()`/`lstat()` behavior, directory symlink support, and non-elevated creation when Developer Mode is enabled. The series defaults to `core.symlinks=false` for safety but includes thorough handling of edge cases like phantom symlinks and MAX_PATH size reporting. With maintainer sign-off already in place from earlier preparatory work, this appears ready for integration.

### ANSI Escape Sequence Security Fixes

Johannes Schindelin's v2 series addresses ANSI escape sequence injection vulnerabilities (CVE-2024-32002, CVE-2024-52005) with a layered defense approach. The implementation introduces a `sideband.allowControlCharacters` configuration with three states (default/color/full) and comprehensive sanitization in `strbuf_add_sanitized()`. The series has evolved through review to accommodate legitimate color output in hooks while maintaining protections against more dangerous sequences. Test coverage in `t5409-colorize-remote-messages.sh` verifies both security enforcement and configuration flexibility. The changes focus on HTTPS channels, explicitly acknowledging SSH's limitations for this protection.

### Object Size Reporting Complete

Justin Tobler's object metrics series reached its final iteration (v5) after addressing i18n refinements and documentation feedback. The implementation now provides comprehensive size reporting (counts, inflated sizes, disk usage) across all `git repo structure` output formats, with human-readable units in table view and precise numbers in machine formats. The careful architectural evolution through preparatory refactoring, utility function additions, and incremental metric integration demonstrates Git's disciplined approach to feature development. Patrick Steinhardt and Junio Hamano's reviews confirm the series is ready for merging.

## In brief

**Submodule gitdir path encoding** -- Adrian Ratiu addresses a configuration handling bug where `extensions.submodulePathConfig` wasn't properly creating required gitdir entries during `--recurse-submodules` clones, planning fixes in v7.

**Pre-push hook status filtering** -- Junio Hamano aligns with Patrick Steinhardt's position against optimizing status filtering in the hook refactoring series, preserving backward compatibility where hooks may depend on seeing attempted-but-rejected ref updates.

**PID file support for lockfiles** -- A new debugging feature creates companion `.pid.lock` files showing which process holds each lock, configurable per component (index, refs, etc.) via `core.lockfilePid`.

**Promisor object optimization** -- Aaron Plattner's two-patch series optimizing promisor pack handling by skipping unnecessary blob parsing graduated to 'next' after thorough review from Jeff King and Junio Hamano.

**Sign-off policy documentation** -- Final tweaks to documentation clarifying Git's stance against automatic sign-offs (beyond grandfathered `format.signoff`) complete, with Johannes Sixt's suggested phrasing improvements accepted.

**Windows test suite fixes** -- Johannes Schindelin's v4 series preparing tests for Windows symlink support is now fully merged, addressing long-standing issues like the 2009 Perl test path handling problem.

## On the radar

**Performance regression in object streaming** -- Aaron Plattner reported a severe slowdown (0.38s to 1508s) in repositories with large promisor packs, traced to a logic error in the recently merged `ps/object-read-stream` series' `packfile_store_read_object_info()`. This may warrant a follow-up fix.