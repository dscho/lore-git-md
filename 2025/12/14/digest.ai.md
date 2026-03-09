# Git Mailing List Digest — 2025/12/14

**The day in brief.** A moderately active Sunday with 18 emails across 6 threads, dominated by technical refinements to macOS build system configuration and philosophical debates about `git reset --hard` behavior. The most substantive discussions centered on Homebrew integration in the build system and the legal implications of automatic Signed-off-by trailers, while an off-topic blockchain proposal accidentally landed in the list.

## Notable threads

### macOS build system refinements for Homebrew

The ongoing work to properly integrate Homebrew into Git's build system saw significant discussion today, with Torsten Bögershausen, René Scharfe, and Junio Hamano refining the implementation details. The thread focuses on introducing `HOMEBREW_*` variables to control linking against Homebrew-installed libraries, similar to existing flags for MacPorts and Fink. Key points include:

- Architecture-specific default paths (/opt/homebrew for ARM, /usr/local for Intel)
- Proposal for explicit component flags (`USE_HOMEBREW_GETTEXT`, `USE_HOMEBREW_MSGFMT`, `USE_HOMEBREW_LIBICONV`)
- Need for clear documentation distinguishing Git-specific variables from Homebrew's environment variables
- Backward compatibility considerations for the `NO_HOMEBREW` flag

The philosophical debate centers on whether to maintain the current ad-hoc detection or move to a more structured approach with explicit component selection. Junio's latest response suggests leaning toward the latter while preserving existing behavior for compatibility.

### `git reset --hard` behavior debate reaches impasse

The discussion about modifying `git reset --hard` to prevent accidental loss of staged-but-uncommitted content appears to have reached a stalemate. Junio Hamano firmly rejected even the narrowed proposal to prevent `--hard` operations when the destination tree is empty, arguing this would break legitimate cleanup scripts that rely on the current behavior. Original author Stefanos Koutsouflakis proposed alternative mitigation approaches, but Hamano's response suggests the maintainer position is settled:

"Any deviation from `reset --hard`'s documented destructive behavior would be perceived as a bug by users who rely on its current behavior for cleanup workflows."

The exchange highlights the tension between Git's design principles (predictable destructive operations) and user experience concerns (accidental data loss prevention), with the philosophical divide proving unbridgeable in this case.

### Legal concerns block automatic Signed-off-by configuration

A proposal to extend `commit.signOff` configuration beyond `git format-patch` to other commands like `commit` and `merge` ran into strong opposition based on legal concerns. Junio Hamano referenced a 2020 documentation patch explaining that automatic Signed-off-by (SOB) trailers could weaken the legal standing of the Developer Certificate of Origin by making it appear the sign-off was done by "inertia" rather than intentional certification. Collin Funk contributed a documentation refinement suggestion to spell out "Signed-off-by" instead of using the abbreviation "SoB" for consistency.

The discussion reveals Git's careful balancing act between user convenience (particularly for tools like lazygit) and preserving the legal and cultural significance of SOB trailers in open source contribution workflows.

## In brief

**Diff machinery bugfix** -- A patch addresses copy detection in `diff-files` when comparing working tree to index, fixing a regression from 2008 that broke copy detection for reverse diffs showing deletions from the index.

**Patch application troubleshooting** -- Jeff King helped diagnose why `git-am` failed to properly apply patches in a Linux kernel dynamic debug series, tracing the issue to mbox preparation methods rather than a git-am bug.

## On the radar

**Off-topic submissions** -- An unrelated proposal about blockchain and monastic principles was mistakenly sent to the list, highlighting the ongoing challenge of keeping technical discussions focused. List administrators may need to follow up with the sender.