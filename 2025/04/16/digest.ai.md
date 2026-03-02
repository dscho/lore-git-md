Here's the Git mailing list daily digest for April 16, 2025:

### The day in brief
A moderately busy day with 86 emails across 17 threads, featuring significant progress on build system modernization, integer handling safety, and documentation updates. Key developments include Ramsay Jones' build system alignment series reaching v3, Patrick Steinhardt's parse-options hardening completing review, and the conclusion of the `--no-hooks` debate in favor of documentation improvements.

### Notable threads

**Build system alignment reaches v3**  
Ramsay Jones' extensive build system modernization series progressed to v3 with 13 patches focusing on Makefile/meson consistency and Cygwin compatibility. The changes include Cygwin-specific additions like enabling `clock_gettime` and `sysinfo()`, CSPRNG standardization to `getrandom` on Linux, and removal of obsolete platform compatibility code. The series shows careful attention to cross-platform behavior while reducing legacy support burden, with Junio Hamano and other maintainers aligned on the technical approach.

**Integer handling safety complete**  
Patrick Steinhardt's v3 series hardening integer handling in parse-options infrastructure addressed all review feedback, adding comprehensive `errno` checks, proper unsigned type usage, and bounded integer options. The 7-patch series now provides compile-time and runtime validation for all numeric option types while maintaining backward compatibility. Junio Hamano approved the technical approach though questioned whether bounded options were truly necessary given no current callers.

**Perl removal series finalized**  
Eric Wong's series making Perl optional for core functionality reached v2 with all components reviewed. The changes convert `git-filter-branch`, `git-request-pull`, and documentation build scripts from Perl to shell, following earlier test infrastructure conversions. Patrick Steinhardt contributed the command list generation conversion, which shows a 7x performance penalty deemed acceptable for documentation builds. Only gitweb and send-email remain as Perl-dependent components.

**Wildcard pathspec investigation deepens**  
Analysis of wildcard pathspec handling inconsistencies expanded to compare behavior across multiple commands (`add`, `commit`, `ls-files`, `grep`). Junio Hamano provided historical context about `git add`'s special case handling of paths not yet in the index, suggesting the current behavior may stem from deliberate early choices. The thread is now examining implementation differences in how commands populate and process pathspec structures.

**Hook disabling concludes with docs**  
Derrick Stolee abandoned the `--no-hooks` proposal after maintainer pushback, instead documenting the existing `core.hooksPath=/dev/null` workaround. Junio Hamano had rejected the feature on design principle grounds, arguing workflow problems should be solved through architectural improvements rather than new bypass options. The final patch adds clear documentation about the `/dev/null` approach while recommending command-specific rather than global use.

### In brief

**Fedora CI awk workaround** -- Johannes Schindelin implemented a robust fix for Fedora 42 CI failures by explicitly installing gawk, following analysis of package management differences between Fedora and AlmaLinux.

**Perforce test improvements** -- Anthony Wang's test reliability patch for Perforce integration reached v7 with all review feedback addressed, replacing fragile text matching with direct ref verification.

**MyFirstContribution updates** -- K Jayatheerth submitted a 4-patch series modernizing the contributor tutorial to demonstrate current practices like repository-aware config access and proper function signatures.

**Azure DevOps cloning issue** -- A Windows-specific regression report identified SSL-related cloning problems with Git 2.44.0+ when accessing Azure DevOps, with guidance provided to file a Git-for-Windows issue.

**Object store refactoring complete** -- Patrick Steinhardt's object storage reorganization received final approvals, with directory utilities moved to path.c and virtual objects made repository-specific.

### On the radar

**Patch tracking debate** -- The discussion about Change-IDs versus Patch Set IDs continues exploring fundamental questions about how patch evolution should relate to Git's object model, with Junio Hamano suggesting a more comprehensive solution might track derivation history.

**MIDX/cruft optimization** -- Taylor Blau's performance series for configurable cruft pack handling in MIDX awaits final wording polish on the commit message before integration, with all technical aspects approved.