# Git Mailing List Digest - 2025/09/30

**The day in brief.** A moderately busy day with 47 emails across 17 threads, featuring significant documentation refinements, ongoing technical discussions about Rust integration and SHA-256 interoperability, and several bugfixes nearing completion. Key highlights include Julia Evans' push documentation series reaching final form, continued debate about Git's Rust future, and the first user feedback on `git whatchanged` deprecation.

## Notable threads

### Push documentation refined

Julia Evans' v4 documentation series (5 patches) clarifying `git-push` behavior and upstream branch relationships has been approved after incorporating Junio Hamano's feedback. The changes:

- Add a new "UPSTREAM BRANCHES" section consolidating scattered information
- Simplify explanations of where/what to push with clearer precedence lists
- Remove inaccurate claims about `push.default=simple` requirements
- Add basic usage examples and standardize section linking
- Improve introductory language to be more accessible

The series represents a significant documentation improvement, particularly around the often-confusing `push.default=simple` behavior. Junio approved the changes while noting one minor wording tweak needed regarding `git branch --track` arguments.

### Rust integration takes unexpected turn

In a surprising development, Ezekiel Newren announced he has dropped his 18-commit Rust integration series, which had previously received positive reviews from brian m. carlson after addressing build system issues. The series included:

- Varint reimplementation in Rust
- Build system integration for both Makefile and meson
- Policy documentation about Rust becoming mandatory
- CI integration components

This reversal comes without explanation, contrasting with the series' previously positive trajectory. Meanwhile, Newren confirmed his separate xdiff cleanup series (removing redundant structures and improving type safety) remains ready for merging.

### SHA-256 interoperability timeline debated

Brian M. Carlson opened a discussion about whether SHA-256 interoperability should delay Git 3.0, noting:

- Only ~100 of 200-400 needed patches have been completed
- Major forges and dependent projects lack full SHA-256 support
- Original timeline would require 75-100 patches per cycle
- Some advocate schedule pressure to motivate adoption while others prefer not to block

The thread is just beginning but will likely involve weighing technical readiness against release management considerations, with ecosystem-wide implications.

## In brief

**Optional file handling finalized** -- D. Ben Knoble's series implementing `:(optional)` prefix support for both config and command-line arguments received final polishing feedback from Phillip Wood, focusing on type safety and test robustness.

**Submodule path encoding** -- Adrian Ratiu's submodule gitdir path encoding series saw several documentation fixes from Kristoffer Haugsbakk, addressing typos in commit messages while the technical implementation progresses.

**Credential URL matching confirmed** -- M Hickford verified that path prefix matching already works in Git's credential configuration using the urlmatch pattern system, moving the discussion to documentation improvements.

**Reflog config handling fixed** -- Michael Lohmann's v3 patch fixes `git reflog write` to properly respect local `user.name` and `user.email` configuration, now with comprehensive test coverage of the precedence hierarchy.

**Git for Windows 2.51.0(2) released** -- Johannes Schindelin announced a Windows-specific release with backported fixes that Junio Hamano confirmed align with upstream priorities.

**Whatchanged deprecation feedback** -- Stephen Harding reported active `git whatchanged` usage via an oh-my-zsh alias, prompting discussion of migration paths to equivalent `git log` commands.

## On the radar

**Worktree behavior debate** -- The discussion about whether worktrees should be created inside the main checkout continues, with Junio Hamano clarifying the original "side-by-side" design intent while acknowledging potential documentation gaps.

**Cherry-pick tracking alternatives** -- Oswald Buddenhagen proposed using git-notes rather than message trailers for cherry-pick tracking, adding another option to Rasmus Villemoes' original hook-based proposal.