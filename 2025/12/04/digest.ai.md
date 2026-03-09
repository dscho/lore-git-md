# Git Mailing List Digest - 2025/12/04

**The day in brief.** A moderately busy Thursday with 38 emails across 15 threads, featuring significant progress on the hook subsystem refactoring and several performance optimizations. The highlight is the v4 submission of the completed hook API conversion series, while notable discussions continue around Windows path handling and test framework improvements.

## Notable threads

**Hook subsystem refactoring completed** -- Adrian Ratiu and Emily Shaffer submit the v4 series (11 patches) that completes the conversion of all major Git hooks to the new structured API. The series standardizes hook execution across Git, enabling future work on config-based hooks and parallel execution. Key additions in this version include flexible stdin handling, output capture callbacks, and sideband support for receive-pack hooks. The implementation maintains backward compatibility while providing the infrastructure needed for future enhancements. With all technical concerns from prior reviews addressed and positive feedback from Ævar Arnfjörð Bjarmason, this series appears ready for merging.

**Promisor object performance optimization** -- A standalone patch dramatically improves performance when initializing promisor objects from large pack files. By skipping full parsing of blob objects (which don't reference other objects), the change reduces processing time for a 176GB promisor pack from 76 minutes to under 2 minutes. The optimization is particularly valuable for partial clone scenarios and maintains correctness by still fully processing trees, commits, and tags. The well-measured patch demonstrates clear benefits with minimal added complexity.

**Windows drive-letter remotes discussion** -- A thread explores edge cases when copying Git repositories between Windows drives, where local path remotes retain their original drive letters. Contributors suggest several workarounds including UNC paths (\\server\share) and NTFS junction points, while clarifying that Git's behavior of preserving exact remote paths is intentional. The discussion highlights platform-specific considerations for Windows users but doesn't propose any core Git changes.

**In brief**

**`git replay` empty commit handling** -- Phillip Wood and Elijah Newren finalize the patch to drop commits that become empty during replay, with only minor test simplifications and commit message tweaks remaining before graduation from RFC status.

**`-z` flag standardization** -- A three-patch series finalizes the addition of `-z` as an alias for `--format=nul` in `git-repo-structure`, bringing it in line with related commands like `git-repo-info` and `git-config`.

**Clar test framework improvements** -- Patrick Steinhardt confirms progress on addressing Jeff King's feedback about assertion macros and TAP output formatting in the clar test framework.

**Gitk window layout fixes** -- Tobias Boesch engages with Johannes Sixt's feedback about horizontal resizing issues in gitk's pane management, planning a v2 patch with corrected scope and proper email formatting.

**Security hardening follow-up** -- Chris Torek suggests a clarifying comment for René Scharfe's `git_mkdtemp()` implementation to prevent confusion about return value handling between file and directory creation paths.

**On the radar**

**Outreachy internship launch** -- Bello Caleb Olamide begins documenting their work on Git's global state reduction effort through weekly blog posts, providing visibility into this complex refactoring project.

**GSoC 2026 preparation** -- Prospective contributor Ayush Jain introduces themselves and signals intent to participate in next year's Google Summer of Code, following Git's onboarding process with initial documentation patches expected soon.