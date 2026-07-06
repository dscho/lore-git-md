Here’s the digest for **July 2, 2026**—a day where post-merge regressions, build system tweaks, and memory hygiene dominated the list, with a few philosophical debates about CLI design and CI infrastructure simmering in the background.

---

### The day in brief
Git’s mailing list saw **68 emails across 21 threads**, a moderate but focused volume. The standout themes: **post-merge regressions in `git replay --linearize`** (now fixed in v6), **memory leak fixes** (with a side of CI infrastructure debate), and **build system races** in Meson. The most urgent work—Toon Claes’s `replay` regression fix—landed, while Jeff King’s hash-context leak series and Patrick Steinhardt’s ODB refactoring advanced toward `next`. A quiet but notable thread emerged about Git’s reflog limitations, hinting at future architectural work.

---

### Notable threads

#### **`git replay --linearize` regression fixed (v6)**
**Headline:** *Toon Claes’s `--linearize` option for `git replay` merged prematurely—then silently dropped commits. Now fixed in v6.*
The thread began with **Johannes Schindelin’s alarm** about a regression in `git replay --linearize`: when replaying a single branch containing merge commits, only the tip commit was preserved, with intermediate commits (including merges) silently dropped. The root cause? A refactoring in v5 inadvertently removed the `replayed_base` mechanism that preserved linearized history. **Toon Claes’s v6** restores this logic, ensuring all replayed commits are flattened into a single linear topology, even across multiple branches. The series also **clarifies the design intent**: `--linearize` produces a *single linear sequence*, duplicating shared history if necessary—a behavior now documented and tested.

**Why it matters:** This was a **data-loss bug** in a newly merged feature. The fix’s thoroughness (regression tests, documentation updates) sets a high bar for post-merge follow-ups. The thread also exposed a **CLI design tension**: Patrick Steinhardt’s push for consistency with `git rebase`’s `--rebase-merges` syntax clashed with Schindelin’s UX-driven preference for `--replay-merges=<mode>`. The debate ended with a **process-level takeaway**: future CLI changes may require **explicit justification in commit messages** to avoid arbitrary divergence.

---

#### **Memory leaks in hash contexts (9-patch series)**
**Headline:** *Jeff King plugs leaks in non-default hash backends (OpenSSL, libgcrypt), exposing CI gaps.*
Peff’s series targets a **niche but critical correctness issue**: Git’s hash contexts (`git_hash_ctx`) leak memory when using non-default backends like OpenSSL SHA-256 or libgcrypt. The leaks surface under `SANITIZE=leak` and affect subsystems like `csum-file`, `patch-id`, and HTTP object requests. The fix introduces a new `git_hash_discard()` primitive and systematically applies it across leaky code paths. **Junio C Hamano approved the first patch**, and the series is now poised for `next`.

**Why it matters:** While the leaks only affect users who explicitly configure non-default hash backends, the series **highlights CI blind spots**. Peff notes that the `linux-sha256` job lacks leak checking, and libgcrypt isn’t tested at all. The thread also sparked a **naming debate** (e.g., `free_hashfile()` vs. `hashfile_discard()`), with Peff proposing a follow-up cleanup. For most users, this is invisible—but for those on OpenSSL ≥ 3.0 or libgcrypt, it’s a **long-overdue fix**.

---

#### **ODB refactoring: `struct object_info` redesign (v2)**
**Headline:** *Patrick Steinhardt’s ODB abstraction effort takes a step forward, replacing `whence` with `source_infop`.*
This v2 series refactors `struct object_info` to use a new `source_infop` field, replacing the coarse `whence` enum with a backend-specific `struct odb_source_info`. The change enables **multi-source object resolution** and is a prerequisite for pluggable ODB backends. **Junio conceptually approved the series**, and Patrick addressed feedback by renaming `sourcep` to `source_infop` to avoid ambiguity. The patches are **mechanical but foundational**, touching 14 files across the ODB and packfile subsystems.

**Why it matters:** This is **infrastructure work** for Git’s long-term ODB abstraction goals. The series is uncontroversial but critical: it paves the way for future backends (e.g., reftable, custom object stores) by making object provenance opt-in and extensible. The only lingering concern is a **pointer assignment pattern** (`oi->source_infop->source = &source->base`) that Junio called "yuck" but accepted as technically safe.

---

#### **Meson build race fixed**
**Headline:** *D. Ben Knoble and Adrian Ratiu squash a race in `hook-list.h` generation.*
A **build-system regression** introduced by commit `2eb541e8f2a9` caused intermittent failures in the Meson build when `hook-list.h` wasn’t generated before `builtin/bugreport.c` was compiled. The fix—moving the header’s custom target definition earlier in `meson.build`—is **minimal and targeted**, with Adrian Ratiu (the regression’s author) confirming the solution. **Patrick Steinhardt’s review** reinforced the correctness of the approach.

**Why it matters:** Build races are **frustrating and hard to debug**. This fix ensures the Meson build remains reliable, a priority as the project continues to invest in Meson alongside the traditional Makefile. The thread also serves as a reminder of the **fragility of build systems** during refactoring.

---

#### **`GIT_TEST_LONG` tests: CI reliability and efficiency (9-patch series)**
**Headline:** *Patrick Steinhardt and Toon Claes make expensive tests CI-friendly—with a side of Chesterton’s fence.*
This series tackles **CI flakiness and inefficiency** in `GIT_TEST_LONG` tests, which were causing pipeline hangs and excessive resource usage. Key changes:
- **CI visibility**: Adds a GitLab CI badge to the README.
- **Test correctness**: Skips broken tests on 32-bit systems (e.g., `t7508-status.sh`).
- **Efficiency**: Replaces slow `dd` loops with `fallocate` (e.g., `t4141-apply-too-large.sh`).
- **CI configuration**: Enables `GIT_TEST_LONG` in GitLab CI and disables it on Windows (due to RAM limits).

**Why it matters:** The series **expands CI coverage** while reducing false positives. **SZEDER Gábor’s review** of patch 3/9 raised a nuanced point: even if a test runs quickly, high memory usage might still warrant the `EXPENSIVE` label. **Jeff King’s follow-up** questioned the portability of `test_copy_bytes`, hinting at future test-suite modernization. The thread underscores the **trade-offs in CI design**: reliability vs. coverage, speed vs. resource usage.

---

### In brief
- **`git history squash` template debate**: Phillip Wood and Junio C Hamano converged on a **minimalist template format** for `--reedit-message`, omitting `fixup!`/`amend!` noise while retaining `squash!` bodies. The series is **code-complete and ready for final review**.
- **Reftable hardening**: Patrick Steinhardt’s 12-patch series to harden the reftable backend against corruption **received surface-level approval** from Christian Couder. The fuzzing infrastructure (libFuzzer + Meson) is now integrated.
- **Git v2.55.0 released**: Junio’s announcement drew a **lighthearted reply** from Weijie Yuan about "Claude Sonnet 4.6" appearing in the contributor list—clearly an AI model.
- **Bug reports**:
  - **`git rm -n *.json` recurses unexpectedly** (Евгений Плискин).
  - **`git rev-list --exclude-first-parent-only` misbehaves with explicit commits** (Michael Hore).
  - **`git apply` leaks memory and corrupts state** (Zephyr Yao, fixed in a standalone patch).
- **Test modernization**: Marcelo Machado Lage updated `t9811-git-p4-label-import.sh` to use modern test helpers (`test_path_is_file`).

---

### On the radar
- **Reflog limitations**: Patrick Steinhardt and Junio C Hamano’s exchange about Git’s reflog **hinted at a future "oplog" (operations log)** to enable atomic undo of multi-ref operations. This is **visionary but long-term**—no concrete patches yet, but the discussion frames a systemic gap in Git’s recoverability tooling.
- **CI job consolidation**: Peff and Patrick debated the value of redundant compiler jobs (clang/gcc) and macOS `TEST-vars` coverage. The consensus leans toward **minimalism**, but the thread may resurface if CI flakiness persists.
- **`git history` recoverability**: Matt Hunter and Phillip Wood’s critique of `--update-refs`’s side effects (e.g., `git reset --hard` not undoing branch moves) remains unresolved. A `--dry-run --verbose` mode or **reflog transaction IDs** could address this.