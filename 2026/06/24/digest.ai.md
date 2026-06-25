Here’s the daily digest for **June 24, 2026** on the Git mailing list:

---

## The day in brief
A **heavy-traffic day** (120 emails, 21 threads) with **two dominant themes**: (1) **security hardening** (reftable parser fixes, GPG/SSH signature normalization) and (2) **ODB abstraction** (Patrick Steinhardt’s refactoring series). The most consequential thread is **Patrick’s 11-patch reftable hardening series**, which lands fuzzing infrastructure and fixes for OOB reads/writes, NULL dereferences, and uninitialized memory. **Junio Hamano** also queued a **GPG/SSH signature fix** and conceptually approved a **`struct object_info` refactoring** for ODB pluggability. Meanwhile, **Harald Nordgren’s `git branch --delete-merged`** and **`git history squash`** series reached **code-complete status**, with only minor design questions remaining.

---

## Notable threads

### **Reftable security hardening (11 patches)**
**Topic**: Patrick Steinhardt’s series hardens Git’s reftable backend against maliciously corrupted files, fixing OOB reads/writes, NULL dereferences, and uninitialized memory.
**Status**: **Ready for merging**. The series introduces **libFuzzer support** (Meson/CI integration) and 9 individual fixes, each with unit tests. The reporter (oxsignal) confirmed the fixes address the originally reported OOB write and a follow-up OOB read.
**Key detail**: The fuzzer now runs for 2+ hours without finding new issues, suggesting the hardening is comprehensive.
**Files touched**: `reftable/basics.c`, `reftable/block.c`, `reftable/record.c`, `reftable/table.c`, build system files, and new unit tests.
**Next steps**: Junio is likely to queue this for `next` in the next integration cycle.

---

### **GPG/SSH signature normalization fix**
**Topic**: Antonio De Stefani fixes an over-aggressive CR-stripping behavior in Git’s GPG/SSH signature handling.
**Status**: **Queued for merging** ("Looking good. Will queue. Thanks." — Junio Hamano).
**Key detail**: The patch renames `remove_cr_after()` to `strip_cr_before_lf()` and preserves lone CR characters, addressing a NEEDSWORK comment from 2008.
**Files touched**: `gpg-interface.c`.
**Next steps**: Graduation to `next`.

---

### **`struct object_info` refactoring for ODB pluggability**
**Topic**: Patrick Steinhardt refactors `struct object_info` to replace the `whence` field with a `source` field, enabling multi-source object resolution and pluggable ODB backends.
**Status**: **Conceptually approved** ("Great." — Junio Hamano). Awaiting substantive review.
**Key detail**: The 6-patch series makes provenance information opt-in, aligning with the ODB abstraction effort.
**Files touched**: `odb.h`, `odb.c`, `packfile.c`, and scattered callers.
**Next steps**: Reviewers to engage with the implementation details.

---

### **`git branch --delete-merged` (v18)**
**Topic**: Harald Nordgren’s series adds safe automated branch cleanup with **refined stacked-branch protection**, per-branch opt-out, and `--dry-run` preview.
**Status**: **Code-complete and ready for Junio’s final assessment**. All design questions (e.g., stacked-branch protection) are resolved.
**Key detail**: The command now **clears upstream configs for kept branches** whose upstreams are deleted, avoiding broken dependency chains.
**Files touched**: `builtin/branch.c`, documentation, and tests.
**Next steps**: Junio’s final review.

---

### **`git history squash` (v5)**
**Topic**: Harald Nordgren’s new subcommand folds a commit range into its oldest commit while preserving descendant history.
**Status**: **Code-complete**. Addresses the last open question (`--ancestry-path` adoption) and adds `--reedit-message` support.
**Key detail**: The command **rejects operations with interior refs by default**, with advice to use `--update-refs=head`.
**Files touched**: `builtin/history.c`, documentation, and tests.
**Next steps**: Junio’s final review.

---

### **`git cat-file --batch-command` security hardening (v13)**
**Topic**: Pablo Sabater’s GSoC series implements a hardened `object-info` capability for remote object metadata queries.
**Status**: **Blocked on two fixes** (memory management issue in patch 10/12, protocol compatibility regression in patch 6/12). All other feedback addressed.
**Key detail**: The series now **reverts an unintentional `GIT_HASH_SHA1_LEGACY` substitution** to restore backward compatibility with pre-v2 servers.
**Next steps**: v14 to address the remaining issues.

---

## In brief
> **Bundle-URI protocol fix** -- Toon Claes and Justin Tobler debate server-side validation for misconfigured bundle-URI data. Consensus reached on combining client-side resilience with server-side filtering, but implementation details remain under discussion.

> **`git replay --linearize`** -- Toon Claes’s series to flatten merge commits into linear history is **blocked on a boolean refactoring debate** (Junio Hamano, Patrick Steinhardt). The `--ancestry-path` requirement is now resolved.

> **`git repo info` path formatting** -- K Jayatheerth’s GSoC series to expose repository paths in `git repo info` is **ready for merging** after Junio’s final review.

> **ODB transaction refactoring** -- Justin Tobler’s series to replace `tmp_objdir` with ODB transactions in `git-receive-pack` is **blocked on error-handling and interface-design feedback** from Patrick Steinhardt.

> **Submodule fetch resilience** -- Ben Knoble and Mike Crowe discuss whether missing submodule commits should block superproject fetches. No patch yet; design discussion ongoing.

---

## On the radar
- **`git branch --set-upstream-to`/`git push` usability improvements** (Harald Nordgren): Junio’s feedback on edge cases and test style nits.
- **External notes commands** (Siddh Raman Pant): Architectural debate with Johannes Sixt about whether Git should support external helpers for notes.
- **`paint_down_to_common()` optimization** (Tian Yuchen): Post-merge discussion about test instrumentation and future-proofing.

---