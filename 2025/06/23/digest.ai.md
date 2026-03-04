# Git Mailing List Digest - 2025/06/23

## The day in brief

A busy day with 72 emails across 19 threads, dominated by performance optimizations for `git fetch --prune` and MIDX/cruft pack handling. Key highlights include Taylor Blau's finalized MIDX optimization series, a major speedup for pruning operations, and continued discussion on the new `repo-info` command's design. Several bugfixes and test infrastructure improvements also landed.

## Notable threads

### **MIDX and cruft pack optimization series finalized**

Taylor Blau's performance optimization series for MIDX and cruft pack interaction during repacking reached its final form with v6. The series standardizes option handling in pack-objects, refactors internal code for better delta compression, and introduces a new `--stdin-packs=follow` mode that properly handles reachability across MIDX layers. The final patch adds a `repack.midxMustContainCruft` config (default true) to reduce MIDX bloat while maintaining correctness. Production testing shows 5-20% speed improvements, with all identified edge cases now addressed including proper handling of incremental MIDX writing. The series has received approvals from Junio Hamano and Elijah Newren, making it ready for merging.

### **Major speedup for `git fetch --prune`**

Phil Hord's optimization patches dramatically improve `git fetch --prune` performance by replacing an O(N^2) dangling ref check with an O(N*logN) approach borrowed from `git remote prune`. The change sorts refs first to enable binary search, reducing runtime from 410 seconds to under 1 second in test cases with large numbers of refs. The series also improves warning messages to explicitly show which deleted ref caused dangling symrefs. After extensive review, the patches are now settling on final wording distinctions between `--dry-run` ("will become dangling") and actual execution ("has become dangling") messages.

### **Submodule remote lookup improvements**

Lidong Yan's v4 series refactors submodule remote handling to be repository-agnostic and more robust against remote name changes. The key addition is URL-based remote matching that falls back to checking the submodule's configured URL against parent project remotes when default name lookup fails. This makes submodule updates work correctly when users rename their default remote (e.g., via `clone.defaultRemoteName`). The series builds on preparatory cleanups to memory management and `the_repository` removal, with comprehensive coverage of edge cases around branch merge configurations and relative URL resolution.

### **`repo-info` command interface debate**

The design discussion around Lucas Seiki Oshiro's new `git repo-info` command continued with Phillip Wood advocating for stricter plumbing command principles. The thread has shifted toward Junio Hamano's preferred opt-out model where the command shows nothing by default unless fields are explicitly requested or `--all` is used. Phillip also raised important questions about JSON output handling of non-UTF8 paths and proposed NUL-terminated output for better script parsing. Lucas acknowledged these concerns and committed to including documentation in the next version while maintaining the command's focused purpose rather than expanding it into a general query tool.

## In brief

**Promisor-remote protocol refactoring** -- Christian Couder's v4 series converts the internal representation of promisor remotes to use a dedicated `struct promisor_info`, preparing for future protocol extensions while maintaining backward compatibility.

**IMAP-send feature series merged** -- After 19 iterations, Aditya Garg's series adding OAuth2.0 support, PLAIN authentication, and folder management to `git imap-send` received final approval from Phillip Wood and Junio Hamano.

**JavaScript diff driver expanded** -- Derick W. de M. Frias' GSoC series added CommonJS/ESModules syntax support to the new JavaScript diff driver, now with 77 test files covering export variants and numeric literals.

**Windows terminal handling fix** -- James Duley's patch addresses an assertion failure in `compat/terminal.c` when toggling DUPLEX mode, with Phillip Wood suggesting additional safeguards against state inconsistencies.

**Historical fsck warnings explained** -- Jeff King and Junio Hamano clarified that "badFilemode" and "missingTaggerEntry" warnings in git.git clones are harmless artifacts from Git's early history, not signs of corruption.

**Test infrastructure improvements** -- Jeff King added a `-f` format option to `test_seq` that eliminates verbose shell loops in 9 test files, making sequence generation more readable and direct.

## On the radar

**Rustification debate** -- Ezekiel Newren's effort to introduce Rust code into Git remains paused pending resolution of platform support concerns, particularly from Randall S. Becker about NonStop compatibility.

**Clang-format line wrapping** -- The discussion about removing line wrapping rules from `.clang-format` in favor of `.editorconfig` continues, with Junio Hamano open to improved rules that only handle excessively long lines.