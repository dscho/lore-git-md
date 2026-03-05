# Git Mailing List Digest — 2025/09/14

## The day in brief

A moderately active Sunday with 27 emails across 8 threads, dominated by final polish on the command deprecation framework series (v5 now ready for integration) and continued discussions about `user.email` validation philosophy. The deprecation series stands out as today's most significant development, having addressed all major technical concerns and now focusing on final user experience refinements.

## Notable threads

### Command deprecation framework reaches v5

Kristoffer Haugsbakk's long-running series to improve Git's command deprecation system has reached its fifth iteration, now addressing all major feedback and ready for integration. The 8-patch series introduces alias shadowing support for deprecated commands like `whatchanged` and `pack-redundant`, allowing users to create transitional aliases while maintaining the existing recursion detection infrastructure. 

Key improvements in v5 include:
- Fixed memory leak identified by Jeff King
- Restored BUG check for mutually exclusive list command options
- Enhanced error messages with concrete alternatives (e.g., suggesting `git log --raw --no-merges` for `whatchanged`)
- Comprehensive test coverage for alias edge cases

The series has maintainer approval for its core technical approach and now focuses on final documentation polish. Notably, it preserves Jeff King's original architectural vision while adding the user-facing improvements that make deprecation warnings more actionable.

### `user.email` validation debate continues

The philosophical discussion about Git's permissive handling of `user.email` values saw continued debate today. Kristoffer Haugsbakk characterized GitHub's use of the field for SSO tokens as "unreasonable," prompting Randall Becker to defend the practice with concrete technical details about the token format (alphanumeric strings without `@` or `.` characters). 

Junio Hamano had previously endorsed the status quo based on such real-world use cases, and today's exchange appears to be winding down with the permissive approach remaining unchallenged at the implementation level. The thread highlights the tension between formal email standards and pragmatic system integration needs that has shaped this longstanding Git behavior.

### Mergetool variable passing clarified

A thread about mergetool environment variables saw important clarification from Junio Hamano, who explained that BASE/LOCAL/REMOTE/MERGED variables are designed as shell variables for tools integrated into Git's mergetool infrastructure (via the `mergetools/` directory), not as environment variables for standalone custom tools. Phillip Wood later noted that custom tools can still access these variables through the `mergetool.<tool>.cmd` configuration mechanism.

The discussion is converging on better documentation as the solution rather than implementation changes, with Junio's explanation providing the architectural context needed to properly frame the documentation updates.

## In brief

**gitk remote branch fix** — Michael Rappazzo's fix for gitk errors after remote branch deletions has been queued by Johannes Sixt, resolving an inconsistency between the `headids` and `upstreamofref` data structures.

**Worktree checkout advice refined** — Gabriel Scherer revised the advice messaging in his worktree safety checks series to better distinguish between `--ignore-other-worktrees` and `--detach` options, addressing Kristoffer Haugsbakk's feedback about clarity.

**Safe directory server-side confirmation** — Marc-Jano Knopp confirmed that the `safe.directory` configuration needs to be set on the server where Git performs ownership checks, not the client machine, closing a thread about remote repository access issues.

**Alias execution refactor** — An unnamed contributor submitted a small refactoring that changes how Git runs commands invoked via aliases, replacing an explicit `strvec_push()` with the standard `git_cmd` flag for subprocess handling.

## On the radar

**`whatchanged` deprecation transition** — The discussion about migrating from `git whatchanged` to `git log --raw --no-merges` has identified that alias shadowing for deprecated commands (enabling `git whatchanged` aliases) will be possible in a future Git version, providing a complete transition path for users valuing the command's brevity.