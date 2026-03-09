Here's the Git mailing list daily digest for January 9, 2026:

**The day in brief.** A busy Friday with 149 emails across 25 threads, dominated by ongoing work on Windows symlink support, ref backend verification, and `the_repository` removal. Key developments include the completion of the Windows symlink series, Patrick Steinhardt's ref verification improvements, and René Scharfe's latest tree parsing conversions in the `the_repository` removal effort.

**Notable threads**

**Windows symlink support finalized**  
Johannes Schindelin and Karsten Blees have completed their 18-patch series implementing comprehensive POSIX-like symlink support on Windows. The v5 series addresses all technical and administrative issues, including proper handling of Karsten's email attribution and final refinements to symlink creation, reading, and error handling. The changes enable non-elevated symlink creation on Windows 10+ and fix long-standing issues with symlink size reporting that affected interoperability with Cygwin/WSL. This represents the culmination of work that began in Git for Windows in 2015.

**Ref verification improvements**  
Patrick Steinhardt sent a 17-patch series that significantly improves ref consistency checks across Git's files and reftable backends. The changes introduce shared validation infrastructure between backends, move fsck checks into the refs subsystem, and add missing verification for root refs like HEAD. The series systematically eliminates backend-specific duplication while maintaining thorough validation, with particular attention to worktree handling in the reftable backend. The changes are well-contained and build on Patrick's extensive ref backend expertise.

**`the_repository` removal continues**  
René Scharfe progressed the long-running effort to eliminate Git's reliance on the global `the_repository` variable with a 10-patch series converting tree parsing functions. The changes introduce repository-aware variants of `parse_tree*()` functions and methodically update 41 call sites across builtins and plumbing commands. The series follows the established pattern of first adding new API variants before converting callers, with the final patch providing a Coccinelle semantic patch to automate future conversions. This continues the multi-year architectural effort to make Git's subsystems properly repository-aware.

**In brief**  

**ANSI escape sequence security** -- Patrick Steinhardt reviewed Johannes Schindelin's patches for granular control of ANSI sequences in sideband channels, confirming the technical approach while suggesting minor documentation improvements.

**git-history command** -- Patrick Steinhardt's v9 series introducing the experimental `git history` command with `reword` subcommand is in final refinements, addressing D. Ben Knoble's feedback on error handling and documentation.

**HTTP authentication fix** -- A bugfix addresses probe_rpc() authentication failures during filtered clones, ensuring Bearer tokens are properly included in subsequent blob fetches after initial authentication succeeds.

**Shallow repository fixes** -- Samo Pogačnik's v2 series fixes memory leaks and relative-depth fetching issues in shallow repositories, with Patrick Steinhardt confirming the leak fix is validated by CI test failures.

**Documentation improvements** -- Kristoffer Haugsbakk's series standardizing `git-patch-id` documentation terminology and clarifying option interactions is ready for merging after positive reviews.

**Test modernization** -- K Jayatheerth updated t7101-reset-empty-subdirs.sh to use modern test helpers like `test_path_is_file` instead of direct `test` operations.

**On the radar**  

**Geometric repacking with promisor remotes** -- Patrick Steinhardt's series enabling geometric repacking with promisor packs is under review, with Taylor Blau providing initial feedback on the pack-objects flag handling.

**Rustification effort** -- Ezekiel Newren's work to introduce Rust code into Git continues in the background, though no new patches appeared today. The discussion around NULL pointer safety in hooks highlighted ongoing tensions around language safety.