# The Git Mailing List Daily Digest for 2026/08/03

## The day in brief
A busy Monday with **56 emails across 18 threads**, dominated by **substantive design discussions** and **integration pipeline updates**. The most urgent item: **Phillip Wood's review of `hn/history-squash`** exposed fundamental correctness flaws, leading to its ejection from `next` and sparking a broader conversation about **review quality and integration pipeline policy**. Meanwhile, **Junio's "What's cooking" report** and follow-up discussions highlighted **64 in-flight topics**, with **13 graduating to `master`** and several others facing design scrutiny. **Pablo Sabater's GSoC series** for `git cat-file --batch-command` object type support reached v3, incorporating architectural feedback from Junio and Peff.

---

## Notable threads

### **`hn/history-squash` ejected from `next` after fundamental flaws exposed**
**Headline:** *Phillip Wood's review reveals critical correctness issues in `git history squash`, leading to ejection from `next` and a broader discussion about review quality and integration pipeline policy.*

The `git history squash` command, designed to collapse commit ranges into a single commit, was **ejected from `next`** after Phillip Wood's review exposed **fundamental flaws in its reachability logic** and **UX inconsistencies**. The issues include:
- **Incorrect handling of `UNINTERESTING` commits** (e.g., `git history squash ^A B` where `A` is `UNINTERESTING`).
- **Failure to handle root commits** (commits with no parents).
- **Multi-tip histories** (e.g., `git history squash ^A C D` in `-A-B-C \ D`).
- **Inconsistent ref safety** (refuses to squash commits pointed to by tags or remote-tracking refs, unlike other `git history` commands).

Phillip is actively developing fixups, but the ejection has sparked a **broader discussion about review quality and integration pipeline policy**. Junio proposed a **temporary moratorium on accepting new topics into `seen`** unless they receive substantive review, aiming to prevent premature promotion to `next`. Matt Hunter reflected on the difficulty of providing substantive positive reviews, while Junio subtly validated the concern while reinforcing the expectation that reviewers demonstrate deeper engagement.

The thread also delved into **message-handling behavior**, with four competing proposals for editor invocation:
1. **Harald Nordgren's proposal**: Remove `--reedit-message` and make re-editing the default behavior.
2. **Matt Hunter's proposal**: Introduce conventional `--edit`/`--no-edit` options.
3. **Phillip Wood's initial proposal**: Dynamic default based on commit message types (editor opens unless commits are primarily `fixup!`/`amend!`).
4. **Phillip Wood's latest proposal**: Refines the dynamic default to handle edge cases like multiple `amend!` commits.

Junio critiqued Phillip's latest proposal, raising concerns about **user confusion** and **ambiguity in `fixup!` vs. `amend!` treatment**, but the discussion remains unresolved. The outcome will influence how future Git commands handle similar trade-offs between scriptability, UX consistency, and user intent.

**Status:** Ejected from `next`; fixups in progress. The broader policy discussion is paused but may resurface in future "What's cooking" reports.

---

### **`git cat-file --batch-command` object type support reaches v3**
**Headline:** *Pablo Sabater's GSoC series extends `git cat-file --batch-command` to support `%(objecttype)` in remote-object-info queries, incorporating architectural feedback from Junio and Peff.*

Pablo Sabater's GSoC series, now at **v3**, extends `git cat-file --batch-command` to support the `%(objecttype)` placeholder in remote-object-info queries. The series is **feature-complete** and addresses all prior review feedback, including a **significant architectural refactoring** in response to feedback from Junio and Peff.

The key change in v3 is the replacement of `struct object_info` with a dedicated `struct fetch_object_info_results`, which holds only the fields the caller requests (currently size and type). This hybrid design avoids sentinel values and boolean flags, using a single array of lightweight structs where the caller controls which fields are populated. The refactoring is spread across four preparatory patches (1–4), with the actual type support added in patches 5–8.

Junio raised two **forward-compatibility concerns**:
1. **Over-length responses**: The client should detect and reject responses with more objects than requested. Pablo proposed a fix: after reading the expected number of objects, the client should check for an immediate flush packet and `die()` if anything else arrives.
2. **Premature `ask_type` flag**: The `ask_type` flag is introduced before the client-side parsing logic and server-side capability advertisement, which could cause a `BUG()` if a user requests `%(objecttype)` and the server supports "type". Pablo plans to move the flag to a later patch.

The series touches `protocol-caps.c`, `fetch-object-info.c`, `serve.c`, `builtin/cat-file.c`, and documentation files, with test updates in `t5701` and `t1017`. The default format string in `cat-file` is unified to `%(objectname) %(objecttype) %(objectsize)` for all queries, removing a temporary workaround.

**Status:** Under review; v3 is feature-complete and merge-ready. The remaining loose ends (over-length responses, premature `ask_type` flag) are minor and will be addressed in the next iteration.

---

### **`git add --resolved` series ready for integration**
**Headline:** *Junio's `git add --resolved` series, which stages only paths whose conflict markers have been removed, is ready for integration after addressing all review feedback.*

Junio's **v3 patch series** introduces `git add --resolved`, a new option that stages only paths whose conflict markers have been removed, allowing users to resolve merge conflicts without accidentally staging unrelated local changes. The series consolidates duplicate conflict-marker detection logic, adds helpers for index removal with flags, and includes a targeted performance optimization for binary files.

The series is **technically complete** and addresses all prior review feedback, including:
- A **logic error in option incompatibility handling** (fixed in v2).
- A **missing test script entry in `t/meson.build`** (fixed in v2).
- **Portability fixes for the test script** (fixed in v3).
- A **performance optimization for binary files** (added in v2).

The final patch (4/4) implements the user-facing `--resolved` option, completing the feature. Junio clarified the correct logic for option incompatibility checks in `die_for_incompatible_opt3()` and identified a broader architectural issue in Git's option-parsing logic for `git add`, where the presence of command-line flags (e.g., `-A`) is conflated with the behaviors they trigger.

Michael Montalbo proposed an alternative design: a `--skip-conflict-markers` flag to allow incremental staging of resolved files (those without conflict markers) while leaving files with remaining markers unstaged. Junio rejected this, arguing that the monolithic design aligns with Git's existing conventions (e.g., `git am --resolved`) and reflects user intent. Jeff King (Peff) raised a **usability concern** about `--resolved` potentially creating a false sense of completeness by staging only files with resolved conflict markers while ignoring related changes (e.g., updates to callers of a modified function). Junio responded that this risk is mitigated by Git's existing merge behavior, which refuses to operate on paths with local modifications.

Michael Montalbo also identified a **correctness issue in the binary-file check** in `has_conflict_markers()`: the current placement of the `buffer_is_binary()` check *after* the conflict-marker detection could cause false positives in binary files whose first line coincidentally matches a conflict-marker pattern. Junio rejected moving the check, arguing that false positives are preferable to false negatives in this context. Montalbo then proposed an alternative: inspecting the *merge driver* used for the file (e.g., `binary`, `union`) to determine whether conflict-marker detection should be skipped, as these drivers cannot emit conflict markers.

**Status:** Ready for integration into `next`. The design debate (monolithic vs. incremental staging) is unresolved but does not block merging.

---

### **`git bisect --reset-when-found` ready for `next`**
**Headline:** *Harald Nordgren's `git bisect --reset-when-found[=<where>]` series is ready for integration after addressing all maintainer feedback.*

Harald Nordgren's **v6 patch series** introduces `--reset-when-found[=<where>]` to `git bisect`, which automatically resets the working tree to a user-specified commit after the bisect identifies the first bad commit. The parameter `<where>` accepts `original` (reset to the pre-bisect commit, the default) or `found` (leave the working tree on the first bad commit). The feature targets automated workflows (scripts, CI pipelines) and interactive debugging by eliminating the need for an explicit `git bisect reset`.

The series is **technically complete** and implements all maintainer feedback, including Junio's final internal clean-ups. The auto-reset logic is centralized in `cmd_bisect()`, which reads `refs/bisect/bad` directly before reset to eliminate the file-scope static variable `first_bad_oid`. The `reset_when_found` variable is initialized to a sentinel value (`RESET_WHEN_FOUND_NONE`) to remove the separate boolean flag `reset_when_found_arg_seen`.

Junio's final review confirmed the patch is ready for `next`, noting minor style distractions (e.g., reflowing function signatures) and a debatable `else` clause, but these are non-blocking. The series touches `bisect.c`, `builtin/bisect.c`, `bisect.h`, `Documentation/git-bisect.adoc`, and `t/t6030-bisect-porcelain.sh`.

**Status:** Ready for integration into `next`.

---

### **`git fast-import` usage refactoring awaits v2**
**Headline:** *Christian Couder's `git fast-import` usage refactoring series awaits v2 to address Junio's feedback on placeholder local variables.*

Christian Couder's **v1 patch series** refactors `git fast-import` to standardize its usage string with its SYNOPSIS documentation, reduce reliance on global variables, and modernize the parse-options API. The series introduces infrastructure (a new `OPT_HIDDEN_GROUP` macro and a `struct fast_import_state`) and documents existing parse-options flags to support future libification work.

Junio provided **substantive feedback on the final patch (7/7)**, expressing discomfort with the intermediate state of the refactor—specifically, the introduction of unused local variables in the `struct option` array as placeholders for future parsing logic. Christian acknowledged the feedback and is preparing v2 to fully integrate the parse-options API, which will remove the placeholder variables and complete the libification step.

The series touches `parse-options.h`, `parse-options.c`, `builtin/fast-import.c`, `Documentation/technical/api-parse-options.txt`, and `Documentation/git-fast-import.adoc`. The `.mailmap` discussion was resolved in v1 and is unrelated to the technical content.

**Status:** Awaiting v2; **Waiting for response** (per "What's cooking").

---

### **`git history squash` message-handling debate advances**
**Headline:** *Phillip Wood proposes a context-sensitive default for editor invocation in `git history squash`, but Junio raises concerns about user confusion and edge cases.*

The `git history squash` thread advanced with **Phillip Wood's proposal for a context-sensitive default editor behavior**: the editor should open by default *unless* the commits being squashed are primarily `fixup!` or `amend!` messages, since those already signal the user's intent. This aims to reduce friction for common cases while preserving flexibility via `--edit`/`--no-edit`.

Junio critiqued the proposal, raising two key concerns:
1. **Ambiguity in `fixup!` vs. `amend!` treatment**: A single `amend!` commit is a clear signal that no further editing is needed, but the same logic may not apply to `fixup!` commits, where the need for a log message change is ambiguous.
2. **Edge case: multiple `amend!` commits**: The intent may not be as clear as with a single `amend!` commit, complicating the dynamic default's logic.
3. **User confusion**: The dynamic behavior could be confusing, as the editor's behavior would depend on subtle, context-dependent rules.

The discussion remains unresolved, with Junio framing it as a tension between usability and predictability. The outcome will influence how future Git commands handle similar trade-offs.

**Status:** Design debate ongoing; no code changes yet.

---

## In brief

**Terminal output corruption fix verified** -- Junio asked Hugo Osvaldo Barrera to confirm whether the fix for terminal output corruption (commit 31e8fcabd8) resolves the issue in Git 2.55 or newer. The fix addresses a bug where ANSI escape sequence handling in `sideband.c` caused character truncation in terminals where line width exactly matched terminal width.

**Trailer parsing bugfix ready for integration** -- Kristoffer Haugsbakk's patch to stop Git from misinterpreting URLs as trailers is ready for integration. The patch modifies `find_separator()` in `trailer.c` to exclude lines containing `://` from trailer parsing, addressing a real-world issue with 245 documented instances in the Linux kernel. Jeff King (Peff) and Junio endorsed the approach, with Peff suggesting a readability improvement (replacing manual pointer arithmetic with `starts_with(c, "://")`).

**HTTP daemon test helpers fix ready for integration** -- Michael Montalbo's v2 patch series fixing race conditions in HTTP daemon test helpers is ready for integration. The series replaces non-atomic shell operations (`test -f` followed by `rm` or `touch`) with atomic operations (`mv` for one-shot markers, `mkdir` for first-request election) to eliminate race conditions under concurrent Apache requests. Junio confirmed all three patches look good.

**Trace2 ancestry test prerequisite refined** -- Jamie Magee's patch refining the `TRACE2_ANCESTRY` prerequisite to skip ancestry tests under user-mode emulation (e.g., QEMU) is under review. Junio noted the lack of engagement from contributors with similar environments and framed the patch as a low-risk housekeeping measure. **Needs review** (per "What's cooking").

**`git diff` path completion fix queued** -- Junio's patch fixing path completion for `git diff` when invoked with `git -C <path>` or when no revision references match the current input is queued for integration. The patch ensures completion suggests only tracked paths from the correct repository, aligning with `git status`/`git add` behavior. SZEDER Gábor identified a regression for `git diff --no-index`, which Junio resolved by noting the current patch already handles it via implicit fallback to Bash-native path completion.

**`git add --resolved` binary-file check debated** -- Michael Montalbo proposed inspecting the *merge driver* used for a file (e.g., `binary`, `union`) to determine whether conflict-marker detection should be skipped, replacing the current `buffer_is_binary()` heuristic. Junio rejected the idea, arguing that false positives (incorrectly identifying a binary file as conflicted) are preferable to false negatives (missing actual conflict markers).

**Promisor pack performance patch stalled on policy** -- Arijit Banerjee's RFC patch to speed up promisor pack link recording by avoiding a per-object mutex is stalled on policy regarding AI-assisted contributions. Brian m. carlson explicitly ruled out acceptance of nontrivial LLM-generated patches in the foreseeable future, citing legal, reputational, ethical, and environmental risks. The performance benefit (15–26% speed-up for partial clones) remains unaddressed.

**Test compatibility fix for `csh`-style shells queued** -- Kenneth Lorber's patch fixing a spurious test failure in `t7528-signed-commit-ssh.sh` when the test suite runs under a `csh`-style shell (e.g., `tcsh`) is queued for integration. The fix explicitly passes `-s` to `ssh-agent` to force Bourne shell syntax, ensuring consistent behavior regardless of the user's login shell. Brian m. carlson confirmed the fix is correct and audited the codebase to ensure no other instances of `ssh-agent` exist.

**`.mailmap` update for Christian Couder queued** -- Christian Couder's patch updating his `.mailmap` entry to reflect his current primary email address (`christian.couder@gmail.com`) is queued for integration. The change is purely administrative and ensures contributions are correctly attributed.

**`git rebase` regression confirmed fixed** -- Markus Geiger confirmed that a regression in `git rebase` where worktree advisory comments in the rebase todo list were hardcoded to `#` instead of respecting `core.commentChar` is already fixed in Git 2.48.0 by commit 94304b9f ("sequencer: comment checked-out branch properly", 2024-11-25).

**`git rebase` bug report for `core.commentChar` conflict** -- Markus Geiger reported a bug in `git rebase` where hardcoded `#` comment characters in todo list advisory lines conflict with `core.commentChar` configuration, causing sequencer parsing failures. The issue affects users who customize `core.commentChar` (e.g., to `;`) and use worktrees, rendering `git rebase` unusable without workarounds. The root cause is isolated to `sequencer.c`, where the literal `#` should be replaced with the resolved `comment_line_char`.

**Git Rev News edition 137 published** -- Christian Couder announced the publication of *Git Rev News* edition 137, the monthly digest of Git development activity. Ben Knoble replied with a brief note of appreciation, mentioning that the digest inspired him to revisit his work on the `USE_NSEC` timestamp feature.

---

## On the radar
- **`hn/branch-delete-merged`** -- Phillip Wood's review flagged two key issues: unreliable upstream push-check logic and a behavior change for merged hierarchies. Junio deferred to community consensus; the topic remains in `next` pending author response. **Needs review** (per "What's cooking").
- **`jm/t0213-skip-emulated-ancestry-tests`** -- Jamie Magee's patch refining the `TRACE2_ANCESTRY` prerequisite to skip ancestry tests under user-mode emulation (e.g., QEMU) is under review. Junio noted the lack of engagement from contributors with similar environments and framed the patch as a low-risk housekeeping measure. **Needs review** (per "What's cooking").
- **`tb/repack-geometric-cruft`** -- Taylor Blau's patch teaching `git repack` to combine `--geometric` with `--cruft` includes a squash commit that still contains a bare `grep !`. **Needs reroll** (per "What's cooking").
- **`hn/checkout-track-fetch`** -- Marked as "Will discard" unless rerolled with justification. **Needs reroll** (per "What's cooking").
- **`za/completion-hide-dotfiles`**, **`hs/rebase-continue-edit`**, **`tc/last-modified-bloom`**, **`mm/lib-httpd-cgi-safe`**, **`mm/diff-process-hunks`**, **`ec/commit-fixup-options`** -- No response to review comments; may be dropped. **Needs response** (per "What's cooking").