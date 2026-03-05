# Git Mailing List Digest - August 18, 2025

**The day in brief.** A busy Monday with 46 emails across 18 threads, featuring significant progress on several fronts. The highlight is Jeff King's comprehensive 5-patch series fixing edge cases in `git describe`'s blob handling, while Junio Hamano's "What's cooking" report and Git v2.51.0 announcement provide key project updates. The Rust integration discussion continues with maintainer guidance on priorities.

## Notable threads

**Rust integration policy discussion evolves**  
Junio Hamano provided key maintainer perspective on Ezekiel Newren's RFC series introducing Rust as a hard dependency. While accepting the need for consistent Rust style (likely via rustfmt) and flexible version requirements, Junio emphasized that initial Rust adoption should focus on framework establishment (FFI patterns, build integration) rather than performance claims. The discussion revealed tension with parallel C-based xdiff optimization work (am/xdiff-hash-tweak), with Junio noting the speed improvements stem from hash function choice rather than Rust itself. Ben Knoble later reinforced consensus on using default rustfmt configurations, solidifying one of the three policy dimensions under discussion.

**`git describe` safety overhaul completed**  
Jeff King (Peff) delivered a comprehensive 5-patch series addressing multiple edge cases in blob handling within `builtin/describe.c`. The fixes include: proper error reporting for unreachable blobs (patch 2), explicit handling of unborn branches (patch 3), protection against corrupted HEAD states (patch 4), and elimination of redundant commit lookups (patch 5). The series originated from René Scharfe's analysis of NULL pointer risks and evolved into a thorough redesign of error handling. Junio has already approved patches 1-3, with the full series representing a significant hardening of this core command.

**Rebase documentation reaches consensus**  
Phillip Wood and Junio finalized the long-running git-rebase man page improvement series after eight iterations. The last discussion point centered on optimal placement of the `--onto` explanation - whether to keep it as an "advanced topic" or promote it earlier given its common usage. The compromise maintains detailed examples in a dedicated section while adding cross-references from basic usage descriptions. The series achieves a 104-line reduction while improving structure and clarity, with all parties now considering it ready for merge.

**Line-log fixes address longstanding bugs**  
SZEDER Gábor submitted a two-patch series fixing multiple issues with `-L` range specifications in `git log`. The first patch resolves an assertion failure, infinite loop, and incorrect output when processing multiple disjoint line ranges, while the second ensures all ranges touched by a diff are properly shown. The bugs had existed since line-log's 2013 introduction, with some test cases having incorrect expected output all along. Junio acknowledged the comprehensive fix, which required careful handling of boundary conditions across several functions.

**Git v2.51.0 released**  
Junio announced Git 2.51.0, featuring 506 non-merge commits from 91 contributors (including 21 new). Key changes include reftable backend maturation (slated to become default in Git 3.0), new `--compact-summary` for merge/pull, and removal of "experimental" labels from `git switch`/`restore`. The release continues progress on removing `the_repository` global and restructuring the object store while deprecating `git whatchanged`. Jenkins plugin maintainers noted they're working to remove their dependency on this deprecated command.

## In brief

**Submodule hash algorithm mismatch** -- Michael Schroeder pushed back against Brian M. Carlson's assertion that mixed-hash submodules shouldn't work, arguing the current behavior (with padding/truncation) already functions for non-shallow clones.

**Stash --keep-index behavior refinement** -- Phillip Wood proposed adding a trailer to stash commit messages when created with `--keep-index`, enabling automatic unstaged-change-only behavior during pop operations.

**Autosquash rebase cleanup inconsistency** -- Phillip Wood identified a 2021 change that made fixup commits ignore commit.cleanup=strip during rebase, proposing to align this behavior with regular "pick" commands.

**Hostname-based config includes** -- Ayush Sharma revived discussion of adding `hostname:` conditions to `includeIf` sections for multi-machine dotfile management, with Junio referencing prior related threads.

**Line numbers in format-patch** -- Seyi Kuforiji proposed a `--with-line-numbers` flag for `git format-patch` to aid review, though Junio noted technical challenges in maintaining patch applicability.

**Nested interactive rebases** -- Isaac Oscar Gariano proposed allowing modification of earlier commits during an ongoing rebase, with Oswald Buddenhagen indicating related work in progress.

## On the radar

**Rust integration framework** -- With Junio's guidance now emphasizing framework establishment over performance claims, watch for the RFC series to pivot toward build integration and FFI patterns.

**Submodule mixed-hash viability** -- The debate continues about whether Git should formally support submodules with different hash algorithms than their superprojects, with clear use cases but architectural concerns.

**Stash workflow improvements** -- The `--keep-index` trailer proposal could significantly improve interactive rebase testing workflows if the serialization concerns can be addressed.