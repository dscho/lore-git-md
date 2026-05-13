# Git Mailing List Daily Digest — 2026/05/12 (Tuesday)

**The day in brief.** A busy Tuesday with 123 emails across 42 threads, dominated by the final push of several long-running series toward integration. The most consequential developments are Junio C Hamano's "What's cooking" report, which shows a large graduation batch moving topics from `next` to `master` for the first time since 2.54-rc2, and the near-completion of Taylor Blau's pseudo-merge bitmap bugfix series, which Junio has queued. The `git url-parse` series received its final sign-off and appears ready for `next`, while the `git branch --prune-merged` series underwent a fundamental redesign in response to maintainer concerns.

---

## Notable threads

**What's cooking in git.git (May 12, 2026).** Junio C Hamano's biweekly status report covers roughly 60 in-flight topics. The most notable move: `jc/neuter-sideband-fixup` graduated to `master` — the stricter form that masks control characters by default, deliberately given real-world testing before a potential loosening at the Git 3.0 boundary. Taylor Blau's `tb/pseudo-merge-bugfixes` (9 commits) and `tb/incremental-midx-part-3.3` (16 commits) are both queued for `next`. Patrick Steinhardt's `ps/odb-in-memory` (18 commits) is waiting on its base topic to be leakfixed. The `hn/branch-prune-merged` series still needs a reroll following Junio's comments. A handful of stale topics were discarded. The overall picture: the project is accelerating after a quiet post-2.54-rc2 period.

**Pseudo-merge bitmap bugfixes (Taylor Blau, v3, 9 patches).** This series, which fixes multiple compounding bugs in the pseudo-merge bitmap implementation — including a binary-search sort order error, a swapped comparator, extended-table offset/parsing bugs, and a silent classification bug that rendered `stableThreshold` and `sampleRate` configuration inert — has been accepted and queued by Junio. The v3 changes were purely test-hygiene fixes (ordering `test_when_finished` above `git init`, avoiding Git on the left side of a pipe). Junio's "Queued. Thanks" closes the thread. The series also fixes a division-by-zero crash when `sampleRate` is exactly 0 and a memory leak on duplicate pattern config keys.

**`git branch --prune-merged` redesign (Harald Nordgren, v8, 5 patches).** This series underwent a fundamental redesign between v7 and v8 in direct response to Junio C Hamano's v7 review. The core safety model was overhauled: instead of checking whether a branch's push destination still exists on the remote, v8 checks whether the local branch's tip is reachable from its configured upstream remote-tracking branch. The previous fallback-to-remote-HEAD logic that Junio questioned is gone entirely — branches with unresolvable upstream refs are silently skipped. The `--forked` listing mode, the `branch.<name>.pruneMerged` opt-out config, and the `--all-remotes` convenience flag are all updated to match the new semantics. The test suite has been rewritten accordingly. The series now awaits Junio's reaction to the redesigned approach.

**`git url-parse` series ready for `next` (Matheus Afonso Martins Moreira, v3, 13 patches).** This series, which adds a new builtin command exposing Git's internal URL parsing logic as a plumbing tool, received its final sign-off. Torsten Bögershausen — whose platform-compatibility expertise carries particular weight given the deferred Windows UNC path handling — explicitly endorsed merging to `next`. Junio had asked the list whether everyone was satisfied, and no objections have been raised. The series is expected to appear in the next "What's cooking" as a topic in `next`.

**Negotiation controls for fetch/push (Derrick Stolee, v3, 7 patches).** Matthew Cheetham completed his review of all seven patches in this series, which introduces `--negotiation-include` and `--negotiation-restrict` for monorepo scenarios. The most technically significant finding was in patch 5/7: a COMMON-flag abstraction breakage where the patch sets bit 6 (fetch-pack.c's COMMON) but the negotiator checks bit 2, potentially causing duplicate "have" lines on protocol v0/v1. Stolee called this "a great catch" and plans a structural fix — adding a callback function pointer to the negotiator struct itself. He also agreed to unify ref-resolution for both options at the builtin layer. A v4 with expanded scope is expected.

**Maintenance mode lockfile fix (Patrick Steinhardt, v2, 2 patches).** This series fixes a real bug where `git maintenance run --detach` was not properly holding the lockfile, allowing concurrent processes. The v2 approach introduces `daemonize_without_exit()` and `lock_file_reassign_owner()` to transfer lock ownership from the parent to the child process. Taylor Blau reviewed patch 1 and advocates for a different implementation strategy — folding the ownership transfer automatically into `daemonize()` rather than at the single call site — and feels "fairly strongly" about it, offering Co-authored-by if Patrick adopts the alternative. Patch 2 (restoring `gc.auto` integer-threshold behavior) is fully settled.

**`git log --follow` subtree merge fix (Miklos Vajna).** A single-patch bugfix addressing a real inconsistency: `git log --follow` fails to follow a file across merge commits when that file first appeared via a subtree merge, while `git blame --follow` already handles this case correctly. The fix adds a per-parent rename-detection loop in `log-tree.c` mirroring the existing logic in `blame.c`. The patch was re-sent after receiving no replies to v1. Clean, well-scoped, and likely ready for review.

---

## In brief

**`git clone --dissociate` crash with commit-graph** -- Daniel Mach at SUSE reported a crash when `git clone --dissociate --reference <old-repo>` is used with a reference repository that has a commit-graph. Jeff King confirmed this is the same bug he independently discovered, and outlined two candidate fix approaches (lazy fallback vs. aggressive pre-filling) but has not settled on one.

**`strbuf_add_uint()` convenience function** -- René Scharfe sent a 4-patch series adding a specialized function for formatting unsigned integers, replacing `strbuf_addf()` calls in `cat-file`, `ls-files`, and `ls-tree`. Jeff King reviewed and endorsed the change as both a performance win and a readability improvement. Benchmarks show 4-7% speedup on `cat-file --batch-all-objects`.

**`git add -p` word-diff interface design** -- Junio C Hamano responded to D. Ben Knoble's `:` prefix proposal for exempting commands from single-key mode, accepting the conceptual framing but asking whether the machinery is worth building for the word-diff use case alone. The thread is approaching a decision point between the `:` prefix, a single-letter `w` command, or reusing existing `p`/`P` with options.

**`git history fixup` ready for `next`** -- Patrick Steinhardt gave explicit go-ahead for the v3 series to proceed, stating no further iteration planned. Junio had asked Elijah Newren, D. Ben Knoble, and Tian Yuchen for input before marking for `next`; none have responded, but the series appears on track for integration.

**`includeIf worktree:` config conditions** -- Chen Linxuan's v3 series adding `worktree` and `worktree/i` conditions received full review from Patrick Steinhardt (three action items: documentation dedup, two additional tests) and a motivation question from Phillip Wood. Junio endorsed Phillip's concern and said the documentation must help users choose between the new feature and the existing `extensions.worktreeConfig` mechanism.

**Approxidate bugfix series (Tuomas Ahola, v2, 3 patches)** -- This series fixing long-standing edge cases in Git's fuzzy date parser received review from both Junio C Hamano and Jeff King. Junio raised coding-style requests on patch 1/3 and a substantive concern about the "today" alias on patch 2/3. Peff raised interface-design observations on the test helper. A v3 is expected.

**`git status` push-branch advice fix** -- A single patch improves the `git pull` advice shown by `git status` when a local branch is compared against a push branch, quoting the correct remote and branch instead of suggesting a bare `git pull` that would follow the upstream.

**`http-walker.c` absolute alternate URL parsing fix** -- Jeff King fixed a bug dating to 2005 where `strchr()` returns NULL when a URL has no path component, producing a garbage length. One-character change: `strchr()` to `strchrnul()`.

**`git diff --word-diff` documentation clarification** -- Michael Montalbo proposed adding a paragraph to the documentation explaining that word diff first computes a line-level diff, so whitespace-only changes can produce word-level output. The original bug reporter confirmed the clarification would address his concern.

**`git log -L` line-log refactoring** -- Michael Montalbo's 3-patch series making `git log -L` flow through the standard diff output pipeline received no reviews. Junio noted the lack of review activity and expressed concern about community review bandwidth.

**`git history` merge commit support (RFC)** -- Johannes Schindelin's RFC series adding merge commit support to `git history` received substantive review from Phillip Wood (silent new conflicts when one parent is rebased) and design feedback from Toon Claes on the test DSL syntax.

**`git branch --forked` listing mode** -- Harald Nordgren's patch 1/5 introducing `--forked <remote>` for listing local branches tracking a given remote is unchanged from v7 and functions as infrastructure for the `--prune-merged` feature.

**`fetch.pruneLocalBranches` config proposal** -- Harald Nordgren sent a standalone patch proposing a "nuclear option" config to delete local branches after fetch when their upstream is gone. The patch arrived in the middle of the `--prune-merged` thread where Junio just rejected the safety model of the related work, and appears to be a brainstorming sketch rather than a serious proposal.

**`git log --decorate` documentation fix** -- Kristoffer Haugsbakk's two-patch series fixing AsciiDoc formatting in the `--decorate` description list was accepted by Junio and marked for `next`.

**Worktree-shared exclude file documentation** -- D. Ben Knoble's v3 patch corrects all stale references to `$GIT_DIR/info/exclude` to `$GIT_COMMON_DIR/info/exclude` across six files, aligning documentation with Git's actual behavior since 2.20.

**`rebase --update-refs` with `rebase.instructionFormat`** -- The v3 bugfix for this issue has Phillip Wood's Reviewed-by and Junio's confirmation that the approach is correct. A parallel code path in `make_script_with_merges()` still needs the same fix, but the core patch is ready.

**`paint_down_to_common` merge-base optimization** -- Kristofer Karlsson's v4 series optimizing single merge-base calculation was accepted and queued by Junio, who called the optimization patch "Very nicely done and well described."

**`git merge` octopus up-to-date check refactoring** -- Kristofer Karlsson sent a single-patch refactoring replacing a roundabout `repo_get_merge_bases()` call with `repo_in_merge_bases()` in the octopus merge path. Clean, well-explained, likely to be picked up quickly.

**`strbuf_add_uint()` implementation patch** -- René Scharfe's patch 1/4 adding the helper function itself. Jeff King reviewed and confirmed the on-stack buffer approach is reasonable after benchmarking alternatives.

**`cat-file` objectsize formatting** -- René Scharfe's patch 2/4 switching `cat-file` to `strbuf_add_uint()`. Peff endorsed it as both a performance win and a readability improvement.

**`ls-files` objectsize formatting** -- René Scharfe's patch 3/4 switching `ls-files --format='%(objectsize)'` to `strbuf_add_uint()`. ~1.5% speedup on Linux kernel tree.

**`ls-tree` objectsize formatting** -- René Scharfe's patch 4/4 switching `ls-tree` to `strbuf_add_uint()`. ~1% speedup on `ls-tree -r`.

**`pretty.c` RFC 2047 overflow removal** -- Jeff King removed a single `strbuf_grow()` line that could overflow on 32-bit systems with extremely large inputs. The line was a dead optimization from 2007.

**`sequencer.c` dead code removal** -- A single patch removing an unused `commit` field from `struct todo_add_branch_context` that has been dead since the struct was introduced in 2022. Patrick Steinhardt confirmed the field has never been read.

**`git diff -U` and `--inter-hunk-context` negative value fix** -- Michael Montalbo's 4-patch series fixing a bug where negative values were silently accepted and produced malformed hunk headers. All four patches technically approved and ready for merging.

**Promisor file handling during repacks** -- Lorenzo Pegorari's v6 series is blocked on expert review from Christian Couder, who Junio explicitly CC'd on the cover letter. No response from Christian yet.

**ODB write-stream file descriptor leak** -- Justin Tobler acknowledged Jeff King's finding of a file descriptor leak in `odb_source_loose_write_stream()` error paths and will send another version to fix it.

**`git checkout --track=fetch` (v9)** -- Harald Nordgren's ninth iteration addresses both issues from Junio's v8 review: hierarchical remote name parsing and control flow simplification. Junio has not yet responded to v9.

**`git branch --forked` (patch 1/5, v8)** -- Unchanged from v7. Infrastructure patch for the `--prune-merged` series.

**`git branch --prune-merged` warning infrastructure (patch 2/5, v8)** -- Mechanical refactoring of `delete_branches()` to support bulk deletion with summarized warnings. Unchanged from v7.

**`git branch --prune-merged` core logic (patch 3/5, v8)** -- The redesigned safety model: checks upstream reachability instead of push-destination existence. No fallback when upstream is gone.

**`branch.<name>.pruneMerged` config (patch 4/5, v8)** -- Per-branch opt-out that `--force` does not override. Updated for v8 semantics.

**`--all-remotes` flag (patch 5/5, v8)** -- Convenience flag for operating on all configured remotes. Updated for v8 semantics.

**`git maintenance --detach` lockfile fix (patch 1/2, v2)** -- Introduces `daemonize_without_exit()` and `lock_file_reassign_owner()`. Taylor Blau advocates for folding the transfer into `daemonize()` automatically.

**`gc.auto` threshold fix (patch 2/2, v2)** -- Restores correct integer-threshold behavior after the transition from `git gc` to `git maintenance`. Fully settled.

**`git log --follow` subtree merge fix** -- Single patch adding per-parent rename detection in merge commits. Mirrors existing `blame.c` logic.

**`git clone --dissociate` crash with commit-graph** -- Bug report with reproducer. Jeff King confirmed the bug and outlined two candidate fix approaches.

**`strbuf_add_uint()` series (4 patches)** -- René Scharfe. Reviewed and endorsed by Jeff King. Ready for maintainer pickup.

**`git add -p` word-diff interface** -- Junio asked whether the `:` prefix machinery is worth building for word-diff alone. Thread approaching decision point.

**`git history fixup` ready for `next`** -- Patrick Steinhardt gave explicit go-ahead. Awaiting responses from CC'd reviewers.

**`includeIf worktree:` config conditions** -- Full review from Patrick Steinhardt. Motivation question from Phillip Wood endorsed by Junio. V4 expected.

**Approxidate bugfix series** -- Junio and Peff reviewed. Coding-style and interface-design concerns raised. V3 expected.

**`git status` push-branch advice** -- Single patch quoting correct remote and branch in `git pull` hint. Clean and well-scoped.

**`http-walker.c` alternate URL parsing** -- One-character fix from Jeff King for a 2005-era bug.

**`git diff --word-diff` documentation** -- Proposed clarification paragraph. Original reporter confirmed it would address his concern.

**`git log -L` line-log refactoring** -- No reviews received. Junio noted lack of community review bandwidth.

**`git history` merge commit support (RFC)** -- Phillip Wood raised silent-conflict edge case. Toon Claes gave DSL design feedback.

**`fetch.pruneLocalBranches` config proposal** -- Standalone sketch arriving in the middle of the `--prune-merged` design debate. Not a serious proposal in current form.

**`git log --decorate` documentation fix** -- Accepted by Junio, marked for `next`.

**Worktree-shared exclude file documentation** -- V3 expands scope to six files. Ready for merge.

**`rebase --update-refs` with `rebase.instructionFormat`** -- Core fix ready. Parallel code path still needs the same fix.

**`paint_down_to_common` merge-base optimization** -- Accepted and queued by Junio.

**`git merge` octopus up-to-date check refactoring** -- Single patch, clean and well-explained.

**`strbuf_add_uint()` implementation** -- Patch 1/4. Reviewed by Peff.

**`cat-file` objectsize formatting** -- Patch 2/4. Endorsed by Peff.

**`ls-files` objectsize formatting** -- Patch 3/4. ~1.5% speedup.

**`ls-tree` objectsize formatting** -- Patch 4/4. ~1% speedup.

**`pretty.c` RFC 2047 overflow removal** -- Single patch from Jeff King. Trivially correct.

**`sequencer.c` dead code removal** -- Single patch. Patrick Steinhardt confirmed the field has never been read.

**`git diff -U` negative value fix** -- 4-patch series. All patches technically approved.

**Promisor file handling during repacks** -- Blocked on Christian Couder's review.

**ODB write-stream file descriptor leak** -- Acknowledged by Justin Tobler. Fix pending.

**`git checkout --track=fetch` (v9)** -- Awaiting Junio's response.

---

## On the radar

**`git clone --dissociate` crash with commit-graph** -- Jeff King has confirmed the bug and described two candidate approaches but has not submitted a polished patch. The thread is at the "root cause confirmed, design decision pending" stage. Anyone tracking clone reliability should watch for a fix patch in the coming days.

**`git history` merge commit support** -- Phillip Wood's silent-conflict edge case (when one parent is rebased and the other is not, new conflicts not overlapping the original ones are silently accepted) remains unresolved. The RFC series is not yet queued, and this correctness concern will need to be addressed before it can progress beyond `seen`.