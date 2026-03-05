# Git Mailing List Digest — 2025/09/14

**The day in brief.** A moderately active Sunday with 27 emails across 8 threads, dominated by final polish on the command deprecation framework series (v5) and continued discussions about `user.email` validation. The deprecation series appears ready for integration after addressing last-minute feedback, while the mergetool variables thread reached consensus on intended behavior.

## Notable threads

### Command deprecation framework reaches v5

Kristoffer Haugsbakk posted the fifth iteration of the command deprecation series, now addressing all outstanding feedback from Jeff King and others. The 8-patch set introduces:

1. Infrastructure for flagging deprecated commands (like `whatchanged` and `pack-redundant`)
2. Alias shadowing support to ease transitions
3. Improved error messages suggesting concrete alternatives
4. Comprehensive test coverage for edge cases

The series has maintainer approval for its technical approach and now focuses on final user experience polish. Notable improvements in v5 include fixing a memory leak, preventing infinite alias loops with deprecated commands, and restructuring advice messages to better guide users toward modern equivalents. The implementation allows transitional aliases while maintaining Git's general prohibition against shadowing builtins.

### Mergetool variable behavior clarified

After several days of discussion about why BASE/LOCAL/REMOTE variables weren't reaching custom merge tools, Junio Hamano confirmed the current behavior is intentional. The variables are shell variables for tools integrated via Git's `mergetools/` directory, not environment variables for standalone scripts. Phillip Wood provided the canonical solution: using `mergetool.<tool>.cmd` configuration to properly expand variables without modifying Git's installation.

The thread revealed a documentation gap that will need addressing, but no code changes appear necessary. The design favors tools that integrate properly with Git's infrastructure over ad-hoc scripts, maintaining consistency with other Git subcommands.

### Worktree safety checks polish

Gabriel Scherer refined the advice messages in his worktree safety series based on Kristoffer Haugsbakk's feedback. The new version better separates the concepts of bypassing checks (`--ignore-other-worktrees`) from detaching HEAD (`--detach`). A separate discussion thread confirmed that testing advice messages requires using `bin-wrappers/git` rather than system aliases, a subtle but important testing methodology point.

## In brief

**`user.email` validation debate continues** -- Randall Becker defends GitHub's use of the field for SSO tokens against Kristoffer Haugsbakk's "unreasonable" characterization, providing concrete format details (alphanumeric strings without `@` or `.`).

**`gitk` remote branch fix approved** -- Johannes Sixt queued Michael Rappazzo's fix for a gitk crash when viewing tags/heads after remote branch deletions.

**`safe.directory` server-side resolution** -- Marc-Jano Knopp confirmed the "dubious ownership" errors occur server-side, requiring configuration there rather than on clients.

**Alias execution refactor** -- An internal cleanup changes how Git runs alias commands to better follow `run-command.h` API guidelines, using `git_cmd` flag instead of explicit "git" push.

## On the radar

**`whatchanged` deprecation transition** -- Kristoffer Haugsbakk clarified that while exact-name aliases aren't yet possible for deprecated commands, that restriction is expected to lift soon, completing the migration path for users attached to the shorter command name.