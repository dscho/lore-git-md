Here's the daily digest for September 25, 2025:

---

### The day in Brief
A busy Thursday with 104 emails across 28 threads saw significant progress on multiple fronts. Key developments include final approval for Phillip Wood's `git add -p` hunk splitting changes, Junio Hamano signaling readiness to merge Patrick Steinhardt's Rust infrastructure series, and a major hook subsystem refactoring nearing completion. The day also featured lively discussions about command architecture and a surprise announcement about `git whatchanged`'s impending removal.

---

### Notable Threads

**Rust infrastructure clears final hurdles**  
Patrick Steinhardt's v7 series introducing Rust support appears headed for merge after Junio Hamano's final review. The 9-patch series now includes:
- Meson/Makefile build system integration
- Varint subsystem proof-of-concept
- Phased adoption timeline (optional in 2.52, default in 2.53, mandatory by 3.0)
- CI validation across platforms

The implementation addresses Windows/MSVC concerns through a crate rename to "gitcore" and defers LTS handover details. While Eric Wong raised questions about modern C safety features as alternatives, the technical work appears complete pending any last-minute feedback.

**Hook subsystem modernization completes**  
Adrian Ratiu's 10-part series refactoring Git's hook infrastructure reached completion, converting all major hooks to use the new `hook.h` interface. The changes introduce:
- Streaming stdin via callbacks
- Sideband output capture
- server hook optimizations
- Unified process management

The series demonstrates significant code reduction (often 50%+ per hook) while maintaining behavior. Junio provided extensive documentation feedback throughout, ensuring interface clarity for this foundational change.

**`git add -p` hunk splitting finalized**  
Phillip Wood's behavioral fix for hunk splitting in interactive patches received maintainer approval. The change makes split hunks always require explicit selection (`UNDECIDED_HUNK` state) rather than inheriting parent state, addressing workflow confusion. Junio endorsed the unconditional approach despite some increased selection overhead, prioritizing interface clarity over convenience.

**`git whatchanged` deprecation announced**  
In a brief but decisive exchange, Junio Hamano stated plans to remove the legacy `git whatchanged` command, recommending `git log --raw` or `--stat` as modern alternatives. The announcement came in response to user praise for the command's utility, suggesting the decision reflects interface cleanup priorities rather than functional deficiencies.

**Repository statistics command matures**  
Justin Tobler's `git repo stats` series reached v3 with comprehensive repository health metrics:
- Reference/object counting
- Multiple output formats (table, keyvalue, NUL)
- Progress reporting
- Thorough test coverage

The implementation follows Git's design patterns while providing actionable data previously requiring external tools like git-sizer. Eric Sunshine noted an i18n consideration in table formatting that will need addressing.

---

### In Brief

**Reftable validation refinements** -- Patrick Steinhardt and Karthik Nayak finalized terminology for update index checks, settling on "strictly monotonically increasing" to precisely describe reftable stack requirements.

**Packfile store abstraction complete** -- Junio marked Patrick Steinhardt's packfile storage refactoring ready for 'next' after all substantive feedback was addressed in v6.

**SHA-1/SHA-256 interoperability** -- brian m. carlson clarified design decisions around GPG signature headers in tag objects during hash algorithm transitions.

**Outreachy mentorship confirmed** -- Christian Couder finalized mentor assignments for the December 2025 internship focusing on global state removal work.

**xdiff modernization wraps up** -- Ezekiel Newren completed the series with final cleanups to similarity matching variables and reconsidered an aliasing change after practical experience.

**Range-diff notes consistency** -- A v2 series fixed `git format-patch` notes display inconsistency between main output and range-diff sections for single commits.

---

### On the Radar

**Pluggable ODB design debate** -- Junio challenged Patrick Steinhardt's position on commit-graph ownership in the ongoing object database abstraction effort, favoring repository-wide over backend-specific graphs.

**`git refs get` architecture discussion** -- Thread continues debating whether new subcommands improve discoverability over extending existing tools like `show-ref`.

**`stash.showIncludeUntracked` default** -- Junio expressed unusual openness to changing this default despite versioning implications, acknowledging the current behavior's potential for confusion.

**`git merge --no-ff` stash failure** -- Early bug report suggests platform-specific issue with stash operations during merges, needing more diagnostic details.

--- 

Today's traffic showed Git's development ecosystem operating at full capacity - from foundational infrastructure work to user-facing refinements, with maintainer decisions shaping the project's future direction. The volume and diversity of activity reflect healthy engagement across the contributor community.