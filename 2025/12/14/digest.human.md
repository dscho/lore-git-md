# Git Mailing List Digest — 2025/12/14

## The day in brief

A moderately active Sunday with 18 emails across 6 threads, dominated by technical refinements to macOS build system configuration and philosophical debates about `git reset --hard` behavior. The most substantive discussions centered on Homebrew integration in the build system and the legal implications of automatic Signed-off-by trailers, while an off-topic blockchain proposal accidentally landed in the list.

## Notable threads

### macOS build system refinements for Homebrew

The ongoing effort to properly integrate Homebrew into Git's build system saw significant refinement today, with Junio Hamano and René Scharfe working through the details of how to handle Homebrew-specific variables. The discussion moved from initial questions about variable purpose (HOMEBREW_PREFIX, HOMEBREW_GETTEXT_PREFIX) to a proposal for structured component flags (USE_HOMEBREW_GETTEXT, USE_HOMEBREW_MSGFMT). This builds on earlier work to standardize macOS package manager support while maintaining compatibility across Intel and Apple Silicon architectures (/usr/local vs /opt/homebrew paths). The thread shows the project carefully balancing build system flexibility with maintainability, particularly for users with non-standard Homebrew installations.

### The `reset --hard` debate reaches impasse

Junio Hamano firmly rejected proposals to modify `git reset --hard`'s behavior to protect staged-but-uncommitted content, even in the narrowed case of empty-tree scenarios. Original author Stefanos Koutsouflakis had argued that scripts relying on this behavior would likely be buggy, but Hamano countered that established workflows (like `reset --hard && clean -f -x`) must remain reliable. The exchange highlights Git's philosophical commitment to predictable destructive operations over safety mechanisms that could introduce inconsistency. With the maintainer's position clearly stated, the discussion appears to have reached its conclusion unless new technical arguments emerge.

### Legal concerns block automatic Signed-off-by configuration

A proposal to extend `commit.signOff` configuration beyond `format-patch` to commands like `commit` and `merge` ran into Git's longstanding position on Signed-off-by trailers. Junio Hamano resurrected a 2020 documentation patch explaining that automatic addition could weaken the legal standing of the Developer Certificate of Origin by enabling "inertia" sign-offs. Collin Funk contributed a documentation refinement (spelling out "Signed-off-by" instead of "SoB" for clarity), but the core position remains unchanged. This leaves tools like lazygit needing to implement their own signoff handling rather than relying on Git configuration.

## In brief

**Diff copy detection fix** — A bugfix addresses a 2008 regression in `diff-files` copy detection when comparing working tree to index, particularly with `--find-copies-harder` and index preloading. The patch properly handles up-to-date and skip-worktree entries in this scenario.

**Patch application troubleshooting** — Jeff King helped diagnose why `git am` failed to apply patches in the correct order for a Linux kernel dynamic debug fix series, suggesting the issue may lie in mbox preparation rather than Git itself.

## On the radar

**MacOS iconv compatibility** — The Homebrew build system changes are part of a larger effort to properly handle character encoding conversion on macOS across different package managers, with more refinements likely coming.