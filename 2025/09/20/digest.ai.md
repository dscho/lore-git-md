# Git Mailing List Digest - 2025/09/20

**The day in brief.** A moderately busy Saturday with 34 emails across 11 threads, dominated by technical refinements to ongoing efforts. Key discussions include platform compatibility concerns in the Rust adoption debate, documentation clarifications for SHA-1/SHA-256 interoperability, and final administrative cleanup of the xdiff refactoring series. The most contentious thread remains the philosophical debate around semantic conflict handling in rebase operations.

## Notable threads

**Rust build system PIC/PIE handling**  
Junio Hamano refines the discussion around position-independent code handling differences between Make and Meson builds in the Rust integration effort. While Rust's default PIC behavior is understood, Junio questions whether Meson's simpler approach works portably across all platforms or is coincidental to specific test environments. The thread shifts from understanding the behavior difference to evaluating solution robustness, with implications for both build systems' future maintenance. This technical discussion remains focused on determining whether Meson approach can be safely standardized or if Make needs to maintain explicit PIE flags conditionally.

**xdiff refactoring administrative cleanup**  
Multiple emails wrap up administrative details from the now-merged xdiff refactoring series (patches 1-10). Discussions center on two minor points: an unusual "Best-viewed-with" trailer in commit messages (revealed to be an original suggestion without clear precedent) and a missed indentation issue that slipped through review. Junio plans to fix these via rebase rather than requesting a resend. The exchange provides insight into review processes and tooling limitations, particularly how the very `--color-moved` tool that helped spot the issue had previously been used by reviewers without catching it. No technical changes are needed - this represents final housekeeping for the completed preparatory work.

**SHA-1/SHA-256 interoperability documentation**  
Brian m. carlson and Junio Hamano refine documentation details in the SHA-1/SHA-256 interoperability series. Key clarifications include: confirming use of full 32-byte SHA-256 checksums in pack index v3 (rather than truncated 20-byte values), and precise terminology around "current" vs "other" algorithm in tag signature handling. The discussion reveals historical context about early design considerations while aligning documentation to current implementation reality. These exchanges demonstrate the series' progression from design discussion to concrete documentation updates, with both technical and historical context informing final decisions.

**Semantic conflict handling in rebase**  
Elijah Newren provides a substantive critique of Antonio Mennillo's `git-rebase-clean` proposal, raising fundamental concerns about its approach to semantic conflicts. While acknowledging the technical innovation in v2.0.0's content-agnostic diff/apply mechanism, Newren questions whether resolving conflicts in a squashed state then reconstructing commits can work reliably across various conflict types. The response identifies several technical edge cases and strongly disagrees with the philosophical premise that atomic commits are unachievable, advocating for teaching proper commit practices rather than accommodating fragmented history. This represents a fundamental disagreement about Git's role in enforcing versus accommodating development practices.

**Platform compatibility in Rust adoption**  
The mandatory Rust discussion expands with concrete platform compatibility concerns from PowerPC Darwin and NonStop maintainers. Florian Märkl and Randall Becker detail technical constraints where Rust toolchains are unavailable or broken, including mrustc's hidden GCC dependencies and C17 incompatibility on NonStop. Ezekiel Newren explores potential cross-compiler solutions, but responses indicate these platforms lack working C++17 compilers needed for gccrs. The thread highlights the concrete technical impasse between memory safety goals and platform support realities, with no immediately apparent resolution that satisfies both requirements.

## In brief

**gitk macOS focus behavior** -- Robert Scott reports gitk fails when `osascript` is unavailable in PATH on macOS. Eric Sunshine points to an unmerged 2018 patch that would make the call non-fatal.

**gitk window persistence** -- A two-patch series improves window geometry persistence for both main window and Tags/Heads view, fixing position restoration and switching to absolute positioning.

**strvec memory management** -- Jeff King clarifies commit message wording in the `setup_revisions_from_strvec()` wrapper API patch after Eric Sunshine's review.

**git whatchanged replacement** -- Ben Knoble suggests simpler `git log -- path` as alternative to deprecated `git whatchanged` for finding deleted files.

## On the radar

**Rust toolchain maturity** -- The mandatory Rust debate awaits gccrs development or other implementation improvements to address platform support gaps, particularly for NonStop and PowerPC Darwin.