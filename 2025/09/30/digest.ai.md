# Git Mailing List Digest - 2025/09/30

## The day in brief

A moderately busy day with 47 emails across 17 threads, featuring significant documentation refinements, ongoing infrastructure discussions, and a major decision regarding Rust integration. Key highlights include the completion of Julia Evans' push.default=simple documentation series, a surprise reversal on Rust integration work, and opening discussions about Git 3.0 release timing.

## Notable threads

### Documentation refinements for push.default=simple behavior

Julia Evans' v4 documentation series (5 parts) clarifying `git-push` behavior and upstream branch relationships was approved and merged after incorporating Junio Hamano's feedback. The changes:

- Add a new "UPSTREAM BRANCHES" section consolidating scattered information
- Simplify explanations of where/what to push using clearer precedence lists
- Remove inaccurate claims about `push.default=simple` requiring upstream config
- Add basic usage examples and standardize section linking
- Improve introductory language to be more accessible

The series represents a significant improvement in documenting Git's complex push behavior, particularly around the nuances of `push.default=simple` which has confused users since its 2013 introduction. Junio approved the changes after multiple rounds of review focused on both technical accuracy and pedagogical clarity.

### Rust integration work abruptly dropped

In a surprising reversal, Ezekiel Newren announced he has dropped his 18-commit Rust integration work, including:

- The misc::varint Rust implementation
- Build-helper crate and Cargo.lock integration
- Policy documentation about Rust becoming mandatory
- Xdiff/reftable library merging into libgit.a

This comes just after brian m. carlson had confirmed the series was technically sound and working with both Makefile and meson builds. No rationale was provided for the decision, which represents a significant setback for Rust adoption in the Git codebase. The parallel xdiff cleanup series (12 commits) remains approved and ready for merging.

### Git 3.0 release timing discussion opens

Brian M. Carlson initiated discussion about Git 3.0 release timing, with SHA-256 interoperability as the primary consideration. Key points:

- Original timeline (~1 year, ~4 releases away) may be aggressive
- 200-400 total SHA-256 patches needed, with only ~100 completed
- Incomplete SHA-256 support in major forges and dependent projects
- Debate between schedule pressure vs. feature completeness

The thread is weighing whether to stick to the original timeline to motivate ecosystem or delay until SHA-256 work is complete. Carlson personally prefers not to let SHA-256 block the release due to limited contributor bandwidth.

## In brief

**Optional file specifications** -- Phillip Wood provided final review feedback on D. Ben Knoble's implementation of `:(optional)` prefix support, suggesting type safety improvements (int->bool) and test case refinements.

**Submodule path encoding** -- Kristoffer Haugsbakk contributed several typo fixes to Adrian Ratiu's submodule gitdir path encoding series, including corrections to "collisions", "unnecessarily", and "across".

**Atomic ref updates in git replay** -- Christian Couder and Phillip Wood provided final polish feedback on Siddharth Asthana's patch to make atomic updates default, focusing on commit message structure and option naming.

**SHA-1/SHA-256 interoperability docs** -- brian m. carlson responded to Patrick Steinhardt's review of loose objects format documentation, agreeing to defer map file details and improve sharding explanations.

**Credential URL matching** -- M Hickford confirmed existing urlmatch patterns already provide the requested path prefix matching functionality, planning to document this capability.

**Reflog write config handling** -- Michael Lohmann's v3 patch fixing `git reflog write`'s config handling was approved after addressing Junio's feedback about config reading order and adding comprehensive precedence tests.

**Cherry-pick message alternatives** -- Oswald Buddenhagen proposed using git-notes rather than message trailers for cherry-pick tracking, while brian m. carlson suggested exploring existing hooks.

**Git for Windows 2.51.0(2)** -- Junio Hamano acknowledged Johannes Schindelin's backported fixes as sensible and aligned with upstream priorities.

**git whatchanged deprecation** -- User reports confirmed active usage via oh-my-zsh aliases, with Junio providing migration guidance to equivalent `git log` commands.

## On the radar

**Worktree behavior discussion** -- Junio Hamano provided historical context about worktree design assumptions as the thread explores documentation gaps and cross-VCS comparisons.

**Patch workflow optimization** -- Taylor Blau and Junio continued discussing review requirements and newcomer experience, with Junio clarifying that 'seen' branch inclusion implies no approval.