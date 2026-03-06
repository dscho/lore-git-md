# Git Mailing List Digest - 2025/09/27

**The day in brief.** A moderately active Saturday with 32 emails across 8 threads, dominated by continued development of the `git repo stats` feature and Rust configuration handling. Key developments include v4 of the repository statistics series nearing completion and an expanded Rust FFI implementation for config parsing. Junio Hamano provided substantive feedback on both fronts, while discussions about workflow optimization and worktree behavior continued.

## Notable threads

### Repository statistics feature matures in v4

Justin Tobler's `git repo stats` series reached v4, now implementing all core functionality for repository health reporting. The series has progressed through multiple iterations to include reference counting, object type traversal, machine-readable output formats (keyvalue and NUL-delimited), and progress reporting. Junio Hamano's review raised important design questions about whether the tool should focus more on actionable metrics (like delta chains) rather than historical statistics, though the technical implementation appears sound. The series has incorporated feedback throughout, most recently switching to `utf8_strwidth()` for proper i18n support in table formatting. With all technical components now submitted, the series appears ready for final review pending documentation completion.

### Rust config handling expands scope

A series implementing boolean configuration parsing in Rust's FFI layer expanded to include unsigned long` and pathname handling in v3. Following Junio's direction to wrap Git's native functions rather than reimplementing in Rust, the series now provides comprehensive FFI bindings for `git_configset_get_bool()`, `git_configset_get_ulong()`, and `git_configset_get_pathname()`. The implementation includes thorough test coverage of edge cases and maintains Rust's memory safety practices. While the boolean handling addressed all prior feedback, the expanded scope in v3 would benefit from explicit maintainer approval. Junio also suggested organizational improvements, recommending the documentation fix in patch 1 be split out and the implementation patches be cleaned up to show the ideal final state.

### Worktree visibility debate continues

A user question about why worktree directories appear as untracked files led to an extended discussion of Git's design choices. Junio Hamano explained the behavior is intentional - worktrees are treated like any directory containing a `.git` subdirectory. Michal Suchánek challenged this rationale by pointing out an inconsistency: while a worktree's own `.git` is hidden, sibling worktrees' `.git` files appear as untracked. The thread explored whether this asymmetry warrants reconsideration, though practical concerns about collision scenarios reinforced the current recommendation to create worktrees in sibling directories rather than nested within the main worktree.

## In brief

**Documentation formatting fix** -- Jean-Noël Avila corrected Asciidoctor rendering issues in config/extensions.adoc and pretty-formats.adoc, working around a bug that incorrectly rendered "+" characters after nested lists.

**Push.default documentation discrepancy** -- A user highlighted inconsistent descriptions of `push.default=simple` between git-config and git-push documentation, where the former misleadingly suggests upstream branch configuration is only sometimes needed.

**Credential wildcard matching proposal** -- A feature request suggested adding partial URL matching for Git credential configuration, allowing different credentials under the same domain (like `example.com/org1/*` vs `example.com/org2/*`).

## On the radar

**Workflow optimization RFC** -- Taylor Blau's response to Junio's workflow RFC suggested requiring positive acks before merging and analyzed maintainer expansion models, moving the discussion toward concrete policy proposals.