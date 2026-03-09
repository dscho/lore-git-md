# Git Mailing List Digest - December 9, 2025

**The day in brief.** December 9 saw active discussion across multiple fronts with 73 emails spanning 20 threads. The day was dominated by technical refinements to ongoing efforts - particularly Taylor Blau's MIDX compaction series and Patrick Steinhardt's ODB alternates refactoring - alongside significant debate about `git history`'s multi-branch commit handling. Junio's "What's cooking" report provided a comprehensive snapshot of the project's current state.

## Notable threads

### `git history` multi-branch commit handling debate intensifies

The philosophical divide over how `git history` should handle commits appearing in multiple branches deepened with contributions from Martin von Zweigbergk and Kristoffer Haugsbakk. Both strongly supported Elijah Newren's position that the command should rewrite all affected branches by default, aligning with Jujutsu's behavior. Patrick Steinhardt acknowledged the workflow differences but remains concerned about implementation efficiency, particularly for repositories with hundreds of thousands of references. The discussion revealed fundamental tension between Git's traditional branch-centric model and Jujutsu-inspired approaches that treat commits more independently.

### MIDX compaction series nears completion

Taylor Blau's 17-patch MIDX compaction series received extensive review from Patrick Steinhardt, with discussions covering:
- Checksum function naming conventions (`_hex` vs `_raw` suffixes)
- Backward compatibility considerations with libgit2
- Bitmap handling during compaction
- API design for MIDX writing operations

The series appears technically sound but requires a version bump for the relaxed pack ordering change after confirming libgit2's strict requirements. Review feedback focused on final polishing touches around edge case handling and test coverage.

### ODB alternates refactoring progresses

Patrick Steinhardt's systematic refactoring of alternates handling continued with constructive review from Justin Tobler. The series moves toward source-based operations while preserving existing functionality, with patches 5-8 addressing:
- Elimination of mutual recursion between alternates processing functions
- Clearer separation of parsing versus adding alternates
- Interface design for future pluggable backends

Justin raised important architectural questions about how alternates are represented in `struct object_database`, suggesting the current single-list approach may need refinement for write operations.

### macOS iconv workaround discussion

Multiple contributors investigated macOS 15's regression in stateful encoding conversion (ISO-2022-JP), proposing three solution approaches:
1. Code-based workarounds forcing full-string reconversion (René Scharfe)
2. Build-system detection preferring Homebrew's libiconv (René Scharfe)
3. Platform-specific state reset handling (Torsten Bögershausen)

The discussion narrowed to implementation details around state management and platform detection, with consensus that this is a macOS-specific issue requiring a workaround.

## In brief

**HTTP authentication follow-up** -- Ashlesh Gawande proposed a modified approach to 403 response handling that preserves credentials while allowing prompting when none exist, addressing brian m. carlson's earlier concerns.

**HTTP 429 review feedback** -- Taylor Blau suggested several improvements to the rate limiting series, focusing on error message translation, function organization, and test structure.

**Windows symlink test prep finalized** -- Junio queued Johannes Schindelen's v2 series preparing Git's test suite for Windows symlink support after positive reviews from Patrick Steinhardt.

**Promisor object optimization** -- A v3 series introduced performance improvements for `--exclude-promisor-objects` by skipping hash verification when processing promisor objects.

**Shallow fetch depth fix** -- Samo Pogačnik addressed relative-depth fetching issues in shallow repositories by unifying absolute and relative depth handling.

**`repo info --keys` format support** -- A v2 series added format compatibility (`-z`/`--format`) to the new `--keys` option for `git repo info`.

**SSH client visibility proposal** -- Ryan Johnson suggested showing which SSH binary Git uses on authentication failures to help diagnose configuration issues.

## On the radar

**`--staged` vs `--cached` terminology** -- Junio questioned whether standardizing on `--staged` is preferable to simply removing the "obsolete" label from `--cached`, shifting the discussion's focus.

**Ref location and batched tag fixes** -- Karthik Nayak updated status on two ref-related series, with one needing review responses and the other awaiting a minor typo fix.

**Repository structure metrics** -- A 6-patch series extended `git repo structure` with detailed object size reporting (both inflated and on-disk) and improved output formatting.