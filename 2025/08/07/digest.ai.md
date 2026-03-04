# Git Mailing List Digest - 2025/08/07

**The day in brief.** A busy day with 97 emails across 19 threads, featuring significant progress on multiple fronts. Key highlights include the completion of the `git repo info` GSoC project, final refinements to the `--max-depth` diff feature, and extensive work on ODB/MIDX integration. Junio also released Git v2.51.0-rc1 and the monthly "What's cooking" report.

## Notable threads

### `git repo info` reaches completion

The GSoC project to migrate repository metadata reporting from `git rev-parse` to a new `git repo info` command has reached its final form with version 9 of the series. The implementation now includes all planned functionality: reporting ref storage format (`references.format`), repository layout (`layout.bare` and `layout.shallow`), with both key-value and nul-terminated output formats. The v9 changes are purely cosmetic, standardizing test output formatting and error message style as requested by mentors Karthik Nayak and Patrick Steinhardt. With full mentor approval and all substantive issues addressed, this represents successful completion of the project objectives.

### Diff content filtering finalized

The long-running thread about making content-ignoring diff options (`-w`, `-I`) work consistently with metadata outputs (`--name-status`, `--raw`, `--name-only`) has reached resolution with version 4 of the fix. The implementation introduces a `dry_run` optimization mode in `struct diff_options` and a `quick_consume()` callback to detect changes without full processing. The solution has evolved through multiple versions, with the current approach being simpler than earlier attempts that used `/dev/null` redirection. Jeff King has signed off on the implementation, which now awaits final merge after addressing Junio's minor code organization suggestions.

### MIDX/ODB integration advances

Patrick Steinhardt's 9-part series refactoring multi-pack index (MIDX) handling to better integrate with Git's object database (ODB) source system has progressed to version 2. The changes systematically replace direct path and repository handling with ODB source pointers throughout MIDX operations. Reviewers Taylor Blau and Karthik Nayak have provided positive feedback, with only minor questions remaining about handling of user-provided object directories. The series represents a significant step in abstracting Git's object storage layer and follows the pattern of the ongoing `the_repository` removal effort.

### `--max-depth` diff feature refined

Toon Claes has posted version 2 of the `--max-depth` diff feature, which limits recursive tree traversal depth similar to `git grep --max-depth`. The implementation adds `max_depth` and `max_depth_valid` fields to `struct diff_options` and modifies core diff infrastructure to respect depth limits. Notable v2 improvements include using `-1` as "no limit" value, making error messages translatable, and adding comprehensive unit tests for the `within_depth()` function. The series builds on Jeff King's original work and addresses all feedback from Patrick Steinhardt's review, showing thorough test coverage across various depth combinations and pathspec edge cases.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Rebase merge commit handling** -- Johannes Sixt fixes an oversight in interactive rebase that prevented dropping merge commits, with Phillip Wood confirming the solution.

**ZIP archive deflate fixes** -- Toon Claes's series addressing corruption issues in `git archive` ZIP output receives positive reviews from René Scharfe, with only minor zlib API compliance tweaks remaining.

**Commit-graph type safety** -- Patrick Steinhardt's series removing `the_repository` usage from commit-graph operations continues with refined integer type handling based on Junio's feedback.

**Diff stdin handling fix** -- A heap overflow in `git diff --no-index` when comparing stdin from subdirectories is addressed with a proper working directory context fix from Junio Hamano.

## On the radar

**Partial submodule cloning** -- Petar Vutov proposes extending partial clone filters to submodule operations via `.gitmodules` configuration, seeking feedback before implementation.

**Type safety discussion** -- The proposal for a `count_t` typedef faces pushback from Junio and Phillip Wood, who suggest alternative approaches to integer safety through better allocation macros and naming conventions.

**Stash timestamp issue** -- A report surfaces about unwanted `.pack` file timestamp changes during `git stash --include-untracked`, potentially affecting backup systems.