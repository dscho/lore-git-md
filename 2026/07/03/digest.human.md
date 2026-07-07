Here’s the daily digest for **July 3, 2026**, covering the Git mailing list traffic:

---

### **The day in brief**
A busy but focused day on the Git mailing list, with **106 emails across 26 threads**. The standout developments:
1. **`git history drop` merged** after a final whitespace nit was resolved, marking a milestone for Patrick Steinhardt’s experimental history-editing command.
2. **`git replay --linearize` debate** continues, with Junio C Hamano raising procedural questions about authorship and documentation clarity.
3. **`USE_NSEC` discussion** heats up, with D. Ben Knoble defending the build-time knob while Patrick Steinhardt proposes runtime configuration.
4. **Reftable hardening series** (12 patches) nears completion, with all feedback addressed and Junio approving several fixes.

The day’s tone was **pragmatic and incremental**, with most threads converging on technical refinements rather than contentious design debates.

---

### **Notable threads**

#### **`git history drop` lands in `next`**
**Headline**: Patrick Steinhardt’s 11-patch series adding `git history drop` was **merged after Junio C Hamano fixed a final whitespace nit** in `reset.c`. The series modernizes the reset API, advances `the_repository` removal, and includes 561 lines of test coverage for edge cases like detached HEAD and bare repositories. All prior feedback—including a critical ref-resolution dispute—has been resolved. The command is now **ready for the next release cycle**.
**Why it matters**: This is a significant step for Git’s history-editing toolkit, offering a more intuitive alternative to `git rebase -i` for removing commits. The series also demonstrates the project’s rigorous review process, with 8 iterations and contributions from 5 reviewers.

#### **`git replay --linearize`: Authorship and documentation questions**
**Headline**: Junio C Hamano raised two **procedural concerns** about Toon Claes’s `git replay --linearize` series (v6):
1. **Authorship**: Should Johannes Schindelin remain the primary author (and thus responsible for latent bugs) given the extensive changes in this iteration?
2. **Documentation**: Should the incompatibility between `--linearize` and `--revert` be justified or moved to a BUGS/LIMITATIONS section?
The series itself is **technically complete**, with all prior feedback addressed, but these questions may delay its merge.
**Why it matters**: The thread highlights the project’s attention to **commit attribution** and **user-facing clarity**, even for experimental features. The `--linearize` option is a key part of Git’s replay machinery, and its design (standalone flag vs. `git rebase`-style `--rebase-merges=<mode>`) has been debated for months.

#### **`USE_NSEC`: Build-time vs. runtime configuration**
**Headline**: The long-running discussion about `USE_NSEC` (sub-second timestamp tracking) **took a new turn** as Patrick Steinhardt proposed **runtime configuration** (e.g., `core.useNsec`) to replace the current build-time knob. D. Ben Knoble defended the existing Meson option, arguing it provides **build-time parity** for distributions like Gentoo. Jeff King’s testing confirmed that modern Linux filesystems (ext4, XFS, etc.) preserve nanosecond timestamps reliably, but **interoperability risks** (e.g., Git/JGit mixing) remain unresolved.
**Why it matters**: This debate touches on **platform compatibility**, **user experience**, and **Git’s configuration philosophy**. The outcome could influence how Git handles filesystem-specific behaviors in the future.

#### **Reftable hardening: 12 patches ready for `next`**
**Headline**: Patrick Steinhardt’s **12-patch series hardening the reftable backend** against corrupted files is **technically complete**, with all feedback addressed. The series includes:
- **Fuzzing infrastructure** (libFuzzer + Meson support)
- **Test helper** (`cl_reftable_write_block`) to reduce boilerplate
- **10 security fixes** for out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage
Junio C Hamano has **approved 6 of the fixes** as "obviously correct," and the series is now **ready for integration**.
**Why it matters**: The reftable backend is a critical part of Git’s future, and this series ensures it’s **resilient to malicious input**. The fuzzing infrastructure may also be reused for other subsystems.

---

### **In brief**
- **`git refs` subcommands merged**: Patrick Steinhardt’s 5-patch series adding `create`, `delete`, `update`, and `rename` to `git refs` is **ready for `next`**, consolidating reference manipulation under a unified interface.
- **`GIT_TEST_LONG` tests now run in GitLab CI**: Patrick Steinhardt’s 9-patch series enables expensive tests for GitLab CI pushes to integration branches, matching GitHub Actions behavior.
- **`git rm` pathspec behavior clarified**: A thread about `git rm -n *.json` unexpectedly recursing into subdirectories **converged on documentation improvements**, with Patrick Steinhardt explaining the `:(glob)` modifier and Junio endorsing `failglob` as a user-side workaround.
- **Memory leaks plugged**: Jeff King’s 2-patch series fixing leaks in `git format-patch` and the test harness was **merged in principle**, with CI infrastructure improvements proposed as follow-up.
- **`greplint.pl` series ready**: Michael Montalbo’s 6-patch series introducing a linter to convert bare `grep` to `test_grep` is **technically complete**, with all feedback addressed. The series fixes 3 pre-existing test bugs and adds shared shell parser infrastructure.

---

### **On the radar**
- **`git replay --linearize`**: Awaiting Toon Claes’s response to Junio’s procedural questions.
- **`USE_NSEC` runtime configuration**: Patrick Steinhardt’s proposal needs further discussion about defaults and interoperability.
- **`git repo info` prefix queries**: Junio’s suggestion to support glob patterns may require a reroll.
- **`git history` signing**: Souma’s 3-patch series teaching `git history` to sign rewritten commits is under review.

---

### **Editorial note**
Today’s traffic reflected Git’s **steady progress on multiple fronts**:
- **History editing** (`git history drop`) and **replay machinery** (`git replay --linearize`) are maturing, with the former now merged and the latter nearing completion.
- **Build system and platform compatibility** (`USE_NSEC`, macOS fixes) remain active topics, with pragmatic trade-offs between user experience and technical debt.
- **Security hardening** (reftable, memory leaks) continues to be a priority, with fuzzing infrastructure now integrated into CI.

The day’s discussions were **technical and collaborative**, with reviewers like Patrick Steinhardt, Jeff King, and Junio C Hamano providing detailed feedback that advanced the state of the art without derailing progress. The project’s **rigorous review process** was on full display, particularly in the `git history drop` and reftable hardening threads.