Here’s the daily digest for **July 21, 2026**, covering the Git mailing list traffic:

---

### **The day in brief**
A **busy but focused** day on the Git mailing list, with **46 emails across 20 threads**. The standout themes:
1. **Performance regressions** took center stage, with a deep dive into the ODB abstraction’s impact on `git receive-pack` connectivity checks (now reframed as an opportunity for targeted optimizations).
2. **Feature refinements** dominated, including a v3 reroll of the `git history squash` series (now ready for `next`) and a heated debate over `git rebase --continue --[no-]edit`.
3. **Maintainer signals** were clear: Junio’s "What’s cooking" report (July 2026 #09) provided a **comprehensive snapshot** of in-flight topics, while several series (e.g., Swift userdiff, `git stash push` optimization) graduated to `next`.

---

### **Notable threads**

#### **1. ODB abstraction regression: Connectivity checks in `git receive-pack`**
**Headline**: *Performance regression in `git receive-pack` after ODB rework (commit 8384cbcb4c)*
- **What’s happening**: A **12% slowdown** in server-side Git operations (pushes) was traced to the ODB abstraction rework, which changed the object lookup order from "pack-first, loose-second" *per backend* to "pack-loose, pack-loose" *across backends*. This causes redundant loose object checks, especially costly on NFS.
- **Key insights**:
  - **Root cause**: Commit `a593373b09` (part of the ODB rework) altered `find_pack_entry()` to iterate backends sequentially, checking both packs and loose objects in each before moving to the next.
  - **Proposed fixes**:
    - **Two-phase lookup** (Taylor Blau): Check all packs first, then all loose objects (restores performance but violates ODB abstraction).
    - **Backend-level "fast/slow" lookups** (Junio): Distinguish between pack and loose object checks within backends to enable cross-backend optimizations.
    - **Pre-loaded loose object table** (Junio): Cache loose object names in memory to short-circuit expensive `lstat()` calls during connectivity checks.
  - **Design tension**: The ODB abstraction’s goal of independent backends clashes with the need for performance optimizations in specialized use cases (e.g., connectivity checks, where false positives are tolerable).
- **Status**: Awaiting Patrick Steinhardt’s (ODB rework author) response to evaluate whether the abstraction can accommodate cross-backend optimizations or requires a broader redesign.

---

#### **2. `git history squash` v10: Ready for `next`**
**Headline**: *`git history squash` v10 addresses all feedback, including reflog accuracy and `rev-list` option sanitization*
- **What’s happening**: Harald Nordgren’s **10th iteration** of the `git history squash` series (a faster alternative to `git rebase --autosquash`) is now **technically complete** and **ready for integration**. The series folds a commit range into its oldest commit while preserving descendant history, avoiding the repeated conflict stops of rebase.
- **Key improvements in v10**:
  - **Reflog accuracy**: Now records the **full revision expression** (e.g., `squash: updating start..HEAD ^keep`) instead of truncating to the first argument.
  - **`rev-list` option sanitization**: Preserves the `boundary` walk invariant while enforcing `--reverse --topo-order`.
  - **`amend!` message handling**: Incorporates `amend!` messages targeting the first folded commit into the default squashed message.
  - **Test suite**: Expanded to cover 30+ branches, `--independent`, and `--merge-base` modes.
- **Open question**: Whether `--reedit-message` (or `--edit`) should be the default (minor usability tweak, not a blocker).
- **Status**: **Matt Hunter’s explicit v10 sign-off** confirms all prior concerns are resolved. Junio’s "Will replace" sign-off from v7 signals intent to merge. The series is **poised for `next`**.

---

#### **3. `git rebase --continue --[no-]edit`: A feature too far?**
**Headline**: *Junio questions the necessity of `--[no-]edit` for `git rebase --continue`*
- **What’s happening**: Hugo Sales proposed a `--[no-]edit` option for `git rebase --continue` to skip the editor prompt when accepting the existing commit message. The patch includes a new config option, `rebase.noEdit`.
- **Maintainer pushback**:
  - **Necessity**: Junio dismissed the feature as redundant, noting the existing workaround (`GIT_SEQUENCE_EDITOR=: git rebase --continue`).
  - **Config scope**: The `rebase.noEdit` config option is "especially concerning" because it could silently override user intent during interactive rebase steps (e.g., `edit`, `reword`).
  - **Edge cases**: The patch doesn’t address how the feature behaves when the rebase stops for reasons other than conflict resolution (e.g., `edit`/`reword` steps).
- **Status**: The author has not yet responded to Junio’s feedback. The thread highlights a **tension between workflow efficiency and Git’s design principles** (e.g., explicit user intent).

---

#### **4. `git cat-file --batch-command remote-object-info` v20: Dynamic capability negotiation**
**Headline**: *`git cat-file --batch-command remote-object-info` v20 adds dynamic capability negotiation, resolving edge cases*
- **What’s happening**: Pablo Sabater’s **20th iteration** of the security-hardened `remote-object-info` series is **queued for `next`**. The series enables Git clients to query object metadata (e.g., size) from remotes without downloading the full object, using the `object-info` protocol v2 capability.
- **Key improvements in v20**:
  - **Dynamic capability negotiation**: The client now requests only the metadata fields (`size`, `type`) that the server advertises support for, ensuring forward compatibility.
  - **Build fix**: Resolves a signed/unsigned comparison warning in `write_fetch_command_and_capabilities()`.
  - **Edge-case handling**: Bare OID requests trigger an unconditional existence check on the server, avoiding lazy-fetching in partial clones.
- **Status**: **Technically complete** and **cleared for graduation to `next`**. Minor cosmetic nits (e.g., documentation caveats) remain unresolved but are non-blocking.

---

#### **5. Swift userdiff driver: Graduating to `next`**
**Headline**: *Swift userdiff driver v2 promoted to `next` after addressing review feedback*
- **What’s happening**: Shlok Kulshreshtha’s **v2 patch** adding a built-in userdiff driver for Swift is now **queued in `next`**. The patch enables meaningful hunk headers and word-level diffs for `.swift` files.
- **Key details**:
  - **Funcname pattern**: Covers all 26 Swift declaration forms (attributes → modifiers → declaration keyword) with boundary conditions to prevent false matches.
  - **Word regex**: Handles Swift-specific literals (hexadecimal, octal, binary, floating-point) and operators (`<<=`, `??`, range operators).
  - **Test coverage**: 13 fixtures in `t/t4018/` covering classes, structs, enums, protocols, extensions, actors, and edge cases.
- **Status**: **Ready for integration testing** unless last-minute feedback arises from Swift users on the list.

---

### **In brief**
- **`git rev-list --exclude-first-parent-only` fix**: Junio’s patch to fix misbehavior with explicit commits received a **Reviewed-by** from Jerry Zhang (the option’s original author). The fix is **ready to cook**.
- **`git send-email` completion**: A patch to include filesystem paths in completion lists sparked a **usability debate**. SZEDER Gábor argued that shorter, focused completion lists (offering only refs) are more usable, while Yury Norov defended the change as addressing a real logic flaw.
- **`git repo info` path keys**: K Jayatheerth’s series adding path-related keys (e.g., `path.grafts`, `path.index`) to `git repo info` resolved the `path.grafts` design discussion (unconditional path reporting is intentional). The `path.index` behavior in bare repositories remains unresolved.
- **`git stash push` optimization**: Ted Nyman’s patch to avoid sparse-index expansion for cone-internal pathspecs **graduated to `next`** after Junio’s merge signal. The series includes a **heap-buffer-overflow fix** and delivers a **100x performance improvement** in cone mode.
- **`git config --global` and `include.*`**: The thread resolved that the current behavior (not respecting `include.*` by default) is intentional, but **documentation clarity** remains an open question.
- **`git submodule` URL alias fix**: Éric NICOLAS’s patch to fix `git submodule update --remote` with `url.<base>.insteadOf` aliases is **under review**. The fix resolves a 2023 regression by resolving aliases before matching remotes.

---

### **On the radar**
- **`paint_down_to_common()` optimization**: Kristofer Karlsson’s series (100-1000x speedup for asymmetric merge-base queries) is **queued in Junio’s tree** but stalled due to **reviewer availability**. Proposed simplifications (e.g., deferring fallback cleanup) aim to accelerate review.
- **`git show-branch` slab conversion**: Gatla Vishweshwar Reddy’s v10 patch (eliminating `object.flags` reliance) **fixed a critical regression** (division-by-zero crash) and is **ready for Junio’s confirmation**.
- **`git fsmonitor` macOS fix**: Tamir Duberstein’s patch to flush pending FSEvents before cookie wait **addresses a performance regression** on loaded macOS systems, reducing timeouts from ~89% to zero.
- **`git remote` push-tracking v3**: Harald Nordgren’s series to restore push-tracking display after remote renames **addressed Junio’s technical gap** (inclusion of `pushurl` and `pushInsteadOf` in URL-matching logic). A **documentation inconsistency** (claiming only `url` is used) remains unresolved.

---

### **Editorial note**
Today’s traffic underscored **Git’s evolving priorities**:
1. **Performance at scale**: The ODB regression thread revealed how architectural changes (e.g., ODB abstraction) can have **unintended consequences** for large deployments (e.g., NFS-backed servers). The discussion is shifting toward **targeted optimizations** for specialized use cases (e.g., connectivity checks).
2. **Usability vs. design principles**: The `git rebase --continue --[no-]edit` debate highlighted the **tension between workflow efficiency and Git’s emphasis on explicit user intent**. Junio’s skepticism suggests that **not all convenience features align with the project’s philosophy**.
3. **Review bandwidth**: Several series (e.g., `paint_down_to_common()`, `git show-branch` slab conversion) are **technically complete but stalled** due to reviewer availability, a recurring challenge for the project.