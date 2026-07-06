# The Git Project Mailing List Daily Digest
## 2026/07/05 (Sunday)

**The day in brief.**
A quiet Sunday on the list, with 29 emails across 9 threads. The standout developments: Junio queued two long-running patch series for merging—Patrick Steinhardt’s ODB refactor and Johannes Schindelin’s Coverity-driven leak fixes—while a build-system regression report sparked discussion about Rust’s growing role in Git. The only controversy of the day was Junio’s principled objection to a proposed `git rebase -i -x` feature, questioning whether it aligns with rebase’s core semantics.

---

## Notable threads

### **Rustification build system for macOS graduates to `next`**
Junio C Hamano signed off on the final v4 of Shardul Natu and Koji Nakamaru’s two-patch series, which enables macOS Universal Binary support for Rust components and fixes a parallel-build race condition in `git-credential-osxkeychain`. The series—touching only the `Makefile` and introducing the `RUST_TARGETS` environment variable—is now queued for merging. Patrick Steinhardt’s earlier concerns about Universal Binary completeness remain unresolved in principle but are accepted as out of scope. This is the first Rust-related build-system change to land since Git 2.55.0 made Rust components opt-out rather than opt-in, a shift that surfaced in a separate thread today when a user reported build failures on AlmaLinux 10.

---

### **`greplint.pl` series approved, test-suite hygiene improved**
Junio accepted Michael Montalbo’s six-patch series introducing `greplint.pl`, a test-suite linter that converts bare `grep` assertions to `test_grep` for better debuggability. The series, which evolved from a simple infrastructure improvement to a systemic audit of test bugs, is now approved for merging after addressing all prior concerns: auditing `# lint-ok:` exemptions, fixing edge cases (reftable backend compatibility, NTFS 8.3 short names), and documenting `test_grep`’s requirements in `t/README`. The linter’s shared shell parser (`lib-shell-parser.pl`) also paves the way for future test-suite hygiene efforts. Expect this to graduate to `next` shortly.

---

### **Coverity-driven leak fixes land in v2**
Johannes Schindelin posted the final v2 of his 12-patch series plugging resource leaks and error-path bugs flagged by Coverity. The series—reviewed by Patrick Steinhardt, Jeff King, and Eric Sunshine—addresses memory, file-descriptor, and process-handle leaks across loose-object handling, bundle URI downloads, run-command API, diff machinery, and Windows-specific code. Junio has stated he will queue the series for merging, indicating strong maintainer approval. The patches are mechanical, well-scoped, and carry minimal regression risk, making them ideal for early-cycle integration. A latent bug in `read_one_dir()` (uninitialized slots in `->dirs` and `->untracked`) is noted but will be addressed separately.

---

### **`git rebase -i -x`: A feature in search of a justification**
Trevor Gross’s patch introducing a `-x` flag to `git rebase -i`’s `pick`, `reword`, and `edit` commands—mirroring `git cherry-pick -x`—drew a sharp design critique from Junio. The maintainer argued that rebase (moving history) and cherry-pick (duplicating history) are fundamentally different workflows, and the `-x` flag’s purpose is undermined when original commits are discarded. Junio’s objection remains unresolved, though Matt Hunter’s review surfaced a pre-existing uninitialized memory bug in `do_pick_commit()` that Trevor’s patch exposes. The thread’s future hinges on whether the author can reconcile the feature with rebase’s core semantics or propose an alternative design.

---

## In brief

**ODB refactor queued for merging** -- Junio confirmed that Patrick Steinhardt’s six-patch series refactoring `struct object_info` to use an opt-in `source_infop` field is now in his integration pipeline. The series, part of the ODB abstraction effort, replaces the `whence` field with `struct odb_source_info` to enable multi-source object resolution and pluggable backends.

**CI PID limits raised for private repositories** -- Junio marked Johannes Schindelin’s patch raising PID, process, and file descriptor limits in Dockerized GitHub Actions jobs for private repositories as ready for `next`. The change addresses the "cannot create async thread: Resource temporarily unavailable" error that consistently fails tests in the t5xxx-t6xxx range.

**Guilt v0.37.1 released** -- Frediano Ziglio announced a new minor release of Guilt, a third-party tool that emulates Mercurial Queues workflows in Git. The update introduces new commands (`guilt rename`, `guilt goto`) and options, along with performance optimizations. Guilt remains a niche but long-standing tool in the Git ecosystem.

**Build system regression sparks Rust discussion** -- A user reported that Git 2.55.0 now unconditionally requires `cargo` to build, breaking the build on AlmaLinux 10. D. Ben Knoble clarified that Rust is opt-out for now (`NO_RUST=1` works) but will become mandatory in Git 3.0. The thread highlights the growing tension between Rustification and platform support, particularly for non-mainstream platforms like NonStop.

---

## On the radar

**`git rebase -i -x` design debate** -- Junio’s principled objection to Trevor Gross’s `-x` flag patch remains unresolved. The thread is worth tracking for its implications on rebase’s design philosophy and whether the feature can be justified despite the conceptual mismatch with cherry-pick.