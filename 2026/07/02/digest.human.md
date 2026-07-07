Here’s the digest for **July 2, 2026**, covering the Git mailing list’s key developments:

---

### **The day in brief**
A busy day on the list, with **68 emails across 21 threads**. The standout themes: **post-merge bugfixes** (notably in `git rev-parse --parseopt` and `git apply`), **CI/test infrastructure improvements** (memory leaks, `GIT_TEST_LONG` reliability), and **feature finalization** (`git replay --linearize` and `git history squash`). The `git replay --linearize` series reached v6, while `git history squash` v7 addressed all prior feedback—both now await Junio’s final review. A lighthearted moment: "Claude Sonnet 4.6" appeared in the Git 2.55.0 contributor list, sparking amusement.

---

### **Notable threads**

#### **1. `git rev-parse --parseopt` exit code fix: post-merge fallout**
**Headline**: Bugfix series for `git rev-parse --parseopt`’s inconsistent exit codes (129 vs. 0 for `--help`) is **fully merged**, but three follow-up issues emerged:
- **Shell-quoting bug in `SVN` test prerequisites**: Jeff King (Peff) identified a quoting error in `t/t1517-outside-repo.sh` that caused Perl to miscompare version strings (e.g., `1.14.5` < `1.2.0` lexicographically). Brian m. carlson will simplify the prerequisite logic.
- **Refactoring gap in `usage_with_options_internal()`**: Peff flagged a missing update to distinguish between direct help requests (`--help`) and error-triggered help. A one-line fix is proposed.
- **Behavioral trade-offs**: Peff documented edge cases where scripts wrapping Git commands (e.g., `git log --my-options "$@"`) might now process nonsense output due to `--help` exiting 0 instead of 129. Junio acknowledged this as a real but acceptable risk.

**Why it matters**: The series resolves a 17-year-old inconsistency, but the follow-ups highlight how even "simple" fixes can ripple through the codebase. The `SVN` prerequisite fix is particularly notable for its subtle shell/Perl interaction.

---

#### **2. `git replay --linearize` v6: feature complete**
**Headline**: Toon Claes’s `--linearize` option for `git replay` (flattening merge commits into a linear history) reached **v6**, addressing all prior feedback:
- **Regression fix**: Restored the `replayed_base` parameter to ensure all commits are linearized into a single topology, even when replaying a single branch with merges.
- **Interface design**: The series diverges from `git rebase --rebase-merges=<mode>` syntax, using `--linearize` as a standalone flag. The commit message now justifies this choice, meeting Patrick Steinhardt’s requirement.
- **Test coverage**: Added scenarios for replaying multiple divergent branches and verifying the behavioral difference from `git rebase --no-rebase-merges`.

**Why it matters**: This is a **predictable, all-or-nothing alternative** to rebase’s merge-handling modes. The series is technically complete and ready for Junio’s final review, though the interface design debate may resurface.

---

#### **3. `git history squash` v7: ready for merge**
**Headline**: Harald Nordgren’s `git history squash` (collapsing a commit range into its oldest commit) **v7 addressed all prior feedback**:
- **Template alignment**: Adopted `git rebase -i`’s squash-message template, with `fixup!` messages fully commented out and `squash!`/`amend!` bodies retained.
- **Input validation**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).
- **Documentation**: Expanded to clarify merge limitations and `--reedit-message` behavior.

**Open question**: Phillip Wood and Matt Hunter argue `--reedit-message` (or `--edit`) should be the default to encourage commit hygiene. Junio has not weighed in yet.

**Why it matters**: This is a **long-awaited feature** for efficiently collapsing history without rebase’s conflict stops. The series is now **code-complete**, with only minor usability tweaks pending.

---

#### **4. Memory leak fixes: `git apply` and hash backends**
**Headline**: Two bugfix series target memory leaks:
- **`git apply`**: Zephyr Yao fixed a leak in `find_header()` where abandoned Git-style diff headers corrupted subsequent parsing. The fix uses a temporary `struct patch` to isolate state.
- **Hash backends**: Jeff King’s 9-patch series plugs leaks in non-default hash implementations (OpenSSL, libgcrypt) by introducing `git_hash_discard()`. Junio approved patch 1/9, and Patrick Steinhardt suggested renaming `git_hash_discard()` to `git_hash_release()` for idiomatic consistency.

**Why it matters**: These fixes are **low-risk but high-impact** for users of non-default hash backends or `git apply` in scripts. The hash backend series also lays groundwork for future Rust integration.

---

#### **5. CI/test infrastructure: `GIT_TEST_LONG` reliability**
**Headline**: Patrick Steinhardt’s 9-patch series makes `GIT_TEST_LONG` tests **reliable and efficient** for CI:
- **Broken tests fixed**: Skipped `t0021` and `t7508` on incompatible platforms (64-bit and 32-bit, respectively).
- **Performance improvements**: Replaced slow `dd` pipelines with `genzeros` in `t4141`, reducing runtime from 6 minutes to under a second.
- **CI coverage**: Enabled `GIT_TEST_LONG` in GitLab CI for integration branches, matching GitHub Actions.

**Controversy**: SZEDER Gábor argued that `t4141`’s high memory usage (>1 GiB) still justifies the `EXPENSIVE` prerequisite, even after the performance fix. Patrick conceded and restored the label.

**Why it matters**: This series **unblocks CI coverage** for long-running tests, a critical step for catching regressions early. The debate over `EXPENSIVE`’s definition (runtime vs. resource usage) may influence future test labeling.

---

### **In brief**
- **`git rm -n *.json` recursion**: Евгений Плискин reported unexpected recursion in `git rm -n *.json`. The behavior is correct (pathspecs match `/` by default), but the discussion shifted to **documentation clarity** in `git-rm(1)`.
- **`git rev-list --exclude-first-parent-only` bug**: Michael Hore reported a bug where explicit commits on the command line cause incorrect exclusions. Junio proposed a fix targeting `process_parents()` in `revision.c`.
- **Git 2.55.0 released**: Junio announced the release, with 505 commits from 100 contributors. Lighthearted note: "Claude Sonnet 4.6" appeared in the contributor list.
- **Git for Windows 2.55.0(2)**: Johannes Schindelin released a hotfix re-enabling NTLM authentication as an opt-in deprecated feature.
- **Test modernization**: Marcelo Machado Lage updated `t9811-git-p4-label-import.sh` to use modern test helpers (`test_path_is_file`/`test_path_is_missing`). Patrick Steinhardt requested minor improvements to the commit message.

---

### **On the radar**
- **`git history squash` default behavior**: Will `--reedit-message` become the default? Phillip Wood’s argument for commit hygiene is compelling but needs Junio’s input.
- **Oplog proposal**: Patrick Steinhardt’s vision for an "operations log" to enable atomic undo of multi-ref operations (e.g., `git history squash --update-refs`) remains a **long-term architectural goal**.
- **`git replay --linearize` interface**: The divergence from `git rebase`’s `--rebase-merges=<mode>` syntax may attract further debate, though the series is otherwise ready.

---