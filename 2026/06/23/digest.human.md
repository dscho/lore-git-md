# The Git Mailing List Daily Digest for 2026/06/23

## The day in brief.

A moderate-volume day (30 emails across 19 threads) with a mix of finalized features, performance optimizations, and usability improvements. The standout developments: Phillip Wood’s `git status` rebase-todo display fixes are now merged; Patrick Steinhardt’s ODB abstraction stack advances with connectivity-check refactoring; and Junio’s “What’s cooking” report (#09) signals deep feature-freeze for Git 2.55-rc2. Two GSoC updates and a submodule fetch bug report round out the day’s substantive work.

---

## Notable threads

### `git status` rebase-todo display fixes now merged
Phillip Wood’s two-patch series (v4) fixing long-standing usability bugs in `git status` during interactive rebase is now fully merged to `master`. The patches address two specific issues: commit object IDs in “fixup -C” and “merge -C” commands were left unabbreviated, and refnames (e.g., “reset main”) were incorrectly replaced with their pointed-to object IDs. The preparatory patch refactors todo-command parsing into a shared `sequencer_parse_todo_command()` function, establishing infrastructure for future sequencer improvements. Junio’s final review confirmed the label-before-OID-abbreviation safety check, and all CI failures have been resolved. The series demonstrates careful handling of object ID/refname interaction while preserving backward compatibility.

### `--track=fetch` for `git checkout`/`git switch` remains contentious
Harald Nordgren’s `--track=fetch` feature for `git checkout` and `git switch` continues to face maintainer skepticism despite technical completion. The v14 series (2 patches) extends `--track` to automatically fetch the remote branch when creating a tracking branch, addressing a workflow pain point where users must manually fetch before checking out. Phillip Wood’s latest review requests clearer workflow justification, while Junio remains concerned about network operations during checkout without user preview. Harald’s response argues the feature reduces merge conflicts and duplicated work by aligning Git’s behavior with user expectations—users intuitively expect `git checkout -b topic origin/master` to start from the *latest* version of `origin/master`. The series is technically approved but unlikely to merge without broader consensus on its workflow value.

### ODB abstraction stack advances with connectivity-check refactoring
Patrick Steinhardt’s four-patch series refactoring connectivity checks to use the ODB interface for promisor objects (`ps/connected-generic-promisor-checks`) received substantive review from Christian Couder. The series removes the packfile dependency in connectivity checks by replacing `find_pack_entry_one()` with `odb_for_each_object_ext()` and the `ODB_FOR_EACH_OBJECT_PROMISOR_ONLY` flag. Christian’s review flags two key concerns: the removal of `odb_reprepare()`, which could cause connectivity checks to miss promisor objects in packs added after iteration begins, and the patch’s overly complex control flow. He proposes splitting `check_connected()` into two focused functions (`check_connected_promisor()` and `check_connected_rev_list()`) to improve maintainability. The series is part of Steinhardt’s broader ODB abstraction effort, which aims to support alternative backends like reftable.

### `paint_down_to_common()` optimization nears completion
Tian Yuchen’s six-patch series optimizing `paint_down_to_common()` for one-sided histories is now technically complete for v2. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Derrick Stolee and Kristofer Karlsson finalized the termination condition (`(!queue.p1_count || !queue.p2_count) && !queue.pending_merge_bases`) and state encapsulation in the `paint_state` struct. The optimization is expected to land in `next` shortly, with hyperfine benchmarks confirming performance improvements in `git merge-base` for asymmetric histories.

### Git 2.55-rc2 enters deep feature-freeze
Junio’s “What’s cooking in git.git” report (#09) signals deep feature-freeze for Git 2.55-rc2, with only regression fixes and a few late topics expected to graduate before the final release. Key merged features include Patrick Steinhardt’s ODB abstraction stack (`ps/odb-source-packed`), Taylor Blau’s MIDX incremental repacking fixes (`tb/midx-incremental-custom-base`), and Han-Wen Nienhuys’ `--delete-merged` for `git branch`. The report highlights 32 cooking topics, including Steinhardt’s ref backend refactoring (`ps/refs-avoid-chdir-notify-reparent`), Karthik K’s merge-base optimization (`kk/merge-base-exhaustion`), and Michael Montalbo’s external hunk provider support (`mm/diff-process-hunks`). Three topics remain stalled, including Jonathan Tan’s config-lock-timeout work.

---

## In brief

**`git history squash` series unblocked** -- Harald Nordgren addressed the last open question about `--ancestry=` support in the `git history squash` series, committing to either implement the option or provide a rationale for omitting it. The series (4 patches) folds commit ranges into a single commit while preserving descendant history and is now procedurally unblocked for Junio’s final assessment.

**Interactive typo correction preserves arguments** -- Jishnu C K’s v2 patch for `help.autoCorrect=prompt` now preserves the user’s original arguments in the autocorrect prompt (e.g., `Run 'git checkout neo' instead [y/N]?`). Junio’s review flags a critical correctness issue: the prompt must shell-quote arguments to avoid misinterpretation when cut-and-pasted. The patch is narrowly scoped and uncontroversial but requires a reroll to address the quoting bug.

**Line-log gains range-scoped diff stat and whitespace checking** -- Karthik Nayak’s seven-patch series extends `git log -L` to support range-scoped diff stat formats (`--stat`, `--numstat`, `--shortstat`), whitespace checking (`--check`), and the `-G` pickaxe. Junio’s review of patch 5 (range-scoped diff stat) suggested minor documentation rewording, which Michael Montalbo agreed to. The implementation is approved in principle and touches `diff.c`, `diffcore-pickaxe.c`, and `revision.c`.

**GSoC updates: `git repo` and partial-clone disk recovery** -- Two GSoC participants shared progress updates. K Jayatheerth’s `git repo` command development remains in the research/design phase, with no technical details shared on-list. Siddharth Shrimali’s work on improving disk space recovery for partial clones continues to prototype promisor object pruning, targeting the `git maintenance` task scheduler and promisor packfile machinery.

**Submodule fetch fails on dangling commits** -- Mike Crowe reported a bug where `git fetch --recurse-submodules` fails with a hard error when encountering a submodule commit that doesn’t exist on the remote, even if the commit isn’t referenced by the current tracking branch. The issue disrupts CI workflows, and Mike suggests two potential solutions: limiting recursive fetches to commits referenced by the current branch or treating submodule fetch failures as non-fatal. The thread is in the problem-statement phase, with no patches yet.

**GPG interface bugfix addresses over-aggressive `\r` stripping** -- DSAntonio08’s patch fixes a helper function in `gpg-interface.c` that was stripping *all* `\r` characters from GPG signature output, even when they were legitimate parts of the payload. Junio’s review identified a logic bug (failing to preserve a lone `\r` at the end of the buffer) and requested historical context in the commit message. The patch is blocked on addressing these issues.

---

## On the radar

**`--track=fetch` workflow debate** -- Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains in limbo despite technical approval. Junio’s sustained reservations about network operations during checkout without user preview suggest the series may stall unless broader consensus emerges. Phillip Wood’s request for clearer workflow justification has been addressed, but the maintainer’s final decision is pending.

**ODB abstraction connectivity-check refactoring** -- Patrick Steinhardt’s `ps/connected-generic-promisor-checks` series awaits a reroll to address Christian Couder’s feedback on `odb_reprepare()` removal and control flow complexity. The series is foundational for ODB abstraction but carries regression risk if the promisor object search logic is not airtight.

**`git history drop` replay safety concerns** -- Phillip Wood and Elijah Newren’s reviews of Patrick Steinhardt’s `git history drop` series (10 commits) raised replay safety concerns, particularly around how descendants are replayed onto a commit’s parent. The series is marked “expecting reroll” in Junio’s report, with v4 needed to address these issues.