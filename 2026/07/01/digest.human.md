Here’s the daily digest for the Git mailing list on **2026/07/01 (Wednesday)**:

---

### **The day in brief**
A **heavy-traffic day** (134 emails, 29 threads) dominated by **feature finalizations, regression fixes, and philosophical debates** about Git’s design. The standout developments:
1. **`git history drop` lands** – Patrick Steinhardt’s 11-patch series introducing the `drop` subcommand to `git history` was **merged after resolving a critical ref-resolution bug**, marking a milestone for the experimental history-editing tool.
2. **`git replay --linearize` controversy** – A **post-merge regression** in Toon Claes’s `--linearize` option sparked a **philosophical debate** about CLI consistency vs. UX clarity, with Patrick Steinhardt demanding **explicit justification for divergences** from `git rebase`’s syntax.
3. **Rustification progress** – Shardul Natu’s **universal binary support for macOS** in the Rust credential helper advanced the build system integration, while a **build dependency debate** (Jan Palus vs. brian m. carlson) highlighted tensions between Rust and C interoperability.
4. **Security and performance** – Pablo Sabater’s **`git cat-file --batch-command` remote object metadata series** (v15) is **ready to merge**, and Kristofer Karlsson’s **quadratic-time fix for reftable** (100x speedup) emerged as a **high-impact optimization**.

---

### **Notable threads**

#### **1. `git history drop` lands after critical fix**
**Headline**: *`git history drop` subcommand merged to `next`*
**Author**: Patrick Steinhardt
**Status**: **Merged** (11/11 patches)
**Key details**:
- The series adds a `drop` subcommand to `git history` to **remove a commit and replay its descendants**, with **conflict detection, bare repository support, and `--update-refs` handling**.
- A **critical ref-resolution bug** in `find_head_tree_change()` (identified by Junio) was fixed in v8, ensuring index/worktree updates aren’t skipped when HEAD moves.
- **Test coverage**: 537 lines in `t/t3454-history-drop.sh`, covering edge cases like detached HEAD and bare repos.
- **Broader impact**: Advances `the_repository` removal and modernizes the reset API (e.g., `reset_working_tree()` with opt-in HEAD updates).
**Why it matters**: This is a **major step for Git’s history-editing toolkit**, offering a safer alternative to `git rebase -i` for commit removal. The fix for the ref-resolution bug ensures the feature is **production-ready**.

---

#### **2. `git replay --linearize` regression sparks CLI design debate**
**Headline**: *Post-merge regression in `--linearize` triggers philosophical debate*
**Author**: Toon Claes (original), Johannes Schindelin (regression report), Patrick Steinhardt (CLI critique)
**Status**: **Merged to `master` (v5)**, but **regression identified** (silent commit dropping in single-branch replay)
**Key details**:
- **Regression**: `--linearize` now **drops intermediate commits** when replaying a single branch with merges (e.g., `git replay --linearize master~2..master` replays only the tip commit, losing "Git 2.55-rc2").
- **Root cause**: v5’s base-selection logic **removed the `replayed_base` mechanism**, breaking single-branch replay.
- **CLI debate**:
  - **Johannes Schindelin** argues `--linearize` should **flatten all replayed commits into a single linear history**, calling this "predictable behavior."
  - **Patrick Steinhardt** demands **explicit justification** for diverging from `git rebase`’s `--rebase-merges` syntax, framing it as a **process-level principle** to avoid arbitrary inconsistencies.
  - **Toon Claes** defends `--linearize` as the **correct name** for the feature’s scope (flattening all commits, not just merges).
- **Follow-up**: A **patch to restore `replayed_base`** is urgently needed, and the CLI debate may require a **documented design decision**.
**Why it matters**: This thread exposes a **tension between UX clarity and CLI consistency**, with Patrick’s demand for **explicit justification** setting a precedent for future divergences. The regression is **high-priority** for Git 2.56.

---

#### **3. Rustification: Universal binaries and build system tensions**
**Headline**: *Rust credential helper gains macOS universal binary support*
**Author**: Shardul Natu
**Status**: **Ready to merge** (v1)
**Key details**:
- Adds **`RUST_TARGETS` support** to the Makefile, allowing users to specify multiple target triples (e.g., `aarch64-apple-darwin,x86_64-apple-darwin`) for **universal static libraries**.
- Uses `lipo` to combine target-specific libraries, addressing a **real-world need** for macOS users.
- **Build system debate**: Jan Palus’s patch to **remove a spurious `$(LIB_FILE)` dependency** was **rejected by brian m. carlson**, who clarified that the Rust code **does depend on `libgit.a`** (e.g., for `src/hash.rs` and `cargo test`).
**Why it matters**: This is a **clean step forward for Rust integration**, but the build system debate highlights **ongoing challenges** in Rust/C interoperability. The universal binary support is **particularly valuable for macOS users**.

---

#### **4. `git cat-file --batch-command` remote object metadata (v15)**
**Headline**: *Security-hardened remote object metadata feature ready to merge*
**Author**: Pablo Sabater
**Status**: **Ready to merge** (15/15 patches)
**Key details**:
- Adds a **`remote-object-info` command** to `git cat-file --batch-command`, allowing clients to **query object metadata (e.g., size) from remotes without downloading full objects**.
- **Security measures**: Dynamic capability-based validation, 10,000-object batching limits, and strict protocol v2 enforcement.
- **Protocol extensibility**: Future metadata (e.g., `objecttype`) will be appended to the `object-info` capability (e.g., `object-info=size type`).
- **Error handling**: Silent continuation for missing metadata (no consensus on explicit failure).
- **Test coverage**: 680 lines in `t/t1017-cat-file-remote-object-info.sh`.
**Why it matters**: This is a **long-awaited feature** for partial clones and large repositories, with **strong security hardening** and **forward-compatible design**. The series is **mature and ready for integration**.

---

#### **5. Reftable quadratic-time fix (100x speedup)**
**Headline**: *Kristofer Karlsson fixes reftable’s quadratic-time bottleneck*
**Author**: Kristofer Karlsson
**Status**: **Patch proposed** (not yet submitted)
**Key details**:
- **Problem**: Bulk ref deletion/re-creation (e.g., `git for-each-ref | git update-ref --stdin`) scales **quadratically** (8,000 refs: 1.8s → 16,000 refs: 7.1s).
- **Root cause**: Tombstone iteration in `merged_iter_next_entry()` and `refs_verify_refname_available()`.
- **Fix**: Exposes tombstones to iterator bounds checks, enabling **early termination** (16,000 refs: 0.258s).
- **Trade-off**: Adds tombstone-skipping loops at call sites, increasing code complexity.
**Why it matters**: This is a **high-impact optimization** for large repositories, addressing a **real-world scalability issue**. The patch is **data-driven and well-motivated**, but its adoption may hinge on **code clarity trade-offs**.

---

### **In brief**
- **`git blame -b` usability fix**: René Scharfe’s patch to **align hash lengths with `core.abbrev`** was **confirmed by the original reporter** (Laszlo Ersek) and is ready to merge.
- **`git history squash` template debate**: Phillip Wood and Junio debated whether `--reedit-message` should **omit `fixup!` noise** for clarity, with Harald Nordgren deferring to `git rebase -i`’s format for consistency.
- **Coverity fixes**: Johannes Schindelin’s 13-patch series **plugging resource leaks** received **mixed reviews** (Patrick Steinhardt flagged **ineffective fixes** in patches 5/13 and 7/13).
- **CI improvements**: Johannes Schindelin’s patch to **adjust PID limits in private GitHub repos** was **clarified by Junio** (it raises, not reduces, the limit).
- **`excludes_file` libification**: Tian Yuchen’s series **completed Phase 2** (removing the guardrail), fulfilling Junio’s **hard requirement for merging**.

---

### **On the radar**
1. **`git replay --linearize` regression**: Urgent patch needed to restore `replayed_base` logic.
2. **CLI design consistency**: Patrick Steinhardt’s demand for **explicit justification** in commit messages may influence future interface changes.
3. **Reftable quadratic-time fix**: Kristofer Karlsson’s patch could land soon, pending code clarity discussions.
4. **`git history squash` default behavior**: Whether `--reedit-message` should be the default remains unresolved.
5. **Rust/C build system tensions**: The `$(LIB_FILE)` dependency debate highlights **interoperability challenges**.

---

### **Editorial note**
Today’s traffic reflected a **mature project grappling with design trade-offs**—not just technical fixes, but **philosophical questions** about consistency, UX, and process. The `git replay --linearize` debate, in particular, may set a **precedent for how Git handles CLI divergences** in the future. Meanwhile, **performance optimizations** (reftable, merge-base) and **security-hardened features** (`cat-file --batch-command`) show the project’s **continued evolution** toward scalability and safety.