# Git Mailing List Digest - 2026/04/11

**The day in brief.** A moderately active Saturday with 17 emails across 6 threads, featuring final refinements to Harald Nordgren's autostash series, technical reviews of Lorenzo Pegorari's promisor repack work, and several documentation fixes. The standout development is the resolution of the `CHECKOUT_AUTOSTASH_HEAD` naming convention, marking the last hurdle before merging a long-running feature branch.

## Notable threads

### Autostash refname convention settled

Harald Nordgren's v9 series implementing unified conflict resolution in `git checkout -m` reached its final refinement today as Jeff King confirmed the correct naming convention for the internal autostash ref. The series had already received maintainer approval but needed one last adjustment: renaming `CHECKOUT_AUTOSTASH` to `CHECKOUT_AUTOSTASH_HEAD` to comply with Git's documented rules for root-level refs. While verbose, the new name properly follows the requirement that such refs end with "_HEAD". This purely naming-related polish doesn't affect the core functionality, which enhances branch switching conflict resolution through integrated autostash behavior and customizable conflict marker labels. With this last issue resolved, the series appears ready for merging after nine iterations.

### Promisor repack series nears completion

Lorenzo Pegorari's GSoC work on promisor file handling during repacks saw detailed technical review today as the v5 series approaches merging. Tian Yuchen provided implementation feedback on both the core functionality and test cases, pointing out shell scripting anti-patterns in the test implementation and suggesting memory efficiency improvements. This follows Junio C Hamano's earlier fixes for a memory leak that had caused CI failures. The discussion revealed deeper questions about packfile API design, with Lorenzo exploring alternative approaches using `packfile_store_load_pack()` instead of `parse_pack_index()`. With all major technical issues resolved and the series already in Junio's 'seen' branch, this appears to be final polishing before Eric Sunshine's review as GSoC mentor.

### Documentation formatting fixes

Jeff King led a focused discussion on AsciiDoc formatting in the Git glossary, addressing indentation issues in entries with sub-lists. The thread progressed from problem identification through cross-tool analysis (revealing differences between asciidoc and asciidoctor behavior) to a robust solution combining blank lines and "--" markers for consistent rendering. Meanwhile, Kristoffer Haugsbakk submitted two documentation patches: one correcting a misstatement about `git am --message-id` behavior (reverting part of his own earlier commit) and another fixing a minor option reference format in the `git am` man page. These documentation threads demonstrate Git's attention to both content accuracy and presentation details.

## In brief

**RFC stash create with untracked files** -- Shabbir Bhojani followed up on his RFC patch adding `--include-untracked` support to `git stash create`, politely requesting review after two weeks without feedback. The patch remains unchanged from its initial submission with comprehensive tests and documentation.

**Message-ID documentation correction** -- Kristoffer Haugsbakk reverted part of his earlier commit that incorrectly described `git am --message-id` as adding a Git trailer, clarifying it actually appends the Message-ID line directly after the message body.

**Man page option formatting fix** -- In a separate patch, Kristoffer corrected a minor inconsistency in the `git am` man page, ensuring the `--no-message-id` option is properly formatted with leading double-dashes.