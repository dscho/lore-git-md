Here’s the daily digest for the Git mailing list on **2026/07/01 (Wednesday)**:

---

### **The day in brief**
A **heavy-traffic day** (134 emails, 29 threads) with **two major milestones** and **one contentious debate**. The **`git history drop`** subcommand series was **unblocked and merged**, resolving a critical ref-resolution dispute. Meanwhile, the **`git replay --linearize`** series, already merged, faced **post-merge scrutiny** for a **silent commit-dropping regression** and a **CLI design debate** that escalated into a **philosophical discussion** about UX consistency. A **new quadratic-time scalability issue** in the reftable backend emerged, with a **proposed fix** from Kristofer Karlsson. The day also saw **CI infrastructure discussions**, **memory-leak fixes**, and **platform-specific bug reports** for Git for Windows.

---

### **Notable threads**

#### **1. `git history drop` subcommand merged**
**Headline**: Patrick Steinhardt’s 11-patch series adding `git history drop` (remove a commit and replay descendants) was **unblocked and merged** after resolving a **critical ref-resolution bug** in `find_head_tree_change()`.
**What happened**: Junio C Hamano approved v8 after Patrick fixed the bug (adding `RESOLVE_REF_READING` to `refs_resolve_ref_unsafe()`) and added a test for detached-HEAD scenarios. The series modernizes the reset API, advances `the_repository` removal, and adds conflict detection.
**Why it matters**: This is a **major feature** for Git’s experimental history-editing toolkit, enabling safer commit removal without the repeated conflict stops of a rebase-based approach.
**Next steps**: The series is now in `next` and will graduate to `master` in the next cycle.

---

#### **2. `git replay --linearize` post-merge regression and CLI debate**
**Headline**: The **already-merged** `git replay --linearize` series (Toon Claes) faces **two critical issues**: a **silent commit-dropping regression** in single-branch replay and a **CLI design debate** that escalated into a **philosophical discussion** about UX consistency.
**What happened**:
- **Regression**: Johannes Schindelin identified a **data-loss bug** where `--linearize` drops intermediate commits when replaying a single branch with merges. Toon Claes acknowledged the issue and agreed the command should produce a **single linear sequence** regardless of input branches.
- **CLI debate**: Patrick Steinhardt argued for consistency with `git rebase`’s `--rebase-merges` syntax, while Johannes Schindelin **rejected `git rebase` as a usability anti-pattern** and advocated for **intentional divergence** (e.g., `--replay-merges=<mode>`). The debate now hinges on **whether to document the divergence** in commit messages.
**Why it matters**: The regression is a **high-priority fix** for Git 2.56, while the CLI debate reflects **broader tensions** between consistency and UX clarity. The outcome may set a precedent for future CLI changes.
**Next steps**: Toon Claes will send a **follow-up patch** to restore the `replayed_base` logic and clarify the intended behavior. The CLI design question remains unresolved.

---

#### **3. Reftable backend quadratic-time scalability issue**
**Headline**: A **new quadratic-time scalability issue** in the reftable backend was identified during bulk ref deletion/re-creation, with a **proposed fix** from Kristofer Karlsson.
**What happened**: Jeff King and Patrick Steinhardt diagnosed the issue (tombstone iteration inefficiency) in `refs_verify_refname_available()` and `reftable_be_transaction_prepare()`. Kristofer Karlsson proposed a fix exposing tombstones to iterator bounds checks, reducing runtime from **O(n²) to O(n)** (e.g., 16,000 refs: 7.1s → 0.258s).
**Why it matters**: This is a **real-world performance cliff** for workflows like `git for-each-ref | git update-ref --stdin`, though it primarily affects artificial benchmarks.
**Next steps**: Kristofer will send a formal patch for review, with discussion likely focusing on **code clarity vs. performance trade-offs**.

---

#### **4. CI infrastructure discussions**
**Headline**: Jeff King and Patrick Steinhardt debated **consolidating CI jobs** for leak detection and reftable testing.
**What happened**: Peff proposed merging `linux-reftables-leaks` and `linux-TEST-vars` into a single job (`linux-TEST-vars-leaks`), while Patrick suggested extending the approach to macOS. The discussion also touched on **renaming jobs** for clarity (e.g., `linux-changed-defaults`).
**Why it matters**: This is a **process-level improvement** to reduce CI maintenance overhead and improve leak detection coverage.
**Next steps**: The consolidation will proceed, with macOS support likely added later.

---

#### **5. Memory-leak fixes**
**Headline**: Two series targeting memory leaks were posted:
- **Jeff King’s 3-patch series** fixing leaks in Bloom-filter code (merged to `next`).
- **Johannes Schindelin’s 13-patch Coverity-driven cleanup** (awaiting review).
**Why it matters**: These are **low-risk, high-impact fixes** that improve Git’s robustness, especially under leak sanitizers.

---

### **In brief**
- **`git cat-file --batch-command` remote object metadata**: Pablo Sabater’s **15th iteration** of the GSoC series is **ready for merging**, with all prior feedback addressed. The series adds `remote-object-info` support for querying object sizes from remotes.
- **Parse-options help exit codes**: Brian m. Carlson’s **4-patch series** standardizing `--help`/`-h` exit codes to 0 (success) was **merged**, resolving a long-standing inconsistency.
- **`excludes_file` libification**: Tian Yuchen’s **3-phase migration** (silent return → `BUG()` → no check) is **complete and merged**, eliminating a global variable.
- **Git for Windows bugs**: Two reports emerged:
  - **HTTP/HTTPS authentication regression** (domain accounts) in 2.55.0.windows.1 (redirected to Git for Windows issue tracker).
  - **`git gui` encoding issue** with non-ASCII home directories (fix proposed).
- **Build system fixes**: Mike Gilbert fixed a **Meson build race condition** for `hook-list.h`.

---

### **On the radar**
- **`git history squash` template debate**: Phillip Wood and Junio C Hamano are debating whether to diverge from `git rebase -i`’s template format for `--reedit-message`. The outcome may influence future UX decisions.
- **Reftable backend optimizations**: Beyond the quadratic-time fix, **redundant `stat()` calls** and **unbuffered reflog writes** remain low-hanging fruit.
- **CI job consolidation**: The proposed `linux-TEST-vars-leaks` job may be extended to macOS.

---

### **Editorial note**
Today’s traffic reflected **Git’s dual nature**: **feature development** (`git history drop`, `git replay --linearize`) and **maintenance** (leak fixes, CI improvements). The **`git replay --linearize` debate** stands out as a **philosophical clash** between consistency and UX clarity, with implications for how Git evolves its CLI. Meanwhile, the **reftable scalability issue** highlights the **trade-offs** in backend design (tombstoning vs. compaction). The **`git history drop` merge** is a bright spot, showing the project’s ability to **resolve critical issues** and deliver new functionality.