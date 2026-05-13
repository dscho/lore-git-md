# Git Mailing List Daily Digest — 2026/05/12 (Tuesday)

**The day in brief.** A busy Tuesday with 123 emails across 42 active threads. The most consequential developments are the graduation of the sideband sanitization fix to `master` (a multi-year security effort now getting real-world testing in its strict form), and the near-completion of the pseudo-merge bitmap bugfix series, which Junio has queued for `next`. Several long-running series — including the `git url-parse` command, the `git history fixup` subcommand, and the fsmonitor Linux daemon — all received green lights to move forward. The day also saw a substantive design debate on the `git branch --prune-merged` safety model, with Junio questioning whether the upstream-fallback logic is defensible at all.

---

## What's cooking: sideband sanitization graduates, large batch moves to `master`

Junio C Hamano's biweekly "What's cooking" report covers roughly 60 topics across the integration branches. The most notable change is the graduation of `jc/neuter-sideband-fixup` to `master` — the multi-year effort to sanitize escape sequences received over the sideband during push/fetch, which could be used for terminal injection attacks. Junio explicitly graduated the *stricter* form (masking control characters by default) ahead of a looser "allow everything for now" final step, giving the stricter form real-world testing before potentially dialing it back at the Git 3.0 boundary. He describes the graduated topic as an "experiment."

A large batch of topics moved from `next` to `master` for the first time since 2.54-rc2, indicating the project is accelerating again after a quiet period. Taylor Blau's `tb/pseudo-merge-bugfixes` (9 commits) and `tb/incremental-midx-part-3.3` (16 commits) are both queued for `next`. Patrick Steinhardt's `ps/odb-in-memory` (18 commits) is waiting on its base topic to be leakfixed. The `hn/branch-prune-merged` series is noted as needing a reroll following Junio's comments. A handful of topics were discarded for inactivity, none apparently controversial.

---

## `git url-parse` series cleared for `next`

The 13-patch series adding a new `git url-parse` builtin command — which exposes Git's internal URL parsing logic as a plumbing tool — has received the final sign-off needed to graduate from `seen` to `next`. Junio C Hamano explicitly asked the list whether everyone was satisfied, and Torsten Bögershausen, a long-time contributor with particular expertise in platform compatibility, replied with explicit approval. The series has a known and documented gap: Windows UNC path reconstruction (`file://server/share/repo`) is deferred to follow-up work, with the current parser returning the path component as-is and relying on downstream code in `connect.c`. Torsten's sign-off carries particular weight because Windows path handling was the domain most affected by the deferral. No objections have been raised.

---

## `git history fixup` gets maintainer green light

Patrick Steinhardt's three-patch series adding a `git history fixup <commit>` subcommand — which amends staged changes to an existing commit while automatically rebasing dependent branches — is on track for `next`. Junio sent a procedural nudge to Elijah Newren, D. Ben Knoble, and Tian Yuchen asking whether the topic should be marked for merging, noting that v2 and v3 received no review comments and that his own cursory look found nothing curious. Patrick replied that he had no plans for another iteration and thanked Junio — an explicit green light. The series has had very light review scrutiny overall; correctness beyond the empty-commit infrastructure and documentation framing has not been independently verified by anyone outside the author and maintainer.

---

## fsmonitor Linux series cleared for re-promotion

The 15-iteration cross-platform filesystem monitoring (fsmonitor) series, temporarily reverted from `next` to `seen` due to a Gentoo build failure, has been formally cleared for re-promotion. D. Ben Knoble confirmed in response to Junio's request for a verdict that the only issue was a downstream Gentoo packaging conflict (a local patch clash), not an upstream bug, and that he would not hold the series back from `next` for Gentoo's sake. Junio now has the confirmation he asked for, and the series — which implements comprehensive fsmonitor support with Linux inotify, Windows compatibility, and enhanced process isolation — is expected to return to `next` absent a new issue.

---

## `git branch --prune-merged`: safety model under debate

Harald Nordgren's v8 series adding `git branch --prune-merged` for safe automated local branch cleanup arrived just one day after v7, directly addressing Junio C Hamano's design concerns. The core redesign replaces the v7 approach (checking whether a branch's push destination still exists on the remote) with a fundamentally different safety model: a local branch is safe to delete only if its tip is reachable from its configured upstream remote-tracking branch. The previous fallback-to-remote-default logic that Junio questioned is gone entirely.

However, Junio's review of v7 raised deeper questions that v8 may not fully resolve. He argued that the entire upstream-fallback rationale is questionable: a remote's default branch is not something you expect to disappear, and if it did, falling back to some other arbitrary branch cannot provide safety. He also questioned the `no_head_fallback` parameterization of `branch_merged()`. The series has been through eight iterations and the test coverage is thorough, but the maintainer's concerns go to the heart of whether the feature's safety model is sound. A separate `fetch.pruneLocalBranches` config proposal that appeared in the same thread — offering a simpler "nuclear option" of deleting any local branch whose upstream is gone — was thin and under-motivated, and does not advance the core design debate.

---

## Pseudo-merge bitmap bugfixes queued

Taylor Blau's nine-patch series fixing multiple compounding bugs in the pseudo-merge bitmap implementation has been accepted and queued by Junio. The series addresses a binary-search sort order, a swapped comparator, extended-table offset/parsing errors, group selection logic that rendered `stableThreshold` and `sampleRate` configuration inert, a latent division-by-zero crash, stale documentation, and a memory leak. The existing test suite never exercised `apply_pseudo_merges_for_commit()` before this series. Junio signaled the series was "so close to the finish line" in his draft "What's cooking," and the v3 test-hygiene fixups (ordering `test_when_finished` above `git init`, avoiding Git on the left side of a pipe) were the final polish. The series is now in `next`.

---

## Commit-signing regression fix queued

brian m. carlson's two-patch series fixing a commit-signing regression introduced in Git v2.45.0 and present through at least v2.53.0 has been queued for `next` by Junio. The regression caused signatures to be computed over pre-UTF-8-transcoding content, rendering signed commits with non-ASCII messages unverifiable if no `i18n.commitEncoding` was configured. The fix reorders operations in `commit_tree_extended()` so that UTF-8 validation and transcoding run *before* signing, and also fixes a latent logical short-circuit bug in the original code. The fix covers all signature backends (GnuPG, SSH, X.509). Elijah Newren provided thorough substantive review in the previous round.

---

## In brief

- **Hierarchical alias regression fix queued** — Junio has marked the topic for `next` that restores backward compatibility for dotted aliases (e.g., `pull.sub`) broken in Git 2.54.0. The fix preserves the new three-level alias syntax while treating dotted aliases as a legacy feature with possible future deprecation post-3.0.

- **`git log --follow` subtree merge fix** — Miklos Vajna submitted a single-patch bugfix making `git log --follow` correctly follow files across merge commits when the file first appeared via a subtree merge. The fix mirrors the existing handling in `blame.c`.

- **`git status` push-branch advice fix** — A single patch improves the `git pull` advice shown by `git status` when a local branch is compared against a push branch, quoting the correct remote and branch rather than suggesting a bare `git pull` that would follow the upstream.

- **`git maintenance --detach` lockfile fix** — Patrick Steinhardt's two-patch series fixing a real bug where `git maintenance run --detach` was not properly holding the lockfile, allowing concurrent processes. Taylor Blau has advocated for a different implementation strategy (folding the ownership transfer into `daemonize()` automatically), and the design debate is ongoing.

- **`gc.auto=0` regression fix** — Patrick Steinhardt's second patch restores the correct behavior where setting `gc.auto=0` suppresses automatic maintenance, broken when the default maintenance strategy stopped shelling out to `git gc`. Both Jeff King and Junio independently confirmed the bug.

- **`strbuf_add_uint()` series** — René Scharfe's four-patch series replacing `strbuf_addf()` calls with a specialized `strbuf_add_uint()` function in `cat-file`, `ls-files`, and `ls-tree` received positive review from Jeff King, who endorsed the change as both a performance win and a readability improvement. Benchmarks show 4-7% speedup on `cat-file --batch-all-objects`.

- **`git log -L` line-log refactoring** — Michael Montalbo's three-patch series making `git log -L` flow through the standard diff output pipeline remains awaiting first review. Junio noted the lack of review activity with concern.

- **`includeIf worktree:` config conditions** — Chen Linxuan's v3 series adding `worktree` and `worktree/i` `includeIf` conditions received full review from Patrick Steinhardt (three action items: documentation dedup, two additional tests) and a motivation question from Phillip Wood that Junio has now endorsed as a formal requirement for v4.

- **`git diff --word-diff` documentation** — Michael Montalbo proposed adding a paragraph to the documentation explaining that word diff first computes a line-level diff, which determines the word-level output. The original bug reporter confirmed the clarification would address his concern.

- **`http-walker.c` alternate URL parsing fix** — Jeff King fixed a bug from 2005 where `strchr()` returns NULL when a URL has no path component, producing a garbage length value. One-character change: `strchr()` to `strchrnul()`.

- **RFC 2047 encoder overflow fix** — Jeff King removed a single `strbuf_grow()` line from `pretty.c` that could overflow on 32-bit systems with extremely large inputs. The line was a pure optimization from 2007 that is no longer needed.

- **Worktree-shared exclude file documentation** — D. Ben Knoble's v3 patch corrects all stale references to `$GIT_DIR/info/exclude` to `$GIT_COMMON_DIR/info/exclude` across six files, aligning documentation with Git's actual behavior since 2.20.

- **`git log --decorate` formatting fix** — Kristoffer Haugsbakk's two-patch series fixing AsciiDoc formatting in the `--decorate` description list was accepted by Junio as "a solid improvement" and marked for `next`.

- **Negative diff option values fix** — Michael Montalbo's four-patch series rejecting negative values for `-U` and `--inter-hunk-context` is complete and ready for merging. The fix converts both options to unsigned types and adds a defensive `BUG()` guard in xdiff.

- **`rebase --update-refs` bugfix** — The v3 patch fixing incorrect update-ref instructions when `rebase.instructionFormat` includes `%d` decorations has Phillip Wood's Reviewed-by and Junio's confirmation that the approach is correct. A parallel code path in `make_script_with_merges()` still needs the same fix.

- **`git clone --dissociate` commit-graph crash** — Daniel Mach at SUSE reported a crash when `git clone --dissociate` is used with a reference repository that has a commit-graph. Jeff King confirmed this is the same bug he independently discovered, and outlined two candidate fix approaches but has not submitted a polished patch.

- **Merge-base optimization queued** — Kristofer Karlsson's two-patch series speeding up single merge-base calculation by exiting `paint_down_to_common()` early was accepted by Junio with "Very nicely done and well described." Reported 100x speedup (5s to 24ms) in a 2.2M-commit monorepo.

- **Octopus merge up-to-date check refactored** — Kristofer Karlsson replaced a roundabout `repo_get_merge_bases()` call in the octopus merge path with the simpler `repo_in_merge_bases()`, a -14/+4 line cleanup that also provides a small performance win on deep histories.

- **Unused `commit` field removed** — A single-patch cleanup removing a `commit` field from `struct todo_add_branch_context` in `sequencer.c` that has been dead code since it was introduced in 2022. Patrick Steinhardt confirmed the field was never read.

- **`git history` merge support RFC** — Johannes Schindelin's RFC series adding merge commit support to `git history` received substantive review from Phillip Wood (who identified a silent-conflict edge case when one parent is rebased) and design feedback from Toon Claes on the test infrastructure DSL.

- **`git add -p` word-diff interface** — The interface design discussion reached a decision point: Junio accepted the conceptual framing of D. Ben Knoble's `:` prefix proposal for exempting commands from single-key mode, but asked whether the machinery is worth building for the word-diff use case alone. The burden of proof is now on proponents to justify the complexity.

- **`git pack-objects --path-walk --filter` series** — Taylor Blau, co-author of the 12-patch series, gave a social sign-off stating he has read the full series and expects v4 to be "pretty close to the finish line." The series has received zero independent review from anyone outside the two co-authors.

- **ODB write-stream series** — Justin Tobler acknowledged Jeff King's finding of a file descriptor leak in the ODB write-stream series and will send another version to fix it. The series is otherwise approved by Patrick Steinhardt.

- **`git push` remote group support** — Usman Akinyemi's v5 series adding remote group support to `git push` received only documentation formatting nits from Kristoffer Haugsbakk. The series appears architecturally settled and ready for merging.

- **Promisor repack series blocked on expert review** — Lorenzo Pegorari's GSoC promisor repack series is functionally ready but blocked on review from Christian Couder, the domain expert for the promisor-remote subsystem. Junio explicitly CC'd Christian to draw his attention to the work.

- **Negotiation control series** — Derrick Stolee's v3 series adding `--negotiation-include` and `--negotiation-restrict` to fetch/push received thorough review from Matthew Cheetham, who identified a COMMON-flag abstraction breakage that Stolee plans to fix with a new negotiator callback mechanism. The v4 scope has expanded to include structural rework of the negotiator abstraction.