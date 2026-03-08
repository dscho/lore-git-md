# Git Mailing List Digest - 2025/11/16

## The day in brief

A moderately active Sunday with 14 emails across 8 threads, featuring ongoing discussions about Git's author/committer model, UTF-8 alignment testing, and CI infrastructure improvements. Notable developments include Junio Hamano's definitive stance on the `--committer` proposal and resolution of macOS Perforce test flakiness through architecture-aware binary selection.

## Notable threads

### Author/committer asymmetry debate reaches impasse

Junio Hamano provided a detailed defense of Git's intentional asymmetry between author and committer fields, responding to ZheNing Hu's proposal for a `--committer` option in `git commit`. The maintainer emphasized this isn't an inconsistency but reflects real-world workflow differences, where multiple authors typically contribute through fewer committers (like maintainers applying patches). Technical arguments centered on the rarity of needing explicit committer specification versus the common need for author attribution. Matej Dujava later contributed practical examples of multi-identity management using `includeIf` configs, but the core philosophical disagreement about interface design remains unresolved.

### UTF-8 alignment testing strategy settled

Jiang Xin and Junio Hamano converged on testing methodology for CJK character width calculations in `builtin/repo.c`. While Jiang Xin demonstrated a working Chinese localization test case, both agreed translation-dependent tests would be too fragile across release cycles. The decision favors unit testing `strbuf_utf8_align()` directly rather than verifying localized output, resolving one of the last open questions in this thread. This leaves only minor implementation preferences (manual padding vs strbuf utility) and Phillip Wood's encoding concerns as remaining discussion points.

### macOS Perforce CI tests get architecture fix

Koji Nakamaru diagnosed and proposed a solution for flaky Perforce tests on GitHub's arm64 macOS runners, where x86_64 binaries requiring Rosetta translation were causing instability. After confirming native arm64 binaries work reliably, the discussion refined the implementation details - notably reverting to the original r23.2 version once its arm64 support was verified. Junio Hamano's final patch (co-authored with Nakamaru) implements this architecture-aware binary selection, preserving test coverage while improving reliability as GitHub Actions transitions to arm64-based macOS-14 runners.

## In brief

**Lisp userdiff driver unification** -- Junio Hamano suggested expanding the proposed "lisp" driver to cover Scheme, Emacs Lisp and Common Lisp with unified regex patterns, arguing the "(def" prefix would be safe across dialects.

**Bash prompt test portability** -- Kiril Ivanov and Junio Hamano agreed the quiet mode feature's tests need shell-agnostic rewrites despite current bash-isms working accidentally under plain sh, keeping with git-prompt.sh's historical multi-shell support.

**Submodule segfault fix** -- A NULL pointer dereference in `git submodule add` when handling incomplete .gitmodules entries was fixed with proper path checking, plus a NEEDSWORK note about related assumptions in update_submodule().

**Translation updates integrated** -- Jiang Xin's batch of 10-language localization updates (13k+ lines changed) for Git 2.52.0 was pulled by Junio Hamano following standard l10n workflow.