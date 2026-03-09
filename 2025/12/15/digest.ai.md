# Git Mailing List Digest - 2025/12/15

**The day in brief.** A busy Monday with 64 emails across 23 threads, featuring significant progress on the ODB abstraction effort, multiple documentation improvements, and ongoing refinements to the `git replay` and `repo structure` commands. The most notable developments include Patrick Steinhardt's 10-part packfile refactoring series and Justin Tobler's comprehensive size reporting enhancements for `git repo structure`.

## Notable threads

**Packfile storage refactored for ODB abstraction  
**Patrick Steinhardt's 10-patch series completes a major step in the object database abstraction effort by moving packfile storage tracking from the ODB level to individual ODB sources. This architectural change enables each source (main repo, alternates, etc.) to manage its own packfiles independently. The series includes mechanical but important changes to packfile store initialization, MIDX handling, and kept-pack caching. Justin Tobler provided positive feedback on the early patches, noting the transitional state where stores are still ODB-owned but initialized via sources. The changes touch core packfile handling across 13 files but maintain existing behavior while paving the way for pluggable ODBs.

**`git repo structure` gains comprehensive size reporting  
**Justin Tobler's v3 series (7 patches) extends the experimental `repo structure` command with both inflated content sizes and on-disk storage measurements. The implementation now properly separates numeric values from unit strings for internationalization support and introduces a `HUMANISE_COMPACT` flag for abbreviated displays. The series builds on earlier feedback from Junio Hamano and Patrick Steinhardt, particularly around i18n concerns and test verification methods. The human-readable table output now shows both raw counts and scaled values with appropriate units (decimal SI for counts, binary IEC for sizes), while machine-readable formats gain new `inflated_size` and `disk_size` fields.

**`git replay` empty commit handling refined  
**Phillip Wood's patch to make `git replay` drop commits that become empty during operation received maintainer approval from Junio Hamano after addressing review feedback. The change matches `git rebase --reapply-cherry-picks --empty=drop` behavior and includes test coverage demonstrating branch pointer handling when commits are dropped. A separate thread addressed a segfault regression when using non-existent `--onto` targets, with René Scharfe's fix being acknowledged by both the original reporter and Phillip Wood. Kristoffer Haugsbakk followed up with a proposal to improve error messaging for invalid commit-ish arguments, though this remains in early stages.

**Documentation codifies Git's sign-off policy  
**Junio Hamano and brian m. carlson collaborated on documentation that formally establishes Git's stance against automatic sign-off configuration beyond the existing `format.signoff` exception. The new text explains the legal rationale - that automatic sign-offs could weaken the Developer Certificate of Origin's validity in court by enabling "inertia" certifications. The identical explanation appears in both the FAQ and signoff option documentation to provide authoritative reference material. This concludes a long-running discussion by transforming it into permanent project policy.

**"Rebaser-flow" workflow proposal emerges  
**Milos Markovic introduced a novel DevOps-oriented workflow concept called "rebaser-flow" that prioritizes branches differently than traditional Git models (production > QA > dev). The initial shell script implementation drew criticism for Bash-specific syntax, prompting a proposal for core Git integration via a `--follow-branch` option that would automate hierarchical rebasing during pull operations. The discussion remains at an early stage without concrete use cases or performance data, but represents an interesting departure from developer-centric workflows.

## In brief

**Scalar configuration documentation** -- Derrick Stolee and Henrique Ferreiro continue refining Scalar's configuration documentation, with Junio Hamano providing final polish on the `index.threads` explanation phrasing.

**`git diff` vs merge rename detection** -- Luca Balsanelli identified an inconsistency where `git diff` detects renames between branch heads that `git merge` misses at the same threshold, despite earlier resolution of a related issue.

**MacOS iconv workaround** -- A 3-patch series proposes fixes for ISO-2022-JP conversion issues in macOS 14/15, introducing an `ICONV_RESTART_RESET` flag and consolidating Darwin build configuration.

**`git reset --hard` behavior discussion closes** -- Stefanos Koutsouflakis formally withdrew his RFC proposal to modify `reset --hard`'s destructive behavior after Junio Hamano's firm rejection based on established workflows.

**Documentation style fixes** -- Jean-Noël Avila submitted various man page improvements noticed during French translation work, including imperative mood enforcement and asciidoc markup fixes.

**`rev-list` filter documentation** -- Justin Tobler clarified `git rev-list --filter` behavior when objects are explicitly provided, documenting the `--filter-provided-objects` option's role.

## On the radar

**MEMZERO_ARRAY refactoring** -- Patrick Steinhardt strongly endorsed Junio Hamano's change to make Coccinelle patch failures more debuggable in CI, suggesting this small but impactful change is ready for merging.

**Build system fix for Rust on macOS** -- The solution to GNU/BSD sed incompatibility affecting Rust builds is agreed upon but awaits implementation from D. Ben Knoble or another contributor.

**Copy detection in `diff-files`** -- SZEDER Gábor noted a build dependency issue with René Scharfe's fix for copy detection, reminding maintainers that two changes must be applied in sequence.