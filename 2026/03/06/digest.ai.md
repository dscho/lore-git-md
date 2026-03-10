# Git Mailing List Digest - 2026/03/06

**The day in brief.** A busy day with 108 emails across 32 threads, featuring significant technical discussions and several patch series nearing completion. Key developments include the `git replay` revert functionality reaching consensus, configurable partial clone filters being finalized, and multiple refactoring efforts progressing. The day also saw active discussion around test infrastructure issues and several GSoC proposals.

## Notable threads

### `git replay` revert functionality matures

The long-running effort to add revert capability to `git replay` reached a significant milestone today, with the technical implementation now complete after addressing all review feedback. Siddharth Asthana's series has evolved through multiple iterations to handle complex cases like non-linear histories and selective commit reversal while maintaining clear boundaries between plumbing and porcelain behavior.

The latest patches consolidate message formatting logic as suggested by Phillip Wood, resolving the last major technical questions. While an interface design discussion (subcommands vs flags) remains open, the core functionality is now merge-ready with comprehensive test coverage. This feature, primarily for GitLab's Gitaly service, represents a substantial enhancement to bare repository operations.

### Configurable partial clone filters finalized

Alan Braithwaite's series implementing URL-based partial clone filter configuration (`clone.<url>.defaultObjectFilter`) is now complete after addressing all maintainer feedback. The implementation uses Git's urlmatch infrastructure with three specificity levels (domain/namespace/full path) and maintains safety boundaries by limiting the feature to clone operations.

While brian m. carlson raised late concerns about script reliability, Junio Hamano expressed preference for keeping this in core Git given the clear override options (`--filter` and `--no-filter`). The final version includes comprehensive tests and documentation, with only minor wording tweaks remaining before merging.

### Rebase trailer support ready for merging

Phillip Wood and Li Chen's series adding `--trailer` support to `git rebase` has completed its eighth iteration with all technical questions resolved. The implementation refactors trailer handling into reusable components and maintains Git's established trailer ordering (signoffs after other trailers). The feature works across all rebase modes while preserving trailers through conflicts and properly handling fixup/squash operations.

Junio Hamano has approved the approach, and the series now only awaits minor documentation updates. This represents a significant enhancement to rebase's capabilities, enabling in-process trailer manipulation without forking `git interpret-trailers`.

## In brief

**`the_repository` removal in wt-status.c** -- Shreyansh Paliwal's three-part series eliminating global state from `wt-status.c` is now fully merged, marking another step in Git's ongoing architectural cleanup.

**Repository statistics feature** -- Justin Tobler's series adding comprehensive object store metrics to `git repo` has been acked by Junio and is queued for merging, completing its evolution through five iterations.

**Cover letter formatting** -- Mirko Faina's configurable cover letter formatting for `git format-patch` is in final form (v7) with new `%(count)` and `%(total)` placeholders and thorough documentation.

**Pre-add hook design** -- Chandra Kethi-Reddy's `pre-add` hook series faces fundamental reconsideration despite technical completion, as Junio questions whether its limited scope provides sufficient value.

**Branch name prefixing** -- Yoann Valeri's feature for automatic branch name prefixing has been restructured based on maintainer feedback, now using a more flexible string-based approach rather than booleans.

**Fast-import re-signing** -- Justin Tobler's `re-sign-if-invalid` mode for `git fast-import` has been updated with key specification support and proper failure handling for interoperability cases.

**Test failures on /dev/shm** -- Multiple credential and HTTP tests are failing on Arch/Artix Linux due to `noexec` mounts, sparking discussion about test infrastructure requirements versus portability.

**Git-gui maintenance** -- Wolfgang Faust's series modernizes git-gui's maintenance behavior by integrating with Git's built-in auto maintenance and removing legacy GC warnings.

## On the radar

**Xdiff refactoring** -- Ezekiel Newren's xdiff refactoring series aimed at Rust interoperability may be revived after being paused due to merge conflicts, though Phillip Wood's implementation concerns remain unaddressed.

**Worktree metadata** -- The discussion about adding creation timestamps to worktrees has evolved into a broader proposal for general worktree metadata, following Junio's suggestion to model it after `.git/description`.

**Stash pathspec documentation** -- An emerging discussion suggests clarifying `git stash push`'s pathspec behavior in documentation, as it currently uses current-directory-relative paths unlike some other commands.