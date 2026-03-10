# Git Mailing List Digest - 2026/01/29

**The day in brief.** A moderately busy Thursday with 55 emails across 17 threads, featuring final refinements to several significant features (`--maximal-only` for rev-list, git-gui tab alignment), ongoing architectural discussions around ODB abstraction, and new proposals for default branch references. Junio's "What's cooking" report provides a comprehensive snapshot of in-flight topics.

## Notable threads

### **Final approval for rev-list's --maximal-only**

Derrick Stolee's `--maximal-only` option for `git rev-list` has reached full consensus after Junio Hamano confirmed the long-term direction. This feature selects "frontier" commits (maximal in reachability order) while supporting negative revisions - a capability distinguishing it from the existing `--independent` flag. Junio explicitly endorsed the name and suggested this should eventually replace `--independent` across Git commands. The implementation uses bit 28 (`CHILD_VISITED` flag) with comprehensive test coverage and is now ready for merging.

### **Git-gui tab alignment fix concludes**

After nearly two decades of inconsistency between git-gui and gitk's tab display on Windows, a solution has been finalized. The patch uses Tk's native `apply_tab_size` configuration with different tabstop offsets (1 for regular diffs, 2 for combined diffs) to compensate for diff markers while preserving staging functionality. Maintainer Johannes Sixt confirmed git-gui only needs to handle 3-way diffs (not octopus merges), resolving the last open question. Junio provided final commit message guidance, marking this routine bugfix ready for merging.

### **ODB transaction handling refactoring**

Justin Tobler's series to refactor object database transaction handling for pluggable backends saw extensive architectural discussion. Consensus emerged on making temporary object directory management an internal implementation detail of the files backend rather than exposing it as a separate API. Junio strongly agreed that current `tmp_objdir_create()` users (remerge-diffs, receive-pack, and ODB transactions) should use the ODB transaction abstraction directly. The series appears to be rerolled with this improved architecture before merging.

### **String-list API improvements progress**

Amisha Chhajed's RFC series introducing `string_list_sort_u` (a combined sort+deduplicate utility) added comprehensive test coverage for seven previously untested string-list functions. The tests follow established patterns and verify edge cases and memory management. While the core functionality remains uncontroversial, edge cases in fetch.c (remote consultation order) and help.c (key processing) require resolution before the series can progress from RFC status.

## In brief

**Histogram diff edge case fix** -- Yee Cheng Chin's patch addressing redundant matching lines in histogram diff output has addressed all technical feedback and awaits final documentation polish before merging.

**Repository config migration** -- Olamide Caleb Bello's `the_repository` removal work saw detailed review from Junio, validating the technical approach while keeping broader initialization architecture questions open.

**GSoC 2026 project scoping** -- Christian Couder proposed eight potential projects including `the_repository` removal continuation and `git cat-file` improvements, with Lucas Seiki Oshiro offering expertise on `git repo` subcommands.

**Windows linker conflict resolution** -- brian m. carlson identified the root cause of CI failures in their SHA-1/SHA-256 interoperability work (competing `link.exe` implementations) and is preparing a fix.

**Path normalization refactoring** -- Pushkar Singh's attempt to split `normalize_path_copy_len()` into helpers received mixed feedback, with Junio finding only the slash-skipping helper clearly beneficial change.

## On the radar

**Default branch shorthand proposal** -- Harald Nordgren's `@{default}` syntax for referring to a repository's default branch faces skepticism from Junio about its real-world utility and edge case handling.

**Bitmap tip selection documentation** -- Taylor Blau and Junio are refining explanations of `pack.preferBitmapTips` behavior regarding window-based selection and hierarchical ref matching.