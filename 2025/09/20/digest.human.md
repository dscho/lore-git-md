# Git Mailing List Digest - 2025/09/20

**The day in brief.** A moderately busy Saturday with 34 emails across 11 threads, dominated by technical refinements to ongoing efforts (xdiff refactoring, SHA-1/SHA-256 interoperability) and continued debate about Rust adoption's platform compatibility challenges. Notable discussions include Elijah Newren's fundamental critique of semantic conflict handling in rebase tools and concrete platform constraints from NonStop and PowerPC maintainers.

## Notable threads

**Rust build system PIC/PIE handling**  
Junio Hamano refines the discussion about position-independent code handling differences between Make and Meson builds for Rust integration. While Rust's default PIC behavior is understood, Junio questions whether Meson's simpler approach works portably or just coincidentally in test environments. The thread shifts from explaining the behavior to evaluating solution robustness, with implications for both build systems' future maintenance. This technical clarification moves the Rust integration effort forward by identifying the next key question to resolve before standardizing the approach.

**xdiff refactoring cleanups**  
A series of administrative follow-ups document minor issues in the now-merged xdiff preparatory refactoring (patches 1-10). The discussion covers a formatting oversight in brace indentation and an unusual "Best-viewed-with" trailer that sparked meta-discussion about commit message conventions. While technically complete, these exchanges reveal interesting insights about review processes - how both reviewers missed an indentation issue despite using `--color-moved`, and how minor presentation choices can generate discussion even after substantive work is merged. The thread shows the project's attention to detail in both code and documentation standards.

**Semantic conflict handling in rebase tools**  
Elijah Newren delivers a fundamental critique of Antonio Mennillo's `git-rebase-clean` proposal, questioning whether resolving conflicts in a squashed state then reconstructing commits can work across various conflict types. Newren identifies technical edge cases (overlapping line conflicts, non-content conflicts, intermediate-state conflicts) and philosophically disagrees with accommodating fragmented history, advocating instead for teaching proper commit practices. The response frames this as a core disagreement about Git's role in enforcing versus accommodating development workflows, with implications for how tools approach semantic conflict resolution.

**SHA-1/SHA-256 interoperability documentation**  
Brian m. carlson and Junio Hamano refine documentation details in the interoperability series, resolving questions about pack index checksum length (confirming 32-byte SHA-256 checksums) and clarifying terminology around tag signature fields. The exchanges show the series progressing from design discussion to precise documentation, with historical context informing final decisions. Particularly notable is the resolution to use full-length hashes throughout, matching implementation reality rather than early design considerations of truncated checksums.

**Platform constraints in Rust adoption debate**  
Multiple contributors provide concrete technical constraints about making Rust mandatory, with new details about PowerPC Darwin and OpenBSD/macppc limitations from Florian Märkl, and deeper analysis of NonStop's toolchain issues from Randall Becker. The thread reveals fundamental incompatibilities - mrustc's hidden GCC dependencies, C17 requirements, and loader complexities that currently prevent Rust solution on NonStop. Ezekiel Newren's attempts to find cross-compiler paths highlight the technical impasse between memory safety goals and real-world platform support gaps.

## In brief

**gitk window focusing on macOS** -- Robert Scott reports gitk fails when `osascript` is unavailable in PATH, with Eric Sunshine pointing to an unmerged 2018 patch that would make the call non-fatal.

**gitk window persistence** -- A two-patch series improves geometry persistence for both main and "Tags and Heads" windows, fixing position restoration and switching to absolute positioning for reliable session-to-session behavior.

**strvec memory management** -- Jeff King clarifies documentation for the new `setup_revisions_from_strvec()` API in response to Eric Sunshine's review, maintaining the technical accuracy while improving sentence structure.

**git whatchanged replacement** -- Ben Knoble suggests a simpler `git log -- path` alternative to deprecated `git whatchanged` for finding deleted files, continuing the thread's pattern of documenting migration paths.

## On the radar

**Rust build system decisions** -- The PIC/PIE discussion now moves to evaluating Meson's approach for portable adoption, a key consideration before standardizing Rust integration's build requirements.

**NonStop platform constraints** -- The technical impasse around Rust toolchain availability on NonStop systems remains unresolved, with no clear path yet that satisfies both memory safety goals and platform compatibility requirements.