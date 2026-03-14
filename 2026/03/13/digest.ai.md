# Git Mailing List Digest - 2026/03/13

**The day in brief.** A busy Friday with 121 emails across 40 threads saw significant progress on several fronts: Patrick Steinhardt's `git replay --revert` feature reached technical completion, the `the_repository` removal effort advanced with worktree subsystem patches, and Junio signaled readiness to merge multiple series. Notable discussions included interface design debates for experimental commands and careful attention to test portability.

## Notable threads

**`replay --revert` reaches technical completion**  
Siddharth Asthana's v4 series adding revert capability to `git replay` has addressed all substantive review feedback, with Junio Hamano approving the final technical piece - consolidation of message formatting logic into `sequencer_format_revert_message()`. The implementation now handles reverts via distinct operation mode while preserving commit-by-commit reversal with individual messages. While higher-level interface questions (subcommands vs flags) remain open, the consensus is to defer those to a follow-up series. This feature is particularly valuable for GitLab's Gitaly service operating on bare repositories.

**Histogram diff bugfix approved**  
Yee Cheng Chin's fix for redundant output in the diff algorithm's compaction phase has been approved for merging after thorough review from Phillip Wood and Junio. The patch addresses an edge case where shifted change groups could produce matching lines in both files, affecting <1% of diffs but meaningfully improving output quality when triggered. Johannes Sixt validated the platform-specific edge case handling, particularly around zero-byte write behavior.

**`pre-add` hook faces design reconsideration**  
Despite technical completion, Chandra Kethi-Reddy's `pre-add` hook series now faces fundamental questions about whether its limited scope provides sufficient value. Phillip Wood raised key concerns about dry-run behavior, interactive mode exclusions, and whether the hook should validate `git commit` staging. The thread shows how even well-implemented features may stall if reviewers question the core concept's utility, with Junio suggesting enhanced `pre-commit` hooks might be preferable.

**Cover letter formatting finalized**  
Mirko Faina's eight-iteration series adding configurable cover letter formatting has reached consensus with a DWIM (Do What I Mean) compromise on format string prefixes. While the technical "log:" prefix remains required, it can be omitted when the string contains %-placeholders, following `git log --pretty` precedent. The implementation cleanly separates shortlog generation while introducing `generate_commit_list_cover()` for alternative formats, with thorough test coverage now progressing through integration.

**`fast-import` signature handling merged**  
Justin Tobler's v6 series adding `sign-if-invalid` mode to `git fast-import` has been merged after addressing final review feedback. The changes enable re-signing commits with invalid signatures while valid ones remain unchanged, building on Christian Couder's earlier work. The implementation shares infrastructure with existing `strip-if-invalid` mode but adds signing capability through a consolidated `sign_buffer()` API, with comprehensive test coverage for OpenPGP, X.509, and SSH signatures.

**Upload-pack performance optimizations**  
Patrick Steinhardt's 10-patch series addressing write contention in `git-upload-pack` has been approved, demonstrating 40% build speed improvements through `writev()`-based buffering. The changes reduce write syscalls from ~400k to ~44k when cloning Linux by implementing batched writes across multiple subsystems. Johannes Sixt validated the cross-platform implementations (POSIX/Windows/NonStop), particularly around edge case handling, making this impactful for high-concurrency server environments.

## In brief

**Reftable compatibility with PCH** -- Phillip Wood noted the reftable backend avoids `git-compat-util.h` by design, suggesting precompiling just `compat/posix.h` might better maintain architectural separation while still providing compilation speedups.

**Subtree recursion limit fix** -- Junio called for review of Colin Stagner's patch addressing shell recursion limits in `git subtree split` on Debian/Ubuntu, noting the lack of feedback after two iterations may indicate disinterest in the subtree feature itself.

**`git apply -p` test portability** -- Jeff King identified Windows test failures due to CRLF handling in the recently merged argument parsing fix, leading to consensus on inlining test patch data rather than using external files.

**`git maintenance list` command** -- Pablo Sabater suggested refactoring to reduce code repetition in config file handling logic for Rémy Léone's new subcommand that displays registered maintenance repositories.

**GSOC microproject concluded** -- Junio acknowledged completion of Arsh Srivastava's work improving error messages for uncommitted changes during branch switches, demonstrating Git's educational onboarding approach.

**Reference-transaction hook extended** -- Eric Ju's patch adding a "preparing" phase before reference locks was approved, enabling distributed systems like Gitaly to coordinate writes earlier in the transaction flow.

## On the radar

**`git history split` interface** -- Junio checked for blockers on Patrick Steinhardt's new subcommand, which introduces commit splitting while maintaining tree consistency, with design questions about empty split handling still open.

**Partial clone edge cases** -- Paul Tarjan provided packet traces confirming recursive lazy-fetch behavior in blob:none clones, validating the `GIT_NO_LAZY_FETCH=1` solution while revealing deeper questions about server-side pack generation.

**Conditional stashing in checkout** -- Harald Nordgren's series unifying `-m` and `--autostash` behavior faces submodule handling challenges, as `git stash` lacks `--recurse-submodules` support that `checkout -m` currently provides.