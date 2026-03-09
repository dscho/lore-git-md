# Git Mailing List Digest - 2026/01/01

**The day in brief.** A busy New Year's Day in Git development sees the conclusion of two major documentation efforts, a flurry of test improvements, and Junio Hamano's monthly "What's cooking" report. The standout threads include the finalized `git reset` documentation rewrite and the 10th iteration of a `git status` divergence tracking feature that has now found its final form.

## Notable threads

### `git reset` documentation reaches final form

D. Ben Knoble shepherds the completion of a comprehensive rewrite of the `git reset` man page, incorporating Julia Evans' pedagogical approach and Junio Hamano's technical refinements. The four-patch series (now at v3) systematically reorganizes the documentation, with the final patch clarifying pathspec behavior - particularly the interactive (`-p`) mode's index modification behavior. The collaborative effort resolves all outstanding questions, including Junio's edge case feedback about `reset -p` directionality. With all technical concerns addressed and sign-offs from both the original contributor and maintainer, this documentation modernization is now merge-ready.

### `git status` divergence tracking concludes after 10 iterations

Harald Nordgren's feature to show divergence from push/tracking branches in `git status` output reaches its final form after significant design evolution. The v10 series abandons the original `status.goalBranch` configuration approach (rejected by Junio) in favor of using Git's existing branch tracking mechanisms (`branch.<name>.pushRemote` and tracking branches) as suggested by Ben Knoble. While Junio notes the commit organization could be cleaner, he approves the technical direction. The implementation now shows divergence from `upstream/main`-style output when the push target differs from the tracking branch, with robust test coverage (296 lines across 12 scenarios). The thread demonstrates effective community collaboration, with the author successfully adapting the feature while preserving its utility through significant design changes.

### Test reliability improvements land

Multiple test fixes address platform-specific flakiness. Johannes Schindelin reviews and approves a difftool test fix that modifies content sizes to avoid Windows filesystem timestamp quirks. A separate fsmonitor test series introduces a `retry_grep()` helper to handle event detection races, replacing an earlier `test-tool` delay approach. Both changes are minimal, well-documented, and maintain existing coverage while improving reliability. Junio has already accepted the difftool fix while musing about broader solutions for racily clean index entries.

## In brief

**GSoC contributor introductions** -- Two prospective Google Summer of Code participants (Samuel Abraham Adekunle and Pushkar Singh) introduce themselves, having studied Git's contribution guidelines and begun exploring the codebase. Both follow the community's recommended onboarding path of starting with documentation and test improvements.

**Skip-worktree test coverage** -- K Jayatheerth converts TODO placeholders into actual tests for skip-worktree behavior in t7012, covering git-apply, git-mv, and git-checkout operations. The tests confirm three working cases and two expected failures around skip-worktree handling.

**Interactive patch UI improvement** -- A GSoC contribution modifies `git add -p` and related commands to show previous hunk decisions when navigating between hunks. The v2 version extends support across all interactive patch modes but may need test adjustments as Junio flags potential t3701 breakage.

**Git Rev News #130 published** -- Christian Couder announces the latest edition of the monthly Git development newsletter, prepared with help from Lucas Seiki Oshiro and David Aguilar. The next edition (#131) is already being tracked in GitHub issue #817.

## On the radar

**"What's cooking" report highlights** -- Junio's monthly status update shows Patrick Steinhardt's Linux fsmonitor implementation and René Scharfe's `the_repository` removal work progressing well, while Johannes Schindelin's Windows symlink support and Taylor Blau's MIDX repacking need more review attention. The report provides a comprehensive snapshot of Git's development pipeline.