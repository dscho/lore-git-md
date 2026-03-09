# Git Mailing List Weekly Digest - 2025/12/15 -- 2025/12/21

**The week in brief.** A busy week with 473 emails across 135 threads, featuring major progress on multiple fronts. The standout developments include the long-awaited completion of Windows symlink support, security fixes for ANSI escape sequences, and significant architectural work on the ODB abstraction. Other notable threads covered the hook subsystem refactoring, submodule gitdir path encoding, and the new `git history` command proposal. The week saw several large patch sets reach maturity after extensive review cycles.

## Key developments

### Windows symlink support finalized

After nearly a decade of development, Johannes Schindelin and Karsten Blees' comprehensive Windows symlink support series (18 patches) received final approval. The implementation provides POSIX-compatible symlink operations through direct Win32 API calls, including proper `stat()`/`lstat()` behavior and directory symlink support. While defaulting to `core.symlinks=false` for safety, the changes enable proper symlink handling when Developer Mode is enabled, resolving long-standing compatibility issues. The series includes thorough handling of edge cases like phantom symlinks and MAX_PATH size reporting, with Johannes Sixt providing final review notes on documentation and attribution.

### ANSI escape sequence security fixes

Johannes Schindelin's v2 series addressed CVE-2024-32002 and CVE-2024-52005 with a layered defense against ANSI escape sequence injection. The implementation introduces a `sideband.allowControlCharacters` configuration with three states (default/color/full) and comprehensive sanitization in `strbuf_add_sanitized()`. The discussion evolved into a broader security philosophy debate, with Junio Hamano suggesting URL-specific trust settings while Johannes maintained strict defaults. The refined version accommodates legitimate color output in hooks while protecting against dangerous sequences, with test coverage verifying both security enforcement and configuration flexibility.

### ODB abstraction progresses

Patrick Steinhardt's 10-part series moved packfile storage tracking from the object database (ODB) level to individual ODB sources, completing key architectural groundwork for pluggable object databases. The changes ensure each source (main repo, alternates, etc.) manages its own packfiles independently while maintaining existing behavior. Justin Tobler provided positive feedback on the early patches, noting alignment with the project's direction. A related 7-part series from Patrick improved object info handling in preparation for future `odb_for_each_object()` functionality, showing 9% speedup in size-only request benchmarks.

### Hook subsystem refactoring completed

Emily Shaffer and Ævar Arnfjörð Bjarmason's v5 series converted all major Git hooks to use the new `hook.h` API, standardizing execution with structured stdin handling and output capture. The 11-part implementation maintains backward compatibility while adding capabilities for parallel execution. Notable refinements included reverting unnecessary `REF_STATUS_REJECT_*` additions and adding BUG() assertions for NULL output buffers. The series addressed all substantive review feedback, with only documentation remaining for follow-up work. Adrian Ratiu later addressed final stylistic nits on the receive hook conversion, marking completion of this multi-year effort.

### Submodule gitdir path encoding ready

Josh Steadmon's v7 series implementing `extensions.submodulePathConfig` reached final approval after addressing filesystem conflicts and enabling custom gitdir layouts through runtime configuration. The comprehensive solution includes a four-tier fallback system (plain -> encoded -> numbered -> hashed), case-folding protection, atomic migration, and new conflict detection. Junio Hamano's final ack confirmed readiness for merging after seven iterations of review and refinement. The changes particularly benefit case-insensitive systems while maintaining compatibility with existing submodule configurations.

## In brief

**`git repo structure` reporting** -- Justin Tobler's series adding comprehensive object size metrics to the experimental command was merged after addressing i18n refinements, providing human-readable and machine-readable output formats.

**`git replay` improvements** -- Multiple patches enhanced the experimental command's stability, including René Scharfe's segfault fix for invalid `--onto` targets and Phillip Wood's handling of empty commits.

**Documentation standardization** -- Jean-Noël Avila and Kristoffer Haugsbakk coordinated man page improvements, converting several to consistent synopsis formatting and AsciiDoc tables while clarifying `git reset` behavior.

**HTTP 429 rate limiting** -- A v2 series added comprehensive support for rate limit retries in Git's HTTP client, featuring non-blocking delays and trace2 instrumentation.

**Performance regression fixed** -- Junio Hamano and Patrick Steinhardt addressed a severe slowdown (1508s to 0.38s) in promisor pack handling by removing problematic static variable usage.

**MacOS iconv workaround** -- A 3-patch RFC series proposed fixes for ISO-2022-JP conversion issues in macOS 14/15, introducing an `ICONV_RESTART_RESET` flag.

**Custom remote branch templates** -- A new `checkout.remoteBranchTemplate` feature allows printf-style customization of remote branch name resolution during DWIM operations.

**PID file support for lockfiles** -- New debugging creates companion `.pid.lock` files showing which process holds each lock, configurable per component.

**French translation update** -- Jean-Noël Avila brought the French `.po` file up to date with latest source strings.

**Reftable compaction fix** -- Patrick Steinhardt corrected an edge case that could silently drop refs when two tables share a deletion tombstone.

## Looking ahead

**`git history` command** -- Patrick Steinhardt plans to address functionality for rewriting dependent branches in January following ongoing discussion about branch selection strategy. The series includes significant refactoring of the add-patch subsystem.

**ODB abstraction** -- Patrick Steinhardt's foundational work continues with more changes expected to enable pluggable backends, building on this week's packfile storage refactoring.

**Rust build system** -- D. Ben Knoble's portable sed fix for macOS/BSD systems should land soon, resolving compilation issues in the Rust support code.

**Documentation guidelines** -- Jean-Noël Avila's proposal about formalizing imperative-mood-first documentation guidelines may see discussion in the new year.