Here’s the digest for **2026/08/04**, covering the day’s most significant developments in Git development:

---

### **The day in brief**
August 4th was a **busy, milestone-heavy day** for Git, with **one major series merged to `next`**, a **contentious design debate resolved**, and **critical regressions identified** in two high-profile topics. The standout event: **Harald Nordgren’s `git branch --delete-merged` series (v24) was confirmed ready for merging** after resolving its last technical blockers, marking the culmination of a **14-iteration, 6-month effort** to add safe, automated branch cleanup. Meanwhile, **Phillip Wood conceded a key design point** in the `git history squash` debate, simplifying the command’s editor behavior and clearing a path for its eventual integration. On the regression front, **Junio C Hamano identified a critical bug** in Patrick Steinhardt’s ODB streaming unification series, while **Randall S. Becker narrowed a textconv regression** to binary files on NonStop. The day also saw **Junio’s "What’s cooking" report** spark a procedural discussion about review quality, underscoring the project’s ongoing tension between velocity and correctness.

### Do not miss:

1. **`git branch --delete-merged` v24 merged to `next`** – A long-awaited feature for safe, automated branch cleanup, now implementation-complete.
2. **`git history squash` design resolution** – Phillip Wood’s concession on editor behavior simplifies the command’s UX, removing a major roadblock.
3. **ODB streaming unification bug** – Junio’s discovery of a **critical correctness issue** in Patrick Steinhardt’s series, blocking its progress.

---

### **Notable threads**

#### **1. `git branch --delete-merged` v24: Merged to `next`**
**Headline:** After **14 iterations and 6 months of review**, Harald Nordgren’s `git branch --delete-merged` series (v24) was **confirmed ready for merging** and is now in `next`. The series adds a **safe, automated branch cleanup command** with:
- **Order-independent stacked-branch protection** (now simplified to protect only direct upstreams of surviving branches).
- **Per-branch opt-out** via `branch.<name>.deleteMerged=false`.
- **`--dry-run` preview** (avoids modifying config during dry runs).
- **Repeatable `--delete-merged` arguments** (e.g., `--delete-merged origin/main --delete-merged origin/next`).
- **Push-detection logic fix** (correctly handles `remote.pushDefault` configurations).

## Key developments:

- **Phillip Wood’s final review** confirmed the v24 fixes resolved the last technical blockers (push-detection logic and stacked-branch protection).
- **Junio C Hamano queued the series in `next`**, with only a **surface-level test formatting nit** remaining (unwrapping long test command lines in patch 1/7).
- **Follow-up work agreed**: Extending stacked-branch protection to `git branch -d` (post-merge).

**Why it matters:** This is a **major usability improvement** for Git workflows, particularly in CI/CD and topic-branch management. The series’ **14-iteration journey** also highlights Git’s rigorous review process, with **Phillip Wood and Junio C Hamano** playing pivotal roles in shaping the final design.

**Files touched:** `builtin/branch.c`, `ref-filter.c`, `Documentation/`, `t/t3200-branch.sh`.
**Status:** **Merged to `next`**. Expected to graduate to `master` in the current cycle.

---

#### **2. `git history squash`: Design debate resolved**
**Headline:** The **contentious design debate** over `git history squash`’s message-handling behavior **converged on a simpler approach**, removing a major roadblock for the series. Phillip Wood **conceded his proposed context-sensitive default** (editor opens unless commits are primarily `fixup!`/`amend!`) in favor of an **always-open-editor approach**, resolving Junio C Hamano’s concerns about complexity and user confusion.

## Key developments:

- **Phillip’s concession**: "A simpler, always-open-editor approach may be preferable" to avoid edge cases (e.g., multiple `amend!` commits, `fixup!` vs. `amend!` ambiguity).
- **Unresolved issues remain**:
  - **`--reedit-message` deprecation**: Junio called it "unfortunate" but did not endorse replacement with `--edit`/`--no-edit`.
  - **Template appropriateness**: Phillip’s critique of the rebase squash template as inappropriate for multi-commit squashing is still unaddressed.
  - **Correctness fixes**: Phillip is still developing fixups for reachability logic flaws (e.g., `UNINTERESTING` commits, root commits, multi-tip histories).

**Why it matters:** The resolution **simplifies the command’s UX** while preserving flexibility. The series’ **12 iterations** reflect Git’s iterative design process, with **Junio, Phillip, and Matt Hunter** driving the discussion.

**Files touched:** `builtin/history.c`, `revision.c`, `sequencer.c`, `t/t3455-history-squash.sh`.
**Status:** **Stalled in `seen`** pending Phillip’s fixups. Junio’s "Will replace" sign-off on v7 suggests intent to merge once correctness issues are resolved.

---

#### **3. ODB streaming unification: Critical bug identified**
**Headline:** Junio C Hamano **discovered a critical correctness bug** in Patrick Steinhardt’s **ODB streaming unification series (patch 3/7)**, blocking its progress. The bug causes the **in-memory ODB backend to compute object names (`oid`) using `OBJ_BLOB` even for non-blob objects**, leading to **silent corruption or hash collisions** when streaming large trees or commits.

## Key developments:

- **Bug details**: The patch updates `odb_source_inmemory_write_object()` to accept an `enum object_type` but fails to propagate it to `hash_object_file()`, which still hard-codes `OBJ_BLOB`.
- **Impact**: Non-blob objects (e.g., large trees) streamed into the in-memory backend will have **incorrect object names**, breaking hash-algorithm compatibility.
- **Justin Tobler’s reviews**: Confirmed the series’ structural changes (e.g., unifying `odb_write_stream` and `odb_read_stream`) but did not catch the bug.

**Why it matters:** This is a **blocker for the ODB abstraction effort**, which aims to enable pluggable backends. The bug underscores the **risk of subtle regressions in low-level refactoring** and the need for **targeted testing of edge cases** (e.g., non-blob objects).

**Files touched:** `odb/source-inmemory.c`, `odb/streaming.c`, `object-file.c`.
**Status:** **Blocked** pending fix. Junio’s review is a **must-address** for the series to proceed.

---

#### **4. Textconv regression: Narrowed to binary files on NonStop**
**Headline:** Randall S. Becker **narrowed a textconv regression** to **binary files on the NonStop platform**, where external filters receive **empty temporary files** instead of the actual content. The issue does not affect text files or ELF binaries (which lack leading null bytes), suggesting a **platform-specific I/O or null-byte handling bug**.

## Key developments:

- **Discrepancy with D. Ben Knoble’s report**: Ben observed **populated temp files** on Git 2.53.0, indicating the regression may be **environment-specific** (platform, Git version, or configuration).
- **Root cause hypothesis**: The issue may involve **early truncation or file creation failures** when encountering null bytes in binary files.

**Why it matters:** This is a **user-facing regression** in a niche but critical workflow (external textconv filters for binaries). The **platform-specific nature** complicates debugging, and the lack of maintainer response suggests it may linger unless a contributor steps in.

**Files touched:** None (user report only).
**Status:** **Unresolved**. No patch or root-cause analysis yet.

---

#### **5. `git repack --drop-filtered`: RFC v2 complete**
**Headline:** Siddharth Shrimali **posted RFC v2** of `git repack --drop-filtered`, a feature to **safely reclaim disk space in partial clones** by removing locally cached promisor blobs exceeding a size threshold. The series is **feature-complete** and addresses all prior feedback, including:
- **Safety guards**: Merge/rebase/cherry-pick checks and index validation (though Junio proposes replacing these with a **time-based heuristic**).
- **Protocol robustness**: Client now `die()`s on under-length or over-length server responses.
- **Documentation**: Updated to reflect the new functionality.

## Key developments:

- **Junio’s time-based heuristic proposal**: Avoid culling recently fetched objects (e.g., within 20 minutes) to reduce redundant network operations. Siddharth endorses this as a **natural extension** of the existing framework.
- **Siddharth Asthana’s review**: Endorsed the promisor-object enumeration mechanism and safety guards, while suggesting **softer language** in documentation about "guaranteed recoverability."

**Why it matters:** This is a **performance-critical feature** for partial clones, particularly in large-scale hosting environments (e.g., GitHub, GitLab). The **time-based heuristic debate** highlights the trade-off between **simplicity and smarter heuristics**.

**Files touched:** `builtin/repack.c`, `repack-filtered.c`, `Documentation/`, `t/t7706-repack-drop-filtered.sh`.
**Status:** **RFC v2 complete**. Awaiting maintainer decision on the time-based heuristic.

---

### **In brief**
- **`git fast-import` libification**: Christian Couder’s **12-patch series** refactoring `git fast-import` to use the parse-options API and reduce global variables **received Junio’s approval** and is queued in `seen`.
- **`git last-modified` Bloom filter optimization**: Toon Claes’s series to **reuse Bloom filters for `git last-modified`** is stalled pending resolution of a **design question** about the public API’s return semantics.
- **`git worktree` hooks**: Domen Kožar’s **`post-worktree-*` hook series (v2)** received **Junio’s fundamental design challenge**, questioning whether the hooks are necessary given user-written shell wrappers.
- **`git maintenance` "none" strategy fix**: David Lin’s **bugfix for `git maintenance`** (restoring the `"none"` strategy) is **ready to merge** pending a trivial author identity alignment.
- **Hex object ID case sensitivity**: brian m. carlson’s **RFC to enforce lowercase-only object IDs in Git 3.0** received **Junio’s tentative acceptance** ("Will queue") but remains contentious.
- **Bash completion fixes**: Junio’s **`git diff` path completion fix** (v2) is **ready to merge**, while Vincent Mailhol’s **`git history` completion patch** is under review.

---

### **On the radar**
1. **`git history squash` fixups**: Phillip Wood is developing **correctness fixes** for reachability logic flaws (e.g., `UNINTERESTING` commits, root commits). Expect a v13 reroll.
2. **ODB streaming unification bug**: Patrick Steinhardt must **fix the in-memory ODB backend bug** before the series can proceed.
3. **Textconv regression**: The **NonStop-specific issue** needs a maintainer or contributor to investigate.
4. **Review quality discussion**: Junio’s **proposed moratorium on new topics in `seen`** without substantive review remains open-ended. The `git history squash` ejection from `next` has reignited debate about **reviewer incentives and process gaps**.

---

### **Editorial note**
Today’s digest reflects a **project at a crossroads**. The **merging of `git branch --delete-merged`** is a triumph of persistence and review rigor, but the **ejection of `git history squash`** and the **ODB streaming bug** underscore the **fragility of Git’s low-level refactoring efforts**. The **textconv regression** and **review quality discussion** highlight ongoing challenges in **platform support and contributor onboarding**. As Git approaches its **20th anniversary**, the project’s ability to **balance innovation with stability** will be tested by these tensions.