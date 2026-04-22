Here's the Git mailing list digest for April 21, 2026:

**The day in brief.** A busy Tuesday with 89 emails across 23 threads, dominated by major technical series nearing completion. Key developments include Taylor Blau's incremental MIDX repacking reaching final form, Patrick Steinhardt's `set -e` test suite changes ready for merge, and substantive discussions about source tree reorganization and credential security. Several bugfix series also reached resolution today.

**Notable threads**

**Incremental MIDX repacking finalized**  
Taylor Blau's 16-patch series implementing geometric MIDX chain repacking for large repositories is now complete. The final version adds an append-only mode when used without `--geometric`, providing flexibility between simple growth and sophisticated compaction. The series has undergone thorough review with today's exchanges finalizing edge cases around configuration validation and test organization. This represents a major architectural improvement to Git's scalability, particularly for monorepos that repack frequently.

**Test suite `set -e` compatibility ready**  
Patrick Steinhardt's effort to make Git's test suite compatible with `set -e` (errexit) has concluded after six iterations. The final version enables strict error checking by default for Bash 5+, ensuring test failures surface immediately rather than being silently ignored. Today's discussion resolved the last remaining question about defensive coding in test helpers, with Jeff King confirming the simpler approach was correct. The changes significantly improve test reliability by catching setup/teardown errors and test declaration typos.

**Source tree reorganization planning**  
The discussion about moving libgit.a components into a "lib/" directory progressed from design to implementation concerns. Patrick Steinhardt and Derrick Stolee agreed on an incremental approach, focusing first on the library move while leaving command implementations in place for now. The thread addressed practical transition challenges for downstream forks and in-flight patches, with preliminary approval from Git for Windows maintainer Johannes Schindelen. The change aims to reduce root directory clutter while maintaining build system compatibility.

**Credential helper security issue**  
A security bug report revealed the `git-credential-libsecret` helper leaks credentials to stdout during store operations. The discussion clarified interface boundaries between public and internal implementation, with consensus forming that only `get` operations should output credentials. While a minimal patch would address the immediate issue, the thread raised broader questions about credential helper protocols that may warrant documentation updates. Other credential helpers may need similar audits for stdout leakage.

**In brief**

**`git stash` argument parsing** -- Phillip Wood reviewed the final version of Deveshi Dwivedi's patch to improve `git stash` argument parsing, suggesting minor implementation refinements around error messages and memory handling.

**Pseudo-merge bitmap fixes** -- Taylor Blau submitted v2 of his pseudo-merge bitmap bugfix series, addressing all identified issues including memory leaks in pattern redefinition and incorrect commit date parsing.

**Test modernization** -- Siddharth Shrimali's series to modernize tests in t7004-tag.sh received final approval from Patrick Steinhardt after implementing Junio's "belt-and-suspenders" verification approach.

**Merge-ort hardening** -- Elijah Newren completed a 5-patch series hardening merge-ort against corrupt trees with duplicate entries and improving directory/file conflict detection.

**Git-gui bare repo handling** -- A bugfix improves git-gui's handling of bare repositories and missing worktrees, replacing cryptic Tcl errors with helpful warnings and repository selection dialogs.

**On the radar**

**Git Merge 2026** -- Initial announcement for the September conference in Lisbon, with Contributor's Summit and Jujutsu (jj) events planned alongside the main program. Community input requested on session topics.