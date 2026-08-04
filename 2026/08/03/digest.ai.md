# The Git Mailing List Daily Digest – 2026/08/03

## The day in brief

A busy Monday on the Git mailing list, with **56 emails across 18 threads**—a mix of **feature finalization, bugfixes, design debates, and policy discussions**. The standout developments: **`git history squash` reaches v11** but faces **fundamental design questions** about message handling and reachability logic; **`git cat-file --batch-command` gains `%(objecttype)` support** in a GSoC series now at v3; **`git add --resolved` inches closer to merge** with a new architectural proposal for binary-file handling; and **a policy debate on AI-assisted contributions** reaches a firm conclusion. **Junio’s "What’s cooking" report** also landed, marking the start of a new integration cycle.

---

## Notable threads

### `git history squash` – v11 posted, but design questions linger

**Headline:** Harald Nordgren’s `git history squash` series reaches **v11**, addressing all prior feedback—yet **Phillip Wood’s review** exposes **fundamental design questions** about message handling and reachability logic, leading to a **lively debate** about user experience and correctness.

The series introduces a new `git history squash` subcommand to collapse a commit range into its oldest commit while preserving descendant history. **v11 is functionally complete**, with **stricter range validation**, **merge-parent preservation**, **ref-handling refinements**, and **default message editing** (resolving the usability debate from v10). The command now **rejects root commits**, **multi-tip histories**, and **ranges with external merge parents**, and it **protects all local branches descended from the squashed range**.

However, **Phillip Wood’s review** raises **two major design questions**:
1. **Message handling:** Should the editor open by default (Harald’s proposal), never open by default (Matt Hunter’s `--no-edit` default), or **open conditionally based on commit message types** (Phillip’s latest proposal)? Phillip argues that the editor should open by default *unless* the commits are primarily `fixup!` or `amend!` messages, since those already signal the user’s intent. Junio **critiques this dynamic default**, questioning whether `fixup!` commits should trigger the same behavior as `amend!` and raising a new edge case: **multiple `amend!` commits**, where the intent may not be as clear.
2. **Reachability logic:** The current implementation **fails to handle `UNINTERESTING` commits**, **root commits**, and **multi-tip histories** correctly. Phillip is developing **fixup patches** to address these flaws, but the series has been **ejected from `next`** pending resolution.

**Key takeaway:** The series is **technically sound** but **UX-controversial**. The debate now centers on **balancing scriptability, user intent, and predictability**—a classic Git design tension. Expect **further iteration** before this lands.

---

### `git cat-file --batch-command` gains `%(objecttype)` support (GSoC v3)

**Headline:** Pablo Sabater’s **GSoC series** to extend `git cat-file --batch-command` with `%(objecttype)` support reaches **v3**, completing the **architectural refactoring** and **end-to-end implementation** for object type metadata.

The series adds support for querying object type metadata via remote-object-info, the last piece of metadata guaranteed to match between client and server after a fetch. **v3 introduces a new `struct fetch_object_info_results`** to replace the inefficient `struct object_info`, avoiding sentinel values and boolean flags. The design uses a **single array of lightweight structs** where the caller controls which fields are populated, aligning with the protocol’s transmission of raw strings (e.g., `"blob"`, `"1234"`).

**Key changes in v3:**
- **Safety checks** to `die()` if the server sends fewer object-info lines than requested OIDs.
- **Removal of `struct object_info_args`**, passing its members directly to `fetch_object_info()`.
- **Introduction of `struct fetch_object_info_results`**, replacing `object_info_options` and `remote_atom_map[]`.
- **Server-side capability advertisement** for "type" (patch 7/8).
- **Unified default format** for all queries: `%(objectname) %(objecttype) %(objectsize)` (patch 8/8).

**Open questions:**
- **Protocol robustness:** Junio asks whether the client should detect and reject **over-length responses** (server sending more objects than requested). Pablo proposes a fix: after reading the expected number of objects, the client should check for an immediate flush packet and `die()` if anything else arrives.
- **Premature `ask_type` flag:** The `ask_type` flag is introduced in patch 4/8 but not used until patch 6/8. If a user requests `%(objecttype)` at this stage and the server supports "type", the client will hit a `BUG()`. Pablo plans to move the flag to patch 6/8.

**Key takeaway:** The series is **feature-complete** and **merge-ready**, with only **minor protocol hygiene issues** remaining. The architectural refactoring is a **significant improvement** over v2, and the implementation follows established patterns from the predecessor series.

---

### `git add --resolved` – Binary-file handling debate

**Headline:** Junio’s `git add --resolved` series (v3) inches closer to merge, but **Michael Montalbo proposes an alternative approach** to binary-file handling that could **replace the current heuristic**.

The series introduces `git add --resolved`, which stages only paths whose conflict markers have been removed, allowing users to resolve merge conflicts without accidentally staging unrelated local changes. **v3 is technically complete**, with **stricter conflict-marker rules**, **consolidated logic**, and **performance optimizations** for binary files.

**The debate:** The current implementation uses a **`buffer_is_binary()` heuristic** to skip conflict-marker detection for binary files, but this check is placed *after* the marker detection loop, risking **false positives** (incorrectly identifying a binary file as conflicted). Junio **rejects moving the check earlier**, arguing that false positives are preferable to false negatives in this context.

**Montalbo’s proposal:** Instead of relying on the heuristic, inspect the **merge driver** used for the file (e.g., `binary`, `union`). Since these drivers cannot emit conflict markers, this could provide a **more deterministic** way to skip detection. The approach is **conceptually clearer** but may be **fragile** if new merge drivers are added in the future.

**Key takeaway:** The series is **ready for integration**, but the binary-file handling debate highlights a **broader architectural question**: should Git’s attribute system or merge-driver inspection influence conflict-marker detection? This could affect other conflict-marker consumers (`diff.c`, `rerere.c`).

---

### Policy debate: AI-assisted contributions

**Headline:** A **firm policy stance** emerges on AI-assisted contributions, with **brian m. carlson explicitly ruling out acceptance** of nontrivial LLM-generated patches in the foreseeable future.

The thread began with **Arijit Banerjee’s RFC patch** to speed up promisor pack link recording by eliminating a per-object mutex. The patch was **technically sound** (15–26% speed-up for partial clones) but **disclosed AI assistance**, prompting a **policy debate**.

**Key arguments:**
- **brian m. carlson:** Rejects nontrivial LLM-generated patches due to **legal exposure**, **code quality concerns**, **project reputation**, **environmental impact**, and **ethical considerations** around uncredited use of open-source code. Dismisses speculative governance ideas (e.g., experimental release trains) as unproductive.
- **Junio C Hamano (earlier):** Clarified that Git’s policy is **not an outright ban** but a **cautious, case-by-case approach** with heightened scrutiny, citing Linux kernel guidance and LLVM’s rejection of extractive contributions.
- **Collin Funk:** Agrees with brian’s risk assessment but questions whether **legal exposure in sanctioned countries** (e.g., Russia) is a realistic concern for individual contributors.

**Key takeaway:** The project’s stance is now **clear and firm**: nontrivial LLM-generated patches **will not be accepted** in the foreseeable future. The performance benefit of the promisor pack patch **does not warrant an exception**. This sets a **precedent for future AI-assisted contributions**.

---

### `git bisect --reset-when-found` – Ready for `next`

**Headline:** Harald Nordgren’s `git bisect --reset-when-found[=<where>]` series is **technically complete** and **ready for `next`**, with Junio’s final review confirming the patch is **sound and well-motivated**.

The series adds a `--reset-when-found[=<where>]` option to `git bisect` that automatically resets the working tree after identifying the first bad commit. The parameter `<where>` accepts `original` (reset to the pre-bisect commit, the default) or `found` (leave the working tree on the first bad commit). The feature targets **automated workflows** (scripts, CI pipelines) and **interactive debugging** by eliminating the need for an explicit `git bisect reset`.

**Key changes in v6:**
- **Centralized auto-reset logic** in `cmd_bisect()`, reading `refs/bisect/bad` directly before reset.
- **Sentinel value** (`RESET_WHEN_FOUND_NONE`) for `reset_when_found`, removing the separate boolean flag.
- **Deferred reset for `git bisect run`**, ensuring captured output is printed and the `BISECT_RUN` file is closed before reset.

**Key takeaway:** The series is **merge-ready**, with no further changes expected. The feature is a **useful addition** for automated workflows and aligns with Git’s existing conventions (e.g., `git am --resolved`).

---

## In brief

- **`git fast-import` usage refactor** – Christian Couder’s series to standardize the usage string and modernize the parse-options API is **under review**, with Junio’s feedback on patch 7/7 (unused local variables) addressed in v2.
- **`git branch --delete-merged`** – Han-Wen Nienhuys’ series to add `--delete-merged` to `git branch` remains in `next` but faces **unresolved design questions** about upstream push-check logic and merged-hierarchy behavior. Junio defers to **community consensus**.
- **`git rebase` regression fix** – Markus Geiger’s bug report about hardcoded `#` in worktree advisory comments is **already fixed** in Git 2.48.0 (commit 94304b9f).
- **`git diff` path completion** – Junio’s bash completion fix for `git diff` is **queued for integration**, with a **regression for `--no-index`** identified by SZEDER Gábor. Junio proposes a **fallback to Bash-native path completion** to resolve the issue.
- **`git add --resolved` test fix** – Kenneth Lorber’s patch to fix `t7528` for csh-like shells is **queued for integration**, with brian m. carlson’s review confirming the fix is correct.
- **`git interpret-trailers` overhaul** – Kristoffer Haugsbakk’s series to stop misinterpreting URLs as trailers is **under review**, with Peff and Junio endorsing the `://` heuristic as **robust and readable**.
- **`git repo info` extended** – Karthik Nayak’s series to add path keys to `git repo info` is **cooking in `next`**, with no open questions.
- **`git rebase` fixup/squash bugs** – Phillip Wood’s series to fix two bugs in `git rebase` is **cooking in `next`**, with no open questions.

---

## On the radar

- **`hn/history-squash` message handling debate** – The discussion about **dynamic defaults** for editor behavior (Phillip’s proposal) and **preserving `amend!`/`squash!` messages** is ongoing. Expect **further iteration** before the series lands.
- **`ps/cat-file-remote-object-info-type` protocol robustness** – Pablo Sabater’s proposal to **detect over-length responses** is a minor but important **protocol hygiene** improvement. Expect an **incremental update** to patch 2/8.
- **`git add --resolved` binary-file handling** – Michael Montalbo’s **merge-driver proposal** could replace the current `buffer_is_binary()` heuristic. This may inspire **follow-up patches** or a **broader discussion** about using Git’s attribute system for conflict-marker detection.
- **`hn/branch-delete-merged` design questions** – The unresolved questions about **upstream push-check logic** and **merged-hierarchy behavior** may prompt **further review** or **community discussion**.