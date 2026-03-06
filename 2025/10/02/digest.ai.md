# Git Mailing List Digest - 2025/10/02

**The day in brief.** A busy day with 118 emails across 23 threads, dominated by major developments in Rust infrastructure finalization, SHA-256 interoperability groundwork, and build system refactoring. Key highlights include Junio's approval of the Rust integration series, resolution of the commit graph refactoring debate, and significant progress on Git 3.0 planning. Documentation improvements and CI hardening also saw substantial activity.

## Notable threads

**Rust infrastructure finalized** -- Patrick Steinhardt's 9-part series establishing Rust support in Git core reached its final form (v8) with typo fixes and maintainer approvals from Junio Hamano and Ezekiel Newren. The implementation includes build system modernization, CI validation, and a varint subsystem proof-of-concept demonstrating C/Rust interoperability. The policy documentation formalizes a phased adoption timeline (optional in 2.52, default in 2.53, mandatory by 3.0 in H2 2026) with escape hatches for distributors. This marks a major milestone in Git's Rust transition after 8 iterations of review.

**Commit graph refactoring compromise** -- The long-running debate about commit graph storage in the pluggable ODB effort reached resolution with Patrick Steinhardt proposing and Junio Hamano approving a new direction: making commit graph caching pluggable at the repository level while keeping graph data repository-wide accessible. This addresses Junio's core concern about maintaining repository-wide access while allowing implementation flexibility. The first five uncontroversial patches will merge while the problematic sixth patch tying graphs to ODB sources is dropped.

**SHA-256 interoperability groundwork** -- brian m. carlson posted part 1 of a multi-part series establishing documentation and test infrastructure for SHA-1/SHA-256 interoperability. The 9 patches focus on pack format documentation corrections, loose object storage specification, and test prerequisites (`COMPAT_HASH`, `BROKEN_OBJECTS`) for compatibility testing. The series represents foundational work enabling future interoperability patches, with all architectural decisions now settled. Technical highlights include pack index v3 format updates and GPG signature validation for both hash algorithms.

**Build system consolidation complete** -- Ezekiel Newren's series restructuring Git's static library build process was successfully integrated into 'seen' after resolving merge conflicts. The changes fold xdiff and reftable functionality directly into libgit.a, matching meson's behavior and supporting future Rust integration. The thread revealed interesting design discussions about Makefile organization (keeping intermediate *_OBJS variables vs direct LIB_OBJS inclusion) before settling on the simpler direct inclusion approach preferred by Junio. This represents both an immediate build improvement and important groundwork for Rust integration.

**Atomic ref updates for git replay** -- Siddharth Asthana's series to make atomic reference updates the default in `git replay` reached its polishing phase, addressing extensive review from Elijah Newren and Junio Hamano. Key changes include removing the `--allow-partial` option (due to lack of use cases), improving test coverage of atomic behavior, and correcting documentation about `git update-ref --stdin` already providing atomicity. The thread also explored philosophical questions about output format design, with Kristoffer Haugsbakk suggesting a `git-for-each-ref`-style `--format` approach for future consideration.

## In brief

**Documentation policy discussion** -- Ben Knoble introduced a philosophical tangent about LLM capabilities in the AI-generated contributions thread, referencing the "LLMentalist Effect" concept while acknowledging it as separate from the ongoing policy formulation.

**git-history copyedits** -- Kristoffer Haugsbakk provided several documentation polish suggestions for Patrick Steinhardt's `git-history` series, correcting terms like "worktree" to "working tree" and fixing typos in commit messages.

**Reftable fsck validation** -- Karthik Nayak's reftable consistency check series received final review feedback from Patrick Steinhardt, with only a missing positive test case noted before merging.

**Worktree documentation improvements** -- Michal Suchánek and Junio Hamano collaborated on patches improving `git worktree` documentation, particularly warning against nested worktrees and demonstrating bare repository workflows.

**Symbolic reference validation** -- The thread about hardening symbolic reference handling in the files backend reached consensus on centralizing validation in `git refs verify` while maintaining `git fsck` detection capabilities.

**CI reliability improvements** -- A 5-part series addressed Windows GitLab CI flakiness by removing problematic workarounds, adding JUnit reporting, and fixing test 8020's non-deterministic tag sorting.

**Documentation formatting fixes** -- Ramsay Jones posted a 4-part series fixing asciidoc warnings across technical docs (`commit-graph.adoc`, `sparse-checkout.adoc`, etc.) to achieve parity between Makefile and meson builds.

## On the radar

**Git 3.0 planning** -- The SHA-256 interoperability discussion evolved into release strategy debates, with Patrick Steinhardt advocating for roadmap-based planning while Junio Hamano questioned real-world adoption constraints.

**Hook subsystem refactoring** -- Adrian Ratiu's series converting hooks to use new infrastructure received detailed technical feedback from Patrick Steinhardt about parallel process stdin handling and API consistency.

**git whatchanged deprecation** -- User feedback confirmed ongoing use of this legacy command, with Kristoffer Haugsbakk providing migration guidance to `git log --no-merges --raw`.

**Interactive add bug report** -- Ulrich Windl reported unexpected behavior with the "J" (join) hunk command in `git add --interactive`, suggesting a regression in hunk selection logic.