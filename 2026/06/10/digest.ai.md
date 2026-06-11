# Git Mailing List Digest - 2026/06/10 (Wednesday)  
**Total emails:** 125  
**Active threads:** 32  

## The day in brief  
A busy Wednesday with significant progress on multiple fronts: Patrick Steinhardt's `git history drop` series reached v4 with reset API refinements, Kristoffer Haugsbakk completed comprehensive trailer documentation, and Junio merged the promisor remote auto-configuration series. Documentation polish dominated the traffic, with multiple series finalizing man page improvements.

## Notable threads  

### **Trailer documentation finalized**  
Kristoffer Haugsbakk's 11-patch series documenting `git-interpret-trailers` reached completion, comprehensively covering trailer key format restrictions (ASCII alphanumeric + hyphen only) that had caused earlier confusion. The series standardizes terminology ("metadata", "trailer block"), removes outdated RFC 822 references, and adds concrete examples of both valid usage and rejection cases. Junio provided final wording polish on trailer block definitions before merging. The work originated from a bug report about silent failures when parsing keys containing periods, now properly documented as intentional behavior.

### **Promisor remote auto-configuration merged**  
Junio merged Christian Couder's URL-based auto-configuration series implementing `promisor.acceptFromServerUrl` after resolving final review discussions with Toon Claes. The feature provides strict URL pattern matching (scheme/port/host boundaries) with auto-generated remote names and comprehensive test coverage in t5710. This completes a parallel effort with Patrick Steinhardt's LOP series, giving organizations flexible control over promisor remote setup while maintaining security boundaries.

### **Reset API modernization for `git history drop`**  
Patrick Steinhardt's v4 series adding a `git history drop` subcommand refined the reset machinery's API, renaming `reset_head()` to `reset_working_tree()` and introducing explicit HEAD update control via `RESET_WORKING_TREE_UPDATE_HEAD`. Phillip Wood confirmed the mechanical changes look correct after verifying all call sites, suggesting only a `BUG()` check for invalid flag combinations. The 10-patch series includes thorough test coverage (537 lines across 4 scripts) for the new drop functionality, which prevents root/merge commit removal and supports dry-run conflict detection.

### **`the_repository` removal advances**  
Two parallel efforts progressed: Tian Yuchen's migration of `protect_hfs`/`protect_ntfs` to repository-specific storage received maintainer approval after addressing Christian Couder's v1 feedback, while Patrick Steinhardt's 7-patch series eliminating `git_work_tree_cfg` and `is_bare_repository_cfg` globals from setup.c underwent thorough review from Justin Tobler. The changes follow the established pattern of first making callers pass repository state explicitly before removing global infrastructure.

## In brief  

**`git replay --linearize` design finalized** -- Toon Claes' series adding merge commit linearization completed design discussion, with Junio confirming the approach to treat root commits differently from merges. The implementation maps merge commits to first parents while maintaining test coverage for edge cases.

**Ref-filter performance regression fixed** -- Tamir Duberstein's v3 patch restored prefix-scoped iteration in ref-filter, addressing a 25x slowdown (2.8s->0.11s) when listing refs with prefixes in repos containing many loose refs, implementing Karthik Nayak's suggested approach.

**`git status` optimization ready** -- Tamir's v3 patch limiting `git describe`'s ref iteration to tags by default demonstrated 17x speedups (171.7ms->9.9ms) in repos with many non-tag refs, now with packed refs in perf tests and concise benchmark reporting.

**Build system silent mode fixed** -- Harald Nordgren's git-gui/Makefile patch addressing macOS linker warnings by restructuring $(LIBS) to maintain correct ordering while eliminating duplicates cleared review and was queued by Johannes Sixt.

**Documentation syntax fixes** -- Tuomas Ahola's v2 series corrected AsciiDoc markup in config documentation, terminating runaway lists and escaping octothorpes per Junio's feedback, completing Jean-Noël Avila's standardization effort.

## On the radar  

**AI contribution policy** -- Junio proposed documenting Git's stance on AI-generated content in SubmittingPatches, reserving the right to reject patches with insufficient human oversight, following discussion sparked by Tamir's `git describe` optimization.

**Reword behavior design** -- The `git history reword` thread converged on modeling committer correction after `git commit --reset-author`, with Junio suggesting a two-phase approach for timestamp handling that may flip defaults at a major version bump.