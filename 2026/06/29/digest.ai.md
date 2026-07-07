The day in brief.
2026-06-29 was a **heavy, milestone-heavy Monday**—Git 2.55.0 shipped, two major refactoring series landed, and the list saw the first substantive Rust-related traffic in months.  A reader who skims nothing else should note: **Git 2.55.0 is out**, **`git history drop` and `git history squash` are now in `next`**, and **the ODB abstraction effort took two big steps forward** with `ps/odb-drop-whence` and `jt/receive-pack-use-odb-transactions` both queued.

---

## Notable threads

### Git 2.55.0 released
Junio Hamano’s announcement of Git 2.55.0 (505 non-merge commits from 100 contributors) was the day’s headline.  The release brings **parallel hooks**, **new built-ins (`git format-rev`, `git url-parse`)**, **Linux fsmonitor**, **Rust enabled by default**, and **over 50 bug fixes**.  The changelog is long; the most consequential items are the **ODB abstraction work** (Patrick Steinhardt) and **Rust readiness** (Ezekiel Newren), both of which will shape the next 12–18 months of development.

---

### `git history drop` (ps/history-drop) – 11 patches, now in `next`
Patrick Steinhardt’s `git history drop` subcommand (remove a commit and replay descendants onto its parent) is now cooking in `next`.  The series refactored the reset machinery to avoid touching HEAD or the index unnecessarily, added dry-run mode, and introduced a two-phase ref-update design that separates analysis from execution.  Junio’s final review surfaced a **logical flaw in `find_head_tree_change()`**—when `--update-refs=head` is used, the function incorrectly concludes HEAD does not move because it looks for the branch name in an array that only contains `HEAD`.  The fix is straightforward and will appear in a reroll.

---

### `git history squash` (hn/history-squash) – 4 patches, now in `next`
Harald Nordgren’s `git history squash` (fold a range of commits into the oldest one, replay descendants on top) also graduated to `next`.  The series added `--reedit-message` with a template identical to `git rebase -i`’s squash behavior, and Junio’s review confirmed the template’s minor deviation (omitting numbered markers) is harmless.  Phillip Wood’s usability critique—whether `--reedit-message` should be the default—remains unresolved but does not block merging.

---

### ODB abstraction: `ps/odb-drop-whence` (7 patches, now in `next`)
Patrick Steinhardt’s series to remove the `whence` field from `struct object_info` and replace it with an opt-in `struct object_info_source` is now in `next`.  The change enables multi-source object resolution, a prerequisite for pluggable ODB backends.  Junio’s conceptual approval was unconditional, but Justin Tobler’s review raised an **architectural question**: should the ODB source be recorded at `packed_git` initialization time rather than passed as an optional parameter?  The discussion is still open, but the series is already queued.

---

### ODB abstraction: `jt/receive-pack-use-odb-transactions` (6 patches, now in `next`)
Justin Tobler’s series to replace `tmp_objdir` usage in `git-receive-pack` with the ODB transaction API is now in `next`.  The refactoring makes `receive-pack` backend-agnostic and paves the way for backend-agnostic object writes.  Junio’s review was surface-level, and Patrick Steinhardt’s feedback focused on error-propagation correctness.  The series is technically complete, with only minor follow-ups (e.g., renaming `flush_loose_object_transaction()`) deferred to a reroll.

---

### Reftable hardening (ps/reftable-hardening) – 12 patches, now in `next`
Patrick Steinhardt’s fuzz-driven hardening of the reftable backend (OOB reads/writes, NULL derefs, aborts) is now in `next`.  The series adds libFuzzer support to Meson and includes a test helper (`cl_reftable_write_block`) to reduce boilerplate in unit tests.  Junio approved the majority of the fixes as “obviously correct,” and the series is ready for final review.

---

### Rust readiness: xdiff Rust prep (no series yet)
Ezekiel Newren’s preparatory work for Rust in the xdiff subsystem surfaced in the 2.55.0 changelog.  The build system now enables Rust by default (opt-out), and the xdiff codebase has been refactored to be Rust-friendly.  No patches were posted today, but the changelog entry signals that the Rustification effort is moving from planning to implementation.

---

## In brief

**`git replay --linearize` (tc/replay-linearize)** – Toon Claes’s series to add `--linearize` to `git replay` (flatten merge commits) is still under review.  Patrick Steinhardt’s latest feedback questions whether the interface should mirror `git rebase`’s `--rebase-merges=<mode>` syntax.  The regression in `--linearize` behavior (commit-dropping when replaying a single branch containing merge commits) is fixed, but the interface debate remains open.

**`git refs` subcommands (ps/refs-writing-subcommands)** – Patrick Steinhardt’s new `git refs create|delete|update|rename` subcommands are now in `next`.  Junio’s final review approved the series, and the only follow-up is a minor typo fix in `git refs create`’s error message.

**`paint_down_to_common()` optimization (kk/merge-base-exhaustion)** – Kristofer Karlsson’s series to optimize merge-base computation by stopping early when one side’s exclusive commits are exhausted is still cooking.  Junio’s “What’s cooking” report lists it as needing review, but no new traffic appeared today.

**`USE_NSEC` debate (dk/meson-enable-use-nsec-build)** – The discussion about whether `USE_NSEC` should be a build-time option, runtime-configurable, or deprecated entirely continues.  Patrick Steinhardt’s latest proposal (always compile nanosecond support, expose via `core.useNsec`) sidesteps auto-detection challenges but leaves the default value unresolved.  The original Meson parity patch remains queued.

**GSoC 2026: partial-clone pruning (Siddharth Shrimali)** – Siddharth’s progress report on disk-space recovery for partial clones linked to a week-5 blog post.  The design proposes a `git maintenance run --task=gc-promisor` subcommand and a `.keep`-file safety mechanism.  No patches yet, but the blog post provides concrete details for review.

---

## On the radar

**`git history` subcommands** – Both `drop` and `squash` are now in `next`, but usability questions remain (e.g., `--reedit-message` default, recoverability).  These will likely resurface as the commands cook.

**ODB abstraction follow-ups** – The `ps/odb-drop-whence` and `jt/receive-pack-use-odb-transactions` series are foundational; expect more ODB-related traffic in July.

**Rustification** – The xdiff prep work in 2.55.0 is the first concrete step toward Rust in core Git.  The next likely target is the xdiff engine itself, with patches expected in the coming weeks.

**Reftable hardening** – The 12-patch series is in `next` and should graduate soon, but the fuzzing infrastructure may uncover new issues.

**`USE_NSEC`** – The debate about build-time vs. runtime configuration is unresolved.  A follow-up patch (runtime config or deprecation) is likely.