# Here’s the daily digest for the Git mailing list on **2026/06/19 (Friday)**:

---

### **The day in brief**
A **heavy-traffic day** (102 emails, 28 threads) dominated by **feature finalization, bugfixes, and design debates**. The standout developments:
1. **`git cat-file --batch-command` remote object info** (Pablo Sabater) reached **v13**, addressing all security and protocol concerns, and is now **merge-ready**.
2. **`git history squash` ref-handling design** (Harald Nordgren) was **finalized**—refs pointing into the squashed range will be **rejected by default** with an `advice()` message.
3. **`git branch --delete-merged` stacked-branch safety** (Harald Nordgren) saw **Junio endorse the abort-and-notify approach**, clearing the last design hurdle.
4. **MIDX incremental repack regression** (Taylor Blau) was fixed with a **3-patch series**, restoring usability for custom bases.
5. **`git log --graph` cascading indentation** (Pablo Sabater) remains blocked by **architectural fragility**, but Kristofer Karlsson proposed a **clean abstraction** to resolve it.

---

### **Notable threads**

#### **1. `git cat-file --batch-command` remote object info (v13) – Pablo Sabater**
**Headline**: Security-hardened remote object metadata queries are now **merge-ready** after 13 iterations.
**What’s new**: The final version introduces **dynamic capability-based validation** to prevent information leaks, fixes memory leaks in `transport-helper`, and adds strict protocol enforcement. All prior review feedback (Junio, Chandra Pratap, Eric Ju) has been addressed.
**Key details**:
- **Protocol v2**: Only "size" is advertised initially; "objecttype" returns empty strings.
- **Security**: Input size limits (8K URL length), backward iteration for safe list compaction, and empty-string returns for unsupported atoms.
- **Tests**: Shared test infrastructure (`lib-cat-file.sh`) and comprehensive coverage for edge cases.
**Status**: **Ready for `next`**. No outstanding objections.

---

#### **2. `git history squash` ref-handling design – Harald Nordgren**
**Headline**: **Finalized design**: Refs pointing into the squashed range will be **rejected by default** with an `advice()` message.
**What’s new**: Junio endorsed Patrick Steinhardt’s proposal to **abort and notify** users when refs (e.g., branches, tags) point to commits inside the squashed range (e.g., `o---A---B---C` → `o---X`). Users can override this with `--update-refs[=head]`.
**Key details**:
- **Behavior**: Rejects operations like `git history squash @~3..` if `feature-branch` points to `B`.
- **Advice**: Suggests `--update-refs` to retarget refs to the squashed commit.
- **Edge cases**: Fixup-commit sequences and merge commits are handled correctly.
**Status**: **Design resolved**; code updates pending. **Blocked by 3 technical concerns** (range-resolution logic) raised by Patrick.

---

#### **3. `git branch --delete-merged` stacked-branch safety – Harald Nordgren**
**Headline**: **Junio endorsed the abort-and-notify approach**, clearing the last design hurdle.
**What’s new**: Junio agreed that deleting a branch used as an upstream for an unmerged stacked branch (e.g., `feature1` if `feature2` stacks on it) would break `@{u}` references and implicit `git rebase` workflows. The default behavior will now **abort with a clear error message**.
**Key details**:
- **User feedback**: "Cannot delete branch 'feature1' because 'feature2' depends on it."
- **Override**: `--retarget-upstreams` flag to automatically update dependent branches.
- **Scope**: Applies to both `--delete-merged` and `-d`.
**Status**: **Design finalized**; implementation pending.

---

#### **4. MIDX incremental repack regression – Taylor Blau**
**Headline**: **3-patch series fixes `git multi-pack-index write --incremental --base`**, restoring reachability closure for bitmaps.
**What’s new**: The regression (introduced in Git 2.55-rc1) caused the command to ignore custom bases, breaking incremental repacking. The fix ensures the selected base is respected.
**Key details**:
- **Impact**: Bitmap generation failed when a custom base was specified.
- **Tests**: New helper (`nth_line()`) and coverage in `t5319-multi-pack-index.sh`.
- **Files**: `midx.c`, `t/t5319-multi-pack-index.sh`.
**Status**: **Needs review** (Jeff King CC’d). Low-risk, well-scoped fix.

---

#### **5. `git log --graph` cascading indentation – Pablo Sabater**
**Headline**: **Kristofer Karlsson proposed a lightweight abstraction** to resolve the architectural fragility.
**What’s new**: The series remains blocked by `graph_peek_next_visible()`’s direct access to `revs->commits`, but Kristofer’s proposal (two new helpers in `revision.c`) would insulate `graph.c` from the walker’s internals.
**Key details**:
- **Proposal**: `revision_has_more_commits()` and `revision_peek_next_commit()` to dispatch based on walk mode.
- **Status**: Pablo to prototype the abstraction. **No progress on the 3 key flags** (`is_next_visible`, `next_has_column`, `is_next_visual_root`).
**Status**: **Blocked**; architectural redesign needed.

---

### **In brief**
- **`git rebase --abort` hints** (Harald Nordgren): Junio questioned whether the hint should be shown for *all* exec failures (e.g., missing files) or only irrecoverable ones. **Design unresolved**.
- **`git log <file>` omits commits** (Vincent Lefevre): Junio clarified the subtree-merge behavior as intentional (workaround: specify all historical paths). **`--reverse` issue remains unaddressed**.
- **Sparse-checkout data loss** (charmocc): `git checkout` in sparse mode **silently overwrites untracked files** and emits a misleading warning. **New bug report**.
- **`ignore_case` libification** (Justin Tobler/Tian Yuchen): v3 series **ready for `next`**; no substantive objections.
- **Pinned references RFC** (Erik Östlund): Junio ruled out modifying `<describeOutput>`; discussion now focuses on a **new command or option**.
- **macOS CI timeouts**: Patrick Steinhardt traced the issue to **orphaned `httpd` processes** in `t5551`/`t5559`. **Root cause identified**; fix pending.

---

### **On the radar**
- **`git replay --linearize`** (Thomas Koutcher): New command in `next`; no recent discussion.
- **`git refs create|delete|update|rename`** (Patrick Steinhardt): Ref backend subcommands; **needs review**.
- **`git history drop`** (Patrick Steinhardt): New command; **needs feedback**.
- **`mm/diff-process-hunks`**: External hunk providers; **stalled due to security/performance concerns**.