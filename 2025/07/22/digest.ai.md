# Git Mailing List Digest - 2025/07/22

**The day in brief.** A busy day with 135 emails across 30 threads, featuring significant discussions on Rust integration, reflog migration fixes, and the stabilization of experimental commands. Notable threads include ongoing debates about Rust adoption tradeoffs, a comprehensive fix for reflog migration edge cases, and the removal of experimental markers from `git-switch` and `git-restore`.

## Notable threads

### Rust integration debate continues

The discussion around Rust adoption in Git's xdiff implementation saw extensive debate today, with platform compatibility concerns taking center stage. Gentoo maintainers provided detailed breakdowns of Rust's limitations across various architectures (HPPA/Alpha/m68k lacking support, s390 limited to binaries). Patrick Steinhardt maintained his position favoring full Rust adoption, while Junio Hamano noted the architectural challenges of partial integration given Git's current structure. The thread revealed deep tensions between performance benefits (5-19% speedups claimed) and practical deployment constraints, with no clear consensus yet on how to proceed.

### Reflog migration fixes reach completion

Patrick Steinhardt's 8-part series addressing reflog migration issues between storage formats (files <-> reftable) reached its final stages. The patches correct subtle bugs in identity handling and old-OID preservation during migration, with comprehensive test coverage added. Junio Hamano provided the final review note, suggesting a minor shell scripting improvement to a test helper function. This series resolves critical edge cases that could cause assertion failures or incorrect merge results when repositories change storage formats, particularly relevant with reftable becoming the default in Git 3.0.

### Experimental commands stabilize

Justin Tobler proposed removing the experimental markers from `git-switch` and `git-restore` after 6 years of stable use. While the documentation updates were uncontroversial, Junio Hamano pushed back against removing the `-m` short flag for `--merge`, citing his own workflow dependencies. The thread revealed these commands represent a "failed experiment" in the sense that hoped-for UI improvements never materialized, but consensus formed that they're now too entrenched to change. The documentation updates will proceed while the flag changes remain under discussion.

### Merge-ort rename detection fixes

Elijah Newren submitted a 6-part series fixing complex edge cases in merge-ort's rename detection, particularly around directory renames interacting with other operations. The patches address scenarios where files could be incorrectly deleted or retained during merges, with over 300 lines of new test coverage. The series shows the continued refinement of merge-ort as it handles increasingly subtle corner cases, building on its position as Git's default merge strategy.

## In brief

**`git last-modified` final polish** -- Toon Claes and Taylor Blau resolved remaining architectural questions about callback structures and performance optimizations in the new blame-tree inspired command.

**Interactive patch context configuration** -- Leon Michalak's series adding `-U/--unified` options to interactive commands reached final approval pending test adjustments to restore accidentally removed coverage.

**`git repo info` subcommand** -- Lucas Seiki Oshiro's GSoC project added repository metadata inspection with key=value output, though debate continues about the machine-readable format's field syntax.

**Partial clone fetch race fix** -- Jeff King identified and fixed a race condition where repacks could make objects temporarily unavailable during fetches in treeless/blobless clones.

**File descriptor leak fixes** -- Hoyoung Lee's series addressing resource leaks in test helpers (`test-truncate`, `test-delta`) progressed after debate about cleanup necessity in error paths.

**IMAP sent-folder archiving** -- Aditya Garg's `git send-email` feature reached naming consensus ("imap-sent-folder") but faced new questions about whether the functionality belongs in core Git.

**Windows permission issues** -- Johannes Schindelin investigated "Permission denied" errors in Git for Windows 2.50.1, with one user confirming downgrade to 2.24.1.2 resolved their issues.

## On the radar

**Rust adoption implications** -- The ongoing thread may see responses to Gentoo's detailed platform support analysis, particularly around gccrs progress and LTS release needs.

**VCS metadata protection** -- Early thread about treating `.jj`/`.sl` directories as precious files may gain traction as more users adopt alternative VCS tools alongside Git.

**`git switch`/`restore` stabilization** -- Watch for revised patches addressing Junio's concerns about `-m` flag removal while proceeding with documentation updates.