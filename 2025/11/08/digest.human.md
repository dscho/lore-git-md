# Git Mailing List Digest — 2025/11/08

**The day in brief.** A moderately busy Saturday with 24 emails across 10 threads saw several long-running patch series reach their final form while new technical issues emerged. The most notable developments include the completion of the atomic ref updates for `git replay` and a serious macOS fsmonitor daemon bug affecting all Git clients on macOS 26. Meanwhile, the `gitdatamodel.adoc` documentation effort appears ready for merging after extensive review.

---

## Notable threads

### Case-folding edge case in submodule gitdir encoding

Aaron Schrab identified a subtle edge case in Adrian Ratiu's submodule gitdir path encoding series. The current implementation handles case-folding collisions by percent-encoding uppercase characters when a collision is detected, but fails when a submodule like "Foo" is added first (with no encoding) and "foo" is added later — there are no uppercase characters to encode in the second name. This appears to be the only remaining issue in an otherwise complete solution with three-way consensus between Adrian, Junio, and Patrick Steinhardt. The series already includes comprehensive tests for path encoding, nesting, and case-folding collisions, but this specific scenario wasn't covered. The discussion will likely result in either a final clarification or documentation of this as a known limitation.

### Atomic ref updates for `git replay` complete

Siddharth Asthana's atomic ref updates series for `git replay` reached its conclusion after seven iterations. The final discussion addressed minor points about test isolation approaches and reflog message formatting, with Phillip Wood and Elijah Newren confirming the series is ready for merging. The implementation makes atomic ref updates the default in `git replay` while adding `--ref-action` and `replay.refAction` config options. All technical concerns have been resolved, marking this as production-ready functionality following Git's standard patterns. The only remaining note was about potentially simplifying reflog messages in a future follow-up.

### `gitdatamodel.adoc documentation finalized

Junio Hamano provided his final recommendations on two remaining wording issues in Julia Evans' documentation series before merge. The discussion centered on de-emphasizing Unix file mode representations in favor of describing Git's 5 file types directly, and clarifying "root tree" versus "directory" terminology. Junio has already applied XML validation fixes to v6 and considers these final wording issues minor enough that they shouldn't block merging, though he provided strong technical justification for his preferred phrasing about branches "recording" commit IDs. The series has reached its final form after extensive review, with only minor editorial preferences remaining.

### macOS 26 fsmonitor daemon termination bug

A serious bug report emerged about fsmonitor daemon processes failing to terminate properly on macOS 26 (Tahoe). The issue affects all Git clients and builds on this macOS version, causing daemon processes to accumulate indefinitely when core.fsmonitor is enabled. These processes spawn in detached mode and never receive termination signals when parent applications quit, leading to resource exhaustion and locked repositories due to lingering index.lock files. The problem is specific to macOS 26+ and doesn't occur on earlier versions. The only current workaround is to disable fsmonitor globally, and the issue has been reported to Apple as FB20956467.

### `git diff` exit code bug with specific flags

A bug report detailed an incorrect exit status in `git diff` when using `--cached --no-ext-diff --find-copies-harder --quiet` — the command incorrectly exits with status 0 when there are staged changes, while the equivalent `--exit-code` variant behaves correctly. Technical analysis revealed the issue stems from how the "quick" flag (set for `--quiet`) interacts with rename detection machinery, causing `has_changes` to get incorrectly reset to 0. The bug affects scripts like git-prompt.sh that rely on accurate exit codes when `diff.renames=copies-harder` is configured, and appears to be a long-standing issue in unpatched Git.

---

## In brief

**`git maintenance is-needed` subcommand** — Karthik Nayak introduces a new subcommand to check whether maintenance tasks require execution without performing them, building on ref backend optimization infrastructure. The 5-patch series has received positive reviews and is technically complete.

**`git fetch` tag handling regression fix** — A two-part series addresses a regression where non-conflicting tags would fail to commit if any conflicts existed in the same operation. The fix ensures transaction commitment happens even when errors occur in other updates.

**Three-way merge behavior discussion** — Ongoing analysis of why `git cherry-pick` and `git apply --3way` succeed where direct `git apply` fails, with technical explanations about merge heuristics considering changes as already applied in certain text pattern scenarios.

**Git for Windows 2.52.0-rc1 released** — Johannes Schindelin announced release candidate builds for Windows users, updating core components to Git v2.52.0-rc1, PCRE2 v10.47, and cURL v8.17.0.

**Fast-import signature stripping discussion** — Elijah Newren addressed concerns about signature validation in history rewriting, proposing potential solutions while defending the feature's utility for common use cases like `git filter-repo`.

---

## On the radar

**Rust support prerequisites** — The signed commits patch series tests need restructuring to avoid SHA-256 repository dependencies that now require Rust support, highlighting ongoing integration challenges.