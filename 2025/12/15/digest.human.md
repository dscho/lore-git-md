# Git Mailing List Digest - 2025/12/15

**The day in brief.** A busy Monday with 64 emails across 23 threads, featuring significant progress on the ODB abstraction effort, multiple documentation improvements, and ongoing refinements to the `git replay` and `repo structure` commands. The standout developments include Patrick Steinhardt's 10-part packfile refactoring series and Justin Tobler's comprehensive size reporting enhancements for `git repo structure`.

## Notable threads

**Packfile storage refactored for ODB abstraction** -- Patrick Steinhardt posted a 10-patch series that moves packfile storage tracking from the object database (ODB) level to individual ODB sources. This foundational change completes key architectural groundwork for pluggable object databases by ensuring each source (main repo, alternates, etc.) manages its own packfiles independently. The series systematically migrates packfile handling through mechanical but important refactors, touching core packfile operations while maintaining existing behavior. Justin Tobler provided positive feedback on the early patches, noting the changes align well with the project's direction.

**`git repo structure` gains comprehensive reporting** -- Justin Tobler's v3 series (7 patches) extends the experimental `repo structure` command with detailed object size reporting in both human-readable and machine-readable formats. The implementation now shows inflated content sizes and on-disk storage sizes with proper unit formatting (KiB/MiB) and internationalization support. The series incorporates feedback from Junio Hamano and Patrick Steinhardt, particularly around test verification methods and output formatting. With all major technical concerns addressed, this feature appears ready for integration.

**`git replay` error handling improvements** -- Multiple threads addressed `git replay` stability and usability. René Scharfe fixed a segfault when using non-existent `--onto` targets, which Phillip Wood approved. Kristoffer Haugsbakk then proposed more explicit error messages for invalid commit arguments, though this remains a work in progress. Separately, Phillip's patch to drop empty commits during replay received maintainer approval after addressing Junio's stylistic feedback. These changes collectively improve the experimental command's robustness as it moves toward wider adoption.

**Documentation standardization efforts** -- Jean-Noël Avila posted a patch series improving several man pages (`git-checkout`, `git-rebase`, etc.) with consistent imperative mood, fixed asciidoc markup, and clarified parameter documentation. Kristoffer Haugsbakk suggested splitting the changes into separate commits for prose versus technical fixes. Meanwhile, Justin Tobler proposed clarifying `git rev-list --filter`'s behavior with explicitly provided objects, documenting that they bypass filters unless `--filter-provided-objects` is used.

**Workflow proposal: "rebaser-flow"** -- Milos Markovic introduced a novel DevOps-oriented workflow that prioritizes branches differently than traditional Git models (production > QA > dev). The initial proposal used Bash-specific scripts, prompting Kristoffer Haugsbakk to identify portability issues. Markovic then suggested core Git integration via a `--follow-branch` option for pull/rebase operations. While intriguing, the proposal remains at an early discussion stage without concrete implementation plans.

## In brief

**Scalar configuration documentation** -- Derrick Stolee and Henrique Ferreiro continued refining Scalar's configuration documentation, addressing questions about non-performance settings and default value duplication. The series is in final polish stages after Junio Hamano's grammatical corrections.

**MacOS iconv workaround** -- A 3-patch RFC series proposes fixes for ISO-2022-JP conversion issues in macOS 14/15, introducing an `ICONV_RESTART_RESET` flag and reorganizing Darwin build configurations.

**MEMZERO_ARRAY refactoring** -- Patrick Steinhardt strongly endorsed Junio Hamano's change to make Coccinelle check failures more debuggable in CI by showing failed patch contents.

**`git reset --hard` discussion concludes** -- Stefanos Koutsouflakis formally withdrew his RFC to modify `reset --hard`'s behavior after Junio Hamano's firm rejection, preserving the established workflow pattern of `reset --hard && clean -f -x`.

**Signed-off-by documentation** -- Junio C Hamano and brian m. carlson finalized documentation explaining Git's stance against automatic sign-off configuration, citing legal concerns about weakening the Developer Certificate of Origin.

## On the radar

**Rust build fixes** -- The macOS sed incompatibility affecting Rust builds now has a confirmed portable solution (`-n` with `p` instead of `-s`), awaiting implementation from D. Ben Knoble or others.

**`git diff` vs merge rename detection** -- Luca Balsanelli identified an inconsistency in rename detection between `git diff` and `git merge` when operating on the same commit pairs, despite using identical thresholds.