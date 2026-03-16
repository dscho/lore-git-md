Here's the daily digest for March 15, 2026:

## The day in brief

March 15 saw moderate traffic with 51 emails across 21 threads, featuring several notable developments. The highlight was the merge of Alan Braithwaite's configurable partial clone filters via URL patterns, while multiple GSoC proposals continued to evolve with mentor feedback. Junio Hamano remained active in several technical discussions, particularly around the parallel hooks series and worktree refactoring.

## Notable threads

**Partial clone filters by URL now merged** -- Alan Braithwaite's feature allowing `clone.<url>.defaultObjectFilter` configuration was merged after addressing all review feedback. The implementation uses Git's urlmatch infrastructure to apply partial clone filters based on repository URLs, with command-line `--filter` and `--no-filter` options taking precedence. Junio noted the patch addresses all technical concerns, including Patrick Steinhardt's catch of a potential NULL pointer issue and test cleanup improvements.

**Parallel hooks progress** -- Junio provided detailed feedback on Adrian Ratiu's series to enhance `git checkout -m` with autostash behavior. The maintainer questioned documentation accuracy, suggested performance optimizations for dry-run checks, and proposed reorganizing the changes into clearer logical steps. Harald Nordgren then submitted v5 of the series, splitting the implementation into 4 patches and adding new stash labeling capabilities while preserving backward compatibility.

**Worktree refactoring approved** -- Phillip Wood's 3-patch series to eliminate `the_repository` usage from worktree functions received maintainer approval. The changes remove implicit repository dependencies in `is_current_worktree()` and harden `get_worktree_git_dir()` by making NULL worktree handling a BUG() condition. Junio confirmed he will merge the series after Phillip addressed documentation discrepancies from v1.

**AI-assisted l10n workflow discussion** -- Jiang Xin's RFC on AI-assisted workflows for Git's localization process saw follow-up about PO file filter standardization. Johannes Sixt sought clarification on which of two filter types should be considered standard for Gitk alignment, prompting Junio to question the language-specific assignment approach as potentially reflecting translator preferences rather than technical requirements.

## In brief

**`rev-parse --parseopt` help exit codes** -- brian m. carlson and Jeff King agreed help requests should exit successfully (status 0) rather than with error code 129, proposing modifications to both `parse-options.c` and `builtin/rev-parse.c`.

**Coccinelle strbuf by-value detection** -- Deveshi Dwivedi submitted a semantic patch to detect `struct strbuf` passed by value, fixing the last remaining case in `builtin/stash.c` after Jeff King's review.

**Test cleanup in t0008** -- Mirko Faina and Junio discussed improving test isolation in `t0008-ignores.sh`, with the maintainer emphasizing broader test hygiene principles beyond the immediate symlink cleanup fix.

**GSoC proposal refinements** -- Multiple students refined proposals including remote object info completion (Pablo Sabater), advice message updates (Arsh Srivastava), and disk space recovery for partial clones (Amisha Chhajed).

**EditorConfig fix** -- Taylor Blau corrected an unintended behavior change in Git's `.editorconfig` that affected C file formatting rules after a path-specific pattern was added.

## On the radar

**Submodule handling in checkout -m** -- The parallel hooks series has deferred discussion about submodule stashing, where Junio opposes expanding `git stash` scope while Phillip Wood has raised concerns about three-way merge behavior.