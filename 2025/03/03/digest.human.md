# Git Mailing List Digest — 2025/03/03

**The day in brief.** A busy Monday with 113 emails across 24 threads saw significant progress on several fronts. Key developments include the finalization of the large object promisors documentation, major steps forward in the `the_repository` removal effort, and productive discussions about partial reference transactions and bash completion performance. Junio Hamano also weighed in on design questions for a proposed `format-patch` enhancement.

## Notable threads

### Large object promisors documentation finalized

Junio Hamano accepted the final patch in Patrick Steinhardt's series documenting the large object promisors (LOP) protocol, marking the completion of this long-running effort. The 656-line technical design document provides concrete examples of LOP storage and clarifies client offloading scenarios, establishing clear principles for handling large objects via promisor remotes. With this acceptance, all components of the promisor-remote protocol v2 capability are now settled and queued for inclusion in the 'next' branch.

### Partial reference transactions near completion

Karthik Nayak's 7-part series implementing partial reference transactions saw extensive discussion and appears ready for merging. The final patches add the user-facing `--allow-partial` flag to `git update-ref`, allowing transactions to continue even when individual reference updates fail. The implementation distinguishes between user errors (skippable) and system errors (fatal), with rejected updates reported in a structured format. While some follow-up work remains (particularly around fetch/receive-pack integration), the core functionality has been thoroughly reviewed and addresses all major design concerns raised earlier in the thread.

### `the_repository` removal advances with object subsystem refactoring

René Scharfe kicked off a major 12-patch series eliminating `the_repository` usage from Git's object-related subsystems. The changes span 87 files with 676 insertions and 613 deletions, systematically converting functions to take explicit repository parameters rather than relying on the global variable. While stopping short of fully removing `the_repository` from object-file.c (which will require a separate series), this work represents significant progress toward enabling pluggable object databases. The series follows the established pattern of mechanical but careful conversions, maintaining behavior while making repository dependencies explicit.

### Bash completion performance optimization debate

An active discussion emerged around optimizing remote ref completion performance in Git's bash completion script, particularly for Windows users where process forking is expensive. Various approaches were debated - from SZEDER Gábor's proposal for a core `for-each-ref` modification to Junio Hamano's suggestion of using bash-specific features to process refs without additional forks. The thread revealed surprising severity in Windows performance issues, where even 4-5 remotes could cause 1+ second delays. While no final solution was settled on, the discussion has clarified the constraints and tradeoffs involved.

### Format-patch subject prefix proposal sparks design discussion

Lucas Seiki Oshiro proposed a new `--subject-extra-prefix` flag for `git format-patch` to help mentoring program participants clearly identify their contributions. While the technical implementation appeared sound, Junio Hamano suggested an alternative approach using the existing `--rfc` flag with a parameter (e.g. `--rfc=GSoC`) rather than adding a new option. This design-level feedback may require significant rework of the series but reflects the maintainer's preference for leveraging existing mechanisms over introducing new flags.

## In brief

The breaking changes documentation was updated to remove references to the abandoned runtime feature.git3 approach in favor of a WITH_BREAKING_CHANGES compile-time switch. Junio Hamano and Patrick Steinhardt agreed the double-negative `without-breaking-changes` Asciidoc attribute naming could be improved.

The `git-diff-pairs` plumbing command series saw its foundational patches reviewed and approved by Junio, with the implementation now focusing on final polish before merging. Documentation references throughout the codebase continued their systematic conversion from .txt to .adoc format in a 34-patch series from Todd Zullinger.

A Windows user reported lock file errors and missing status output after upgrading to Git 2.48.1, with the issues not present in 2.47.1. The behavior appears repository-specific and may indicate a regression in filesystem handling or error reporting.

## On the radar

The ref consistency checks series from shejialuo appears ready for promotion to 'next' pending final review from Patrick Steinhardt and Karthik Nayak. The bundle-uri performance optimization thread resolved questions about tag handling, with Derrick Stolee confirming annotated tags outside refs/tags/ don't violate Git requirements as long as normal fetch behavior is preserved.