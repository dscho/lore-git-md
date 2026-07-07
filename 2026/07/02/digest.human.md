Here’s the digest for **July 2, 2026**, covering the day’s most notable developments in the Git mailing list:

---

### The day in brief
A **moderately busy day** (68 emails, 21 threads) saw progress on long-running feature series, bugfixes, and CI improvements. The **`git replay --linearize`** series reached v6, **`git history squash`** is now technically complete, and a **security-hardening series for reftable** is ready for merging. Memory-leak fixes and test modernizations also featured prominently. The day’s tone was **collaborative**, with reviewers and authors converging on solutions, though one thread revealed a **systemic limitation** in Git’s undo capabilities.

---

### Notable threads

#### **`git replay --linearize` reaches v6**
**Headline**: Toon Claes posted v6 of the `--linearize` option for `git replay`, addressing all prior feedback, including a **regression fix** for single-branch replay with merges. The series now uses `--linearize` as a standalone flag (diverging from `git rebase`'s `--rebase-merges` syntax) with explicit justification in the commit message. Junio C Hamano and Johannes Schindelin support the design, while Patrick Steinhardt’s earlier request for consistency with `git rebase` has been met with a clear rationale. The series is **ready for review** and likely to merge soon.
**Key detail**: The `replayed_base` parameter was restored to ensure all commits are flattened into a single topology, fixing a bug where commits were dropped during single-branch replay.

---

#### **`git history squash` is code-complete**
**Headline**: Harald Nordgren’s **`git history squash`** series (v7) is now **technically complete**, with all feedback addressed, including:
- Adoption of `git rebase -i`'s squash-message template (resolving Phillip Wood’s usability concerns).
- Explicit rejection of ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the target cannot lie inside the range).
- Expanded documentation and test coverage.
**Systemic limitation**: A discussion emerged about Git’s **lack of atomic undo** for multi-ref operations (e.g., `--update-refs`). Patrick Steinhardt proposed an **oplog (operations log)** as a long-term solution, while Junio clarified that the reflog was never designed for this purpose. The `squash` feature itself is sound, but users should be aware that `git reset --hard` may not fully revert operations affecting multiple branches.
**Key detail**: The `--reedit-message` template now matches `git rebase -i`’s behavior, with `fixup!` messages fully commented out and `squash!`/`amend!` bodies retained.

---

#### **Reftable security hardening ready for merging**
**Headline**: Patrick Steinhardt’s **12-patch series** hardening the reftable backend against corrupted files is now **fully reviewed and approved**. The series includes:
- **Fuzzing infrastructure** (Meson + libFuzzer support).
- **Test helper** (`cl_reftable_write_block`) to reduce boilerplate.
- **Individual fixes** for out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage.
Junio C Hamano approved 6 of the 12 patches as "obviously correct," and Christian Couder endorsed the test helper. The series is **ready for `next`** and will ship in Git 2.56.
**Key detail**: The fuzzer has run for 2+ hours without surfacing new issues, and the fixes replace crashes with `REFTABLE_FORMAT_ERROR` returns.

---

#### **Memory-leak fixes land**
**Headline**: Jeff King (Peff) posted a **9-patch series** plugging memory leaks in Git’s hash implementations (OpenSSL, libgcrypt). The series introduces `git_hash_discard()` and systematically applies it to leaky subsystems (`csum-file`, `patch-id`, HTTP object requests). Junio approved the first patch, and Patrick Steinhardt suggested renaming `git_hash_discard()` to `git_hash_release()` for idiomatic consistency. The leaks are **invisible with default hash backends** but critical for users of `OPENSSL_SHA256=1` or `GCRYPT_SHA256=1`.
**Key detail**: Patch 9 replaces the "hacky" `git_hash_discard()` implementation (which called `git_hash_final()` into a dummy buffer) with platform-specific discard functions, eliminating inefficiency.

---

#### **CI and test improvements**
**Headline**: Patrick Steinhardt’s **9-patch series** makes `GIT_TEST_LONG` tests reliable and efficient enough to run in GitLab CI. Key changes:
- **Test correctness fixes**: Skip broken tests on incompatible platforms (e.g., `SIZE_T_IS_64BIT` for `t0021`).
- **Efficiency improvements**: Replace slow `dd` pipelines with `genzeros` (reducing runtime from 6 minutes to <1 second in `t4141`).
- **CI visibility**: Add a GitLab CI badge to the README and enable `GIT_TEST_LONG` for integration branches.
**Debate**: SZEDER Gábor argued that `t4141` should retain the `EXPENSIVE` prerequisite due to high memory usage (>1 GiB RSS), even though it now runs quickly. The discussion highlights a **policy question**: should "expensive" be tied to runtime, resource usage, or both?

---

### In brief
- **`git rev-list --exclude-first-parent-only` bug**: Michael Hore reported a logic flaw where explicit commits on the command line cause incorrect exclusions. Junio proposed a fix targeting `process_parents()` in `revision.c`.
- **`git apply` memory leak**: Zephyr Yao fixed a leak in `find_header()` where abandoned Git-style diff headers corrupted patch state. The fix adds a temporary `struct patch` to isolate parsing.
- **`git rm -n *.json` recursion**: Евгений Плискин reported unexpected recursion into subdirectories. The behavior is **expected** (pathspecs match `/` by default), but the discussion shifted to improving `git-rm(1)`’s documentation of pathspec semantics.
- **ODB refactoring**: Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use `source_infop` is **ready for `next`**. The series replaces the coarse `whence` field with a backend-specific `struct odb_source_info`, enabling future multi-source object resolution.
- **Git v2.55.0 released**: Junio announced the final release, drawing lighthearted commentary about "Claude Sonnet 4.6" appearing in the contributor list.
- **Git for Windows 2.55.0(2)**: Johannes Schindelin released a hotfix re-enabling NTLM authentication as an opt-in deprecated feature, pending its removal in 2026.

---

### On the radar
- **Oplog proposal**: Patrick Steinhardt’s vision for an **operations log** to enable atomic undo of multi-ref operations (e.g., `git history squash --update-refs`) remains a **long-term architectural goal**. The discussion highlighted Git’s current limitations in this area.
- **Test modernization**: Marcelo Machado Lage’s patch modernizing `t9811-git-p4-label-import.sh` awaits a reroll to address Patrick Steinhardt’s feedback on commit message clarity and readability.