# Git Mailing List Digest — 2025/11/22

## The day in brief

A quiet Saturday with 9 emails across 6 threads, mostly focused on performance optimizations and bugfix refinements. The most notable developments include René Scharfe's tempfile benchmark results confirming Git's implementation outperforms system `mkstemp(3)`, and a bugfix for `--shallow-since` edge cases that drew detailed review feedback from Junio Hamano.

## Notable threads

### Tempfile refactoring concludes with benchmark data

René Scharfe provided the final piece of evidence in the tempfile consolidation effort, enhancing the `test-mktemp` tool to compare `git_mkstemp_mode()` against system implementations. Benchmarks show Git's version is 5-24% faster, particularly when setting non-default permissions. The discussion has now narrowed to whether to replace `mkstemp(3)` calls via macro or ban them outright — a stylistic decision on an otherwise settled technical improvement.

### Shallow clone border commit fix needs polish

A patch addressing descendant relationship handling in `--shallow-since` clones received thorough review from Junio Hamano, who approved the technical approach but requested better commit message structure and test coverage. The fix correctly handles cases where border commits include both parents and children, but needs to document the impact more clearly and prove its correctness with regression tests.

### ODB streaming interface naming refined

Patrick Steinhardt's 19-part object database streaming refactor reached its final naming discussion, with Junio suggesting `odb_read_stream_*` variants to maintain consistency with file I/O APIs while improving clarity over the original "istream" terminology. The series has already achieved its major goals of removing `the_repository` usage and improving encapsulation.

## In brief

**Config file handling clarification** — Delilah Ashley Wu confirmed an edge case in her global config inconsistency fix where the issue only manifests when both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` exist, addressing feedback from Kristoffer Haugsbakk and Junio Hamano.

**Diff performance optimization** — René Scharfe followed up on the recent `--quiet` rename detection fix with a `diff_same()` optimization that speeds up `--find-copies-harder` by 1.33x when processing unchanged files.

**Submodule/Gerrit workflow question** — Alon Bar-Lev raised a limitation where submodules require branch names, preventing direct use of Gerrit's `refs/changes/` references during checkout.