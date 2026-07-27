Here’s the digest for **2026/07/26**, a Sunday with a mix of high-impact feature work, regression fixes, and infrastructure improvements. The day’s traffic was **moderate but technically dense**, with several threads reaching key milestones.

---

### The day in brief
Git’s mailing list saw **73 emails across 20 threads**, a typical volume for a weekend but with notable progress on long-running series. The **`diff.<driver>.process` RFC (v6)** reached technical completion, while **`git rebase --update-refs`** and **`git mv`** bugfixes cleared final design hurdles. A **performance regression in ODB alternates handling** gained architectural clarity, and **CI toolchain upgrades** moved closer to integration. The day’s standout: **Junio’s explicit approvals** for multiple series, signaling a wave of near-term merges.

---

### Notable threads

#### **`diff.<driver>.process` RFC reaches technical completion (v6)**
**Headline:** Michael Montalbo’s **9-patch RFC** for external diff/blame tool integration is now **"technically complete and ready for merging"** after resolving the last merge conflict with `mm/line-log-limited-ops`. The series introduces a **long-running subprocess protocol** that lets tools inject hunks into Git’s pipelines while preserving downstream features (word diff, `--stat`, `git blame`, `git log -L`).

### Key updates:

- **Patch 8/9** wired `--stat`/`--numstat` to the external process, ensuring counts align with the tool’s hunks (not Git’s builtin diff).
- **Patch 9/9** completed the integration by making `git log -L` consult the tool, ensuring consistency between commit selection and diff output.
- **Junio’s final sign-off** confirmed the series is unblocked; the only remaining cases using Git’s builtin diff are combined diffs (`--cc`).

**Why it matters:** This is a **major step toward structural diffs** (e.g., AST-aware tools) without sacrificing Git’s existing features. The protocol’s design—**forward-compatible, cache-aware, and robust to tool failures**—sets a new standard for external tool integration.

---

#### **`git rebase --update-refs` symref bugfix nears merge**
**Headline:** Son Luong Ngoc’s **2-patch series** fixing edge cases with symbolic references in `git rebase --update-refs` cleared its last design hurdle. Junio’s **final review** approved the use of `has_symlink_leading_path()` to reject *all* tracked symlinks in the destination path, aligning with `git apply` precedent.

### Key updates:

- **Patch 2/2** now handles **non-branch symrefs** (e.g., tags) and **cross-worktree conflicts**, with **83 new test lines** covering symlinks as both final and intermediate components.
- **Junio’s "sounds good"** signals readiness for `next`; the only acknowledged loose end is a **TOCTOU race** (deemed "the best we can do").

**Why it matters:** This fixes a **real-world pain point** for branch rename migrations and multi-worktree setups, where symrefs could previously cause rebase failures or index corruption.

---

#### **ODB alternates performance regression gains architectural clarity**
**Headline:** The **12% slowdown in `git receive-pack`** (introduced by the ODB rework) now has a **proposed architectural fix**: move alternates handling into the "files" backend. **Peff endorsed Patrick Steinhardt’s redesign**, dismissing concerns about lost flexibility as "unlikely to be practical."

### Key updates:

- **Root cause confirmed**: The ODB rework shifted from "pack-first, loose-second" *per backend* to "pack-loose, pack-loose" *across backends*, causing redundant loose object checks.
- **Proposed fix**: Treat alternates as an **implementation detail of the files backend**, enabling pack-first-loose-second lookups across all alternates while simplifying commit-graph/bitmap loading.
- **Near-term vs. long-term**: A targeted lookup order fix may still land before Patrick’s redesign (planned for late July).

**Why it matters:** This regression affects **large-scale hosting environments** (e.g., GitHub, GitLab) using `GIT_ALTERNATE_OBJECT_DIRECTORIES`. The redesign could **eliminate secondary issues** (e.g., `OBJECT_INFO_SECOND_READ` flag) while sidestepping cross-process coordination challenges.

---

#### **`git mv` misleading error messages fixed**
**Headline:** Lucas Zamboni Orioli’s **2-patch series** fixing misleading `git mv` errors (e.g., blaming the source file when the destination directory is missing) is now **ready for integration**. Junio’s **final review** approved the use of `has_symlink_leading_path()` to reject symlinks in the destination path.

### Key updates:

- **Patch 1/2** improves error messages to include both source and destination paths.
- **Patch 2/2** adds an early existence check for the destination directory, rejecting **missing directories, non-directory components, and tracked symlinks**.
- **Test coverage**: 83 new lines in `t/t7001-mv.sh`, including a regression test for index corruption when moving through symlinks.

**Why it matters:** This is a **user-facing fix** with **no downsides**, addressing a long-standing usability issue while preventing subtle index/worktree inconsistencies.

---

#### **CI toolchain upgrades move forward**
**Headline:** Jeff King’s **2-patch series** bumping the Ubuntu image for static-analysis jobs to `ubuntu-24.04` (picking up Coccinelle 1.3.0+) received **Junio’s approval**. The series also includes a **preparatory cleanup** in `bloom.c` to silence a Coccinelle false positive.

### Key updates:

- **Patch 1/2** refactors an assertion in `bloom.c` to avoid a false positive in Coccinelle’s `CHECK_ASSERTION_SIDE_EFFECTS` rule.
- **Patch 2/2** updates `.github/workflows/main.yml` and `.gitlab-ci.yml` to use `ubuntu-latest`.

**Why it matters:** This removes a **prior in-code workaround** for Coccinelle’s performance regression, improving CI reliability and reducing maintenance burden.

---

### In brief
- **`git stash rename`**: User testimonials (brian m. carlson, erik88) reinforced the justification for the feature, but **Junio’s skepticism persists** about the current stash design. No progress on implementation.
- **`paint_down_to_common()` optimization**: Elijah Newren’s **resumed review** of the technical document surfaced precision gaps around v1 commit-graph handling. Kristofer Karlsson proposed a `commit_graph_generation_topo_ceiling()` helper to clarify invariants.
- **`git repo info` path keys**: K Jayatheerth’s **7-patch series** adding path-related keys (e.g., `path.toplevel`, `path.hooks`) is now **mechanically clean and ready for substantive review**. Junio’s acknowledgment signals openness to the design.
- **`rebase -i` edge cases**: Phillip Wood’s **2-patch bugfix** for skipped fixup/squash commands is **queued for `next`** after Junio’s final approval.
- **UTF-8 string-width refactoring**: Hardik Kumar’s **v2 patch** changing `utf8_strwidth()` to return `size_t` addressed reviewer concerns by updating all callers. The **overflow check removal** remains unresolved.
- **`commit_body()` test helper**: Shlok Kulshreshtha’s **2-patch series** introducing a reusable helper for commit message bodies is **under review**, with no objections yet.

---

### On the radar
- **ODB alternates redesign**: Patrick Steinhardt’s **architectural shift** (moving alternates into the "files" backend) is the most consequential in-flight work, with **Peff’s endorsement** boosting its credibility.
- **`git stash` redesign**: Junio’s suggestion to **tie stashes to branches** could subsume the `rename` feature, but no concrete proposals yet.
- **Rustification**: No updates; the effort remains **contentious** (Randall S. Becker’s NonStop port concerns vs. memory safety goals).

---

### Editorial note
The day’s traffic underscored Git’s **dual focus on stability and innovation**. While **bugfixes and CI improvements** dominated the "in brief" section, the **`diff.<driver>.process` RFC** and **ODB alternates redesign** hint at the project’s long-term architectural ambitions. Junio’s **explicit approvals** for multiple series suggest a **wave of merges** in the coming weeks, particularly for user-facing fixes (`git mv`, `rebase -i`) and infrastructure work (CI, test helpers). The **ODB alternates discussion** remains the most technically complex, with its resolution likely to shape Git’s performance in large-scale deployments for years.