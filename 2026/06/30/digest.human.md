## The day in brief

**2026/06/30** was a **heavy but routine** day on the Git mailing list, with **94 emails across 23 threads**. The standout developments were **two series landing in `next`**—Patrick Steinhardt’s **reftable hardening** and **ODB abstraction refactoring**—while **Phillip Wood’s systemic rebase-dropped-commit fix** and **Harald Nordgren’s `git history squash`** series advanced toward final review. The day also saw **usability debates** flare up around `git blame -b`’s hash abbreviation and `git replay --linearize`’s interface, and **CI infrastructure discussions** about leak detection and macOS test hangs.

---

## Notable threads

### Reftable hardening lands in `next`
**Subject**: [PATCH v3 0/12] reftable: security hardening against corrupted files

Patrick Steinhardt’s **12-patch series** fixing out-of-bounds reads/writes, NULL pointer dereferences, and `abort()` calls in the reftable backend **landed in Junio’s `next` branch**. The series adds a **libFuzzer-based fuzzer** and a test helper (`cl_reftable_write_block`) to prevent regressions. Junio approved the majority of the fixes as "obviously correct," and the series is now **ready for the next release cycle**. This is a **critical security hardening** for the reftable backend, which is increasingly used in large-scale deployments.

---

### ODB abstraction refactoring advances
**Subject**: [PATCH v1 0/2] odb: generalize reprepare() callback for pluggable ODBs

Patrick Steinhardt’s **3-patch series** generalizing the `reprepare()` callback into a more flexible `prepare()` API **received maintainer approval** and will advance to `next`. The series enables `git grep` to work with pluggable ODB backends by decoupling cache invalidation from ODB preparation. Toon Claes’s **substantive review** (questioning downcasting and flag semantics) was fully resolved, and Junio explicitly approved the series. This is a **key step in the ODB abstraction effort**, which aims to make Git’s object storage layer pluggable.

---

### Systemic rebase-dropped-commit fix nears completion
**Subject**: [PATCH 0/11] sequencer: avoid copying notes from dropped commits during rebase

Phillip Wood’s **11-patch systemic overhaul** of how the sequencer handles dropped commits during rebase **addressed all prior feedback** and is now **queued for final review**. The series fixes a long-standing bug where notes from dropped commits were incorrectly copied to the current HEAD, and it resolves additional edge cases (external merge strategy failures, command execution failures, and final fixup cleanup). Junio manually resolved a `b4 am` tooling issue and queued **typofix commits**, signaling the series is **ready for integration**. This is a **major correctness improvement** for the rebase machinery.

---

### `git history squash` series reaches final design
**Subject**: [PATCH v7 0/5] replay: introduce --linearize option

Harald Nordgren’s **5-patch series** adding `git history squash` **resolved all prior feedback**, including **interface design debates** and **recoverability concerns**. The series now **adopts `git rebase -i`’s squash-message template** for `--reedit-message`, addresses **merge commit handling** (rejecting external-parent merges), and **rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`**. A **usability debate** about whether `--reedit-message` should be the default remains unresolved, but the series is **technically complete** and awaiting Junio’s final review. This is a **significant ergonomic improvement** for monorepo workflows.

---

### `git replay --linearize` interface debate continues
**Subject**: [PATCH 0/3] replay: introduce --linearize option

The **interface design debate** for `git replay --linearize` **intensified**, with Johannes Schindelin **strongly opposing** Patrick Steinhardt’s proposal to mirror `git rebase`’s `--rebase-merges=<mode>` syntax. Schindelin argues that `--linearize` better describes the all-or-nothing flattening behavior and that `git rebase`’s interface is "user-hostile." Steinhardt **acknowledged the usability concerns** but insisted any divergence from `git rebase` must be **explicitly justified** in the commit message. Toon Claes **aligned with Schindelin’s technical direction** on the `replayed_base` parameter, but the **interface question remains unresolved**. This is a **high-stakes design decision** with implications for Git’s command-line consistency.

---

### `git blame -b` output formatting fix proposed
**Subject**: git blame -b output formatting and commit hash abbreviation

René Scharfe **posted a patch** fixing a **usability friction** in `git blame -b`: the command reserved an extra hex digit for a caret marker that was never displayed, causing commit hashes to exceed the user’s `core.abbrev` setting. The patch **dynamically calculates mark columns** and ensures the abbreviated hash length aligns with `core.abbrev`. Laszlo Ersek **confirmed the fix resolves his workflow issue**, and the patch is now **ready for review**. This is a **small but impactful usability improvement** for manual workflows involving `git blame` and `git rebase -i`.

---

### CI infrastructure discussions
**Subject**: macOS CI hang in t5551/t5559 – root cause and fix

The **macOS CI hang discussion** saw **two key developments**:
1. Junio **accepted Patrick Steinhardt’s proposal** to add a **GitLab CI status badge** to the project’s `README.md`, addressing a visibility concern.
2. Jeff King **identified a quadratic-time behavior** in the reftable backend during bulk ref deletion/re-creation, shifting the focus from micro-optimizations (redundant `stat()` calls) to a **more impactful performance bottleneck**. The issue is not yet actionable but provides a clear direction for future work.

Separately, **Jeff King and Patrick Steinhardt agreed to consolidate leak-detection CI jobs**, creating a new `linux-TEST-vars-leaks` job to catch memory leaks in non-default code paths. This is a **pragmatic improvement** to Git’s CI infrastructure.

---

## In brief

**`git refs` subcommands merged** -- Patrick Steinhardt’s **5-patch series** adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` **landed in `next`** after a final typo fix. The series consolidates reference manipulation under a unified interface, improving discoverability.

**`USE_NSEC` debate continues** -- The discussion about whether to **convert `USE_NSEC` to a runtime config knob** (e.g., `core.useNsec`) **remains unresolved**. Jeff King **endorsed the idea** but highlighted the lack of reliable auto-detection, while Patrick Steinhardt **proposed an incremental path forward** (always compile nanosecond support, expose it via runtime config). The debate now centers on **default values and interoperability risks**.

**Memory leak fixes** -- Jeff King **posted a 2-patch series** fixing leaks in `git format-patch` and the test harness. The fixes are **mechanical and low-risk**, and the series includes a **CI proposal** to enable LeakSanitizer for targeted jobs.

**Test modernization** -- Bryan B. Lima’s patch **replacing raw test calls** in `t7412-submodule-absorbgitdirs.sh` with descriptive helpers **was queued by Junio**. This is part of the ongoing **community-wide test modernization effort**.

**Rustification build system fix rejected** -- Jan Palus’s patch **removing a spurious Makefile dependency** for the Rust static library was **rejected by Brian m. carlson**, who explained the dependency is **necessary for `cargo test`** and that future Rust code will rely even more on the C library. This highlights the **growing interdependence** between Git’s Rust and C code.

---

## On the radar

**`git replay --linearize` interface design** -- The debate about whether to use `--linearize` or mirror `git rebase`’s `--rebase-merges=<mode>` syntax **remains unresolved**. The next steps will likely involve **further discussion or a maintainer decision**.

**Quadratic-time behavior in reftable** -- Jeff King’s discovery of a **quadratic-time bottleneck** in the reftable backend during bulk ref deletion/re-creation **warrants further investigation**. The issue is not yet actionable but could inform future optimizations.

**`USE_NSEC` runtime configuration** -- The proposal to **convert `USE_NSEC` to a runtime config knob** (e.g., `core.useNsec`) is **gaining traction**, but the **default value and interoperability risks** remain unresolved. A patch may emerge in the coming days.