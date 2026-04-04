# Git Mailing List Digest - 2026/04/03

**The day in brief.** A moderately busy Friday with 59 emails across 21 threads saw significant progress on several fronts. Key developments include Patrick Steinhardt's in-memory ODB backend series nearing completion, Rust becoming enabled by default in the build system, and ongoing refinements to graph rendering and test infrastructure. The day also featured routine translation coordination and Git for Windows release preparations.

## Notable threads

### **In-memory ODB backend takes shape**

Patrick Steinhardt's 16-patch series introducing an in-memory object database source reached completion today, with Junio C Hamano providing initial feedback. This architectural change allows temporary storage of objects purely in memory without persistence, serving as a building block for features like git-blame's temporary worktree commits and potential future re-merge operations. The series methodically implements each ODB callback while cleaning up technical debt, with particular attention to efficient storage via oidtree. Junio's review focused on clarifying the intended use case for truly transient objects while speculating about potential applications like octopus merges.

### **Graph rendering improvements progress**

The thread on improving `git log --graph` output for root commits and excluded parents saw productive discussion today. Junio C Hamano clarified that the issue extends beyond true root commits to any commit with parents outside the displayed range, prompting author Pablo Sabater to adjust his "seems_root" concept accordingly. A test case portability issue (case-sensitive filenames) was identified and will be fixed in v2. The discussion revealed subtle tradeoffs in graph layout clarity versus strict commit relationship accuracy, with the maintainer reinforcing the project's preference for minimal visual changes over new notation.

### **Rust enabled by default**

brian m. carlson's 4-patch series to enable Rust support by default in Git 2.54 was approved by Junio today. The change implements a long-planned transition that missed its original Git 2.53 target, with build system adjustments for Alpine Linux compatibility and proper linking against libdl on older Linux distributions. The series is technically straightforward but marks an important milestone in the contentious Rustification effort, maintaining backward compatibility via CI jobs that still build without Rust.

### **Submodule fetch errors become configurable**

A new 2-patch series introduces configurable handling of submodule fetch failures, allowing users to choose between the current "fail" behavior and a new "warn" mode that continues despite submodule issues. The change addresses real-world pain point when working with upstream branches that reference unpublished submodule commits. The implementation builds on a refactoring that fixes pre-existing early error recording, then adds thorough test coverage for both config and command-line option variants.

## In brief

**Promisor-remote test refinements** -- Christian Couder finalized his 10-patch series with URI encoding improvements and clearer variable naming in test scripts, addressing Junio's feedback about handling special characters in paths.

**Worktree path matching** -- Chen Linxuan's v3 series adding `includeIf` conditions for worktree paths appears ready after refactoring path-matching logic and adding thorough test coverage for the new `worktree` and `worktree/i` patterns.

**Config file locking** -- Jörg Thalheim's patch to add 100ms retries for config file lock contention was accepted, preventing silent failures in operations like `git worktree add -b` when writing tracking information.

**Windows build updates** -- Johannes Schindelin contributed several platform-specific patches, including UCRT64 compatibility for strftime() and CMake support for writev() wrappers, as part of Git for Windows' ongoing modernization.

**Translation coordination** -- Jiang Xin kicked off the localization effort for Git 2.54.0, noting 182 strings need updates and providing updated workflow instructions that now explicitly endorse AI-assisted translation with human review.

**Git for Windows RC** -- Johannes Schindelin announced Git for Windows 2.54.0-rc0, which drops `git svn` from the main distribution due to maintenance challenges while updating bundled components like Bash and OpenSSH.

## On the radar

**`the_repository` removal effort** -- Shreyansh Paliwal's refactoring series narrowed to focus on lock timeouts and hash algorithm usage while deferring more complex branch helper conversions, with Tian Yuchen noting a remaining static variable that could be addressed in future work.

**Test modernization for bare repos** -- Johannes Schindelin's series preparing tests for potential `safe.bareRepository=explicit` behavior saw productive discussion about handling `ext::` URLs, with Junio agreeing the proposed `--git-dir=.` solution is technically sound while raising higher-level questions about consensus on the behavior change.