**The day in brief.**
2026/06/23 was a busy, milestone-heavy day on the Git mailing list. A long-running `git status` rebase-todo parsing fix was finally merged; the `--track=fetch` feature for `git checkout`/`git switch` received substantive review on its workflow justification; and Git 2.55.0-rc2 was announced, marking the start of deep feature-freeze. Two “What’s cooking” reports (#08 and #09) gave a clear snapshot of the integration branches, with the ODB abstraction stack now cooking in `next` and several performance topics ready for final review.

---

## Notable threads

### `git status` rebase-todo parsing fix merged
Phillip Wood’s two-patch series (v4) fixing incorrect display of commit object IDs and refnames in the rebase todo list was merged to `master`. The preparatory patch refactored the sequencer’s parsing logic into a shared `sequencer_parse_todo_command()` helper, while the second patch implemented command-aware display logic that preserves refnames (e.g., “reset main”) and abbreviates OIDs only where appropriate. All review feedback—including Junio C Hamano’s validation of label/OID collision mitigation—has been incorporated, and the series is now closed.

---

### `--track=fetch` for `git checkout`/`git switch`: workflow justification debated
Harald Nordgren’s `--track=fetch` feature, which automatically fetches the remote branch when creating a tracking branch, received a substantive review from Phillip Wood. While the implementation is technically sound, Phillip questioned the feature’s fundamental value, asking for clearer documentation of the workflow benefit. Harald responded by framing the feature as a pragmatic alignment with user expectations: users intuitively expect `git checkout -b topic origin/master` to start from the *latest* version of `origin/master`, not a potentially stale local copy. The exchange sharpens the focus on whether the feature’s convenience outweighs concerns about network operations during checkout without user preview. Junio’s final decision remains pending, but the thread is now unblocked on the workflow rationale.

---

### `git log --follow` merge-commit rename tracking merged
Miklos Vajna’s four-patch series (v4) improving `git log --follow`’s rename tracking across merge commits was administratively closed. The series introduces a `follow_pathspec_slab` to store per-commit pathspec information, replacing the problematic global variable approach. All technical and merge-conflict issues have been resolved, and the series is now queued for the next release. The implementation touches `revision.c`, `log-tree.c`, and `diff.c`, and includes comprehensive test coverage in `t4219-log-follow-merge.sh`.

---

### `git history squash` range-folding command procedurally unblocked
Harald Nordgren’s `git history squash` series (v4) is now procedurally unblocked after Harald addressed the last open question about `--ancestry=` support. The series adds a new subcommand to fold a commit range into its oldest commit while preserving descendant history, avoiding the repeated conflict stops of a rebase-based approach. The finalized behavior rejects operations with refs pointing inside the squashed range by default, with an `advice.historyUpdateRefs` message guiding users to `--update-refs=head`. The series is code-complete and queued in `next`, awaiting Junio’s final assessment.

---

### `paint_down_to_common()` merge-base optimization technically complete
Tian Yuchen’s six-patch series optimizing `paint_down_to_common()` for one-sided histories is now technically complete for v2. The core optimization terminates early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories. All design and technical feedback—including Derrick Stolee’s endorsement of the termination condition and state encapsulation in the `paint_state` struct—has been resolved. The series is ready for Junio’s final review and is expected to land in the next integration cycle.

---

### Git 2.55.0-rc2 released
Junio C Hamano announced Git 2.55.0-rc2, the second release candidate for the upcoming Git 2.55.0. The release includes 486 non-merge commits from 85 contributors (30 new), covering UI/workflow improvements (e.g., parallel hook execution, `git format-rev`, `git history fixup`), performance optimizations (e.g., revision traversal, reachability bitmaps, streaming object handling), and platform compatibility fixes (e.g., Windows build fixes, macOS linker warning suppression). The release notes are finalized in draft form, and the tree is now in deep feature-freeze, with only regression fixes expected before the final release.

---

## In brief

**`git repo` GSoC progress** -- K Jayatheerth’s Google Summer of Code project on the `git repo` command continues in its research/design phase, with weekly blog updates linked to the mailing list. No technical details or patches have been shared on-list yet.

**Interactive typo correction with argument preservation** -- Jishnu C K’s v2 patch for `help.autoCorrect=prompt` now preserves the user’s original arguments in the autocorrect prompt (e.g., `Run 'git checkout neo' instead [y/N]?`). Junio’s review flags a critical correctness issue (shell quoting of arguments) and an unintended regression (removal of interactive prompt logic for single-candidate cases), but the core usability improvement is uncontroversial.

**Range-scoped diff stat, whitespace check, and `-G` pickaxe for `git log -L`** -- Karthik Nayak’s seven-patch series extending `git log -L` to support range-scoped diff stat formats (`--stat`, `--numstat`, `--shortstat`), whitespace checking (`--check`), and the `-G` pickaxe received minor documentation feedback from Junio and Michael Montalbo. The implementation is approved in principle, and the series is ready for final review.

**GPG interface carriage-return stripping fix** -- DSAntonio08’s patch fixing over-aggressive carriage-return stripping in `gpg-interface.c` received substantive review from Junio, who identified a logic bug (failing to preserve a lone `\r` at the end of the buffer) and requested historical context in the commit message. The patch is blocked on addressing Junio’s feedback.

**`git fetch --recurse-submodules` fails on dangling submodule commits** -- Mike Crowe reported a bug where `git fetch --recurse-submodules` fails with a hard error when encountering a submodule commit that doesn’t exist on the remote, even if the commit isn’t referenced by the current tracking branch. The issue disrupts CI workflows, and Mike suggests two potential solutions: limiting recursive fetches to commits referenced by the current tracking branch or treating submodule fetch failures as non-fatal.

**ODB abstraction: promisor object connectivity checks** -- Patrick Steinhardt’s four-patch series refactoring connectivity checks to use the ODB interface for promisor objects received substantive review from Christian Couder, who questioned the removal of `odb_reprepare()` and critiqued the patch’s control flow complexity. The series is part of the ODB abstraction stack and depends on `ps/odb-source-packed`, which is already merged to `next`.

**`git branch --set-upstream-to` and `git push` usability improvements** -- Harald Nordgren’s two-patch series improving error messages for common command-line mistakes (e.g., `git branch --set-upstream-to=origin main` and `git push origin/main`) received maintainer feedback on topic branch naming and implementation details. The series is now proceeding under the topic branch `hn/branch-push-mistake-advise`.

**Build system quieting for gitk and git-gui** -- Harald Nordgren’s two-patch series aligning gitk and git-gui’s translation catalog generation output with core Git’s `make -s` conventions was procedurally completed. The gitk patch was merged by Johannes Sixt, and the git-gui patch was effectively superseded by equivalent changes already present in Sixt’s git-gui fork.

**GSoC: Improving disk space recovery for partial clones** -- Siddharth Shrimali’s Google Summer of Code progress report on improving disk space recovery for partial clones links to a blog post summarizing weeks 3–4 of work. The project remains in the prototyping phase, with no patches or technical details shared on-list yet.

---

## On the radar

**`--track=fetch` for `git checkout`/`git switch`** -- The thread is now focused on the feature’s workflow justification, with Junio’s final decision pending. Contributors tracking this effort should monitor the discussion for a resolution on whether the feature’s convenience outweighs concerns about network operations during checkout.

**`git history drop` replay safety** -- Phillip Wood’s review of `ps/history-drop` (v3) raised replay safety concerns, and the series is marked as “expecting reroll” in the latest “What’s cooking” report. The author is expected to address these concerns in v4 before the series can graduate to `next`.

**`mm/diff-process-hunks` external hunk provider integration** -- The series adding `diff.<driver>.process` to allow external tools to control which lines Git considers changed is marked as “needs review” in the latest “What’s cooking” report. Reviewers have raised concerns about integration complexity and edge cases in hunk selection, and the series is likely to require further iteration.