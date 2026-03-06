# Git Mailing List Digest - 2025/09/25

## The day in brief

A busy day with 104 emails across 28 threads, featuring significant progress on multiple fronts. Key highlights include the Rust infrastructure series nearing completion, major hook subsystem refactoring, and resolution of several long-running discussions. Junio Hamano's "What's cooking" report provides a comprehensive snapshot of the project's current state.

## Notable threads

### Rust infrastructure approaches completion

The Rust infrastructure series (v7) received final review from Junio Hamano, indicating it's likely to merge after addressing all major technical concerns. The series now includes:

- Meson and Makefile build system support
- Varint subsystem implementation as first Rust component
- CI modernization for Rust testing
- Documented adoption timeline (optional in 2.52, default in 2.53, mandatory by 3.0)

Key changes in v7 include renaming the "git" crate to "gitcore" to avoid Windows/MSVC conflicts and refining the BreakingChanges.adoc wording. While Eric Wong raised questions about modern C safety features as alternatives, the technical work appears ready for integration.

### Hook subsystem modernization advances

Adrian Ratiu's 10-part series to refactor Git's hook subsystem made significant progress, converting several key hooks to use the new `hook.h` interface:

- Added stdin callback mechanism for streaming input
- Implemented output capture for server-side hooks
- Converted post-rewrite, pre-push, and receive-pack hooks
- Reduced code size while maintaining behavior

The series demonstrates measurable improvements, with some hook implementations seeing 50%+ line reduction. Junio provided detailed review feedback focusing on documentation quality and interface design, suggesting the technical work is sound but needs clearer explanations before final approval.

### `git add -p` hunk splitting behavior settled

Phillip Wood's series fixing `git add -p`'s hunk splitting behavior reached conclusion with maintainer approval. The final version:

- Makes split hunks always `UNDECIDED_HUNK` (requiring explicit selection)
- Removes conditional `WITH_BREAKING_CHANGES` guard
- Fixes edge cases in edited hunk splitting
- Includes comprehensive test coverage

Junio endorsed the change despite workflow tradeoffs, emphasizing UI clarity over convenience in cases where split hunks might inherit parent state. The series resolves a long-standing interaction design question in the interactive patch interface.

## In brief

**Reftable fsck validation** -- Patrick Steinhardt and Karthik Nayak refined terminology for update index validation rules, settling on "strictly monotonically increasing" to precisely describe the required ordering between tables.

**SHA-1/SHA-256 interoperability** -- brian m. carlson clarified GPG signature handling in tag objects, explaining the design rationale for having either `gpgsig` or `gpgsig-sha256` headers but not both simultaneously.

**git-push documentation** -- Julia Evans and Junio Hamano refined the man page's explanation of upstream branch requirements, debating whether to frame it as a safety measure or fundamental push target determination.

**git stash show behavior** -- A user reported confusion when `git stash show` hides untracked files by default, prompting Junio to consider changing the `stash.showIncludeUntracked` default in Git 3.0.

**git whatchanged deprecation** -- Junio Hamano confirmed plans to remove the `git whatchanged` command, recommending `git log --raw` as the preferred alternative despite user feedback praising whatchanged's utility.

**Outreachy internship planning** -- Christian Couder confirmed mentor assignments for the December 2025 internship, focusing on global state removal work similar to Ayush's 2025 GSoC project.

## On the radar

**Pluggable ODB design debate** -- Junio Hamano challenged Patrick Steinhardt's position that commit graphs should be backend-specific, arguing they represent fundamental repository metadata needed by core operations regardless of storage backend.

**git refs get architecture** -- Discussion continues about whether to consolidate ref operations under a new `git-refs` command family or extend existing tools like `show-ref`, with Junio skeptical of the discoverability benefits.