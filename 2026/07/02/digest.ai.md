Here’s the digest for **July 2, 2026**, covering the Git mailing list’s key developments:

---

### **The day in brief**
A busy day on the list, with **68 emails across 21 threads**. The standout themes: **post-merge bugfixes** (notably in `git rev-parse --parseopt` and `git apply`), **CI/test infrastructure improvements**, and **finalized feature series** (`git replay --linearize`, `git history squash`). The `git replay --linearize` series reached v6, while `git history squash` v7 addressed all prior feedback—though a debate about **recoverability** (or lack thereof) in Git’s reflog sparked a visionary proposal for an **oplog** (operations log). Meanwhile, Junio’s **"What’s cooking"** report clarified the status of three ODB/ref topics, and a **Git for Windows hotfix** re-enabled NTLM authentication as an opt-in deprecated feature.

---

### **Notable threads**

#### **`git rev-parse --parseopt` exit code fix: post-merge fallout**
The **four-patch series** standardizing help-flag exit codes (merged in Git 3.0) continues to generate follow-up discussion. Today, **Jeff King (Peff)** identified a **shell-quoting bug** in the `SVN` test prerequisite logic, where Perl’s string comparison of version numbers (e.g., `1.14.5 gt '1.1.0'`) fails due to ASCII lexicographical ordering. **brian m. carlson** confirmed plans to simplify the prerequisite by removing the version check entirely, leaving only the `PERL` prerequisite and SVN module availability. Peff also flagged a **refactoring gap** in `usage_with_options_internal()` and documented **behavioral trade-offs** for scripts wrapping Git commands (e.g., `git log --my-options "$@"`), where `--help` now exits 0 instead of 129. The thread underscores how even "simple" bugfixes can reveal deeper design tensions.

#### **`git history squash` v7: ready for merge, but recoverability questions linger**
Harald Nordgren’s **six-patch series** adding `git history squash` (a faster alternative to `git rebase -i` for collapsing ranges) reached **v7**, addressing all prior feedback:
- Adopted `git rebase -i`’s **squash-message template** for `--reedit-message`.
- Rejected ranges where the oldest commit is a `fixup!`/`squash!`/`amend!` (target cannot be inside the range).
- Clarified merge commit handling (rejects external-parent merges; allows fully contained merges).

However, **Phillip Wood** and **Matt Hunter** raised a **systemic issue**: Git’s reflog lacks visibility into multi-ref operations (e.g., `--update-refs`), making `git reset --hard` insufficient for undo. **Patrick Steinhardt** proposed an **oplog** (operations log) to enable atomic undo, while **Junio C Hamano** clarified that the reflog was never designed for this purpose. The series is **technically complete**, but the discussion highlights a broader gap in Git’s undo affordances.

#### **`git replay --linearize` v6: merge-ready**
Toon Claes’s **three-patch series** teaching `git replay` a `--linearize` option (to flatten merge commits) reached **v6**, fixing a regression where replaying a single branch with merges would drop commits. The series now:
- Restores the `replayed_base` parameter to ensure all commits are linearized into a single topology.
- Documents the **behavioral difference** from `git rebase --no-rebase-merges`.
- Justifies the `--linearize` flag (vs. `git rebase`’s `--rebase-merges=<mode>`) in the commit message.

**Junio C Hamano** suggested justifying the `--linearize`/`--revert` incompatibility or moving it to a BUGS section. The series is **ready for review**, with no unresolved technical objections.

#### **ODB/ref topics: Junio’s "What’s cooking" update**
In response to Junio’s **"What’s cooking"** report, **Patrick Steinhardt** clarified the status of three ODB/ref topics:
- **`ps/odb-drop-whence`**: Ready for `next` after a minor reroll (renaming `sourcep` to `source_infop`).
- **`ps/odb-generalize-prepare`**: Ready for `next` with no changes.
- **`ps/refs-writing-subcommands`**: Ready for `next` with no changes.

These topics are part of Patrick’s **ODB abstraction effort**, which aims to make Git’s object storage layer more modular. The updates remove ambiguity about their merge status.

#### **Git for Windows 2.55.0(2): NTLM hotfix**
**Johannes Schindelin** announced a **Git for Windows hotfix** re-enabling NTLM authentication as an **opt-in deprecated feature**, after it was prematurely disabled in 2.55.0. This is a temporary measure pending NTLM’s removal in 2026. The release also marks the **final version to support Windows 8.1**.

---

### **In brief**
- **`git apply` memory leak fix**: Zephyr Yao’s **standalone patch** fixes a leak in `find_header()` where abandoned Git-style diff headers corrupted subsequent parsing. The fix uses a temporary `struct patch` to isolate state.
- **`git rm -n *.json` recursion**: Евгений Плискин reported that `git rm -n *.json` recursively matches subdirectories, contrary to expectations. The behavior is **correct** (pathspecs match `/` by default), but **Patrick Steinhardt** and **Mikael Magnusson** noted the **documentation gap** in `git-rm(1)`.
- **`git rev-list --exclude-first-parent-only` bug**: Michael Hore reported a bug where the option incorrectly excludes commits when additional commits are specified. **Junio C Hamano** proposed a fix targeting `process_parents()` in `revision.c`.
- **Hash implementation leaks**: Jeff King’s **nine-patch series** plugs memory leaks in non-default hash backends (OpenSSL, libgcrypt). The series introduces `git_hash_discard()` and systematically applies it to leaky subsystems (`csum-file`, `patch-id`, HTTP object requests). **Junio approved patch 1/9**, while **Patrick Steinhardt** suggested renaming `git_hash_discard()` to `git_hash_release()` for idiomatic consistency.
- **CI/test reliability**: Toon Claes’s **nine-patch series** (v3) makes `GIT_TEST_LONG` tests reliable and efficient for CI. Key changes:
  - Skips broken tests on incompatible platforms (e.g., `SIZE_T_IS_64BIT`).
  - Reduces disk/CPU usage in `t4141-apply-too-large.sh` and `t5608-clone-2gb.sh`.
  - Enables `GIT_TEST_LONG` in GitLab CI (excluding Windows runners).
  - **SZEDER Gábor** objected to removing the `EXPENSIVE` prerequisite from `t4141` due to high memory usage (RSS > 1 GiB).
- **Test modernization**: Marcelo Machado Lage’s patch modernizes `t9811-git-p4-label-import.sh` by replacing `test -f` with `test_path_is_file`. **Patrick Steinhardt** requested minor improvements to the commit message and readability.

---

### **On the radar**
- **Oplog proposal**: Patrick Steinhardt’s vision for an **operations log** to enable atomic undo of multi-ref operations (e.g., `git history squash --update-refs`). This is a **long-term architectural change** that could benefit many Git commands.
- **`git replay --linearize` interface design**: The debate over `--linearize` vs. `git rebase`’s `--rebase-merges=<mode>` syntax may resurface during review.
- **`git history squash` default behavior**: Phillip Wood’s suggestion to make `--reedit-message` (or `--edit`) the default to encourage commit hygiene.