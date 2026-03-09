# Git Mailing List Digest - 2025/12/20

**The day in brief.** A moderately busy Friday with 32 emails across 9 threads, dominated by the v7 submission of the submodule gitdir path encoding series and ongoing discussions about worktree documentation. Key developments include the near-final submodule path encoding feature reaching architectural consensus and productive collaboration on clarifying worktree command behavior.

## Notable threads

**Submodule gitdir path encoding reaches v7**  
Josh Steadmon submits the seventh iteration of his series implementing `extensions.submodulePathConfig`, a feature that provides unified gitdir path resolution for submodules. The 11-patch series addresses filesystem conflicts (particularly case-insensitive systems) and enables custom gitdir layouts through runtime configuration. V7 incorporates feedback from Patrick Steinhardt, Junio Hamano, and others, with key improvements including simplified code structure, restricted extension enablement to new repositories, enhanced validation for existing submodules, and expanded test coverage. The series now has comprehensive case-folding protection, a complete fallback strategy (plain -> encoded -> numbered -> hashed paths), and atomic migration support. With all major design questions resolved and only final polish remaining, this long-running effort appears ready for merging.

**Worktree documentation refinements**  
An active discussion continues around Sam Bostock's documentation improvements for `git worktree` commands. Junio Hamano and Eric Sunshine collaborate on precise wording to distinguish between three scenarios: `remove` (intentional deletion), `prune` (missing worktrees), and `repair` (moved worktrees). The conversation has evolved from terminology standardization ("missing" vs "unused") to crafting documentation that clearly explains when each command should be used. Junio initially proposed text with redundant "without touching working tree" phrasing, which Eric correctly noted was misleading since the working tree is already missing in prune scenarios. The maintainers are now converging on final wording that explicitly ties `prune` to missing working trees while mentioning `remove` and `repair` alternatives.

**git-history command design debate continues**  
Elijah Newren continues his critique of Patrick Steinhardt's HEAD-to-commit walk approach in the proposed `git-history` command, advocating instead for comprehensive branch walking that includes all local branches (refs/heads/*) but excludes remotes and tags. Newren argues this would provide more consistent history states and simpler implementation, citing Jujutsu's successful use of this approach. The discussion highlights a fundamental tension between performance concerns favoring the current implementation and UX considerations favoring comprehensive branch walking. This design debate remains unresolved but has clarified implementation boundaries, with Newren specifying that only local branches should be considered for rewriting operations.

## In brief

**Hook subsystem refactoring completes** -- Adrian Ratiu addresses final stylistic nits from Patrick Steinhardt on the receive hook conversion, marking the completion point for the multi-year hook API standardization effort.

**Documentation improvements for git replay** -- Junio Hamano provides retrospective feedback on commit message clarity in the recently merged documentation improvements for `git replay`, emphasizing the importance of self-contained explanations in commit messages.

**curl 8.18.0 test compatibility** -- Junio confirms dropping the third patch from the series as curl maintainers have implemented proper fixes, concluding cross-project coordination on HTTP header handling.

**--no-indicators diff flag discussion** -- The thread sees new alternatives proposed (including `git difftool` from Johannes Sixt) while maintainers continue questioning the feature's necessity compared to existing solutions.

**Documentation standardization series** -- Jean-Noël Avila submits a 5-patch series converting several manual pages to synopsis style format, including an RFC proposal for converting `git-status` ASCII-art tables to AsciiDoc format tables.

## On the radar

**git-history command development** -- Patrick Steinhardt notes in his status update that while technical consensus was reached on the core functionality, the series still requires implementation of dependent branch rewriting, planned for January.

**Object database layer fixes** -- Patrick also flags `ps/odb-misc-fixes` and `ps/clar-integers` as ready pending final review, indicating ongoing maintenance work in Git's storage layer.