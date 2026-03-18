# Git Mailing List Digest - 2026/03/16

**The day in brief.** A busy day with 128 emails across 45 threads, featuring significant progress on several fronts. Key highlights include final approvals for the `git replay --revert` feature and parallel hook execution, plus multiple GSoC proposals moving forward with mentor feedback. Infrastructure work dominated with build system improvements and ODB abstraction patches, while documentation and test modernizations continued steadily.

## Notable threads

**`replay --revert` reaches final approval**  
The long-running effort to add revert capability to `git replay` has concluded with Phillip Wood's sign-off on Siddharth Asthana's v4 implementation. The series enables server-side commit reversal by treating reverts as merges with swapped arguments, matching `sequencer.c`'s approach. Key refinements include newest-first processing, empty commit dropping, and consolidated message formatting in `sequencer_format_revert_message()`. While interface design questions (flags vs subcommands) remain open, the technical implementation is now merge-ready after extensive review from Patrick Steinhardt, Phillip Wood, and Junio Hamano.

**Parallel hooks configuration refined**  
Adrian Ratiu's parallel hook execution series saw maintainer feedback on per-event job count configuration naming. Junio Hamano approved the technical approach while suggesting validation for misconfigured `hook.foo.jobs` settings. The discussion revealed potential future extensions like an `hook.<event>.enabled` master switch, though the current patch remains focused on core functionality. Output stream handling also prompted debate about whether stdout/stderr merging should be repository-wide or per-hook, with Junio noting this might warrant a Git 3.0 breaking change.

**AI-assisted l10n workflows mature**  
Jiang Xin's v4 series standardizing PO file filters for Git's localization process has addressed all technical concerns, including Johannes Sixt's alignment needs for Gitk. The implementation introduces two filter types (`gettext-no-location` and `gettext-no-line-number`) with clear technical rationale and measurable workflow improvements (82% fewer API turns). Documentation now cleanly separates human and AI workflows in `po/AGENTS.md`, though legal/DCO questions around AI-generated content remain unresolved. The series demonstrates how specialized tooling can enhance translation efficiency while maintaining human oversight.

**Build system reorganization advances**  
Patrick Steinhardt's build tooling consolidation and precompiled header support is now queued for merging after Phillip Wood confirmed v2 changes look good. The series relocates essential build infrastructure to a new `tools/` directory and implements Meson PCH support for git-compat-util.h, demonstrating 40% build speed improvements. Final discussions resolved PCH compatibility concerns with the reftable backend, opting to monitor for issues rather than preemptively splitting out compat/posix.h. The changes exemplify Git's ongoing infrastructure modernization while maintaining cross-platform compatibility.

**ODB abstraction groundwork laid**  
Justin Tobler's 2-patch series introduces `odb_source_files_try()` to safely handle mixed object storage backends, converting 22 call sites across core object access paths. This foundational work for pluggable ODB backends follows patterns from the reftable ref backend effort, gracefully handling non-files sources during iteration. While some display/optimization functions will need follow-up, the changes enable future alternative storage implementations by removing assumptions that all object sources are files-based. The mechanical conversions demonstrate careful attention to type safety boundaries.

## In brief

**Zombie process fixes finalized** -- Junio and Jeff King resolved final const-correctness issues in Andrew Au's zombie process cleanup and Jeff's incidental transport color memory leak fix, marking both ready for merging.

**Subcommand autocorrection polished** -- Jiamu Sun's parse-options autocorrection series received final maintainer feedback about edit distance thresholds and subcommand documentation wording before expected merging.

**`interpret-trailers` docs converted** -- Kristoffer Haugsbakk's v2 series standardizing the command's documentation to synopsis style is complete, including source code terminology updates to match.

**Reference transaction hooks extended** -- Eric Ju's v2 adding a "preparing" phase for distributed coordination received Patrick Steinhardt's approval, with only error message formatting polish remaining.

**Test isolation improvements** -- Mirko Faina's fix for `t0008-ignores.sh` interference between symlink and large-file tests was queued by Junio after verifying the cleanup approach.

**`strbuf` by-value detection** -- Deveshi Dwivedi's Coccinelle rule to catch dangerous `struct strbuf` pass-by-value patterns was approved, completing cleanup of this anti-pattern.

## On the radar

**Partial clone filter follow-up** -- Patrick Steinhardt noted minor post-merge questions about test file "large.bin" in the recently merged URL-based filter configuration feature.

**`git am` line number reporting** -- Jialong Wang's GSoC project to fix incorrect line numbers in corrupt patch errors has mentor approval and is proceeding to implementation.

**Graph output column limits** -- Pablo Sabater's RFC for `--graph-col-limit` sparked design discussion about naming and behavior, with v2 incorporating use case clarifications.