Here’s the daily digest for **July 2, 2026**, covering the Git mailing list’s key developments:

---

### **The day in brief**
A busy day (68 emails, 21 threads) with **critical post-merge fixes** for `git replay --linearize`, **build system and CI improvements**, and **memory leak cleanups** dominating the discussion. The most urgent thread addresses a **silent commit-dropping regression** in the newly merged `git replay --linearize` feature, while a 9-patch series from Jeff King plugs memory leaks in Git’s hash implementations. Meanwhile, Patrick Steinhardt’s ODB refactoring and Toon Claes’s CI/test efficiency work continue to progress, with several series now ready for `next`.

---

### **Notable threads**

#### **`git replay --linearize` regression: silent commit dropping**
**Headline:** *Post-merge fix for `git replay --linearize` restores `replayed_base` logic*
**Author:** Toon Claes
**Status:** v6 series posted, addressing a **critical regression** in the recently merged `--linearize` feature. When replaying a single branch containing merge commits, v5 inadvertently dropped all intermediate commits, replaying only the tip directly onto `--onto`. The fix (patch 3/3) restores the `replayed_base` parameter to ensure all commits are flattened into a single linear topology. The series also clarifies the feature’s design intent: `--linearize` produces a **single linear sequence** regardless of input branches, even if it duplicates shared history. Documentation and test coverage have been expanded to expose this behavior.
**Key detail:** The regression was confirmed by Johannes Schindelin with a test case (`master~2..master`), showing v5 replayed only the tip commit, dropping "Git 2.55-rc2" entirely. The fix aligns with Schindelin’s argument for **predictable behavior** over branch independence.
**Next steps:** Series is ready for review; no unresolved technical objections remain.

---

#### **Memory leaks in hash implementations**
**Headline:** *9-patch series plugs leaks in OpenSSL/libgcrypt hash backends*
**Author:** Jeff King
**Status:** Series complete, targeting leaks in Git’s hash algorithm handling when using non-default backends (e.g., OpenSSL SHA-256). The leaks were discovered via `SANITIZE=leak` and affect subsystems like `csum-file`, `patch-id`, and HTTP object requests. Patch 2/9 introduces `git_hash_discard()` as a cleanup primitive, while later patches apply it systematically. Patch 9/9 replaces the initial "hacky" implementation with platform-specific discard functions for efficiency.
**Key detail:** Leaks only manifest with non-default hash backends, so the impact is limited to users who explicitly configure Git to use OpenSSL or libgcrypt. Junio C Hamano approved the direction of patch 1/9, and the series is uncontroversial.
**Next steps:** Awaiting substantive review; likely to merge as-is.

---

#### **ODB refactoring: `struct object_info` redesign**
**Headline:** *v2 series replaces `whence` with `source_infop` for multi-source resolution*
**Author:** Patrick Steinhardt
**Status:** v2 posted, addressing feedback from v1. The series replaces the coarse `whence` field in `struct object_info` with a new `struct odb_source_info` that carries backend-specific provenance data. This enables future multi-source object resolution and pluggable ODB backends. The v2 renames `sourcep` to `source_infop` to avoid ambiguity with the actual ODB source.
**Key detail:** Junio C Hamano conceptually approved the series ("everything makes sense"), and the only remaining follow-up is a minor readability concern about an implicit upcast in patch 3/6. The series is a critical intermediate step in Patrick’s ODB abstraction effort.
**Next steps:** Ready for `next` after a small reroll to address the upcast concern.

---

#### **CI and test efficiency improvements**
**Headline:** *9-patch series makes `GIT_TEST_LONG` tests reliable for CI*
**Author:** Toon Claes (with Patrick Steinhardt)
**Status:** Series complete, addressing broken and inefficient `GIT_TEST_LONG` tests in CI. Key changes:
- **Fixes:** Skip tests that hang on 64-bit systems (`t0021-conversion.sh`, `t7508-status.sh`).
- **Efficiency:** Refactor slow tests (`t4141-apply-too-large.sh`, `t5608-clone-2gb.sh`) to use less disk space/CPU time.
- **CI:** Enable `GIT_TEST_LONG` in GitLab CI and add a GitLab CI badge to the README.
**Key detail:** SZEDER Gábor raised a concern about patch 3/9, noting that while the test now runs quickly, it still uses >1 GiB of memory, which may qualify as "expensive" regardless of runtime. Jeff King endorsed the patch but questioned the portability of `test_copy_bytes`.
**Next steps:** Awaiting resolution of Gábor’s concern; otherwise ready for `next`.

---

#### **Reftable hardening and fuzzing**
**Headline:** *12-patch series hardens reftable backend against corruption*
**Author:** Patrick Steinhardt
**Status:** v2 posted, adding fuzzing infrastructure (libFuzzer + Meson support) and fixes for vulnerabilities discovered by the fuzzer (out-of-bounds reads/writes, NULL pointer dereferences, etc.). The series is split into three parts: fuzzing infrastructure (patches 1–2), a test helper (patch 5), and individual fixes (patches 3–4, 6–12).
**Key detail:** Christian Couder’s feedback on v1 (extract a helper function to reduce test boilerplate) was fully addressed in v2. Junio C Hamano acknowledged the test helper improvement.
**Next steps:** Ready for substantive review of the security fixes.

---

### **In brief**
- **`git format-patch` leak fix** -- Jeff King plugs a memory leak in `--base` option handling, present since 2016. Patch merged in principle.
- **Meson build race fix** -- D. Ben Knoble’s patch ensures `hook-list.h` is generated before `builtin/bugreport.c` is compiled. Approved by Adrian Ratiu and Patrick Steinhardt.
- **Git v2.55.0 released** -- Junio C Hamano’s announcement drew a lighthearted note from Weijie Yuan about "Claude Sonnet 4.6" appearing in the contributor list.
- **Git for Windows 2.55.0(2)** -- Johannes Schindelin released a hotfix re-enabling NTLM authentication, previewing its planned deprecation later in 2026.
- **Test modernization** -- Marcelo Machado Lage updated `t9811-git-p4-label-import.sh` to use modern test helpers (`test_path_is_file`).
- **Git Rev News #136** -- Christian Couder announced the latest edition, with contributions from Toon Claes, Štěpán Němec, and Paulo Gomes.

---

### **On the radar**
- **`git rev-list --exclude-first-parent-only` bug** -- Michael Hore reported unexpected behavior when additional commits are specified on the command line. The issue appears to be a logic flaw in `process_parents()` in `revision.c`.
- **`git rm -n *.json` recursion bug** -- Евгений Плискин reported that `git rm -n *.json` recursively removes JSON files from subdirectories, contrary to documentation. This may be a core Git issue rather than platform-specific.
- **`git apply` header parsing bug** -- Zephyr Yao fixed a memory leak and state corruption in `find_header()`, where abandoned Git-style diff headers could interfere with subsequent "---/+++" header parsing.
- **CI job consolidation** -- Jeff King and Patrick Steinhardt discussed consolidating Git’s CI jobs, with consensus to merge `linux-reftables-leaks` and `linux-TEST-vars` into a single job (`linux-TEST-vars-leaks`). Skepticism remains about extending the pattern to macOS or compiler-specific jobs.