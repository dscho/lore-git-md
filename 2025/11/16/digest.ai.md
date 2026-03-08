# Git Mailing List Digest - 2025/11/16

## The day in brief

A moderately active Sunday with 14 emails across 8 threads, featuring continued debate about Git's author/committer model, resolution of several technical discussions, and routine translation updates ahead of Git 2.52.0. The most notable developments include Junio Hamano's detailed defense of Git's committer semantics and a fix for Perflorce test flakiness on macOS CI runners.

## Notable threads

### Maintainer clarifies Git's author/committer design philosophy

Junio Hamano provided a thorough explanation of Git's fundamental asymmetry between author and committer fields, responding to ZheNing Hu's proposal for a `--committer` option in `git commit`. The maintainer emphasized this isn't an inconsistency but reflects real-world workflow differences - authors create content while committers physically create commit objects. Junio argued the proposed option addresses an extremely niche use case that doesn't justify interface complexity, despite technical soundness of the implementation. Matej Dujava later contributed practical example of multi-identity management using `includeIf` configs, though this didn't fundamentally change the maintainer's position.

### UTF-8 alignment testing strategy resolved

Jiang Xin and Junio Hamano converged on testing methodology for CJK character width calculations in `builtin/repo.c`. While Jiang Xin demonstrated a working Chinese localization test case, both agreed to rely on `strbuf_utf8_align()` unit tests rather than fragile end-to-end localized output verification. This resolves a key open question from the UTF-8 alignment thread while maintaining the technical approach established in v2 of the series.

### macOS Perforce test architecture fix

Koji Nakamaru diagnosed and proposed a solution for flaky Perforce tests on GitHub's arm64 macOS runners. The issue stemmed from using x86_64 binaries requiring Rosetta emulation. Nakamaru's patch switches to native arm64 Perforce binaries, with Junio subsequently confirming the original r23.2 version already supported arm64, making a version bump unnecessary. This provides a clean solution that preserves test coverage while improving reliability.

## In brief

**Submodule segfault fix** -- A patch addresses a NULL pointer dereference in `git submodule add` when encountering incomplete .gitmodules entries, adding proper path checks and a new test case.

**Lisp userdiff driver discussion** -- Junio Hamano suggested unifying Lisp dialect handling in the userdiff driver, arguing "(def" patterns would be safe across Scheme, Emacs Lisp and Common Lisp despite Johannes Sixt's concerns.

**Bash prompt test portability** -- Kiril Ivanov and Junio Hamano agreed to remove bash-isms from tests for the multi-shell prompt feature, though the quiet mode functionality itself remains unchanged.

**Translation updates merged** -- Jiang Xin submitted comprehensive updates to 10 language translations, totaling over 13,000 lines changed, which Junio promptly merged for Git 2.52.0.