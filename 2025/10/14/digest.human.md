# Git Mailing List Digest - 2025/10/14

**The day in brief.** A busy Tuesday with 97 emails across 28 threads saw significant progress on several fronts. The `git-history` command series advanced through maintainer integration, the SHA-1/SHA-256 interoperability work reached final approval, and a major `--trailer` rebase feature series entered its fourth iteration. Documentation discussions around Git's data model and reference hierarchy continued to refine technical accuracy while maintaining pedagogical clarity.

## Notable threads

### `git-history` command integration progresses

Patrick Steinhardt's `git-history` series (v4, 12 patches) has been rebased onto the latest `seen` branch by Junio Hamano with only minor conflicts. The series introduces new `reword` and `split` subcommands for history editing, building on shared `replay.c` infrastructure. Karthik Nayak completed a thorough technical review of the implementation, confirming core functionality while identifying minor style nits and one UX consideration about commit message placement during splits. The series appears ready for potential inclusion pending resolution of these final polish items.

### SHA-1/SHA-256 interoperability approved

The foundational SHA-1/SHA-256 interoperability work received final approval from Patrick Steinhardt after Junio Hamano's stability confirmation and brian m. carlson's readiness declaration. This 9-patch subset includes pack index format updates, loose object storage clarifications, tag signature behavior documentation, new plumbing commands for hash reporting, enhanced fsck validation, and comprehensive test infrastructure. The approval marks a significant step in the multi-phase effort to enable hash algorithm compatibility.

### Rebase trailer support refined

Li Chen's 29-patch series adding `--trailer` support to `git rebase` entered its fourth iteration, having addressed reviewer feedback from previous versions. The core change moves trailer manipulation from the external `interpret-trailers` helper into in-process Git code via a new `amend_strbuf_with_trailers()` function. The implementation forces use of the merge backend, automatically enables `--force-rebase` for rewritten commits, and includes comprehensive test coverage. Review feedback from Kristoffer Haugsbakk focused on documentation formatting and test presentation details rather than core functionality.

### Git data model documentation evolves

Julia Evans' `gitdatamodel.adoc` man page reached version 3, incorporating insights from 48 test users. The document explains Git's four core data types (objects, references, index, and reflogs) while avoiding implementation details. Version 3 adds concrete examples for tags and reflogs, clarifies index/staging relationships, and addresses user confusion points like detached HEAD state. The thread included extensive discussion about reference hierarchy explanations, with Junio Hamano noting differing mental models of where the hierarchy begins (`refs/` vs root level).

## In brief

**Stash config documentation clarification** -- D. Ben Knoble and Kristoffer Haugsbakk resolved final questions about merge commit references in documentation for the already-merged `stash.index` config series.

**Refs optimization naming finalized** -- Junio Hamano confirmed `--check-for-auto` as the flag name for `git refs optimize` functionality that determines if optimization is worthwhile, concluding the naming discussion.

**Hook parallelization file descriptor handling** -- Adrian Ratiu reversed course on changing file descriptor marking in the hook subsystem, maintaining the existing `0` convention due to broader run-command API constraints.

**Patch-id hash algorithm debate** -- brian m. carlson and Junio Hamano discussed whether patch IDs should use SHA-1 (for stability) or respect repository hash algorithms (for future compatibility), proposing a configurable approach.

**Symlink ref deprecation warning** -- A patch proposes deprecating `core.preferSymlinkRefs` for Git 3.0, warning when symbolic refs are written as symlinks rather than modern "ref: " files.

**Test modernization corrections** -- Junio Hamano provided detailed feedback on proper use of test_path helpers when converting shell checks in t2401-worktree-prune.sh, emphasizing diagnostic improvements over readability.

**HTTP 403 authentication behavior** -- A proposed change to treat 403 responses like 401s for credential prompting faced pushback from brian m. carlson who argued it violates HTTP semantics and suggested alternative solutions.

## On the radar

**Rustification effort** -- Ezekiel Newren's Rust integration work remains active but faces platform support concerns from Randall S. Becker regarding NonStop compatibility.

**`the_repository` removal** -- Outreachy applicants are being onboarded to this ongoing effort, with guidance to examine `builtin/` files for refactoring opportunities.

**Documentation synopsis conversion** -- Jean-Noël Avila's man page modernization work continues in the background, with recent patches like Kristoffer Haugsbakk's `git-patch-id` update following the new style.