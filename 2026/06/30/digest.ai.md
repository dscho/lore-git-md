Here is the digest for June 30, 2026:

---

**The day in brief.**
June 30 was a heavy-traffic day (94 emails, 23 threads) dominated by **integration milestones** and **usability refinements**. The standout event was Junio C Hamano’s **“What’s cooking” report**, which queued **11 topics for `next`** and clarified the deep freeze for Git 2.55-rc2. Two long-running series—**Patrick Steinhardt’s ODB abstraction stack** and **Phillip Wood’s rebase-dropped-commit overhaul**—reached **final review**, while **Harald Nordgren’s `git history squash`** and **Patrick’s reftable hardening** were **merged to `next`**. Smaller but impactful patches addressed **memory leaks**, **test modernization**, and **`git blame` output formatting**, with one usability fix (**René Scharfe’s `git blame -b` patch**) already **endorsed by its original reporter**.

---

### Notable threads

#### ODB abstraction stack reaches final review
Patrick Steinhardt’s **13-patch series** to separate repository discovery from setup in `setup.c` was **queued for `next`** after Junio fixed a typo in patch 2/13. The series introduces `struct repo_discovery` to consolidate discovery results, eliminating scattered setup logic and reducing global state. Key changes include:
- Moving the repository prefix from `startup_info` into `struct repository` (patch 7/13).
- Extracting discovery into a new `repo_discover()` function (patch 10/13).
- Making the worktree path an explicit parameter to `init_db()` (patch 12/13).
The refactoring is foundational for the **`the_repository` removal effort** and **ODB abstraction**, with no behavior changes intended. Reviewers may focus on the new `repo_discovery` API’s scalability for edge cases (e.g., environment variable overrides, worktree configuration).

#### Rebase-dropped-commit overhaul completes review
Phillip Wood’s **11-patch series** to fix how the sequencer handles dropped commits during rebase was **queued for `next`** after Junio resolved a `b4` tooling glitch. The series addresses a long-standing bug where notes from dropped commits were incorrectly copied to HEAD, corrupting note history. Key changes:
- Introduces `PICK_RESULT_DROPPED` to `enum pick_result` (patch 11/11).
- Skips `record_in_rewritten()` for dropped commits, preventing note corruption.
- Expands test coverage for edge cases (e.g., fixup sequences, external merge failures).
The series is **self-contained and ready for merging**, with no unresolved technical objections. A known limitation (the "edit" command still records dropped commits as rewritten) is documented for future work.

#### `git history squash` merged to `next`
Harald Nordgren’s **5-patch series** adding `git history squash` to fold a range of commits into one was **merged to `next`** after addressing all prior feedback. The feature:
- Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (to avoid ambiguity).
- Adopts `git rebase -i`’s squash-message template for `--reedit-message`.
- Rejects operations with interior refs by default, advising `--update-refs=head`.
The series sparked **usability debates** about default behavior (e.g., whether `--reedit-message` should be the default) and recoverability (e.g., `git reset --hard`’s limitations with `--update-refs`). These remain open for future discussion.

#### Reftable hardening series merged
Patrick Steinhardt’s **12-patch series** hardening Git’s reftable backend against corrupted files was **merged to `next`**. The series:
- Fixes OOB reads/writes, NULL pointer dereferences, and `abort()` calls during parsing.
- Adds a **libFuzzer-based fuzzer** to prevent regressions.
- Introduces a test helper (`cl_reftable_write_block`) to reduce boilerplate.
All patches are **technically complete**, with Junio approving the majority as "obviously correct." The fuzzing infrastructure is now integrated into Git’s CI.

#### `git blame -b` output formatting fixed
René Scharfe’s **patch** to align `git blame -b`’s abbreviated commit hashes with `core.abbrev` was **endorsed by its original reporter (Laszlo Ersek)**. The fix:
- Removes an extra hex digit reserved for an unused caret marker.
- Refactors mark-handling logic to count and print marks only when shown.
- Updates documentation and test expectations.
The patch resolves a **usability friction** in manual workflows (e.g., pasting hashes from `git blame -b` into `git rebase -i`). No backward-compatibility concerns were raised, but reviewers may check for edge cases (e.g., multiple marks per line).

---

### In brief
- **Memory leaks**: Jeff King fixed leaks in `git format-patch` and the test harness, with Patrick Steinhardt proposing to enable LeakSanitizer for the `linux-TEST-vars` CI job.
- **Test modernization**: Bryan B. Lima replaced raw test calls in `t7412-submodule-absorbgitdirs.sh` with descriptive helpers (`test_path_is_file`, etc.).
- **Config parser**: A patch to fix case-sensitivity mismatches in old-style subsection headers awaits review from Johannes Schindelin.
- **Rustification**: Jan Palus’s patch to remove a spurious Makefile dependency was **rejected by Brian m. carlson**, who clarified the dependency is necessary for `cargo test`.
- **Git Rev News**: Christian Couder called for contributions to edition 136, due July 2.

---

### On the radar
- **`git replay --linearize`**: Toon Claes and Johannes Schindelin’s interface design debate remains unresolved, with Patrick Steinhardt requiring explicit justification for diverging from `git rebase`’s syntax.
- **Reftable performance**: Jeff King identified a **quadratic-time behavior** in bulk ref deletion/re-creation, but its real-world impact is unclear. Kristofer Karlsson’s fix is **under reconsideration**.
- **CI resource allocation**: Patrick Steinhardt’s proposal to enable `GIT_TEST_LONG` for GitLab CI is **on hold** pending further discussion.