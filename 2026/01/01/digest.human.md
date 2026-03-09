# Git Mailing List Digest - 2026/01/01

**The day in brief.** A moderately active New Year's Day with 25 emails across 12 threads, featuring the conclusion of two long-running efforts (documentation improvements for `git reset` and `git status` divergence tracking), several test fixes, and Junio's monthly "What's cooking" report. The most notable developments are the finalization of the `git reset` documentation series and the v10 iteration of the `git status` divergence feature.

## Notable threads

### `git reset` documentation finalized

After multiple iterations spanning months, the collaborative effort to improve `git reset` documentation has reached completion. The v3 series, shepherded by D. Ben Knoble with contributions from Julia Evans and Junio Hamano, systematically reorganizes and clarifies the man page. Key improvements include reordered command forms to match common usage, simplified terminology (replacing "tree-ish" with "commit or tree"), and precise descriptions of interactive (`-p`) behavior. The final patch specifically corrects the description of `reset -p` to accurately reflect that it modifies the index contents based on the target commit (HEAD vs. others), using neutral language that matches the contextual prompts users see. With all technical concerns addressed and Junio's sign-off, this series represents a significant documentation upgrade ready for merging.

### `git status` divergence tracking reaches v10

Harald Nordgren's feature to show divergence from push/tracking branches in `git status` output has undergone a remarkable 10-iteration evolution, culminating in a design that uses Git's existing branch tracking mechanisms rather than introducing new configuration. The final version leverages `branch.<name>.pushRemote` when available, falling back to tracking branches otherwise, with robust test coverage (296 lines across 12 scenarios). Junio notes the commit organization could be cleaner but approves the technical approach, marking this long-running effort as ready for inclusion. The thread demonstrates exemplary community collaboration, with the author successfully adapting the feature to maintainer feedback while preserving its utility.

### Flaky test fixes

Two test reliability issues saw attention today. Johannes Schindelin confirmed a fix for racy difftool tests on Windows, where identical file sizes (12 bytes) caused detection failures due to NTFS quirks. The solution uses different-sized content (17 bytes) to ensure reliable change detection. Separately, a new `retry_grep()` helper was introduced to address fsmonitor test flakiness, replacing an earlier delay mechanism with a more robust 5-second timeout approach for event trace verification. Both fixes are minimal, well-justified, and maintainer-approved.

## In brief

**"What's cooking" report** -- Junio Hamano's monthly status update highlights Patrick Steinhardt's Linux fsmonitor implementation, René Scharfe's `the_repository` removal work, and various test modernizations as notable in-progress topics, while flagging some series needing rework.

**Skip-worktree test coverage** -- K Jayatheerth (GSoC) implemented TODO tests for skip-worktree behavior, verifying git-apply, git-mv, and git-checkout interactions with marked files, with two expected failures documented.

**Interactive patch UI improvement** -- A GSoC contribution enhances `git add -p` and related commands to show previous hunk decisions when navigating, though Junio notes potential test impact in t3701 that needs addressing.

**New contributor introductions** -- Two prospective GSoC 2026 participants (Samuel Abraham Adekunle and Pushkar Singh) made initial contacts, both demonstrating proper onboarding by studying documentation before engaging technically.

## On the radar

**Git Rev News #130** -- The monthly newsletter edition was published today, providing a curated overview of recent development activity for those not closely following the list.