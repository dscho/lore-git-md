Here’s the daily digest for **2026/07/03** (UTC), covering the Git mailing list’s traffic:

---

### The day in brief
A **heavy but focused** day on the Git mailing list, with **106 emails across 26 threads**. The standout themes were **security hardening** (Patrick Steinhardt’s 12-patch reftable series), **performance and correctness** (Jeff King’s memory-leak fixes and `USE_NSEC` debate), and **feature finalization** (Toon Claes’s `--linearize` for `git replay` and Patrick’s `git refs` subcommands). One **long-running series reached resolution** (`git history drop`), while another (`paint_down_to_common()` optimization) inched closer to merging. The day also saw **user-facing usability discussions** (e.g., `git rm` pathspecs, `git repo info` glob patterns) and **CI infrastructure tweaks** (GitLab badge, `GIT_TEST_LONG` reliability).

---

### Notable threads

#### **Reftable security hardening (12 patches) – Patrick Steinhardt**
**Headline**: Git’s reftable backend now fuzzes itself—and passes.
Patrick Steinhardt’s **12-patch series** systematically hardens the reftable backend against maliciously corrupted files, fixing out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage. The series also introduces **Meson-based libFuzzer support**, now integrated into Git’s CI. Junio C Hamano approved six patches as "obviously correct," and Toon Claes’s review of two test cases led to minor improvements (e.g., dynamic corruption values). The series is **technically complete** and ready for merging, with no open objections. The fuzzer has run for 2+ hours without surfacing new issues, signaling the hardening is effective.
**Why it matters**: Reftable files aren’t transferred over the network, but local corruption could still crash Git. This series ensures robustness while providing a reusable fuzzing framework for future hardening work.

#### **Memory-leak fixes (9 patches) – Jeff King**
**Headline**: Git’s non-default hash backends leak memory—now fixed.
Jeff King’s **9-patch series** plugs memory leaks in Git’s OpenSSL and libgcrypt hash implementations, which only surface under `SANITIZE=leak`. The series introduces `git_hash_discard()` (proposed as `git_hash_release()`) and applies it across subsystems (`csum-file`, `patch-id`, HTTP object requests). Patrick Steinhardt and Brian M. Carlson **endorsed making the API idempotent**, citing Rust integration benefits, while Junio approved the first patch. The series is **ready for review**, with idempotency slated for a follow-up.
**Why it matters**: Leaks in non-default backends are invisible to most users but critical for those who rely on OpenSSL or libgcrypt. The series also modernizes Git’s resource-cleanup APIs, aligning with the project’s memory-safety goals.

#### **`git replay --linearize` (v6) – Toon Claes**
**Headline**: `--linearize` for `git replay` is ready—but the interface debate lingers.
Toon Claes’s **6-patch series** adds `--linearize` to `git replay`, flattening merge commits into a linear history. The series is **technically complete** after fixing a regression in v6, but Patrick Steinhardt’s earlier interface critique (standalone `--linearize` vs. `git rebase`-style `--rebase-merges=<mode>`) remains unresolved. Junio C Hamano raised a **procedural nit** about authorship attribution, while the **documentation for `--linearize`/`--revert` incompatibility** needs justification or relocation. The series is **under active review** and likely to merge once the interface question is settled.
**Why it matters**: `--linearize` offers a simpler alternative to Johannes Schindelin’s merge-replay implementation, but the interface debate highlights Git’s tension between consistency and usability.

#### **`git history drop` (v8) – Patrick Steinhardt**
**Headline**: `git history drop` is merged—after 11 patches and a whitespace nit.
Patrick Steinhardt’s **11-patch series** adding `git history drop` (remove a commit and replay descendants) is **now fully merged** after Junio C Hamano fixed a single whitespace nit. The series modernizes the reset API, advances `the_repository` removal, and includes **561 lines of test coverage**. Christian Couder’s review was the final hurdle, confirming the series was unblocked after resolving a high-impact ref-resolution dispute.
**Why it matters**: `git history drop` expands Git’s history-editing toolkit, offering a more intuitive alternative to `git rebase -i` for targeted commit removal.

#### **`USE_NSEC` debate – D. Ben Knoble, Jeff King, Patrick Steinhardt**
**Headline**: Nanosecond timestamps are reliable on Linux—but should `USE_NSEC` be runtime-configurable?
The thread re-evaluated Git’s `USE_NSEC` knob, which enables sub-second timestamp tracking. Jeff King’s testing confirmed nanosecond timestamps are preserved on modern Linux filesystems (ext4, XFS, vfat), obsoleting historical "racy Git" concerns. Patrick Steinhardt proposed **making `USE_NSEC` runtime-configurable** (e.g., `core.useNsec`), while D. Ben Knoble defended the build-time option for parity. The discussion remains unresolved, with no consensus on whether to flip the default, deprecate the knob, or add runtime configuration.
**Why it matters**: `USE_NSEC` affects performance and interoperability (e.g., JGit compatibility), but the debate underscores Git’s challenge in balancing backward compatibility with modern filesystem capabilities.

---

### In brief
- **`git refs` subcommands (v3) – Patrick Steinhardt**: The 5-patch series adding `create`, `delete`, `update`, and `rename` to `git refs` is **ready for `next`**, with no open objections. Toon Claes’s usability feedback on symref handling and `--no-deref` was addressed.
- **`greplint.pl` (v3) – Michael Montalbo**: A 6-patch series introducing a linter to convert bare `grep` to `test_grep` is **approved for merging**. The series fixed pre-existing test bugs and added shared shell-parser infrastructure.
- **`GIT_TEST_LONG` reliability (v3) – Patrick Steinhardt**: A 9-patch series making `GIT_TEST_LONG` tests reliable and efficient for CI is **ready for integration**, with all feedback addressed. GitLab CI now runs expensive tests on integration branches.
- **`git rm` pathspecs – Евгений Плискин**: The thread clarified that `git rm -n *.json` recurses into subdirectories due to Git’s pathspec rules, not a bug. The discussion shifted to **documentation improvements** for `:(glob)` and shell-Git interaction.
- **`git repo info` glob patterns – Junio C Hamano**: Junio critiqued a GSoC patch for only supporting prefix matching, arguing glob patterns (e.g., `layout.*`) would be more expressive. The author has not yet responded.
- **`git history` signing (v1) – Souma**: A 3-patch series teaching `git history` to sign rewritten commits (`fixup`, `reword`, `split`) is **under review**, with no objections yet.

---

### On the radar
- **`paint_down_to_common()` optimization (v6) – Tian Yuchen**: The series is **ready for merging** after a procedural rebase, but Junio has not yet queued it.
- **`git rev-list --exclude-first-parent-only` fix – Junio C Hamano**: A standalone patch fixing a bug with explicit commits is **under review**, with no objections yet.
- **macOS UTF-8 filename crash (v2) – Ihar Hrachyshka**: The patch fixing a crash in `precompose_utf8` is **ready for merging** after addressing all feedback, including Junio’s request to avoid Perl in the test.

---

### Editorial notes
- **Volume**: 106 emails is **above average** for a weekday, driven by Patrick Steinhardt’s reftable hardening series (12 patches + reviews) and Jeff King’s memory-leak fixes (9 patches).
- **Tone**: The day was **collaborative and technical**, with reviewers (e.g., Toon Claes, Patrick Steinhardt) providing substantive feedback on edge cases (e.g., symref handling, idempotency) rather than surface-level nits.
- **Milestones**: `git history drop`’s merge marks a **significant expansion** of Git’s history-editing capabilities, while the reftable hardening series **sets a new bar** for backend robustness.
- **Controversy**: The `USE_NSEC` debate and `git replay --linearize` interface question highlight **Git’s tension between consistency and usability**, with no clear resolution yet.