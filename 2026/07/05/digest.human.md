# The Git Project Mailing List Daily Digest
## 2026/07/05 (Sunday)

## The day in brief.

A quiet Sunday on the list, with 29 emails across 9 threads. Two long-running patch series reached the finish line: the Rustification build system adjustments for macOS and the `greplint.pl` test-suite linter both received final maintainer sign-off and are now queued for merging. The most substantive discussion centered on a proposed `-x` flag for `git rebase -i`, where Junio C Hamano raised fundamental questions about the feature’s alignment with rebase’s core semantics.

---

## Notable threads

### macOS Rustification build system adjustments finalized
The two-patch series from Shardul Natu and Koji Nakamaru that enables Universal Binary support for Rust components and fixes a parallel build race condition in `git-credential-osxkeychain` received its final sign-off from Junio C Hamano. The series, now at v4, is uncontroversial and touches only the `Makefile`, introducing the `RUST_TARGETS` environment variable for multi-architecture builds. A minor follow-up clarified a cosmetic inconsistency in the patch description, confirming the series is complete and ready for integration. This is the last administrative step before the changes land in `next`.

### `greplint.pl` test-suite linter approved for merging
Michael Montalbo’s six-patch series introducing `greplint.pl`, a linter that converts bare `grep` assertions to `test_grep` in the test suite, received maintainer approval after a thorough audit of edge cases and `# lint-ok:` exemptions. Junio C Hamano’s only quibble—a minor phrasing issue in the `t/README` documentation—was addressed in a follow-up, and the series is now queued for merging. The linter, which uncovered 10+ pre-existing test bugs during development, is integrated into the build system with new `test-greplint` and `check-greplint` targets. This is a significant infrastructure improvement that will improve test debuggability and reduce latent bugs in the test suite.

### Coverity-driven leak fixes land in v2
Johannes Schindelin posted a 12-patch v2 series addressing resource leaks and error-path bugs identified by Coverity in Git’s codebase. The series, which Junio C Hamano has stated he will queue for merging, plugs memory, file-descriptor, and process-handle leaks across subsystems including loose-object handling, bundle URI downloads, run-command API, diff machinery, and Windows-specific code. Key fixes include replacing unsafe `errno`-based error detection with explicit `ferror(fp)` checks, consolidating cleanup paths in `get_superproject_working_tree()`, and enforcing clear ownership semantics in Windows process termination logic. The patches are mechanical, well-scoped, and carry Reviewed-by tags from Patrick Steinhardt, Jeff King, and Eric Sunshine.

### `git rebase -i -x`: design questions remain
Trevor Gross’s patch introducing a `-x` flag to `git rebase -i`’s `pick`, `reword`, and `edit` commands to append "(cherry picked from commit ...)" lines sparked a substantive design discussion. Junio C Hamano raised fundamental questions about the feature’s alignment with rebase’s core semantics, arguing that rebase (which moves history) and cherry-pick (which duplicates history) serve fundamentally different workflows. The `-x` flag’s purpose is undermined when original commits are discarded, and the feature could encourage workflows that contradict rebase’s design. Matt Hunter also identified a pre-existing uninitialized memory bug in `do_pick_commit()` that the patch exposes, suggesting the initialization fix should be split into a separate patch. The thread’s future hinges on whether the author can reconcile the feature with rebase’s core semantics or propose an alternative design.

---

## In brief

**`struct object_info` refactor queued for merging** -- Patrick Steinhardt’s six-patch series refactoring `struct object_info` to use an opt-in `source_infop` field for multi-source object resolution was queued by Junio C Hamano. The series, part of the ODB abstraction effort, replaces the `whence` field with `struct odb_source_info` and is now in the integration pipeline.

**CI PID limits raised for private repositories** -- Johannes Schindelin’s patch raising PID, process, and file descriptor limits in Dockerized CI jobs for private GitHub repositories was marked for the `next` branch. The change addresses the "cannot create async thread: Resource temporarily unavailable" error in the t5xxx-t6xxx test range.

**Rust build requirement clarified** -- D. Ben Knoble clarified that Git 2.55.0 defaults to building with Rust enabled, but the Rust components remain optional via `NO_RUST=1`. The change aligns with the Rustification effort, though Rust is not yet a requirement (planned for Git 3.0).

**`--sstat` alias withdrawn** -- Junio C Hamano withdrew his patch introducing `--sstat` as a hidden synonym for `git diff --compact-summary` after Phillip Wood’s substantive review objected to the alias’s cryptic nature and UI clutter. The thread is now closed.

**Guilt v0.37.1 released** -- Frediano Ziglio announced Guilt v0.37.1, a third-party tool that emulates Mercurial Queues workflows in Git. The release introduces new commands (`guilt rename`, `guilt goto`) and options, along with performance optimizations.

---

## On the radar

**Rustification timeline** -- The discussion about Git 2.55.0’s default Rust build requirement highlights the ongoing Rustification effort. While Rust is not yet mandatory, the timeline for Git 3.0 (where Rust may become a requirement) remains a topic to watch, particularly for platforms like NonStop that lack Rust support. Randall S. Becker’s concerns about platform compatibility may resurface as the effort progresses.