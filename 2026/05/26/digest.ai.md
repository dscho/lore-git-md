Here's the daily digest for May 26, 2026:

**The day in brief.** A moderately busy day with 46 emails across 18 threads, featuring several patch series reaching maturity and one new feature proposal. Key developments include final refinements to the `interpret-trailers` documentation, performance optimizations for sparse index operations in `git restore`, and a new `--refresh-stat-only` option for `git update-index`. Junio's "What's cooking" report provides a comprehensive overview of the current development pipeline.

**Notable threads**

**`git restore` sparse index optimization (v2)**  
Derrick Stolee's two-patch series optimizing `git restore --staged` for sparse checkouts has reached v2, now incorporating Junio's feedback about code organization. The changes avoid unnecessary index expansion when using `..` pathspecs by introducing a new `try_update_sparse_directory()` helper that handles tree entries directly rather than expanding to blob paths. The series demonstrates Stolee's characteristic test-first approach, with comprehensive coverage verifying both the optimization and preservation of existing behavior. With all feedback addressed, this appears ready for merging.

**`update-index --refresh-stat-only` proposal**  
A new two-patch series introduces a `--refresh-stat-only` option for `git update-index`, targeting CI/container workflows where filesystem metadata changes while content remains identical. The implementation skips content hashing while updating mtime/ctime/inode/device data, with careful documentation of the tradeoffs (speed vs potential stale object IDs). The first patch fixes a preload-index bug with `--really-refresh`, while the second adds the new flag with thorough test coverage. The design thoughtfully addresses historical concerns by making this a one-shot operation rather than sticky configuration.

**`git son` feature proposal**  
Evan Haque proposes a new `git son` command to create independent child repositories with loose parent-child relationships (distinct from submodules). The well-structured RFC includes implementation, documentation, and comprehensive tests, but faces substantive questioning from D. Ben Knoble about whether existing mechanisms (`git clone .`, worktrees, subtrees) already cover this use case. The discussion now centers on whether the convenience merits a new porcelain command versus a contrib script.

**In brief**

**`interpret-trailers` documentation** -- Kristoffer Haugsbakk and D. Ben Knoble finalize wording improvements to the merged documentation series about trailer key format rules, consolidating redundant paragraphs about default appending behavior.

**`git commit --fixup` message options** -- Erik Cervin-Edin's v2 series extending `--fixup` to support all message specification options (`-m`, `-F`, `-c`, `-C`) now routes all variants through `prepare_to_commit()` per Junio's feedback.

**`push.default=simple` clarification** -- Ivan Baluta's v2 documentation patch clarifies behavior in triangular workflows, restructuring the explanation to be more technically precise while maintaining clarity.

**Repository initialization refactoring** -- Patrick Steinhardt's 8-patch series centralizing object database creation concludes with patch 8/8 moving ODB initialization into `apply_repository_format()`, completing this phase of the ODB abstraction effort.

**`diff.<driver>.process` protocol** -- Junio provides substantive review feedback about error handling and global state management in the diff subprocess protocol implementation, pushing for clearer failure semantics.

**Shell completion dotfiles** -- Zakariyah Ali's v2 patch modifies path completion to hide dotfiles by default (aligning with shell conventions), now with historical context about long-standing TODO comments expecting this behavior.

**`git mv --index-only` discussion** -- The thread explores whether recording moves in the index without filesystem operations merits a new flag, with technical consensus forming that `git add old new` provides equivalent functionality.

**On the radar**

**Commit-reach optimization** -- Kristofer Karlsson's O(1) solution for merge-base calculations via `nonstale_queue` appears technically sound but awaits final maintainer approval after extensive discussion of alternative approaches.