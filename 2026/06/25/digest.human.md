# The Git Mailing List Daily Digest for 2026-06-25

## The day in brief.

A heavy day on the list with 66 emails across 13 threads. The standout event: Pablo Sabater’s 14-patch `git cat-file --batch-command` security series reached final readiness and is poised for merging after 14 iterations. Other highlights: Patrick Steinhardt’s ref backend lazy-loading refactor was merged into `next`, Harald Nordgren’s `--track=fetch` debate took a philosophical turn, and Junio Hamano approved a long-overdue documentation fix for `git log --no-follow`.

---

## Notable threads

### `git cat-file --batch-command` security series reaches final readiness
Pablo Sabater’s 14-patch series implementing `git cat-file --batch-command` support for querying remote object metadata (initially just object sizes) without full downloads is now ready for merging. The series introduces a new `remote-object-info` command that lets clients request metadata for up to 10,000 objects in a single command, reducing network overhead compared to the existing `info` command.

Key improvements in v14 include dynamic capability-based validation of format placeholders, memory safety fixes (including a leak in `disconnect_helper()`), and a new `strtoul_szt()` helper for safe `size_t` conversion. The series has addressed all prior review feedback from Junio Hamano, Karthik Nayak, Jonathan Tan, and Christian Couder, with only a minor documentation nit remaining. The implementation is security-hardened, with strict protocol v2 enforcement, input validation, and comprehensive test coverage (680 lines in `t1017-cat-file-remote-object-info.sh`).

The series is a significant step forward for partial clone workflows and represents a well-executed GSoC project that has navigated complex protocol and security requirements.

---

### Ref backend lazy-loading refactor merged into `next`
Patrick Steinhardt’s 11-patch series modernizing Git’s reference backend infrastructure was merged into Junio Hamano’s `next` branch. The series implements a lazy-loading design to resolve recursive initialization issues caused by `includeif.onbranch` conditions, replacing an earlier workaround with deferred write-config parsing until the first write operation.

The redesign touches 34 files, including core ref backends (`files-backend.c`, `reftable-backend.c`), repository setup, and new tests for "onbranch" edge cases. Key changes include dynamic write option passing for the reftable backend, a recursion guard in `get_main_ref_store()`, and fixes for latent memory leaks in the `chdir_notify` subsystem. The series was architecturally approved by Jeff King and received final approval from Justin Tobler after addressing all review feedback.

This is a foundational improvement for ref backend modularity and config system interactions, with no user-visible behavior changes unless `includeif.onbranch` is used. The series is poised to graduate to `master` in the next integration cycle.

---

### `--track=fetch` debate takes philosophical turn
Harald Nordgren’s `--track=fetch` feature series, which would automatically fetch from the remote when creating a tracking branch, remains stalled on workflow concerns despite being technically complete. The debate took a philosophical turn today as Harald challenged Git’s current workflow norms, arguing they are overly idealized and out of touch with the chaotic reality of most development environments.

Harald framed the feature as a pragmatic concession to common practice, citing his own experience as an individual contributor where stale branches routinely caused pain. Kristofer Karlsson countered by praising Git’s design as a clean foundation of composable primitives, suggesting the feature could be implemented as a Git alias rather than cluttering the core. D. Ben Knoble added nuance by sharing anecdotal evidence from corporate environments where developers routinely skip inspecting upstream changes when forking, instead relying on frequent fetches and rapid integration.

The thread remains unresolved, with Junio Hamano yet to respond to the latest reframing. The series is technically approved but lacks the broader consensus needed for merging.

---

### `git log --no-follow` documentation finally merged
Tamir Duberstein’s four-iteration documentation series clarifying the behavior of `git log --no-follow` and its interaction with `log.follow` was merged into `next`. The patch documents the long-standing but previously undocumented `--no-follow` option, which has existed since 2012, and establishes the CLI/config precedence hierarchy (`--no-follow` overrides both `--follow` and `log.follow`).

The series resolves a long-standing gap in Git’s documentation and represents a reference case for technical documentation reviews. Junio Hamano approved the final version with only a minor stylistic preference noted but not enforced. The patch touches only documentation files (`Documentation/git-log.adoc` and `Documentation/config/log.adoc`) and includes references to the original implementation commits.

---

### `git history drop` series nears completion
Patrick Steinhardt’s 10-patch series adding a `git history drop` subcommand to remove commits from history and replay their descendants reached its sixth iteration. The series modernizes the reset API, advances the `the_repository` removal effort, and improves reference update architecture.

All review feedback has been addressed, with only one medium-weight concern remaining: code duplication between `compute_pending_ref_updates()` (in `builtin/history.c`) and `replay_result_queue_update()` (in `replay.c`). Christian Couder flagged this as a maintainability concern, but the series is otherwise technically complete and approved. Junio Hamano marked the series as "Waiting for response(s) to review comment(s)" in his tracking system.

The series is a significant addition to the experimental `git history` command and represents a well-executed architectural improvement.

---

## In brief

**Promisor object connectivity checks generalized** -- Patrick Steinhardt’s four-patch series refactoring Git’s connectivity checks to search for promisor objects generically (rather than relying on packfile internals) was approved by Junio Hamano. The series is a critical step for the ODB abstraction effort and enables backend-agnostic connectivity checks.

**`excludes_file` libification** -- Tian Yuchen’s two-patch series moving the global `excludes_file` string into `struct repo_config_values` as part of the libification effort received substantive review from Junio Hamano. The series introduces proper heap memory management and a cleanup function, with two mechanical issues identified for the next iteration.

**TAP compliance regression fixed** -- A no-op test in `t/t4216-log-bloom.sh` that broke TAP compliance on non-x86 architectures was removed, resolving a regression that blocked the Git 2.55.0 release. Junio Hamano merged the deletion patch, prioritizing the regression fix over a debate about the test’s original intent.

**`git history` file descriptor leak fixed** -- Toon Claes’s patch fixing a file descriptor leak in `git history reword` and `git history fixup` received a substantive review from Junio Hamano, who proposed a more thorough refactoring to eliminate redundant file operations. The discussion now centers on code quality and maintainability.

**SSH signature inspection relaxed** -- A new patch allows SSH signature inspection without requiring `gpg.ssh.allowedSignersFile` to be configured. The change enables users to see basic signature details (key type, fingerprint, validity) while making principal validation optional, aligning with the principle of progressive enhancement.

**SubmittingPatches guidance updated** -- Weijie Yuan’s documentation patch codifying Git’s implicit patch review norms was fully approved and is ready for merging. The update advises a default reroll cadence of no more than one per day and encourages explicit signaling of invalidated parts when substantial rework is needed.

**Usability improvements for branch/push mistakes** -- Harald Nordgren’s two-patch series improving error messages for common command-line mistakes (`git branch --set-upstream-to=origin main` and `git push origin/main`) is ready for merging. The series introduces new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and is uncontroversial.

---

## On the radar

**`git history drop` code duplication** -- The only remaining concern for Patrick Steinhardt’s `git history drop` series is code duplication between `compute_pending_ref_updates()` and `replay_result_queue_update()`. The decision on whether to refactor the duplicate logic or document the intentional duplication will determine the series’ final readiness.

**`--track=fetch` workflow debate** -- The philosophical debate about whether Git should prioritize the needs of users in fast-paced environments or enforce an idealized workflow remains unresolved. Junio Hamano’s response to the latest reframing will determine whether Harald Nordgren’s `--track=fetch` series is merged or abandoned.

**SSH signature inspection fallback** -- The new patch relaxing SSH signature inspection requirements has not yet received review feedback. The change is uncontroversial but may prompt discussion about whether the hint message is sufficient or if additional documentation is needed.