Here's the daily digest for May 5, 2025:

## The day in brief
May 5 saw steady activity across the Git project with 87 emails spanning 20 threads. The day was dominated by ongoing technical discussions around maintenance task decomposition, platform compatibility fixes, and documentation improvements. Key developments included the near-completion of Patrick Steinhardt's series to fully replace `git gc` with granular maintenance tasks and a new feature series adding interactive diff context controls.

## Notable threads

### Finalizing maintenance task decomposition
Patrick Steinhardt's v4 series to complete the `git gc` decomposition effort saw significant discussion today. The series implements the final two maintenance tasks (worktree pruning and rerere garbage collection) as standalone operations with configurable auto-execution thresholds. Key changes from v3 include simplifying the rerere GC auto-check to a simple existence test rather than entry counting, addressing Junio Hamano's concerns. Eric Sunshine raised API design questions about the `get_worktree_names()` helper that remain open, but the series otherwise has approval from Derrick Stolee and appears ready for merging.

### Platform compatibility and performance
The thread on BSD errno handling in packed-refs operations expanded to include performance considerations after Jeff King and Patrick Steinhardt discussed using mmap() instead of strbuf for large packed-refs files. Patrick provided GitLab.com metrics showing 99% of packed-refs files are under 50MB, but outliers like gitlab-org/gitlab's 2GB file exist. Shejialuo confirmed plans to submit a mmap conversion patch, which would benefit these large-repository cases while maintaining the BSD compatibility fixes that started the thread.

### Interactive diff context controls
Leon Michalak introduced a well-received series adding configurable diff context to interactive commands like `add -p` and `commit -p`. The patches:
1. Make these commands respect `diff.context` and `diff.interHunkContext` configs
2. Add command-line overrides via `-U`/`--unified` and `--inter-hunk-context`
3. Introduce an interactive "context" subcommand to adjust settings mid-session

Kristoffer Haugsbakk provided documentation polish suggestions, and Eric Sunshine noted test cleanup improvements, but the technical approach appears sound. This addresses a long-standing inconsistency where interactive patch commands ignored standard diff configuration.

## In brief

**Windows CI build fixes** -- Johannes Schindelin and Patrick Steinhardt finalized a change to use Release mode in Meson builds for Windows CI, resolving test hangs while acknowledging longer-term goals around assert handling.

**Scalar maintenance control** -- Derrick Stolee's v2 series added `--no-maintenance` to `scalar reconfigure`, completing the feature across Scalar commands. Junio Hamano later questioned whether `reconfigure` should support explicit disable (not just skip) of maintenance.

**`git whatchanged` deprecation** -- Patrick Steinhardt noted missing Meson build system updates for the deprecation, providing a patch to mirror the Makefile handling of breaking changes.

**`git send-email` improvements** -- A v3 series added RFC1035-compliant domain validation and expanded OAuth documentation. Junio provided detailed feedback on commit message structure and workflow examples.

**Build system cleanup** -- Johannes Schindelin's 3-patch series removing obsolete CI prove caching and Visual Studio project generation was queued after consensus that VSCode config files serve a different purpose and should remain.

**Test output directory fix** -- A one-line change ensured Meson builds write test output to the expected location for GitHub Actions integration, verified by multiple contributors.

## On the radar

**Legacy code management** -- The discussion about `git svn` Windows issues expanded into a broader conversation about handling unmaintained code in both `contrib/` and core commands, with Junio Hamano suggesting a framework" of promotion, spin-off, or removal.

**Submodule edge case** -- A bug report demonstrated `.gitmodules` data loss when adding a new submodule with the same name as a moved one, highlighting name-vs-path identification issues.