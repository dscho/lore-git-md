Here is the digest for **2026/06/24**, covering the Git mailing list traffic in a concise, front-page-style overview.

---

## The day in brief
**June 24, 2026** was a **heavy-traffic day** (120 emails, 21 active threads) with a **technical focus**: security hardening, ODB abstraction, and usability refinements dominated. The standout developments were **Patrick Steinhardt’s 11-patch reftable security series** (now complete and ready for review) and **Harald Nordgren’s `git branch --delete-merged` v18** (implementation-complete and awaiting Junio’s final assessment). A **long-running architectural debate** over `git replay --linearize`’s boolean refactoring resurfaced, while **Junio’s maintainer feedback** on multiple series signaled near-term integration plans.

---

## Notable threads

### Reftable security hardening (11 patches) – **Ready for review**
**Patrick Steinhardt** submitted a **comprehensive security-hardening series** for Git’s reftable backend, addressing vulnerabilities discovered via fuzzing (out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage). The series pairs **fixes with new fuzzing infrastructure** (libFuzzer + Meson support) to prevent regressions. **Key patches**:
- **Patch 5/11**: Fixes an OOB write in log block reconstruction (heap-buffer-overflow).
- **Patch 11/11**: Fixes an OOB read on truncated tables.
- **Patches 1–2**: Introduce fuzzing infrastructure and CI integration.
**Status**: All patches posted; no objections yet. The series is **self-contained and ready for review**, with the fuzzer now running for 2+ hours without finding new issues.

---

### `git branch --delete-merged` v18 – **Implementation-complete**
**Harald Nordgren**’s **v18 series** for safe automated branch cleanup is now **code-complete**, addressing the last major design question: **refined stacked-branch protection**. The command now **aborts-and-clears** (keeps only branches needed by unmerged branches, clears stale upstream configs) instead of the v17 "all-or-nothing" approach. **Key features**:
- **Per-branch opt-out** (`branch.<name>.deleteMerged=false`).
- **`--dry-run` preview** (matches real deletion output).
- **`--forked` filter** (composable with `--merged`/`--no-merged`).
**Status**: All feedback incorporated; **awaiting Junio’s final assessment**. The series is **well-tested** (497-line test suite) and **ready for merging**.

---

### `git replay --linearize` v4 – **Architectural debate resurfaces**
**Toon Claes**’s `--linearize` option for `git replay` (flattening merge commits into linear history) hit a **design impasse** over **Patch 1/3’s boolean refactoring**. **Junio Hamano** and **Patrick Steinhardt** raised concerns about **callsite clarity** (e.g., `create_commit(..., true)` obscures intent) and **future extensibility** (risk of painting the code into a corner if a third mode is needed). **Toon’s response** acknowledged the issue but noted the ternary operator in `pick_regular_commit()` remains problematic regardless of boolean/enum choice. **Status**: Debate ongoing; a **v5 iteration** is likely to address the refactoring.

---

### ODB abstraction: `struct object_info` refactoring – **Conceptually approved**
**Patrick Steinhardt**’s **6-patch series** refactoring `struct object_info` to use a `source` field (replacing the coarse `whence` enum) received **Junio’s conceptual approval** ("Great"). The series enables **multi-source object resolution** and is a **foundational step** for pluggable ODB backends. **Key changes**:
- **Patch 1/6**: Threads `struct odb_source_packed *source` through `packed_object_info()`.
- **Patch 6/6**: Adds documentation for `struct object_info` fields.
**Status**: **Awaiting substantive review**; no objections yet.

---

### `git history squash` v5 – **Ready for Junio’s final assessment**
**Harald Nordgren**’s **v5 series** for folding commit ranges into a single commit (with `--reedit-message` support) is now **code-complete**. The series addresses **all prior feedback**, including:
- **`--ancestry-path` adoption** (rejects single revisions like `HEAD`).
- **Ref-handling design** (rejects operations with interior refs by default, advises `--update-refs=head`).
- **`--reedit-message` usability** (seeds editor with all folded-in commit messages).
**Status**: **Technically complete**; **awaiting Junio’s final review**.

---

### `git cat-file --batch-command` security series – **v14 planned**
**Pablo Sabater**’s **security-hardened `git cat-file --batch-command`** series (13 patches) is **blocked on two fixes** for v14:
1. **Protocol compatibility regression**: Revert unintentional `GIT_HASH_SHA1_LEGACY` → `GIT_HASH_SHA1` change (breaks backward compatibility with pre-v2 servers).
2. **Memory management issue**: Remove unnecessary `static` declarations in `parse_cmd_remote_object_info()`.
**Status**: All other feedback addressed; **v14 expected soon**.

---

### `git push`/`git branch` usability improvements – **Test style fix needed**
**Harald Nordgren**’s **v2 series** improving error messages for common command-line slips (e.g., `git push origin/main` → `git push origin main`) is **technically sound** but requires a **test style fix** (replace bare `grep` with `test_grep`). **Junio’s review** approved the core design but flagged an **edge case** in the first patch (local branch exists but upstream argument is mistyped). **Status**: **Ready for merging** once the test style is corrected.

---

## In brief
> **`paint_down_to_common()` optimization** – **Merged**; post-merge discussion focused on **test instrumentation** (Derrick Stolee vs. Kristofer Karlsson on trace2 step-count assertions).
>
> **ODB transactions in `receive-pack`** – **v2 expected**; Patrick Steinhardt identified **blocking issues** (e.g., `NULL`-transaction handling in `odb_transaction_begin()`).
>
> **`git repo info` path formatting** – **Merged**; Junio praised the **technical documentation** (`Documentation/technical/paint-down-to-common.adoc`).
>
> **External notes commands** – **Architectural debate**; Johannes Sixt questioned the **problem framing** (TOCTOU vs. native notes scalability).
>
> **GPG/SSH signature normalization** – **Queued for `next`**; Junio approved Antonio De Stefani’s fix for over-aggressive CR stripping.
>
> **Git for Windows 2.55.0-rc2** – **Released**; drops Windows 8.1 support, tightens `.git` file validation, and ports Word diff helper to Rust.

---

## On the radar
- **`--track=fetch` for `git checkout`/`git switch`**: **Junio’s workflow concerns** (encouraging "blind updates") remain unresolved; Harald Nordgren challenged to present **new arguments**.
- **Bundle-URI protocol bugfix**: **Toon Claes** and **Justin Tobler** converged on a **two-part solution** (client resilience + server validation), but **error reporting mechanisms** are still under discussion.
- **Submodule fetch resilience**: **Design discussion** on whether missing submodule commits should block superproject fetches; no patch yet.