# Git Mailing List Digest - 2025/06/11

**The day in brief.** A busy day with 74 emails across 25 threads, featuring significant progress on several fronts. The standout developments include the completion of the promisor-remote protocol validation series, final approval for the stash branch name fix, and ongoing architectural discussions about repository settings access patterns. The day also saw Windows-specific CI fixes, documentation improvements, and the start of a performance-oriented byte-swapping refactoring series.

## Notable threads

### Promisor-remote protocol validation finalized

Christian Couder's 5-patch series enhancing the promisor-remote protocol with configurable validation of remote attributes reached its final v4 iteration, addressing all review feedback from Patrick Steinhardt, Karthik Nayak, and Junio Hamano. The series introduces structured field handling (`struct promisor_info`) to replace the previous string vector approach, allowing servers to advertise additional fields (currently "partialCloneFilter" and "token") via `promisor.sendFields` while clients can validate these using `promisor.checkFields`. The implementation maintains backward compatibility while improving type safety and documentation clarity. With all technical concerns resolved and CI passing, this appears ready for merging.

### Stash branch name fix approved

K Jayatheerth's fix for `git stash list` branch display corruption in submodule contexts received maintainer approval from Junio Hamano. The solution uses `xstrdup()` to preserve superproject branch names through submodule operations, addressing buffer reuse issues in `refs_resolve_ref_unsafe()`. The patch includes comprehensive tests verifying the stash message correctly reflects the superproject's branch name even when stashing within submodules. Only a minor whitespace issue remains (unrelated to functionality), marking this long-standing bug as effectively resolved.

### Repository settings access patterns debated

An ongoing discussion about migrating `core.sparsecheckout` from a global to the `repo_settings` struct took an architectural turn as Junio Hamano questioned the design of getter/setter functions that internally call `prepare_repo_settings()`. The maintainer clarified two valid patterns for repository configuration access: lazy loading via getter/setters for rarely-used settings versus direct access after initialization for common ones like `core.sparsecheckout`. This guidance provides a framework for future similar migrations in the repository settings work, though it doesn't request immediate changes to the current patch.

### Submodule remote lookup improvements

Jacob Keller posted a 6-patch series refactoring Git's submodule remote lookup logic to first try matching remotes by configured URL before falling back to previous heuristics. The changes introduce URL-based remote matching via a new `repo_remote_from_url()` helper while maintaining all existing fallback paths. The series includes preparatory work like removing `the_repository` dependencies from remote.c and fixing a subtle branch state initialization bug. This addresses real-world pain points where renamed default remotes would cause submodule updates to fail unnecessarily.

### Byte-swapping performance refactoring begins

Sebastian Andrzej Siewior started a 6-patch series revisiting Git's byte-swapping infrastructure after a regression on big-endian systems. The initial patches revert a problematic change and improve platform compatibility by adding `__BYTE_ORDER__` support and fixing MSVC endian detection. Later patches in the series (not yet reviewed) aim to remove redundant x86-specific optimizations now handled well by modern compilers while carefully reintroducing built-in bswap support where beneficial. The changes focus on `compat/bswap.h` with a net reduction of 30 lines.

## In brief

**Git for Windows 2.50.0-rc2** -- Johannes Schindelin announced a platform-specific release candidate with dependency updates (MinTTY 3.7.8, OpenSSH 10.0.P1) and fixes for ReFS drive operations and long branch name handling.

**Stash export documentation polish** -- brian m. carlson addressed a minor style inconsistency in the man page for the new `--print` and `--to-ref` options, aligning them with Git's established documentation conventions.

**Merge compact summary config support** -- Junio Hamano extended the new `--compact-summary` format to work via `merge.stat=compact` configuration, maintaining backward compatibility with boolean values.

**Mailmap bugfix approved** -- A fix for incorrect mailmap application in `git cat-file` received final approval, addressing buffer handling issues when processing commits with different author/committer identities.

**CI Windows Coverity fixes** -- Johannes Schindelin resolved DLL conflicts in Windows CI by adjusting PATH ordering for Coverity tools and added build log output on failure to aid debugging.

**Release notes improvements** -- Kristoffer Haugsbakk submitted editorial changes to the 2.50.0 release notes, standardizing terminology and fixing typos in function names.

**Coding guidelines update** -- Junio Hamano documented the existing practice that global variables should not be explicitly initialized to 0/NULL, letting the BSS section handle zero initialization instead.

## On the radar

**`git repo-info` development** -- Karthik Nayak provided detailed feedback on this GSoC project to extract repository metadata functionality, requesting clearer boundaries for what belongs in this new command versus existing tools like `git refs`.

**Batch branch repointing debate** -- The discussion continues about whether batch branch operations should be implemented via porcelain (`git branch`) or plumbing (`update-ref --stdin`), with Junio Hamano maintaining his plumbing-first stance.

**Rebase trailer integration** -- Phillip Wood agreed with Junio's suggestion to refactor trailer handling into a shared helper before adding rebase integration, putting this series on hold pending his return in two weeks.