Here’s the daily digest for the Git mailing list on **2026/06/22 (UTC)**:

---

### **The day in brief**
A **heavy-traffic Monday** (119 emails, 27 threads) saw **two major series reach critical milestones**: Patrick Steinhardt’s **ref backend lazy-loading redesign (v5)** resolved its last architectural debate, while Taylor Blau’s **`--delta-islands` + `--path-walk` integration** was **merged to `next`**. Controversy flared over **`git branch --delete-merged`’s stacked-branch protection**, and Junio C Hamano **finalized a governance patch** requiring design justification in resubmissions. **Notable absences**: no progress on the stalled `git cat-file --batch-command` security series or Harald Nordgren’s usability improvements for `git branch`/`push` misformattings.

---

### **Notable threads**

#### **Ref backend lazy-loading redesign (v5) lands**
**Topic**: Patrick Steinhardt’s 11-patch series to resolve recursive initialization issues in Git’s reference backend.
**What happened**: The v5 rewrite **abandoned the v4 `ignore_refs` workaround** in favor of a **lazy-loading design** that defers write-config parsing until the first write operation. The series now includes a **recursion guard** in `get_main_ref_store()` and has **resolved all high-weight objections** from Jeff King (Peff) and Derrick Stolee. The reftable backend’s refactor (patches 9–10/11) was the most complex change, requiring dynamic write option passing.
**Why it matters**: This is a **foundational shift** in how ref backends interact with configuration, eliminating a long-standing initialization-order constraint. The redesign aligns with Peff’s advocacy for minimizing early config loading and may serve as a template for future backend work (e.g., ODB-based ref stores).
**Next steps**: The series is **ready for final review** and likely to graduate to `next` soon. Reviewers may focus on edge cases in deferred config parsing (e.g., races in concurrent write operations).

---

#### **`--delta-islands` + `--path-walk` integration merged**
**Topic**: Taylor Blau’s 4-patch series integrating `--delta-islands` with `--path-walk` in `git pack-objects`.
**What happened**: Junio C Hamano **queued the series for `next`**, calling it "very cleanly implemented." Derrick Stolee confirmed the **performance validation** (up to 99.3% speedup in object-counting while preserving path-walk’s compression benefits). The only lingering uncertainty—Junio’s and Stolee’s admitted lack of confidence in the delta-islands logic—was resolved by the empirical data.
**Why it matters**: This removes a **long-standing limitation** in `git pack-objects`, enabling bitmap acceleration for path-limited traversals. The series is now **effectively merged**, with graduation to `master` dependent on its base topics (`ds/path-walk-filters` and `ps/clang-w-glibc-2.43-and-_Generic`).

---

#### **`git branch --delete-merged`’s stacked-branch protection debated**
**Topic**: Harald Nordgren’s 7-patch series adding `--delete-merged` to `git branch`.
**What happened**: The v17 iteration **finalized the stacked-branch protection design** as "abort-and-notify" (failing with an error if a branch is used as an upstream for another branch). However, Phillip Wood **proposed a more nuanced alternative**: keep only the upstreams of unmerged branches and clear the upstream config for merged branches in the chain. The debate centers on whether the current approach is **too restrictive** for workflows involving chains of dependent branches (e.g., `b1→b2→b3`).
**Why it matters**: The series is **feature-complete** and ready for merging, but this design question could delay it. The outcome will set a precedent for how Git handles **branch dependencies** in automated cleanup commands.
**Next steps**: Harald to respond to Phillip’s proposal or proceed with the current design.

---

#### **`git replay --linearize` at architectural crossroads**
**Topic**: Toon Claes’s 3-patch series adding `--linearize` to `git replay`.
**What happened**: Junio C Hamano and Patrick Steinhardt **reopened the debate** over Patch 1/3’s boolean refactoring (converting `enum replay_mode` to a `reverse` flag). Steinhardt also questioned whether `--linearize` should **mirror `git rebase`’s full flexibility** (e.g., `--rebase-merges`, `--rebase-merges=rebase-cousins`) instead of hardcoding `--no-rebase-merges`.
**Why it matters**: The series is **technically complete** but faces **two architectural concerns**:
1. **Boolean vs. enum**: Junio illustrated how `create_commit(..., true)` obscures intent at the callsite, favoring enums for future extensibility.
2. **Interface scope**: Steinhardt’s critique suggests the current design may be **too narrow** to fully emulate `git rebase`.
**Next steps**: Toon to address the boolean refactoring (likely reverting to the enum) and clarify the interface scope.

---

#### **Governance patch merged: Design justification required in resubmissions**
**Topic**: Junio C Hamano’s patch updating `Documentation/SubmittingPatches` to require explicit design justification in resubmissions.
**What happened**: The patch was **merged after Patrick Steinhardt’s LGTM**, codifying the expectation that contributors must address high-level critiques (e.g., "is this feature worth implementing?") before resubmitting. The final text softened prescriptive language and added concrete examples.
**Why it matters**: This aims to reduce "zombie patches" that advance through iterations without resolving fundamental objections. However, Kristoffer Haugsbakk’s earlier critique—that the requirement may **disproportionately burden non-implementers**—remains unaddressed.
**Next steps**: Monitor whether the patch reduces reviewer frustration or creates new barriers for contributors.

---

### **In brief**
- **`git cat-file --batch-command` security series**: Pablo Sabater’s v13 **stalls** after Karthik Nayak flagged a **protocol compatibility regression risk** in Patch 6/12 (replacing `GIT_HASH_SHA1_LEGACY` with `GIT_HASH_SHA1`). The series is otherwise ready for merging.
- **`git log --follow` non-linear history fix**: Miklos Vajna’s series **merged to `master`** after Junio resolved a trivial merge conflict. This removes a long-standing limitation in rename tracking across merge commits.
- **`git history squash`**: Harald Nordgren’s v4 **awaits response** to Patrick Steinhardt’s question about adding an `--ancestry=` option to mirror `git log`’s range interpretation.
- **`git branch --set-upstream-to` usability**: Harald Nordgren’s patch to improve error messages for missing slashes **faces skepticism** from Junio, who argues the current error ("branch 'main' does not exist") is accurate. The discussion now centers on whether the advice should only appear when `origin/main` exists.
- **`paint_down_to_common()` optimization**: Tian Yuchen’s 6-patch series **resolved all high-weight objections** after Kristofer Karlsson accepted Derrick Stolee’s refactoring alternative. The series now awaits v2 with **hyperfine-based performance benchmarks**.
- **macOS CI hangs**: Patrick Steinhardt **diagnosed the root cause** as Apache’s 300-second timeout being hit during `ls-refs` advertisements. The thread now debates whether to **increase the timeout, optimize the test further, or extend keepalives** to cover `ls-refs`.

---

### **On the radar**
- **`USE_NSEC` debate**: Patrick Steinhardt **challenged the existence** of the `nanosec` Meson option, arguing that `USE_NSEC` is subtly broken on some filesystems (e.g., CEPH, CIFS). The discussion could lead to **deprecation or removal** of the knob.
- **`git clone --only/--except` RFC**: Pushkar Singh’s proposal to add sparse-checkout paths to `git clone` **lacks feedback**. The interface design (comma-separated vs. repeated options) remains unresolved.
- **External notes commands**: Siddh Raman Pant’s 4-patch series **remains unreviewed** after two pings. The implementation is technically mature but lacks domain expert evaluation of its security model.