# Git Mailing List Digest - 2026/04/22 (Wednesday)

**The day in brief.** A moderately busy Wednesday with 76 emails across 18 threads, featuring several significant developments. Key highlights include the conclusion of the long-running autostash feature for `git checkout -m`, a new `git history fixup` subcommand proposal, and a security discussion about credential helper behavior. The day also saw multiple patch series reach final polishing stages, including subcommand autocorrection and negotiation control features.

## Notable threads

### `contrib/subtree` modernization reaches deployment phase

The `contrib/subtree` modernization effort has progressed to deployment planning, with Colin Stagner's shell patches for in-tree maintenance and Ian Jackson's Rust/libgit2 version as an external tool now seeking naming approval from original author Avery Pennarun. Johannes Schindelin expressed strong support for the Rust implementation's benefits on Windows, where shell script performance is problematic. The dual-path approach appears settled - maintaining the shell version in-tree while developing the Rust version externally - with current focus shifting to deployment mechanics and naming continuity.

### Subcommand autocorrection ready for merging

Jiamu Sun's subcommand autocorrection feature has reached final polishing stage after multiple iterations. The series adds autocorrection for mistyped subcommands (like `git remote rmtoe` suggesting `rmote`) with behavior matching Git's existing command autocorrection. The implementation includes comprehensive test coverage and thorough API documentation, with all maintainer feedback addressed. The feature will give users consistent autocorrection whether they mistype a main command or subcommand.

### Autostash feature for `git checkout -m` concludes

Harald Nordgren's 17-iteration effort implementing autostash behavior for `git checkout -m` with conflict resolution customization has fully concluded. The feature, which provides unified conflict resolution during branch switching operations, is now merged and stable with all administrative follow-ups complete. The final exchange was a simple acknowledgment between Harald and Phillip Wood regarding test refinements, marking the end of this long-running development effort.

### New `git history fixup` subcommand proposed

Patrick Steinhardt proposed a new `git history fixup` subcommand that allows amending staged changes to an existing commit while automatically rebasing dependent branches. The implementation builds on previous `git history` work and includes extensive tests, though it currently aborts on merge conflicts pending future Git capabilities. Tian Yuchen reported a bug where file deletions aren't properly incorporated into fixup commits, while D. Ben Knoble provided substantive review focusing on documentation clarity.

### Credential helper security discussion clarifies protocol

A security discussion about `git-credential-libsecret` leaking credentials during store operations reached consensus that the issue stems from improper direct invocation of helpers rather than a protocol flaw. Phillip Wood clarified that credential helpers are intentionally not in `$PATH` and users should always use `git credential` rather than invoking helpers directly. While the helper will be fixed to suppress unnecessary output, the broader solution involves reinforcing proper interface usage patterns.

## In brief

**Incremental MIDX/bitmap repacking** -- Taylor Blau's series implementing geometric MIDX chain repacking received final approval from Elijah Newren, confirming all feedback from v1 has been addressed.

**Negotiation control for fetch/push** -- Derrick Stolee's v3 series introducing `--negotiation-include` and `--negotiation-restrict` options appears technically complete, with only minor naming and documentation refinements remaining.

**Pseudo-merge bitmap fixes** -- Elijah Newren reviewed Taylor Blau's bugfix series, identifying one remaining test script issue (duplicate `git init`) in an otherwise complete patch set.

**Source tree reorganization** -- Patrick Steinhardt's proposal to move libgit.a components into a "lib/" directory moved toward consensus, with Derrick Stolee agreeing the incremental approach reduces risk.

**Commit signing regression fix** -- brian m. carlson's patches fixing a UTF-8 handling regression in commit signing received review from Elijah Newren, who suggested minor warning message refinements.

**Ref backend consolidation** -- Karthik Nayak's series standardizing object validation across ref backends progressed through detailed review from Patrick Steinhardt, with all feedback now addressed.

**Rebase todo list parsing** -- Phillip Wood's improvements to `git status` display during interactive rebase were reviewed by Elijah Newren, who identified subtle behavior changes that should be documented.

**Ninja build compatibility** -- Toon Claes' fix for build failures on older Ninja versions was validated by Patrick Steinhardt, with Phillip Wood ensuring portability across sed implementations.

**Italian translation fix** -- A one-line patch corrected misaligned "usage:" and "or:" text in Italian help output by using a shorter equivalent for "or".

**mksh test suite hang** -- Jeff King analyzed and proposed reverting commit dd3693eb08 which causes indefinite hangs when running tests with mksh due to shell process handling differences.

**Documentation inconsistency** -- A straightforward patch fixed a mismatch between `format.commitListFormat` config option and its `--commit-list-format` command-line flag in the docs.