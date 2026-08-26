# Git mailing list daily digest for 2026/08/25

## The day in brief
The Git mailing list saw significant activity around several major patch series. The `git history squash` series reached a key decision point on case-sensitive OID matching, while the trace2 hardening series received detailed technical reviews. A new `git whoami` command sparked debate about project scope, and a `git branch -d` protection feature faced backward-compatibility concerns. The ODB abstraction effort continued with multiple refactoring series.

## Notable threads

### `git history squash` autosquash marker resolution policy
The long-running `git history squash` series by Harald Nordgren reached a critical juncture regarding autosquash marker resolution. Phillip Wood questioned whether the series should enforce strict case-sensitive matching (rejecting uppercase OIDs) or adopt a permissive approach consistent with `git rebase --autosquash`.

**What changed**: Junio C Hamano reaffirmed the project's direction toward strict case-sensitive matching, citing the `bc/restrict-hex-to-lowercase` precedent and Git's historical convention of emitting only lowercase OIDs. The resolution requires a one-line mechanical change in Patch 7/8: replacing `istarts_with()` with `starts_with()` in `squash_check_can_autosquash()`.

**Why it matters**: This decision removes the last technical blocker for the series, which is now functionally complete except for this change and corresponding test coverage. The series aims to efficiently collapse commit ranges while preserving descendant history, avoiding the repeated conflict stops of rebase-based approaches.

**Current status**: The series is ready for integration once the reroll (v15) with the `starts_with()` change and test coverage is posted. Junio's "Will replace" sign-off from v7 indicates intent to merge.

---

### Trace2 hardening against `die()`-triggering helpers
Derrick Stolee's seven-patch series to eliminate `die()`-triggering helpers from the trace2 API received detailed technical reviews. The series addresses crashes caused by memory pressure or system call failures during telemetry operations.

**What changed**: Elijah Newren identified several technical concerns:
- A naming inconsistency in `banned-die.h` (`banned(die)` vs. `BANNED(die)`)
- An include-ordering risk where future includes after `banned-die.h` could silently bypass enforcement
- Incomplete hardening in the config parsing path (still triggers `die()` via `xstrfmt()`)
- A one-byte off-by-one error in manual string concatenation for redaction
- A splitting error in patch 6/7 that introduced a regression when applied in isolation

**Why it matters**: Trace2 is a low-level telemetry layer that should never crash Git, even under memory pressure. The series replaces banned functions with defensive fallbacks that prioritize process stability over telemetry completeness.

**Current status**: The series is technically complete with all feedback addressed in the upcoming v3. The introduction of `banned-die.h` as a general-purpose enforcement mechanism could inform future hardening efforts in other subsystems.

---

### `git whoami` command proposal
Andrew Pleeter proposed a new `git whoami` command to display the current Git identity and signing configuration. The patch sparked debate about whether this functionality belongs in Git core.

**What changed**: Both brian m. carlson and Junio C Hamano expressed skepticism:
- Brian requested machine-readable/NUL-terminated output, removal of `the_repository` usage, support for X.509 signing keys, and justification for why this doesn't belong in `git var`
- Junio questioned the necessity of adding "random subcommands" and identified confusing flag interactions

**Why it matters**: The command consolidates identity and signing configuration information that currently requires checking multiple `git config` or `git var` settings. The debate centers on whether this convenience justifies adding another subcommand to Git's already extensive command set.

**Current status**: The patch remains under review, with the author needing to address technical gaps and justify the command's inclusion in Git core.

---

### Protecting local upstreams from `git branch -d`
Harald Nordgren proposed a two-patch series to protect local branches from deletion via `git branch -d` when they serve as the upstream of another local branch.

**What changed**: Junio C Hamano raised a substantive backward-compatibility concern. The patch changes the long-established semantics of the `-d` flag, which could break existing workflows that rely on the current behavior. The test adjustment in the patch—replacing `git branch -d @{u}` with `git branch -D @{u}`—demonstrates the disruption.

**Why it matters**: The change aims to prevent accidental breakage of dependency chains but alters a fundamental Git command's behavior. Junio explicitly rejected introducing a configuration variable to toggle this behavior, arguing it would complicate predictability.

**Current status**: The series is under review, with the author needing to address the backward-compatibility concern.

---

### ODB abstraction and fsck pluggability
Patrick Steinhardt continued the ODB abstraction effort with two significant series:
1. An 8-patch series refactoring ODB alternates handling during repository creation
2. A 10-patch series making ODB fsck checks pluggable

**What changed**: Both series are now complete:
- The alternates series removes the ability to write alternates after repository creation, simplifying the ODB interface
- The fsck series moves verification logic into backend-specific implementations, preparing for future pluggable ODB backends

**Why it matters**: These refactorings are foundational for the ODB abstraction effort, which aims to give backends more control over their configuration and enable alternative storage systems.

**Current status**: Both series are under review and technically complete, with no remaining open questions.

---

## In brief
- **`git repo info` path keys**: Junio proposed consolidating shared logic with `git rev-parse` into a new helper library to avoid maintainability issues
- **Hex object ID case sensitivity**: Brian provided concrete security scenarios where Git's case-insensitive parsing enables security bypasses
- **`git worktree add` basename handling**: René Scharfe's four-patch series fixing edge cases received maintainer approval pending a commit-message tweak
- **`git format-patch` range-diff notes**: Kristoffer Haugsbakk acknowledged a usability issue with the `--range-diff-notes` option inheriting patch notes settings
- **Stash design**: Vladimir identified a gap in the branch-aware stash design for detached HEAD worktrees
- **HTTP authentication**: Junio called for review of a bugfix preserving `WWW-Authenticate` headers during redirects
- **`git push` from shallow clones**: Elijah rebutted a negotiation-based alternative, arguing it fails in several scenarios
- **Reftable stack reloads**: Patrick Steinhardt expressed mild unease about lock state tracking but endorsed the series
- **MIDX race condition**: Elijah's four-patch series addressing a race condition in geometric repacking is ready for merging