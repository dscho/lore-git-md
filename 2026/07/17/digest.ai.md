Here’s the daily digest for **2026/07/17**, covering the Git mailing list traffic:

---

### **The day in brief**
A **heavy but focused** day on the Git mailing list, with **101 emails across 22 threads**. The standout themes:
1. **Security and robustness** dominated with the **`git cat-file --batch-command` remote object info series (v19)** landing in `next` after 19 iterations, addressing critical security hardening and dynamic server capability negotiation.
2. **Performance optimizations** took center stage with **`git last-modified` Bloom filter reuse** (4× speedup) and **`wt-status` quadratic-time fixes** (O(n²) → O(n log n)).
3. **Design debates** flared around **`git replay --linearize`** (multi-branch ambiguity) and **`git stash rename`** (motivation vs. broader stash redesign).
4. **Refactoring efforts** advanced with **ODB abstraction** (loose object backend), **`the_repository` removal** (refspec, copy.c), and **worktree isolation bugs** in `git history fixup`.
5. **Documentation clarifications** progressed with Junio’s contributor guidance series, now acknowledging reviewer disinterest as a reason for silence.

The day’s tone was **collaborative but cautious**, with maintainers emphasizing **human oversight** in contributions (e.g., Junio’s critique of AI-generated patches) and **incremental progress** over grand redesigns.

---

### **Notable threads**

#### **1. `git cat-file --batch-command` remote object info (v19) lands in `next`**
**Headline**: *Security-hardened remote object metadata queries graduate to `next` after 19 iterations.*
**Author**: Pablo Sabater
**Status**: **Queued for `next`** (technically complete; minor cosmetic nits remain).
**Key details**:
- Adds `remote-object-info` to `git cat-file --batch-command`, enabling clients to query object metadata (size, eventually type) from remotes **without downloading full objects**.
- **Security**: 10,000-object batch limit, 8KB URL cap, dynamic allow-list for server-advertised capabilities (e.g., `size`, `type`).
- **Robustness**: New commit ensures bare OID requests trigger unconditional existence checks on the server, avoiding lazy-fetching in partial clones.
- **Protocol**: Uses `object-info` v2 capability, advertised by servers with `transfer.advertiseobjectinfo=true`.
- **Tests**: 747 lines of new tests covering all smart transports (`git://`, `file://`, `http://`), `--buffer` mode, and edge cases (malformed OIDs, legacy protocols).
**Why it matters**: This is a **major milestone** for partial clones and sparse checkouts, enabling efficient metadata queries while maintaining security. The series’ 19-iteration journey highlights Git’s rigorous review process for protocol-level changes.

---

#### **2. `git replay --linearize` multi-branch ambiguity stalls in `next`**
**Headline**: *Maintainer defends current behavior; author must reconcile competing design philosophies.*
**Author**: Toon Claes
**Status**: **On hold in `next`**, waiting for author response.
**Key debate**:
- **Junio’s position**: The current behavior (concatenating branches into a single linear chain) is **intentional and reasonable**—users explicitly request `--linearize` *and* update multiple branches, so concatenation is the direct result.
- **Elijah Newren’s position**: The behavior violates the **principle of least astonishment**, creates a **recoverability asymmetry** (untangling concatenated branches is harder than concatenating independent ones), and is inconsistent with `--advance`/`--revert`.
- **Toon’s dilemma**: Must choose between:
  1. Defending the status quo (aligning with Junio).
  2. Adopting Elijah’s **minimal fix** (restrict `--linearize` to a single positive ref).
  3. Proposing a third alternative (e.g., Junio’s syntax suggestions).
**Why it matters**: This is a **rare design-level standoff** in Git, where maintainers disagree on fundamental UX principles. The outcome will set a precedent for how Git handles **ambiguous user intent** in complex commands.

---

#### **3. `git last-modified` Bloom filter optimization (4× speedup)**
**Headline**: *Experimental command now matches `git log` speed for wildcard pathspecs.*
**Author**: Toon Claes
**Status**: **Under review** (v1 posted; correctness bugs and test gaps identified).
**Key details**:
- **Performance**: Reduces runtime from **52 ms → 14 ms** on ziglang/zig (3.7× speedup) by reusing Bloom filters instead of re-fetching them.
- **Optimizations**:
  - Patch 2/4: Exposes `revs_maybe_changed_in_bloom()` to avoid redundant filter lookups.
  - Patch 3/4: Wires Bloom-filter pre-filtering into `git last-modified`.
  - Patch 4/4: Fixes wildcard pathspecs (e.g., `*.c`) by restoring `bloom_filter_settings` after `prepare_revision_walk()`.
- **Review feedback**:
  - **Junio**: Identified **unintended behavioral changes** in Bloom filter statistics reporting (patch 2/4).
  - **Taylor Blau**: Found a **correctness bug** with `--show-trees` (patch 3/4) and a **test coverage gap** (patch 4/4).
**Why it matters**: This is a **textbook example of performance optimization**—leveraging existing infrastructure (Bloom filters) to eliminate redundant work. The fixes will benefit **GitLab, Forgejo, and other tools** that batch directory entries.

---

#### **4. ODB abstraction: Loose object backend refactoring**
**Headline**: *Loose object writing logic moves to `odb/source-loose.c`, making the backend self-contained.*
**Author**: Patrick Steinhardt
**Status**: **Under review** (9-patch series posted).
**Key details**:
- **Goal**: Eliminate direct calls to `write_loose_object()` from `object-file.c`, consolidating loose backend internals.
- **Preparatory steps**:
  - Centralize object hash computation and compatibility ID handling.
  - Add mtime support to ODB write/freshen interfaces.
  - Fix a memory leak in `force_object_loose()`.
- **Final patch (9/9)**: Moves all loose-object writing helpers to `odb/source-loose.c`, leaving only `odb_source_loose_write_stream` in `object-file.c` (marked TODO).
**Why it matters**: This is a **critical step** in the ODB abstraction effort, enabling future pluggable backends. The series is **well-structured and incremental**, following Git’s pattern of **refactoring before redesign**.

---

#### **5. `git history fixup` corrupts index in other worktrees**
**Headline**: *New bug report reveals worktree isolation flaw in `git history`.*
**Reporter**: Toon Claes
**Status**: **Newly reported**; no patch yet.
**Key details**:
- **Bug**: Running `git history fixup` in one worktree **leaves spurious staged changes** in other worktrees sharing the same repository.
- **Reproduction**: Minimal (fresh repo, one commit, one worktree added, one `fixup` invocation).
- **Impact**: Silent index corruption in unrelated worktrees.
**Why it matters**: This is a **serious correctness bug** for multi-worktree users. The `git history` command is still new (Git 2.46, 2026), so edge cases like this are expected, but the **worktree isolation flaw** could affect other commands if not fixed.

---

#### **6. Documentation: Contributor guidance clarifies reviewer silence**
**Headline**: *Junio’s series now acknowledges disinterest as a reason for reviewer silence.*
**Author**: Junio C Hamano
**Status**: **Under review** (v2 pending).
**Key addition**:
- Explicitly lists **reviewer disinterest** as a possible reason for silence, alongside busyness and oversight.
- Encourages reviewers to **communicate disinterest** rather than leaving contributors in limbo.
**Why it matters**: This is a **small but significant cultural shift**, addressing a **recurring pain point** for newcomers. The change reflects Git’s **growing emphasis on transparency** in the review process.

---

### **In brief**
- **`git bisect --auto-reset=<where>` (v2)**: Harald Nordgren implements Junio’s parameterized design (`original`/`found`), enabling both automated workflows and interactive debugging. **Queued for `next`**.
- **`git stash rename`**: Junio questions the feature’s practicality (e.g., rewording 211 stashes) and suggests **broader stash redesign** (branch-tied stashes). **Design debate ongoing**.
- **`git repo info` path keys (v2)**: K Jayatheerth’s GSoC series adds 7 new keys (e.g., `path.toplevel.absolute`, `path.hooks.relative`). **Awaiting technical review**.
- **`git show-branch` slab conversion (v9)**: Gatla Vishweshwar Reddy addresses Junio’s cultural concerns by demonstrating **human understanding** of the code paths. **Ready for integration**.
- **`gitweb` index line rendering fix (v2)**: Travor Liu squashes tests and aligns with `test_grep` conventions. **Ready for `next`**.
- **`git rebase -i` fixup/squash edge cases (v1)**: Phillip Wood fixes incorrect commit counts and editor launch failures. **Under review**.
- **Swift userdiff driver**: Shlok Kulshreshtha adds built-in support for Swift. **Queued in `seen`**; awaiting validation from Swift users.

---

### **On the radar**
- **`git replay --linearize`**: Toon Claes must respond to the **multi-branch ambiguity debate** to unblock the series.
- **`git stash rename`**: Author needs to address **motivation and design concerns** or risk the feature being deferred.
- **`git last-modified`**: Toon Claes must fix the **`--show-trees` correctness bug** and **test coverage gap** before the series can progress.
- **ODB abstraction**: Patrick Steinhardt’s loose object backend refactoring awaits **reviewer feedback**.

---

### **Editorial note**
Today’s traffic underscored Git’s **dual priorities**: **security/robustness** (e.g., `git cat-file` hardening) and **incremental improvement** (e.g., ODB refactoring, performance optimizations). The **design debates** (`git replay`, `git stash`) highlight the project’s **cautious approach to UX changes**, while the **documentation updates** reflect a **growing focus on contributor experience**. The **worktree bug** in `git history fixup` is a reminder that **new features must rigorously test edge cases**—especially in multi-worktree setups.