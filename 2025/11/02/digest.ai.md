# Git Mailing List Digest — 2025/11/02

## The day in brief

A moderately active Sunday with 12 emails across 4 threads, featuring security clarifications, final polish on the `:(optional)` path series, and mentoring for a new contributor. The key developments are Git's official position that CVE-2024-52005 is a terminal emulator issue rather than a Git vulnerability, and a completed v2 series addressing all feedback on the `:(optional)` path handling.

## Notable threads

### Git project clarifies CVE-2024-52005 is not a Git vulnerability

Brian M. Carlson responded to Thomas Klausner's inquiry about CVE-2024-52005, stating the Git project considers this a terminal emulator bug rather than a Git security flaw. The issue involves Windows terminal emulators executing arbitrary code when clicking hyperlinks created via terminal codes. While Git for Windows listed this as a CVE in their 2.47.1(2) release, the core project maintains the behavior is outside Git's security model since URL clicks should never result in code execution regardless of context. Randall Becker later suggested adding a configurable filter for non-printable characters as a potential mitigation, though this would not address cases where terminals interpret plain URLs as clickable.

### Final polish for `:(optional)` path series

D. Ben Knoble sent a 5-patch v2 series addressing all feedback from Phillip Wood on the `:(optional)` path handling implementation. The key change fixes command-line behavior to properly ignore only missing files (not empty ones) by using `is_missing_file()` instead of `is_empty_or_missing_file()`. The series also includes documentation clarifications, boolean type conversions for flags, and restoration of a dropped `const` qualifier — all small but meaningful improvements following thorough review. With maintainer Junio Hamano having already incorporated some fixes, this appears to be the final iteration of this feature.

### Mentoring on patch submission quality

Junio Hamano provided detailed feedback to a new contributor about patch submission practices in response to Christian Couder's review of an fsck.c refactoring. Noting malformed headers and incorrect indentation, Hamano emphasized patches should only make changes relevant to their stated purpose and avoid unrelated formatting changes. He suggested contributors test email formatting before sending and study examples from established contributors, marking this guidance as potentially useful for the MyFirstContribution documentation (#leftoverbits).

## In brief

**Documentation clarification for `:(optional)` paths** — D. Ben Knoble updates gitcli.adoc to explicitly state the equivalence between using `:(optional)` on a non-existent file and the default command behavior.

**Type safety improvements in parse-options** — Two patches in Knoble's series convert `is_optional` flags from `int` to `bool` in both parse-options.c and config.c, following Phillip Wood's suggestions for clearer code intent.

**Const correctness restoration** — The final patch in Knoble's series restores a dropped `const` qualifier in parse-options.c, maintaining consistency with historical precedent for option parsing.