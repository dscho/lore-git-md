Here's the Git mailing list digest for September 2, 2025:

## The day in brief

A busy day with 96 emails across 34 threads, dominated by major architectural discussions around Rust adoption timelines and the ongoing packfile store refactoring. Key highlights include Patrick Steinhardt's proposal for an LTS version to ease the transition, completion of the packfile store series, and several security-related patches.

## Notable threads

**Rust transition plan takes shape** -- Patrick Steinhardt proposed designating the last pre-Rust Git version as an LTS release that would receive security fixes until gccrs stabilizes (projected 2026). The suggestion gained support from Gentoo's Sam James but faced pushback from Brian Carlson who argued for faster adoption based on productivity gains. The discussion revealed technical timelines including a possible Git 3.0 milestone for mandatory Rust dependency.

**Packfile store refactoring completes** -- Patrick Steinhardt's 15-patch series to migrate packfile management to `struct packfile_store` reached completion after thorough review. The changes systematically separate packfile state from the object database to enable future pluggable ODB backends. Junio Hamano noted only minor merge conflicts remain before integration.

**Gitweb command injection vulnerabilities addressed** -- Taylor Blau submitted a comprehensive fix adding `--end-of-options` protection throughout Gitweb's Git command invocations. The patch prevents malicious URL parameters from being interpreted as command options, addressing a demonstrated security issue where crafted URLs could write arbitrary files.

**Reftable fsck validation advances** -- Karthik Nayak's v2 series adding integrity checks for the reftable backend received review feedback from Junio Hamano questioning whether some validations belong in runtime code rather than fsck. The series implements four specific stack-level checks while maintaining separation between reftable internals and Git's fsck infrastructure.

**Case-insensitive filesystem fixes** -- A 2-patch series addressed reference update issues on case-insensitive filesystems, fixing both case conflicts between references and file/directory case collisions. The changes improve error handling and prevent lock file leaks while maintaining the recommendation to use reftable as the long-term solution.

## In brief

**ODB source system integration** -- Patrick Steinhardt confirmed the MIDX interface changes in his ODB abstraction series are ready for merging after addressing Derrick Stolee's review feedback.

**Documentation verification tests** -- Junio Hamano submitted test fixups for the allowlist mechanism verifying builtin documentation, adding the missing `adoc-missing` file and correcting prereq logic.

**Submodule path encoding** -- Junio noted the submodule mixed gitdir test should explicitly set `GIT_TEST_DEFAULT_INITIAL_BRANCH_NAME` for future-proofing.

**Meson build reorganization** -- Patrick defended using `subproject()` over `subdir()` for Git's GUI components despite requiring directory restructuring, viewing it as a one-time cost for cleaner architecture.

**git refs exists finalized** -- Meet Soni's `git refs exists` subcommand received final approvals and will be merged, completing the GSoC project to consolidate ref commands.

**Default branch naming** -- Discussion continued on the WITH_BREAKING_CHANGES implementation to switch default branch from "master" to "main", focusing on documentation clarity and test coverage.

**On the radar**

**Rust FFI tooling consensus** -- Ezekiel Newren reversed his position on cbindgen after hands-on experience, now supporting its use for C/Rust interop in the initial integration phase. This aligns with brian m. carlson's earlier recommendations.

**git whatchanged deprecation UX** -- Users expressed dissatisfaction with `--raw` as the recommended replacement, arguing the name doesn't intuitively convey "show changed files" functionality. Junio maintained the 20-year-old option name won't be changed.