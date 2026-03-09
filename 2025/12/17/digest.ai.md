Here's the daily digest for December 17, 2025:

## The day in brief

December 17 saw significant activity with 79 emails across 16 threads, dominated by major platform compatibility work for Windows symlink support and important security fixes. The standout developments include Johannes Schindelin's v2 security series addressing ANSI escape sequence vulnerabilities (CVE-2024-32002, CVE-2024-52005) and the completion of a comprehensive 18-part Windows symlink support series that's been nearly a decade in the making.

## Notable threads

### **Windows symlink support reaches maturity**

After years of development in Git for Windows, Johannes Schindelin and Karsten Blees submitted an 18-part series that brings comprehensive symlink support to the main Git codebase. The changes touch core filesystem operations in `compat/mingw.c`, implementing proper `stat()`/`lstat()` behavior, directory symlink support, and accurate size reporting. Notably, the series defaults `core.symlinks=false` on Windows due to platform limitations, while enabling full support when explicitly configured. The implementation includes careful error handling and maintains POSIX compatibility, addressing long-standing issues like incorrect MAX_PATH size reporting that caused spurious index updates. With maintainer approval, this foundational work is now poised for integration.

### **Security fixes for ANSI escape sequence vulnerabilities**

Johannes Schindelin's v2 patch series addresses critical vulnerabilities (CVE-2024-32002, CVE-2024-52005) involving ANSI escape sequence injection in Git's sideband channel. The implementation introduces a three-state `sideband.allowControlCharacters` configuration (defaulting to `color` mode) with careful sanitization in `handle_ansi_color_sequence()`. The series has evolved through review to accommodate legitimate color output in hooks while maintaining protections against more dangerous sequences. Test coverage in `t5409-colorize-remote-messages.sh` verifies the behavior, though SSH channel limitations remain acknowledged but unaddressed in this series.

### **Object size reporting nears completion**

Justin Tobler's series adding comprehensive object size metrics to `git repo structure` reached its fifth iteration, now focusing on final i18n refinements. The implementation provides both inflated and disk sizes across all output formats (table, keyvalue, nul), with human-readable formatting for the table view. The v5 changes address minor documentation and spacing issues, signaling the series is ready for merging after positive reviews from Patrick Steinhardt and Junio Hamano. This represents a significant enhancement to repository introspection capabilities.

### **Submodule path encoding adjustments continue**

Adrian Ratiu responded to feedback on his submodule gitdir path encoding series, acknowledging two issues in the global config handling of `extensions.submodulePathConfig`. The fixes planned for v7 will ensure proper error throwing when required configs are missing and add comprehensive test coverage for clone scenarios. This work is part of making submodule paths config-driven while preventing inconsistent repository states.

## In brief

**PID file support for lockfiles** -- A new `core.lockfilePid` config creates companion `.pid.lock` files showing which process holds each lock, aiding in debugging stale locks.

**Promisor object optimization cleared** -- Aaron Plattner's two-patch series optimizing promisor pack handling by skipping unnecessary blob parsing was promoted to 'next' after thorough review.

**Sign-off policy documentation finalized** -- After extensive discussion, the project settled on clear documentation explaining Git's stance against automatic sign-offs (except the grandfathered `format.signoff`), with only minor phrasing tweaks remaining.

**Windows test suite preparations** -- Johannes Schindelin's v4 series addressing test failures for upcoming MSYS2 symlink support was accepted, fixing long-standing issues like the 2009 Perl test problem in t9700.

**Performance regression reported** -- Aaron Plattner identified a severe slowdown (0.38s to 1508s) in repositories with large promisor packs, traced to a logic error in the recently merged object streaming series.

## On the radar

**Hook API modernization** -- Adrian Ratiu's hook subsystem changes remain pending Junio's architectural review and Josh Steadmon's subsystem expert review.

**Rust interoperability** -- brian m. carlson's SHA-1/SHA-256 interop work continues as a long-running effort, noted in Junio's "What's cooking" but not generating new traffic today.

**Documentation style conversion** -- Jean-Noël Avila's man page improvements saw follow-up discussion about splitting stylistic changes from technical fixes, part of the ongoing synopsis-style conversion effort.