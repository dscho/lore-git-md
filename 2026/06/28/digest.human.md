# The Git Mailing List Daily Digest for 2026/06/28

**The day in brief.** A Sunday with moderate traffic (44 emails, 16 threads) that leaned technical: a merged-but-regressed `git replay --linearize`, a security-hardened `git cat-file --batch-command` series reaching consensus, and a performance optimization for `paint_down_to_common()` that finally lands. The one thing you should not miss is Johannes Schindelin’s report of silent commit dropping in `git replay --linearize`—a post-merge regression that needs urgent attention.

---

## Notable threads

### `git replay --linearize` silently drops commits after merge
Johannes Schindelin reports that the recently merged `--linearize` option for `git replay` (v5) **silently drops commits** when replaying a single branch that contains merge commits. In a real-world example (`master~2..master`), only the tip commit is replayed, and the “Git 2.55-rc2” commit is lost. The regression stems from the removal of the `replayed_base` logic in `pick_regular_commit()`; the fix will need to restore that mechanism or redesign how multi-branch histories are handled. Toon Claes is expected to send a follow-up patch before the next release.

---

### Remote object metadata queries: `git cat-file --batch-command` v14
Pablo Sabater’s GSoC series implementing `git cat-file --batch-command` for remote object metadata queries reaches **consensus on protocol extensibility** and resolves the last commit-message nits. The series is now **technically complete** (13 patches, 680 lines of new tests) and ready for Junio’s final assessment. Key decisions: future metadata features (e.g., `objecttype`) will be appended to the same space-separated `object-info` capability value, and the client will silently return empty strings for unsupported fields. The only remaining open question—whether the client should fail explicitly or continue silently when metadata is missing—has no consensus, but Pablo has agreed to document the behavior clearly.

---

### `paint_down_to_common()` optimization lands
Tian Yuchen’s eight-patch series optimizing `paint_down_to_common()` for one-sided histories is **fully approved and ready to merge**. The series delivers **100-1000x speedups** for asymmetric merge-base queries by terminating early when one side’s commit queue is exhausted. It includes a regression fix, code clarity improvements, a new technical document (`paint-down-to-common.adoc`), expanded edge-case test coverage, trace2 instrumentation, and dead-code removal. Derrick Stolee’s final approval removes the last open question; the series is now regression-free and fully reviewed.

---

### `greplint.pl` exposes systemic test-suite fragility
Junio C Hamano and Michael Montalbo debug why the `greplint.pl` tool preserves latent test bugs. The root cause is now clear: the linter’s `# lint-ok:` comments bypass `test_grep`’s file-existence check, masking incorrect assertions like `! grep dirty file3` in `t3420-rebase-autostash.sh`. The maintainer demands an audit of all `# lint-ok:` comments to ensure they do not hide similar issues. The thread has shifted from tooling improvements to **test correctness**, exposing a tension between automation and diagnostic rigor.

---

### `git history squash` v6 completes the feature
Harald Nordgren posts the sixth iteration of `git history squash`, a new subcommand that folds a commit range into its oldest commit while preserving descendant history. The series is now **functionally complete**, addressing all prior feedback: stricter input validation, support for multiple revision arguments (e.g., `@~3.. ^topic`), explicit rejection of merges with external parents, and the `--reedit-message` flag that seeds the editor with every folded-in commit message. The ref-handling logic rejects operations with interior refs by default, advising users to use `--update-refs=head` to retarget only the current branch. The series is ready for Junio’s final review.

---

## In brief

**Worktree diff performance** -- Jeff King (Peff) narrows the edge case for unconditional `USE_NSEC`: kernel inode cache eviction could cause inconsistent nanosecond precision, but the failure mode is benign in one direction and improbable in the other. He leans toward making `USE_NSEC` unconditional on Linux.

**HTTPS proxy regression fix** -- Johannes Schindelin’s two-line patch to accept HTTPS proxy URLs is merged into `master` ahead of Git 2.55. The fix restores the intended behavior after a hardening commit incorrectly rejected supported schemes.

**Reftable memory leak** -- Peff fixes a leak in `reftable_writer_new` where the struct was allocated before input validation. The patch is merged with a trivial conflict resolution.

**CI hangs in `t5551`/`t5559`** -- Jeff King’s three-patch series increases Apache’s `Timeout` directive to 600 seconds and isolates the expensive “many-tags” test case. The ref-packing optimization (patch 3/3) is likely to be dropped as redundant.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is queued, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true` or the knob should be deprecated.

**`excludes_file` migration** -- Tian Yuchen’s patch moving the global `excludes_file` into `struct repo_config_values` is ready for `next` after resolving the guardrail debate: `repo_config_values()` is now the sole arbiter of repository initialization.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass alongside the usual string updates. The pull request covers 11 languages and is ready for merging.

---

## On the radar

**`git replay --linearize` regression** -- The silent commit dropping reported by Johannes Schindelin needs a follow-up patch before the next release. Track Toon Claes’s response.