# Git Mailing List Digest — 2025/11/22

## The day in brief

A quiet Saturday with 9 emails across 6 threads, featuring performance optimizations in the diff machinery, final refinements to tempfile handling, and a bugfix for shallow clones. The most notable developments were René Scharfe's benchmark proving Git's tempfile implementation outperforms system calls, and Junio Hamano's detailed review of a shallow clone edge case.

## Notable threads

### Tempfile implementation benchmarks settle debate

René Scharfe provided conclusive benchmark data showing Git's `git_mkstemp_mode()` consistently outperforms system `mkstemp(3)` calls, particularly when setting non-default permissions (24% faster). His enhanced `test-mktemp` tool demonstrated 5-24% performance gains across various scenarios, settling the technical debate about which implementation to standardize on. The only remaining question is stylistic — whether to replace the last `mkstemp()` caller directly or keep the macro indirection.

### Shallow clone border commit fix needs polish

A patch fixing edge cases in `--shallow-since` border commit handling received thorough review from Junio Hamano, who approved the technical approach but requested better commit message context and test coverage. The fix addresses subtle bugs when parent-child relationships exist in the shallow commit list, but needs to document both the broken behavior and solution more clearly before merging.

### ODB streaming interface naming refined

Late in Patrick Steinhardt's 19-part object database refactoring series, Junio suggested renaming the streaming interface functions to follow the familiar `open/read/close` pattern while maintaining ODB-centric terminology (`odb_read_stream_open` etc.). This naming tweak would polish an otherwise complete effort that has successfully eliminated `the_repository` usage and improved type safety.

## In brief

**Diff performance optimization** — René Scharfe introduced `diff_same()` to skip unnecessary processing of unchanged files during `--find-copies-harder`, yielding a 1.33x speedup in Linux repository benchmarks.

**Config file handling clarification** — Delilah Ashley Wu confirmed her upcoming v2 will precisely document when the global config inconsistency occurs (only with both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` present).

**Gerrit submodule limitation** — Alon Bar-Lev identified a workflow pain point where submodules reject Gerrit's `refs/changes/` references, questioning if the branch requirement could be relaxed for checkouts.