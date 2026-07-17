# Here is the daily digest for the Git mailing list on 2026/07/16:

---

## The day in brief

A **heavy-traffic day** (126 emails, 38 active threads) saw **two long-running series reach resolution**, a **new performance optimization land**, and **multiple design debates** surface around usability and safety. The most urgent items: **`git replay --linearize`’s multi-branch ambiguity** remains unresolved after Elijah Newren and Junio C Hamano clashed over design philosophy, and **`git repack --drop-filtered`’s bitmap validation flaw** blocks its RFC series. A **new `git last-modified` optimization** (4× speedup) is now coordinated between Toon Claes and Jeff King.

---

## Notable threads

### `git replay --linearize`’s multi-branch ambiguity
The **v7 series** for `git replay --linearize` (Toon Claes) is **technically complete** but now faces a **fundamental design dispute**. Elijah Newren argues the current implementation **violates the principle of least astonishment** by silently concatenating unrelated branches, creating a **recoverability asymmetry** that favors rare use cases over common ones. Junio C Hamano defends the status quo, framing the concatenation as a **feature** (users can avoid it by invoking `git replay` multiple times) and suggesting **syntax alternatives** to reduce ambiguity. The ball is in Toon’s court to reconcile these competing visions—**restrict the interface** (Elijah’s preference) or **preserve flexibility** (Junio’s). The series remains in `next` but is unlikely to graduate without resolution.

### `git repack --drop-filtered`’s blocking flaw
Siddharth Shrimali’s **RFC v1** for `git repack --drop-filtered` (partial clones) is **stalled** after Junio C Hamano identified a **logical flaw** in the `--drop-filtered` vs `--write-bitmap-index` validation. The check occurs **too late** in the control flow—after `write_bitmaps` may have been set by config—so it cannot distinguish between user-requested bitmaps (which should be rejected) and config-driven bitmaps (which should be silently disabled). The **safety mechanism is ineffective**, and the series cannot proceed without a fix. The author has not yet responded; the RFC remains in limbo.

### `git last-modified` optimization lands
A **performance regression** in `git last-modified` (66 ms vs 26 ms for a hand-rolled `git log` pipeline) is now **resolved** after Jeff King (Peff) posted a **proof-of-concept patch** that reuses Bloom filter logic from `revision.c`. The patch reduces runtime from 64 ms to 16 ms (4× speedup) by pre-filtering commits with the original pathspec. Toon Claes benchmarked it on GitLab’s repositories, confirming **1.13×–1.56× speedups** with no regressions. The two are now coordinating on a **unified patch series**; this optimization is likely to land soon.

### `git stash rename`’s justification debate
Harald Nordgren’s **`git stash rename`** feature (v1) is **contested** after Junio C Hamano dismissed its justification as insufficient. Brian m. Carlson provided a **concrete use case** (frequent stashing with default messages leading to manual inspection overhead), but Junio’s skepticism persists. The **naming** (`rename` vs `reword`) and **behavior** (reset to original commit vs stay on first bad commit) remain unresolved. The series is **not actionable** until the justification is incorporated into the patch’s framing.

### `git bisect --auto-reset`’s design in flux
Harald Nordgren’s **`git bisect --auto-reset`** (v1) is **undergoing redesign** after Junio C Hamano expressed a **strong preference** for leaving the working tree on the first bad commit (the culprit) rather than resetting to the original pre-bisect commit. The author acknowledges the misalignment and is open to changing the behavior, but notes the **option name (`--auto-reset`)** may no longer fit. The series is **paused** pending a decision on the new behavior and naming.

---

## In brief

**`git cat-file --batch-command` remote-object-info** (Pablo Sabater) – The **GSoC v18 series** is **technically complete** and **queued for `next`** after resolving the `%(objectname)`-only request handling. The dynamic allow-list and placeholder handling are finalized.

**`diff.<driver>.process` RFC** (Michael Montalbo) – The **v5 series** is **unblocked** after resolving a merge conflict with `mm/line-log-limited-ops` via rebase. All technical objections are addressed; the series is ready for final review.

**`trust_executable_bit` and `has_symlinks` refactoring** (Tian Yuchen) – The **v6 series** is **under active review** after Junio C Hamano raised a **design-level concern** about the relocation of `ce_mode_from_stat()`. The mingw compatibility layer is now settled, but the `ce_mode_from_stat()` inlining debate continues.

**Memoized commit traversal optimization** (Tamir Duberstein) – The **three-patch series** is **ready for merging** after Jeff King (Peff) explicitly approved all patches. The optimization provides **100× speedups** with generation numbers and **10× without**, and is now **queued for `next`**.

**`git repo info` path keys** (K Jayatheerth) – The **GSoC v1 series** (7 patches) adds **18 new path-related keys** (e.g., `path.toplevel.absolute`, `path.git-prefix`). Junio C Hamano identified **two mechanical issues** in the final patch (include order, commit title); the author will address them in v2.

**`git bisect --auto-reset`** (Harald Nordgren) – The **v1 series** is **paused** pending a decision on whether the working tree should reset to the original commit or stay on the first bad commit. The option name (`--auto-reset`) may also need to change.

**`git stash rename`** (Emin Özata) – The **v1 series** is **contested** after Junio C Hamano dismissed its justification. Brian m. Carlson provided a **concrete use case**, but the maintainer’s skepticism persists. The **naming** (`rename` vs `reword`) and **behavior** remain unresolved.

**`git repack --drop-filtered`** (Siddharth Shrimali) – The **RFC v1 series** is **stalled** after Junio C Hamano identified a **logical flaw** in the `--drop-filtered` vs `--write-bitmap-index` validation. The safety mechanism is ineffective, and the series cannot proceed without a fix.

**`git last-modified` optimization** (Toon Claes/Jeff King) – A **4× speedup** (64 ms → 16 ms) is now coordinated between Toon and Peff. Benchmarks on GitLab’s repositories confirm **1.13×–1.56× speedups** with no regressions. A unified patch series is forthcoming.

**`git fetch` submodule errors** (Paulius Zaleckas) – The **v6 series** is **technically complete** and **ready for `next`**. It introduces `fetch.submoduleErrors` (config) and `--submodule-errors=(fail|warn)` (CLI) to let users choose whether `git fetch --recurse-submodules` should fail hard or merely warn when a submodule’s gitlink points to an unreachable OID.

**`git writev(3p)` wrapper revival** (Patrick Steinhardt) – The **v1 series** is **under review** after Junio C Hamano confirmed the bounds-checking logic is safe. Johannes Sixt (Hannes) objects to the name `git_writev()` for implying POSIX semantics; `git_write_gather()` is suggested as an alternative.

**`git fast-import` usage refactor** (Christian Couder) – The **v1 series** (7 patches) is **under review** after Junio C Hamano flagged the **awkwardness of unused local variables** in the final patch. The series standardizes the usage string and begins libification.

**`git submodule` and sparse-checkout** (Brad Larson) – A **feature request** to make `git submodule update --init` respect sparse-checkout cones is **under discussion**. No patch yet; the maintainers’ appetite for the change is unclear.

---

## On the radar

- **`git replay --linearize`’s multi-branch ambiguity** remains unresolved. Toon Claes must respond to Elijah Newren’s and Junio C Hamano’s competing visions.
- **`git repack --drop-filtered`’s bitmap validation flaw** blocks the RFC series. Siddharth Shrimali must address Junio’s feedback before the series can proceed.
- **`git stash rename`’s justification** is contested. Harald Nordgren must incorporate Brian m. Carlson’s use case into the patch’s framing.
- **`git bisect --auto-reset`’s behavior** is in flux. Harald Nordgren must decide whether to reset to the original commit or stay on the first bad commit.
- **`git writev(3p)` wrapper naming** is disputed. Patrick Steinhardt must address Johannes Sixt’s objection to `git_writev()`.
- **`git fast-import` usage refactor**’s unused variables are flagged. Christian Couder must respond to Junio’s feedback.

---