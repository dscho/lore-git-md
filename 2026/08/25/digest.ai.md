# Git mailing list daily digest for 2026/08/25

## The day in brief
The Git mailing list saw significant activity around several major patch series. The `git history squash` series reached a key decision point on case-sensitive OID matching, while the trace2 hardening series received detailed technical reviews. A new `git whoami` command sparked debate about project scope, and architectural concerns were raised about the `git repo info` path keys. Several bugfix series (reftable stack reloads, HTTP redirects, worktree basename handling) advanced toward integration.

## Notable threads

### `git history squash`: Autosquash marker resolution policy resolved
**What changed?**
The final technical blocker for Harald Nordgren’s `git history squash` series was resolved today. Junio C Hamano reaffirmed the project’s direction toward strict case-sensitive matching for autosquash markers, rejecting Phillip Wood’s suggestion to adopt permissive matching for consistency with `git rebase --autosquash`.

**Technical details**
The resolution requires a one-line mechanical change in Patch 7/8: replacing `istarts_with()` with `starts_with()` in `squash_check_can_autosquash()`. This enforces strict case-sensitive matching for autosquash markers (e.g., `squash!`, `fixup!`, `amend!`), rejecting all uppercase OIDs. The change aligns with Git’s historical convention of emitting only lowercase OIDs and the precedent set by `bc/restrict-hex-to-lowercase`.

**Why it matters**
This decision removes the last technical hurdle for the series, which is now functionally complete except for test coverage for the strict matching policy. The series introduces a new `git history squash` subcommand to efficiently collapse commit ranges while preserving descendant history, avoiding the repeated conflict stops of a rebase-based approach. The strict matching policy ensures consistency with Git’s broader effort to discourage uppercase OID usage, addressing security concerns where tools might check only lowercase forms.

---

### Trace2 hardening: Architectural feedback and correctness bugs
**What changed?**
Elijah Newren provided substantive reviews of Derrick Stolee’s seven-part trace2 hardening series, identifying several correctness issues and architectural concerns. The series aims to eliminate all `die()`-triggering helpers from the trace2 API to prevent Git crashes during telemetry operations.

**Technical details**
Key findings from Elijah’s reviews:
1. **Naming inconsistency**: `banned-die.h` uses `banned(die)` instead of `BANNED(die)`, deviating from Git’s existing convention (e.g., `NO_THE_REPOSITORY_H`).
2. **Include-ordering risk**: `banned-die.h` is included in the middle of the include list in `trace2.c`, which could silently bypass enforcement for future banned functions.
3. **Incomplete hardening**: The third patch’s replacement of `xstrdup()` with `strdup()` does not fully eliminate the risk of crashing in trace2’s config parsing path, as the error path still triggers a `die()` via `xstrfmt()`.
4. **Off-by-one error**: The fifth patch’s manual string concatenation for `:<REDACTED>` omits the trailing `>`, producing `:<REDACTED` instead of `:<REDACTED>`.
5. **Splitting error**: The sixth patch introduces a regression (region stack underflow) when applied in isolation, which is only resolved by the seventh patch.

**Why it matters**
The series addresses a critical stability issue: trace2, a low-level telemetry layer, should never crash Git, even under memory pressure or system call failures. The hardening is particularly important for Git for Windows, where `libintl_vsnprintf()`’s allocation behavior differs from POSIX. Elijah’s feedback highlights the fragility of manual string manipulation and the importance of thorough testing for low-level code. The series is now technically complete but requires a v3 to address these issues.

---

### `git repo info`: Architectural duplication concerns
**What changed?**
Junio C Hamano raised a substantive architectural concern about K Jayatheerth’s `git repo info` series, which adds seven new path-related keys (e.g., `path.cdup`, `path.toplevel`). The concern centers on duplication of logic between `git repo info` and `git rev-parse` for keys that mirror existing functionality.

**Technical details**
Junio proposed consolidating the shared logic into a new helper library (e.g., `repo-info.c`) to avoid long-term maintainability issues. The affected keys include:
- `path.cdup` (mirrors `git rev-parse --show-cdup`)
- `path.toplevel` (mirrors `git rev-parse --show-toplevel`)
- `path.superproject-root` (mirrors `git rev-parse --show-superproject-working-tree`)
- `path.git-prefix` (mirrors `git rev-parse --show-prefix`)

The series is otherwise feature-complete and mechanically clean, with all prior feedback addressed. The new `path.cdup` key, added in v5, resolves the last open design question by providing symmetry with `path.git-prefix`.

**Why it matters**
The concern highlights a broader tension in Git’s architecture: as new porcelain commands are added, they often duplicate logic from existing plumbing commands. Junio’s proposal to consolidate this logic into a shared helper library could reduce maintenance burden and ensure consistency across commands. However, it would require a non-trivial refactor of the series, potentially delaying integration. The discussion now centers on whether the duplication is acceptable or if a v6 is needed to address the architectural concern.

---

### `git whoami`: Project scope and usability debate
**What changed?**
A new patch introducing `git whoami` sparked debate about whether this functionality belongs in Git core. The command consolidates Git’s identity and signing configuration (author/committer names/emails, GPG/SSH keys, `commit.gpgsign` status) into a single, user-friendly output.

**Technical details**
Brian m. carlson and Junio C Hamano raised several concerns:
1. **Project scope**: Why doesn’t this functionality belong in `git var` or another existing command?
2. **Machine-readable output**: The command should support `key=value` pairs and NUL-terminated output (`-z`) for scripting.
3. **Flag design**: The current `--author`/`-a`, `--committer`/`-c`, `--name`/`-n`, and `--email`/`-e` interactions are counterintuitive. Junio proposed a more additive design where flags combine predictably (e.g., `-a -e -n` shows full author identity).
4. **Technical gaps**: The patch uses `the_repository` (violating the ongoing removal effort) and lacks support for X.509 signing keys.

**Why it matters**
The debate touches on Git’s design philosophy: should the project avoid "random subcommands" unless absolutely necessary? The command addresses a real usability pain point (users often juggle multiple identities or signing keys), but its inclusion in Git core is now contested. The discussion will likely hinge on whether the author can justify the command’s necessity and redesign the flag interactions to align with Git’s existing conventions.

---

### Worktree basename handling: Bugfixes and cleanups
**What changed?**
René Scharfe’s four-patch series fixing `git worktree add` basename handling advanced toward integration. The series addresses potential data corruption, rejects malformed paths, improves trailing-slash handling, and simplifies the code.

**Technical details**
Key fixes:
1. **Out-of-bounds read**: Patch 1 adds a bounds check to prevent corruption when the input path has zero length.
2. **Separator-only paths**: Patch 2 rejects paths consisting solely of separators (e.g., `"/"` or `"//"`), which previously triggered a BUG assertion.
3. **Trailing slashes**: Patch 3 trims trailing slashes from derived branch names, fixing a bug where `git worktree add waffle/bit/` attempted to create a branch named `bit/` (invalid due to trailing slash).
4. **Memory management**: Patch 4 renames `worktree_basename()` to `worktree_basename_dup()` and makes it return a freshly allocated string copy, simplifying memory management.

Junio C Hamano reviewed all patches and signaled approval, requesting only a minor commit-message tweak in Patch 4 to clarify that two callers already make their own copies of the returned string.

**Why it matters**
The series addresses real-world issues reported in Git for Windows (issue #6346) and improves the robustness of `git worktree add`. The fixes are narrowly scoped and well-tested, making them low-risk candidates for integration. The series also demonstrates the value of thorough edge-case testing in low-level code.

---

## In brief
- **`git history squash`**: Autosquash marker resolution policy resolved in favor of strict case-sensitive matching; series now functionally complete.
- **Trace2 hardening**: Elijah Newren identified correctness bugs and architectural concerns; v3 expected to address these.
- **`git repo info`**: Junio raised architectural duplication concerns; discussion ongoing about consolidating shared logic.
- **`git whoami`**: New command sparked debate about project scope and usability; flag design and technical gaps remain unresolved.
- **Worktree basename handling**: René Scharfe’s series advanced toward integration with minor commit-message tweak requested.
- **ODB alternates refactoring**: Patrick Steinhardt’s 8-patch series completed, removing unused `write_alternates()` infrastructure.
- **ODB fsck pluggability**: Patrick Steinhardt’s 10-patch series completed, moving fsck logic into backend-specific callbacks.
- **`git branch -d` protection**: Junio raised backward-compatibility concerns about protecting local upstreams from deletion.
- **HTTP redirects**: Junio requested broader review for Aaron Plattner’s bugfix preserving `WWW-Authenticate` headers.
- **Reftable stack reloads**: Patrick Steinhardt expressed mild unease about lock-state tracking but endorsed the series.
- **Stash design**: Branch-aware stash design refined, but detached HEAD gap remains unresolved.
- **Range-diff notes**: Kristoffer Haugsbakk acknowledged usability issue with `--range-diff-notes` inheriting patch notes setting.