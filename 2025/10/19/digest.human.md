# Git Mailing List Digest — 2025/10/19

## The day in brief  

A moderately active day with 16 emails across 8 threads, featuring substantive discussions on Scalar configuration, diff output behavior fixes, and SSH key handling refactoring. The most active thread saw multiple iterations of a comprehensive fix for dry-run diff output, while Junio provided key reviews on several technical changes. A misdirected personal email provided the day's only off-topic moment.

## Notable threads  

### Scalar configuration review advances  

Matthew Hughes advanced the Scalar configuration discussion by shifting focus from implementation details to a substantive review of which configs should be set by default. He identified several potentially outdated Windows-specific settings (core.FSCache, credential.validate) and probed the rationale behind CRLF handling, GC automation disablement, and index optimizations. This line of questioning aligns well with the thread's open question about reviewing historical Scalar configs for relevance, though no concrete proposals emerged today.

### Dry-run diff output reaches resolution  

The multi-day discussion about fixing inconsistent diff output in dry-run mode reached a clear resolution path. Junio Hamano confirmed a two-phase approach: an immediate regression fix via /dev/null redirection for the maintenance branch (which he'll handle), followed by Lidong Yan's comprehensive code fixes for proper dry-run behavior (submitted as v3/v4 patches today). Johannes Schindelin contributed a sophisticated CodeQL query to analyze potential output paths, suggesting tools could help validate and eventually remove the temporary workaround. The technical solution is now agreed upon with clear responsibilities divided between maintainer and contributor.

### SSH key handling refactoring reviewed  

Olamide Caleb Bello's series converting SSH key fingerprint handling from `strbuf_split*()` to `string_list_split*()` in `gpg-interface.c` received final reviews from Junio. While approving the technically sound conversion, Junio mused whether simpler string operations might ultimately be preferable to either splitting approach. The patches carefully handle API semantic differences (notably split counting) while maintaining safety checks, with part 1 adjusting fingerprint parsing to expect 2 splits yielding 3 tokens and part 2 modifying default key retrieval to use 1 split for 2 tokens. The series appears ready for inclusion despite the architectural questions raised.

## In brief  

**Bash completion improvements** — KIYOTA Fumiya added `--exclude=` completion for `git log`/`shortlog` and fixed inconsistent `--committer` handling between the commands. Junio approved with minor wording tweaks while noting the underlying UI inconsistency merits future discussion.

**`git diff --stat` path display proposal** — A new feature request seeks to prevent path truncation in diff stats, either by extending `diff.statGraphWidth` behavior or adding a new option to force full path display regardless of terminal width.

**Submodule ignore behavior** — Phillip Wood questioned a patch changing `git add`'s handling of `ignore=all` submodules, noting historical context and suggesting simpler implementation alternatives than the proposed `--force` approach.

## On the radar  

**Misdirected personal email** — A clearly off-topic message about family travel plans accidentally hit the list, serving as a reminder of the occasional noise in the venerable mailing list workflow. No action needed beyond perhaps a polite "wrong list" reply if the sender follows up.