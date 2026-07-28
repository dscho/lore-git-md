## The day in brief

**2026-07-27** was a busy day on the Git mailing list, with **52 emails across 19 threads**. The day’s traffic was a mix of **bugfixes, design discussions, and procedural follow-ups**, with two threads standing out: the **`git replay --linearize` design debate** and the **`git stash reword` implementation safety review**. Both threads saw significant progress, though neither has reached a final resolution. Other notable activity included a **pack-bitmap bugfix**, a **UTF-8 string-width refactoring withdrawal**, and a **performance regression discussion** that shifted toward a major architectural redesign.

---

## Notable threads

### `git replay --linearize` design debate reaches a crossroads
**Headline:** *Toon Claes proposes `--ref` syntax to resolve multi-branch ambiguity in `git replay --linearize`*

The `git replay --linearize` series, which adds a `--linearize` option to flatten merge commits, remains stalled over the **multi-branch ambiguity**—how the command should behave when multiple branches are specified. Toon Claes’s latest email proposes a **new `--ref` syntax** (e.g., `git replay --linearize --onto master branchA branchB --ref branchA`) to explicitly specify which ref to update, eliminating emergent reachability (where commits from one branch become reachable from another). The proposal is a middle ground between Junio’s defense of the status quo (concatenating all branches into a single linear chain) and Elijah Newren’s call to restrict `--linearize` to a single positive ref.

Toon’s email is a **design exploration**, not a patch, and the ball is now in Elijah’s and Junio’s courts to weigh the trade-offs. The thread’s next steps will likely hinge on whether the maintainers accept the `--ref` syntax as a reasonable compromise or push for a more restrictive approach. The series remains **queued in `next` but on hold**, with no clear timeline for resolution.

---

### `git stash reword` implementation safety review escalates
**Headline:** *Junio C Hamano identifies data-loss risk in `git stash reword`’s reflog rewrite strategy*

The `git stash reword` feature, which allows users to edit stash messages after creation, hit a **major roadblock** when Junio C Hamano identified a **data-loss risk** in its implementation. The current approach slurps the entire reflog into memory, deletes it on-disk, and rewrites it with a single entry modified—creating a critical window where the reflog exists only in memory. If the process is killed after `refs_delete_reflog()` but before the transaction commits, the reflog is permanently lost.

Junio proposes extending the reflog API to support **atomic replacement of individual entries**, which would eliminate the risk and make the operation truly atomic. The discussion has shifted from "does this feature make sense?" (resolved in favor of the feature) to "how do we implement it safely?" This is a **significant escalation** in review depth, and the patch is now blocked until the API extension is designed and implemented. Emin Özata’s v2 patch, which addressed earlier atomicity concerns, is now **invalidated by this correctness issue**.

---

### Pack-bitmap bugfix addresses off-by-one edge case
**Headline:** *David Lin fixes redundant work in `find_objects()` for objects at bitmap position zero*

A **subtle bug** in the pack-bitmap subsystem, where objects at position zero in the base bitmap were incorrectly excluded from being recognized as already represented, was fixed by David Lin. The patch changes a conditional check from `pos > 0` to `pos >= 0` in `find_objects()`, eliminating redundant work (reloading reachability bitmaps or adding to the not-mapped list). Taylor Blau’s review confirmed the fix’s correctness and suggested expanding test coverage to include non-pseudo-merge scenarios. The patch is **low-risk and ready for integration**, with the only remaining action being the test addition.

---

### UTF-8 string-width refactoring withdrawn after sustained opposition
**Headline:** *Hardik Kumar abandons `utf8_strwidth()` return type change after reviewer consensus*

A **three-iteration refactoring effort** to change the return type of `utf8_strwidth()` and `utf8_strnwidth()` from `int` to `size_t` was **withdrawn** after sustained opposition from Junio C Hamano, Pablo Sabater, and Phillip Wood. The reviewers argued that the change introduced unnecessary complexity and risk without meaningful benefit, as the functions’ results are almost always used in contexts requiring `int` (e.g., `printf()` padding calculations). The effort concluded with a **documentation patch** explaining why the functions return `int` and why the `cast_size_t_to_int()` helper is necessary to prevent overflows. The thread is now **closed**, with no further technical discussion expected.

---

### ODB performance regression discussion shifts toward architectural redesign
**Headline:** *Patrick Steinhardt proposes moving alternates handling into the "files" backend*

The discussion around a **performance regression in `git receive-pack`** (introduced by the ODB abstraction rework) took a major turn when Patrick Steinhardt proposed a **fundamental architectural shift**: moving alternates handling into the "files" backend itself. This would treat alternates as an implementation detail rather than a cross-backend concern, enabling pack-first-loose-second lookups across all alternates while simplifying other subsystems (commit-graph, bitmap, maintenance). The proposal has gained **explicit endorsements from Peff and Justin Tobler**, who agree that the original motivation for abstracting alternates was to hide implementation details, not to enable arbitrary combinations of sources.

The discussion is now focused on **two parallel efforts**: Patrick’s alternates-in-files-backend redesign (planned for late July) and Justin Tobler’s plan to **decouple transaction handling from the ODB source list**. Both aim to simplify the ODB layer while resolving the performance regression. A near-term fix (e.g., targeted lookup order adjustment) may still be implemented, but the architectural direction is now the primary focus.

---

## In brief

**`git cat-file --batch-command` assertion failure** -- Alan Stokes reported a crash in `git cat-file --batch-command` when the output format string omits `%(objecttype)`. Jeff King (Peff) confirmed the bug and suggested a targeted fix, which Pablo Sabater implemented and tested. The next step is a formal patch with test coverage in `t1006-cat-file.sh`.

**`git repo info` path keys series** -- K Jayatheerth’s series adding seven new path-related keys to `git repo info` (e.g., `path.toplevel`, `path.hooks`) remains under review. Junio C Hamano identified a **forward-compatibility issue** in patch 1/7 (use of `startup_info->prefix`, a struct member recently removed from `master`), blocking the series until it is rebased. The design for `path.hooks`, `path.index`, and `path.grafts` is settled, with all three keys returning paths unconditionally (even when the file or directory does not exist or the repository is bare).

**`git stash reword` justification resolved** -- Junio C Hamano no longer objects to the `git stash reword` feature in principle, now that two users (brian m. carlson and erik88) have provided concrete workflow pain points. The patch remains under review, with the next step being a technical review of the implementation.

**`git mv` symlink-handling series** -- Lucas Zamboni Orioli’s series fixing misleading errors in `git mv` (when the destination’s parent directory does not exist) is **ready for integration**. Junio C Hamano approved the core logic but requested a **minor refactoring** to deduplicate a small piece of logic. The series now rejects all tracked symlinks in the destination path, aligning with `git apply` precedent.

**`commit_body()` test helper series** -- Shlok Kulshreshtha’s series introducing a `commit_body()` helper to replace duplicated `git cat-file commit ... | sed ...` patterns in the test suite is **under review**. Junio C Hamano identified two missed call sites in v1, which were addressed in v2. The series is a clean refactoring with no user-visible impact.

**`writev(3p)` wrapper revival** -- Patrick Steinhardt’s effort to reintroduce a `writev(3p)`-like wrapper for performance gains saw progress when Junio C Hamano endorsed Johannes Sixt’s suggestion to rename the wrapper to `git_write_gather()` (or similar) to avoid implying POSIX semantics. The discussion is now focused on API design and platform compatibility, with no immediate reroll expected (Patrick is at an offsite).

**Swift userdiff driver** -- Shlok Kulshreshtha’s patch adding a built-in userdiff driver for Swift is **queued in `next`**, fully reviewed, and technically complete. Michael Montalbo suggested adding a word-level diff test, but Johannes Sixt declined, citing the complexity of such tests relative to the microproject’s scope.

**`git last-modified` Bloom filter optimization** -- Toon Claes’s series using Bloom filters to optimize `git last-modified` is **stalled**, waiting for a response to feedback. The series is part of a broader effort to improve performance in large repositories.

---

## On the radar

- **`tb/repack-geometric-cruft`** -- Taylor Blau’s RFC series combining `git repack --geometric` and `--cruft` remains stalled, with unresolved feedback on the two-phase traversal logic in `--stdin-packs=follow-reachable`. Junio C Hamano’s follow-up email asked whether the series is considered complete or if further polishing is planned.
- **`tb/pack-with-duplicates`** -- Taylor Blau’s series hardening packfile handling for duplicate object entries needs review. The series updates reverse index lookup, delta cycle recovery, MIDX verification, and pack reuse.
- **`ps/cat-file-remote-object-info-type`** -- Patrick Steinhardt’s series extending `git cat-file --batch-command`’s `remote-object-info` to support `%(objecttype)` needs review.
- **Alternates-in-files-backend redesign** -- Patrick Steinhardt’s proposed architectural shift to move alternates handling into the "files" backend is gaining traction but has no concrete implementation yet. The effort is planned for late July.