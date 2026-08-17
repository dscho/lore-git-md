# Git mailing list daily digest for 2026/08/16

## The day in brief
Tian Yuchen clarified the legitimacy of repository-less `git index-pack` invocations, reinforcing the design of a ready-to-queue patch series moving `fetch_if_missing` into `struct repository`. Lucas Seiki Oshiro raised a substantive design objection to the `path.git-prefix` key in the `git repo info` series, questioning its conceptual fit. A new RFC proposed a `git squash` command, sparking discussion about redundancy with existing tools like `git rebase --autosquash` and a long-running `git history squash` patch series.

## Notable threads

### Move `fetch_if_missing` into `struct repository` (v6)
**What changed?** The v6 patch series moves the global `fetch_if_missing` variable into `struct repository`, making it a per-repository flag that controls whether missing objects trigger lazy fetches from promisor remotes. This continues Git’s libification effort to eliminate global state.

**Today’s development:** Tian Yuchen [2026/08/16/06-20-50] cited the `git-index-pack` man page to confirm that repository-less invocations of `git index-pack` are a documented and intended use case. The man page states that the packed archive and its index "can then be placed in the objects/pack/ directory of a Git repository," implying the command is designed to operate on standalone `.pack` files regardless of immediate repository placement. This reinforces the series’ approach of preserving existing behavior while moving the flag into `struct repository`.

**Why it matters:** The exchange resolves a lingering design question about whether `fetch_if_missing` belongs in `struct repository` at all. Junio C Hamano previously questioned whether running `git index-pack` outside a repository is a legitimate use case or a user error, but the man page confirms it is intentional. The series is now technically complete and ready for queuing, with no further changes expected from the author.

---

### Add path-related keys to `git repo info` (v4)
**What changed?** The v4 series adds six new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`) to the `git repo info` command, exposing filesystem locations of repository components in a scriptable format.

**Today’s development:** Lucas Seiki Oshiro [2026/08/16/00-53-49] raised a substantive design objection to the `path.git-prefix` key, arguing it represents "current directory information" rather than "repository information" and thus does not belong in `git repo info`. The key exposes the relative path from the repository’s working tree root to the current directory, matching `git rev-parse --show-prefix`. This is the first pushback on the series’ design beyond mechanical or naming concerns, and it directly challenges the patch’s inclusion.

**Why it matters:** The objection shifts the discussion from implementation details to the command’s conceptual scope. If accepted, it could block the `path.git-prefix` patch or require a broader rethink of what `git repo info` should expose. The series is otherwise mechanically clean and ready for technical review, but this design question may delay its progress.

---

### Proposal: new `git squash` command
**What changed?** A new RFC proposed a built-in `git squash <n>` command to combine the last `<n>` commits into a single new commit, equivalent to `git reset --soft HEAD~<n> && git commit`. An optional `-s` flag would add a Signed-off-by trailer.

**Today’s developments:**
- The Super Techie [2026/08/16/18-49-50] proposed the command, motivated by convenience and DCO compliance. The email does not address edge cases (e.g., merge commits, conflicts, or partial staging) or how the command would interact with existing Git workflows.
- Matej Dujava [2026/08/16/21-52-00] noted that `git rebase --autosquash` with `fixup!`/`squash!` commits already provides a more flexible squashing workflow. Matej also linked to a long-running `git history squash` patch series (v13), which preserves the message of the first commit in the squashed range, suggesting the idea has been discussed before.

**Why it matters:** The thread now encompasses two distinct approaches: the original simple `git squash <n>` and the more nuanced `git history squash` from the linked series. The discussion highlights a tension between convenience and redundancy, as existing tools (`rebase --autosquash`) already address the use case. No maintainer has weighed in yet, leaving the proposal’s fate uncertain.

---

### Documentation for `bitmapPseudoMerge.<name>.*` configuration
**What changed?** A new patch introduced documentation for experimental `bitmapPseudoMerge.<name>.*` configuration options, which control how Git groups commits into pseudo-merge bitmaps for reachability query optimization in large repositories.

**Today’s development:** kapicharly@icloud.com [2026/08/16/09-11-53] added detailed man-page entries explaining the purpose and behavior of each option. The documentation covers:
- `bitmapPseudoMerge.<name>.pattern`: regex patterns for grouping references (e.g., tags, remote branches).
- `bitmapPseudoMerge.<name>.decay`: decay rates for adjusting group sizes over time.
- `bitmapPseudoMerge.<name>.threshold` and `bitmapPseudoMerge.<name>.stableThreshold`: age-based thresholds for commit eligibility.
- The distinction between "unstable" and "stable" pseudo-merge bitmaps, with the latter targeting older commits and imposing a one-time generation cost.

**Why it matters:** The documentation provides clear guidance for users experimenting with these settings, though its relevance depends on the status of the pseudo-merge bitmap feature itself. If the feature is still in flux, the documentation may require future revisions. The patch is purely additive and does not introduce code changes.

---

## In brief
- **Fix out-of-bounds read in `git worktree add`**: René Scharfe [2026/08/16/17-51-32] finalized the fix for the memory-safety flaw in `worktree_basename()`, replacing a pointer-decrement loop with an integer-index loop to eliminate underflow risk. The patch is ready for `next` and addresses a critical bug that could lead to data loss. Junio C Hamano [2026/08/16/19-30-55] discussed authorship logistics, suggesting René take sole authorship for DCO compliance, but René deferred to Matthias Aßhauer’s original work.
- **Enthusiastic support for `git squash`**: Bradley Morgan [2026/08/16/23-38-05] endorsed the proposed `git squash` command, expressing frustration with the current `git rebase` workflow. The reply does not add technical detail but signals user interest in a convenience feature.