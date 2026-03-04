# Git Mailing List Digest - 2025/07/16

**The day in brief.** A moderately active day with 75 emails across 19 threads, featuring significant progress on several fronts. Key developments include the finalization of Git's pseudonym policy documentation, performance optimizations for commit traversal, and the near-completion of the `git last-modified` plumbing command. Routine maintenance patches and style cleanups rounded out the day's traffic.

## Notable threads

### `git last-modified` plumbing command reaches v5

The new `git last-modified` command, which tracks tree-level modifications (as opposed to line-level blame), has progressed to its fifth iteration with all major technical decisions settled. The series now includes Bloom filter support (providing 50-55% speedups for certain operations) and an `--extended` output option designed for forge integration. Junio raised important design questions about the extended format's message termination handling, suggesting the output specification may need refinement before final merge. The command is positioned as plumbing for tools rather than end users, with all core functionality reviewed and approved.

### Pseudonym policy finalized after extensive review

After six review cycles spanning legal and technical considerations, the project has finalized its policy on pseudonyms in Signed-off-by trailers. The v2 documentation patch explicitly allows distinctive, identifying pseudonyms while prohibiting anonymous contributions, aligning with Linux kernel precedent. The policy balances legal requirements (DCO compliance, OFAC sanctions) with contributor privacy needs, citing valid use cases like gender identity concerns. With approvals from Jeff King, Patrick Steinhardt, and Junio Hamano, this concludes a lengthy discussion about identity verification in Git project contributions.

### Priority queue optimization for commit traversal

René Scharfe's series converting commit traversal from linked lists to priority queues received thorough technical validation, confirming it improves worst-case performance from O(n²) to O(log n) while maintaining stable ordering for commits with identical timestamps. The optimization particularly benefits operations like `:/` rev-parse and fetch's mark_recent_complete_commits() with many refs. Jeff King analyzed the tie-breaking behavior, verifying the new implementation matches the original's FIFO ordering for same-date commits. The series appears ready for merge pending potential test additions to codify the stability guarantees.

### Comment character handling fixes approved

Ayush Chandekar's bugfix series addressing `core.commentChar="auto"` issues during conflict resolution received final sign-off from Phillip Wood, whose architectural solution was incorporated. The patches prevent conflict markers from being incorrectly considered as comment characters and standardize behavior by resetting to '#' as the baseline in auto mode. This serves as an interim solution until the feature's planned removal in Git 3.0. The thread concluded with agreement that Wood will rebase his deprecation work on top of these fixes.

### `git repo info` output format discussions

The ongoing `git repo info` implementation saw continued debate about output format design, with Patrick Steinhardt proposing `--format=key-value` and `--format=nul` options instead of Justin Tobler's suggested `-z` flag for NUL-termination. The discussion revealed differing perspectives on future extensibility versus command-line convention adherence. Junio later contributed style cleanups to the implementation, standardizing function formatting and comment style in `builtin/repo.c` as the series approaches finalization.

## In brief

**CI test failure reporting fix** -- Junio Hamano corrected a shell syntax error in GitHub Actions artifact collection that caused "Bad substitution" errors during test runs.

**SHA-256 support for git-gui** -- Takashi Iwai's v3 series adding SHA-256 repository support to the graphical interface was queued by maintainer Johannes Sixt after minor typo fixes.

**C99 bool standardization** -- Phillip Wood's series formalizing `bool` usage for predicate functions concluded with final style nits addressed, marking the successful end of an 18-month experimental period.

**Config validation proposals** -- Lidong Yan suggested a registration-based system for config key validation, while Johannes Sixt advocated simpler documentation fixes for specific issues like the undocumented `pull.autostash` key.

**Meson build fixes** -- Carlo Arenas and Eli Schwartz refined PCRE2 dependency handling for macOS, adding header verification to detect broken system installations.

**On the radar**

**Compound literals policy** -- Junio and Phillip Wood continued discussing documentation for the limited C99 compound literals usage in reftable, with resource cleanup patterns being a key consideration.

**Config parsing refactoring** -- Jeff King approved Phillip Wood's removal of redundant `path` field from config parsing structures while fixing a use-after-free issue.