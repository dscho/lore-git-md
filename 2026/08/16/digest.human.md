# Git mailing list daily digest for 2026/08/16

## The day in brief

The Git mailing list saw clarifications and design discussions today, with one patch series nearing completion and another facing a substantive design challenge. A new documentation patch for experimental bitmap configuration was introduced, and a long-running bugfix saw authorship logistics resolved. The most consequential development was a design objection to a proposed `git repo info` key, while a proposal for a new `git squash` command sparked debate about redundancy with existing tools.

## Notable threads

### `fetch_if_missing` moves into `struct repository` (v6)
**What changed**: A two-patch series that moves the global `fetch_if_missing` variable into `struct repository`, continuing Git’s libification effort to eliminate global state.

**Problem/goal**: The global `fetch_if_missing` flag controls whether missing objects trigger lazy fetches from promisor remotes. Moving it into `struct repository` makes it a per-repository setting, preserving existing behavior (default remains enabled) while aligning with the project’s architectural goals.

**Today’s development**: Tian Yuchen [2026/08/16] cited the `git-index-pack` man page to confirm that repository-less invocations of `git index-pack` are a documented and intended use case. The man page states that packed archives and their indices "can then be placed in the objects/pack/ directory of a Git repository," implying the command is designed to operate on standalone `.pack` files regardless of immediate repository placement. This reinforces the legitimacy of the series’ approach to preserve existing behavior while moving the flag into `struct repository`.

**Status**: The series is technically complete and ready for queuing, with Junio C Hamano’s explicit approval. No further changes are expected from the author.

---

### `git repo info` path keys (v4): Design objection to `path.git-prefix`
**What changed**: A seven-patch series adding six new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`) to the `git repo info` command, exposing filesystem locations of repository components in a scriptable format.

**Problem/goal**: Provide user-facing, scriptable access to repository component paths (e.g., hooks directory, index file) without requiring direct access to Git’s internals. The new keys support suffix-based formatting (e.g., absolute vs. relative paths).

**Today’s development**: Lucas Seiki Oshiro [2026/08/16] raised a substantive design objection to the `path.git-prefix` key, arguing it represents "current directory information" rather than "repository information" and thus conceptually mismatches the command’s intended scope. The key exposes the relative path from the repository’s working tree root to the current directory, matching `git rev-parse --show-prefix`. This is the first pushback on the series’ design beyond mechanical or naming concerns and may delay or block the patch’s acceptance, even if the implementation is correct.

**Status**: The v4 series is mechanically clean and ready for substantive technical review. The `path.git-prefix` patch is technically straightforward but now faces a design-level challenge. The author, K Jayatheerth, has not yet responded to the objection.

---

### Documentation for experimental `bitmapPseudoMerge.<name>.*` options
**What changed**: A standalone documentation patch introducing experimental configuration options for Git’s pseudo-merge bitmap feature, which optimizes reachability queries in large repositories.

**Problem/goal**: Document the `bitmapPseudoMerge.<name>.*` settings, which control how Git groups commits into pseudo-merge bitmaps. The options include patterns for grouping references (e.g., tags, remote branches), decay rates for group size adjustment, and age-based thresholds for commit eligibility.

**Today’s development**: kapicharly@icloud.com [2026/08/16] introduced the documentation, explaining each option’s purpose and behavior. The patch clarifies the distinction between "unstable" and "stable" pseudo-merge bitmaps, with examples for grouping tags by remote. The commit message explicitly notes the options are experimental and may change or be removed, linking to `gitpacking(7)` for broader context.

**Status**: The patch is under review. Its relevance depends on the status of the pseudo-merge bitmap feature itself, which may still be in flux.

---

### Proposal for new `git squash` command
**What changed**: A feature proposal for a new `git squash <n>` command to combine the last `<n>` commits into a single new commit, equivalent to `git reset --soft HEAD~<n> && git commit`. An optional `-s` flag would add a Signed-off-by trailer.

**Problem/goal**: Simplify the process of squashing commits for developers who frequently use this workflow, particularly in projects requiring a Developer Certificate of Origin (DCO).

### Today’s development

- The Super Techie [2026/08/16] proposed the command, motivated by convenience and time-saving. The proposal does not address edge cases (e.g., merge commits, conflicts) or interaction with existing Git workflows.
- Matej Dujava [2026/08/16] noted that `git rebase --autosquash` with `fixup!`/`squash!` commits already provides a more flexible solution, and linked to a v13 `git history squash` patch series as prior discussion. This shifts the conversation toward whether a new command is necessary or if existing tools suffice.

**Status**: Proposal stage; no patches posted. The thread now encompasses two distinct approaches: the original simple `git squash <n>` and the more flexible `git history squash` from the linked series. No maintainer guidance has emerged.

---

## In brief
- **`git worktree add` out-of-bounds read fix**: René Scharfe [2026/08/16] finalized the fix for the memory-safety flaw in `worktree_basename()`, replacing a pointer-decrement loop with an integer-index loop to eliminate underflow risk. The patch is ready for `next` and addresses a critical bug that could lead to data loss. Junio C Hamano [2026/08/16] engaged in a procedural discussion about authorship logistics, ultimately accepting Matthias Aßhauer’s sign-off. The second patch in the series (rejecting empty strings in `git worktree add "" HEAD`) remains pending revision.