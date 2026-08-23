# Git mailing list daily digest for 2026/08/22

## The day in brief
The Git mailing list saw active discussion on several fronts today. A long-standing bug in `git config --global` moved closer to resolution with Delilah Ashley Wu confirming cross-command testing for a v2 patch series. The `contrib/credential/libsecret` bugfix thread continued with Daniel Martí defending his design approach against Junio C Hamano’s alternative. A usability improvement for ambiguous remote branch names in `git worktree add`, `git checkout`, and `git switch` received final maintainer approval and will be queued for `next`. Meanwhile, a platform-compatibility series to replace `utime()` with `utimensat()` was blocked by a policy objection to its AI-generated origin. Finally, a new bug report described spurious conflicts when rebasing after a `git subtree add --squash`.

## Notable threads

### `git config --global` reads only one file despite docs claiming it reads both
**What changed**: Delilah Ashley Wu confirmed she is preparing a v2 patch series to make `git config --global` read both `~/.gitconfig` and the XDG config file (`~/.config/git/config`), aligning the implementation with the documentation and resolving inconsistencies with `git config --get --global` and `--global --no-includes`.

**Problem/goal**: The `--global` flag is documented to read both files, but the implementation reads only one (prioritizing `~/.gitconfig` if readable, even if empty). This creates user confusion, tooling failures, and inconsistencies with `git config --get` (which does read both files).

**Subsystem**: Configuration handling (`config.c`).

**Kind of work**: Bugfix.

**Impact**: Resolves a long-standing discrepancy between documented and actual behavior, improving consistency across commands and fixing edge cases like `--global --no-includes`. The change preserves backward compatibility while enabling access to settings in the XDG file.

Today, Delilah Ashley Wu [2026/08/22/09-38-32] noted she will test both `git config --get --global` and `git config list --global` to ensure cross-command consistency before sending v2. This follows Nils Fahldieck’s offer to review the series and strengthens the case for merging the code change, which targets the `git_global_config()` function in `config.c`.

---

### Bugfix for `contrib/credential/libsecret`: avoid crash and silent data loss
**What changed**: Daniel Martí reiterated his design rationale for replacing the `SECRET_SEARCH_LOAD_SECRETS` flag with an explicit `secret_item_load_secret_sync()` call, arguing that the current two-step process (SearchItems + GetSecrets) silently discards errors and triggers assertion failures.

**Problem/goal**: Fix a crash and silent data loss in the `contrib/credential/libsecret` credential helper. The current implementation uses `SECRET_SEARCH_LOAD_SECRETS`, which silently discards failures (e.g., locked or concurrently deleted items), leading to NULL secret values and assertion failures in `secret_value_get_text()` or `secret_value_unref()`.

**Subsystem**: Credential helpers (`contrib/credential/libsecret`).

**Kind of work**: Bugfix.

**Impact**: Prevents silent data loss and assertion crashes by properly reporting errors for inaccessible items. The fix is uncontested in its correctness but remains under discussion due to design trade-offs.

Today, Daniel Martí [2026/08/22/20-47-12] defended his approach, explaining that `SECRET_SEARCH_LOAD_SECRETS` is not a true optimization (it performs two round trips: SearchItems + GetSecrets) and silently discards errors, including client-side skipped items. His patch replaces this with a single SearchItems call plus one explicit GetSecret, keeping the round-trip count identical in the common case while ensuring all errors are surfaced. Junio C Hamano’s alternative (retaining the flag and conditionally loading secrets) would add a third round trip in rare cases and leave the error-prone path untested. The discussion remains focused on design philosophy: optimization vs. simplicity and reliability.

---

### Improving error messages for ambiguous remote branch names
**What changed**: Junio C Hamano approved the v7 series improving user-facing error messages for `git worktree add`, `git checkout`, and `git switch` when an ambiguous remote branch name is provided. The series replaces the terse "matched multiple remote tracking branches" warning with a detailed, actionable advice message listing conflicting remotes.

**Problem/goal**: Replace a generic warning with clear, actionable advice to help users resolve ambiguity when a branch name (e.g., `foo`) matches multiple remote tracking branches (e.g., `origin/foo` and `upstream/foo`). The series also moves an existing `die()` call earlier in the code path to provide a more specific diagnostic.

**Subsystem**: Worktree, checkout, and switch commands (`builtin/worktree.c`, `builtin/checkout.c`, `checkout.c`).

**Kind of work**: Usability improvement.

**Impact**: Improves clarity for users encountering ambiguous branch names, reducing confusion and support burden. The advice is gated behind `advice.checkoutAmbiguousRemoteBranchName` (enabled by default) and is consistent across all three commands.

Today, Junio C Hamano [2026/08/22/18-08-48] approved the series after Yoichi Nakayama addressed two minor fixes: correcting a grammatical error in the advice message ("Branch name '%s' appears in multiple remotes:") and narrowing the scope of the `oid` variable in `builtin/worktree.c`. The series is now ready for `next`. Earlier, Yoichi Nakayama [2026/08/22/00-50-48] confirmed that the one-argument `git worktree add` form (e.g., `git worktree add ../topic-branch`) succeeds regardless of ambiguity and does not require disambiguation logic. Junio C Hamano [2026/08/22/17-22-06] also identified a latent bug in this edge case, suggesting it should error out when ambiguity exists, but this is left for a follow-up effort.

---

### Replace `utime()` with `utimensat()` for POSIX.1-2024 compatibility
**What changed**: The series to replace the deprecated `utime()` with `utimensat()` was blocked by a policy objection to its AI-generated origin. The objection cites the `SubmittingPatches` policy and the Developer Certificate of Origin’s requirement for contributors to know the provenance of their code.

**Problem/goal**: Remove all uses of `utime()` (deprecated in POSIX.1-2024) and replace it with `utimensat()` to ensure long-term compatibility and enable nanosecond-precision timestamps. The series introduces a `git_utimensat()` wrapper and includes MinGW-specific support.

**Subsystem**: Platform compatibility (23 files, including `commit-graph.c`, `object-file.c`, `pack-objects.c`, and build system files).

**Kind of work**: Platform-compatibility cleanup.

**Impact**: Enables nanosecond-precision timestamps and ensures compatibility with POSIX.1-2024. The series is technically complete but cannot proceed until the project clarifies its policy on AI-generated content.

Today, brian m. carlson [2026/08/22/16-03-35] objected to the AI-generated origin of the series, citing legal and ethical concerns. Junio C Hamano [2026/08/22/17-59-09] acknowledged the legal risks, noting the potential for future legal costs and the challenge of distinguishing AI-generated contributions from those of unfamiliar contributors. brian m. carlson [2026/08/22/21-15-35] reinforced the legal rationale, citing global litigation around LLM outputs and the risk of copyright infringement from training data. The thread is now blocked pending a Project Leadership Committee discussion or a broader community amendment to `SubmittingPatches`.

---

### `git subtree add --squash` followed by `git rebase` produces spurious conflict
**What changed**: Gabriel Ford reported a new bug where `git subtree add --squash` followed by `git rebase` produces a spurious conflict between identically named files in separate directories (e.g., `.clang-format` in the main repository root and the subtree prefix directory).

**Problem/goal**: Identify the root cause of spurious conflicts during rebase after a `git subtree add --squash` operation. The files are in separate directories, and the main repository's file has not changed, so no conflict should occur.

**Subsystem**: `contrib/subtree` (shell script) and rebase machinery.

**Kind of work**: Bug report.

**Impact**: Users may encounter unexpected conflicts when rebasing after vendoring an external repository with `git subtree add --squash`, even when the conflicting files are in separate directories.

Today, Gabriel Ford [2026/08/22/16-10-04] provided reproduction steps and hypothesized that the issue stems from how the squash merge records history, causing rebase to treat the two files as conflicting. The report is the first of its kind, and no prior discussion or related commits were referenced.

## In brief
- **`git config --global` documentation-behavior mismatch**: Delilah Ashley Wu [2026/08/22/09-38-32] will test `git config --get --global` and `git config list --global` to ensure cross-command consistency before sending v2 of her patch series. The series aims to make `--global` read both `~/.gitconfig` and the XDG config file, aligning the implementation with the documentation.