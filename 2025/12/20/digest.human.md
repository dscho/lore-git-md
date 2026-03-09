# Git Mailing List Digest — 2025/12/20

**The day in brief.** A moderately busy Saturday with 32 emails across 9 threads, dominated by the v7 submission of the submodule gitdir path encoding series. Other notable activity includes ongoing discussions about the `git history` command's branch selection strategy and documentation refinements for worktree commands. The submodule series has reached architectural consensus and appears ready for merging after final polish.

## Notable threads

### Submodule gitdir path encoding reaches v7

Josh Steadmon submitted the seventh iteration of his series implementing `extensions.submodulePathConfig`, a feature that provides unified gitdir path resolution for submodules. The 11-patch series addresses filesystem conflicts (particularly on case-insensitive systems) and enables custom gitdir layouts through runtime configuration. 

Key improvements in v7 include simplified path resolution code, stricter validation for existing submodules with encoded names, expanded test coverage, and atomic migration of existing configurations. The series now has comprehensive case-folding protection and a four-tier fallback system (plain -> encoded -> numbered -> hashed) for path conflicts. All major design questions appear resolved, with this version addressing the last technical issues reported during review.

### Branch selection debate in git history command

Elijah Newren continued his critique of Patrick Steinhardt's HEAD-to-commit walk approach in the proposed `git history` command, advocating instead for comprehensive walking of all local branches (excluding remotes and tags). Newren argues this would provide more consistent history states and better align with user expectations, citing Jujutsu's implementation as precedent. 

The discussion highlights a design tension between performance (Steinhardt's scaling concerns) and UX (Newren's principle of least astonishment). Newren questions whether any potential performance impact would outweigh the benefits of predictable behavior, connecting this to future plans for immutable branches. The thread remains unresolved but has clarified implementation boundaries (local branches only).

### Worktree documentation refinements

Junio Hamano and Eric Sunshine collaborated on precise wording for `git worktree prune` documentation, building on Sam Bostock's terminology standardization efforts. The discussion has evolved to clearly distinguish three scenarios: proper removal with `remove`, cleanup of lost worktrees with `prune`, and reconnection of moved worktrees with `repair`. 

Junio initially proposed text with redundant "without touching working tree" phrasing, which Eric correctly noted was misleading since the working tree is already missing in prune scenarios. The maintainers appear to be converging on final wording that explicitly ties `prune` to missing working trees while referencing the alternatives.

## In brief

**Hook subsystem refactoring complete** -- Adrian Ratiu addressed final stylistic nits from Patrick Steinhardt on the receive hook conversion, marking the completion point for the multi-year hook API standardization effort.

**Documentation post-mortem** -- Junio provided retrospective feedback on commit message clarity in the recently merged `git replay` documentation improvements, emphasizing the importance of self-contained explanations in commit messages.

**Curl test compatibility resolved** -- Junio confirmed dropping the patch for curl 8.18.0 test compatibility as the maintainers implemented proper fixes at the source, with Jeff King verifying the changes work against both specific curl commits and current master.

**Diff indicator debate continues** -- Harald Nordgren noted GNU diff supports similar functionality to the proposed `--no-indicators` through `--line-format`, while Johannes Sixt suggested `git difftool` as an alternative solution for copying code blocks from diffs.

**Documentation standardization** -- Jean-Noël Avila submitted a 5-patch series converting several manual pages (`git-status`, `git-stage`, `git-remote`) to use consistent synopsis formatting and AsciiDoc tables, improving maintainability and translatability.

## On the radar

**git history command** -- Patrick Steinhardt plans to address functionality for rewriting dependent branches in January, following the current discussion about branch selection strategy. The series includes significant refactoring of the add-patch subsystem.