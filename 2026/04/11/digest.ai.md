# Git Mailing List Digest — 2026/04/11

**The day in brief.** A moderately active day with 17 emails across 6 threads, featuring final refinements to Harald Nordgren's autostash series, progress on Lorenzo Pegorari's promisor file handling, and several focused documentation fixes. The most notable development is the resolution of a refname convention discussion that clears the last hurdle for the autostash feature's inclusion.

## Notable threads

### Autostash refname convention resolved

Harald Nordgren's v9 series implementing unified conflict resolution in `git checkout -m` through autostash behavior has cleared its final technical hurdle. Jeff King identified that the proposed `CHECKOUT_AUTOSTASH` refname violated Git's documented rules for root-level refs (which must end with `_HEAD`). After discussion, the group settled on `CHECKOUT_AUTOSTASH_HEAD` as the correct naming choice. While verbose, it strictly follows Git's refname validation requirements. This post-approval refinement addresses the last open issue before merging, with the core functionality remaining unchanged. The series has already received maintainer approval from Junio C Hamano and represents a significant enhancement to branch switching conflict resolution.

### Promisor file handling nears completion

Lorenzo Pegorari's v5 series on promisor file handling during repacks, part of his GSoC work mentored by Eric Sunshine, has addressed its final technical issues. Junio C Hamano provided specific fixes to resolve memory leaks that were causing CI failures, including proper cleanup sequences after `parse_pack_index()`. Tian Yuchen contributed detailed implementation feedback on both the core functionality and test implementation quality, pointing out shell scripting anti-patterns and memory efficiency concerns. The series now appears technically complete pending final review, with Lorenzo demonstrating growing familiarity with Git's object storage internals through his exploration of alternative API approaches.

### Documentation formatting fixes

Jeff King led a focused discussion on AsciiDoc indentation issues in the Git glossary, particularly around sub-lists in entries like "glob", "pathspec", and irregular refs. The thread progressed from problem identification through toolchain-specific analysis (revealing differences between asciidoc and asciidoctor behavior) to a robust cross-tool solution. Kristoffer Haugsbakk provided thorough validation of the fixes, confirming they produce consistent output across documentation formats. This exchange exemplifies Git's attention to documentation rendering details, even for subtle edge cases.

## In brief

**`git stash create --include-untracked` RFC ping** -- Shabbir Bhojani followed up on his two-week-old RFC patch adding untracked file support to `git stash create`, politely requesting review as the initial submission hadn't yet appeared in "What's cooking" reports.

**`git am --message-id` documentation correction** -- Kristoffer Haugsbakk reverted part of his earlier commit that incorrectly described the option's behavior, clarifying it appends Message-ID directly after the message body rather than as a proper Git trailer.

**`git am` man page option fix** -- Kristoffer Haugsbakk corrected a minor inconsistency in the `--no-message-id` option reference, adding the missing double-dash to align with standard documentation style.