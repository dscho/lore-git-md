# Git Mailing List Digest - 2025/07/21 -- 2025/07/27

**The week in brief.** A busy week with 497 emails across 150 threads saw significant progress on several fronts. Key developments include finalization of the promisor-remote protocol extensions, major strides in the `the_repository` removal effort, and heated debates about Rust integration challenges. The week also saw important infrastructure improvements with Patrick Steinhardt's reflog migration fixes and performance optimizations for `git remote rename`. Notable new features like `git last-modified` and `git repo info` reached maturity, and long-standing issues like `core.commentChar=auto` deprecation neared resolution.

## Key developments

### Promisor-remote protocol finalized

Christian Couder's v6 series extending the promisor-remote protocol to support configurable validation of remote attributes reached consensus after addressing terminology concerns. The implementation allows servers to advertise additional fields (like `partialCloneFilter` and `token`) via `promisor.sendFields`, with clients validating these against local config using `promisor.checkFields`. Junio Hamano questioned whether "field" terminology conflicted with Git's established "configuration variable" concepts, but the technical implementation appears sound and ready for merging after documentation refinements.

### `the_repository` removal advances

Patrick Steinhardt's 21-patch series to eliminate `the_repository` usage from config-related functionality made significant progress, converting `git_config_*()` wrapper functions to explicit `repo_config_*()` variants. The series moves Git-specific config parsing from `config.c` to `environment.c` and addresses sign comparison warnings in `prepare_include_condition_pattern()`. With most of the series already reviewed, this represents a major step in the multi-phase effort to eliminate implicit global state throughout Git's codebase.

### Rust integration debates intensify

Discussions about Rust adoption revealed concrete platform compatibility concerns, particularly for architectures without Rust support (HPPA/Alpha/m68k) and 32-bit systems. Patrick Steinhardt and Elijah Newren advocated for full Rust adoption, while Junio maintainers noted architectural challenges in Git's current code organization. The thread highlighted deployment obstacles including supply chain risks and dependency management, with Brian Carlson sharing real-world security maintenance concerns from Git LFS experience that may influence `Cargo.lock` policy.

### Reflog migration infrastructure matures

Patrick Steinhardt's 8-part series fixing reflog migration between storage formats (files <-> reftable) reached v2 with comprehensive fixes for identity handling and OID preservation. The series introduces `git reflog write` for manual entry creation in reftable and includes thorough test coverage. Jeff King and Junio Hamano provided detailed review feedback on flag handling consistency and atomicity concerns, with the implementation now appearing ready for integration after addressing minor documentation suggestions.

### `git remote rename` performance overhaul

Patrick Steinhardt proposed a major rewrite of `git remote rename` that reduces operation time from hours to seconds for repositories with 100k+ refs. The changes use atomic transactions and avoid packed-refs rewrites, building on Karthik Nayak's reflog migration infrastructure. Jeff King approved the approach while suggesting a secondary optimization using targeted ref iteration. With edge cases like directory/file conflicts now resolved, this long-running improvement appears ready for integration.

## In brief

**`git last-modified` feature** -- Toon Claes and Taylor Blau concluded review of this series (formerly `blame-tree`), showing 50.6% speedups for top-level checks with Bloom filter integration.

**`git repo info` command** -- Lucas Seiki Oshiro's GSoC project reached v6 with structured output formats, though debate continues about when to implement proper value quoting for forward compatibility.

**`core.commentChar=auto` deprecation** -- Phillip Wood's long-running effort to remove this problematic setting nears submission, making Git error out rather than silently fall back to `#`.

**IMAP sent-folder archiving** -- Aditya Garg's v4 patch for `git send-email` added a pure-IMAP mode while debate continues about whether this functionality belongs in core Git.

**Interactive patch commands** -- Leon Michalak's series adding `-U/--unified` and `--inter-hunk-context` options to interactive patch modes resolved final test issues.

**Test infrastructure** -- Jeff King modernized test helpers like `test-delta`, replacing manual memory management with `strbuf` while maintaining SANITIZE=leak compatibility.

**Rebase warning system** -- A proposal to warn when rebase skips commits that become empty faces fundamental detection challenges, with Johannes Sixt highlighting scenarios that would generate false positives.

**Git alias `-h` handling** -- Jeff King identified edge cases where aliases that inject arguments interfere with help mode detection, revealing deeper complexities in Git's alias expansion pipeline.

**Bash completion** -- Nelson Benítez León proposed configurable ref sorting via `GIT_COMPLETION_REFS_SORT_BY_FIELDNAME`, though the patch currently lacks tests.

## Looking ahead

The Rust integration debate will likely continue as platform support timelines become clearer, with GCC-based Rust compilation estimates suggesting libcore may be possible by end of summer but full support remains distant. Several major features that reached maturity this week - including `git last-modified`, the reflog migration infrastructure, and `git remote rename` optimizations - will likely see integration in the coming weeks. The `git repo info` command appears ready for finalization pending resolution of output format stability questions, while the rebase warning feature may need fundamental redesign to address its detection challenges.