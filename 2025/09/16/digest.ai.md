Here's the daily digest for September 16, 2025:

## The day in brief

A busy day with 112 emails across 30 threads, featuring significant progress on several fronts. Key highlights include the completion of the ODB transaction refactoring series, ongoing Rust infrastructure discussions, and multiple documentation improvements. The Git community also saw active debate around user experience issues in `git status` and `git stash` behaviors.

## Notable threads

**ODB transaction refactoring complete** -- Justin Tobler's 6-part series to establish proper transaction interfaces for Git's object database subsystem has reached its final form. The changes relocate transaction code from bulk-checkin to object-file, introduce a pluggable backend infrastructure, and simplify the API by removing nested transaction support in favor of explicit NULL returns when transactions are active. Junio Hamano and other reviewers have approved the series after thorough discussion of transaction lifecycle and safety guarantees.

**Rust infrastructure design debate continues** -- Ezekiel Newren raised significant concerns about Patrick Steinhardt's Rust implementation approach, particularly around build system integration and Windows support. The discussion revealed tensions between an MVP-focused strategy and Newren's vision for a more modular, Cargo-first architecture. Ramsay Jones added practical concerns about Linux distribution packaging challenges, highlighting real-world deployment hurdles that the build system redesign must address.

**`git stash` gets configurable index restoration** -- A 4-part series from D. Ben Knoble introduces a `stash.index` config option that makes `stash apply/pop` restore the index by default when set. This addresses the long-standing inconsistency where `stash push` saves staged changes but `apply/pop` doesn't restore them by default. The feature received maintainer approval after addressing test strategy feedback from Phillip Wood, with Junio noting the current default (false) may be reconsidered based on user feedback.

**Documentation terminology debates** -- Multiple threads saw active discussion about Git's documentation wording choices. Julia Evans defended using more general terms like "data" over technical specifics like "objects" in the `git-push` man page, arguing for beginner accessibility. Meanwhile, Gustavo Velasco-Hernández's clarification of `git push --mirror` behavior was approved after refining the description to enumerate all three mirroring operations (deletion of remote-only refs, force-update of diverged refs, and addition of local-only refs).

**`git status` command suggestion inconsistency** -- A new thread explored why `git status` suggests different commands for unstaging files before versus after the first commit (`git rm --cached` vs `git restore --staged`). Junio Hamano explained this is technically necessary - without any commits, there's no HEAD to restore from, making `rm --cached` the only viable option. While some found the inconsistency unintuitive, the consensus was that the behavior correctly reflects underlying technical constraints.

## In brief

**Tig 2.6.0 released** -- Thomas Koutcher announced tig 2.6.0 with 9 bugfixes and several user-facing enhancements including a new committer column and improved Unicode support.

**Shallow clone limitations in CI** -- Philipp Hahn's discussion about `--shallow-exclude` limitations in GitLab CI workflows led to exploration of blobless clones as an alternative approach when commit hash exclusion isn't supported.

**`git add -p` behavior changes** -- Phillip Wood and Junio debated whether hunk splitting changes should be gated behind `WITH_BREAKING_CHANGES`, ultimately agreeing to make the change unconditionally as it fixes clearly suboptimal behavior.

**Color handling refactoring** -- Jeff King submitted a 13-part series standardizing Git's color handling code, replacing numeric constants with named enums and improving type safety across 32 files.

**HP-UX threading fix** -- A patch addresses pthread detection issues on HP-UX by removing empty string test from configure.ac, forcing explicit pthread linkage as required by the platform.

**SOCKS proxy test failure** -- Brian Carlson reported a test failure in t5564 where libcurl 8.16.0 on Debian unstable appears to misinterpret socks4:// URLs as HTTP proxies.

## On the radar

**Rustification effort** -- The build system redesign discussion continues as the project balances Windows support, crate modularity, and distribution packaging concerns against the Rust transition timeline.

**Command deprecation framework** -- Kristoffer Haugsbakk's series improving messaging for deprecated commands like `git whatchanged` needs minor adjustments before final approval.

**`git-repo-info` interface** -- Debate continues about whether the experimental command should output all fields by default or require an explicit `--all` flag, with Junio expressing concerns about scripting implications.