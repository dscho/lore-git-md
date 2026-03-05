# Git Mailing List Digest - 2025/08/23

**The day in brief.** A busy Saturday with 60 emails across 10 threads, dominated by the ongoing debate about Rust adoption in Git's xdiff infrastructure. The Rust series reached v3 with comprehensive technical implementation but hit a policy roadblock over platform compatibility concerns, particularly for NonStop systems. Meanwhile, Julia Evans' rebase documentation improvements reached final approval, and Johannes Sixt proposed a radical simplification of Git's progress meter signaling.

## Notable threads

**Rust infrastructure for xdiff reaches impasse**  
Ezekiel Newren's 15-part series introducing Rust infrastructure for xdiff reached v3 with all technical components implemented, including C/Rust interop types, build system integration, and the first translated xdiff function (`xdl_trim_ends()`). However, Randall Becker's concerns about mandatory Rust dependencies excluding NonStop systems escalated into a fundamental policy debate. Brian Carlson proposed mrustc as a potential compromise, but platform experts confirmed it wouldn't work for NonStop's constrained environment. The thread now faces a binary choice between modernization and platform support, with no clear resolution path forward. Junio Hamano's late review comments on code style suggest the series is technically merge-ready pending this governance decision.

**Progress meter signaling modernization**  
Carlo Marcelo Arenas Belón proposed replacing `setitimer()` with `alarm()` for progress meter updates, addressing POSIX obsolescence while simplifying platform code. Johannes Sixt countered with a more radical signal-free approach using periodic `getnanotime()` checks, which Junio Hamano initially questioned over performance concerns but later softened his stance on. The discussion revealed Windows-specific signal handling complexities that Sixt's approach would resolve. By day's end, Sixt had a polished prototype ready for CI testing, marking a likely shift in direction for this long-standing subsystem.

**Rebase documentation finalized**  
Julia Evans' v9 series restructuring the `git-rebase` man page received final approval after implementing Junio Hamano's structural refinements. The changes introduce core concepts earlier, consolidate merge conflict instructions, and properly section the `--onto` explanation while maintaining all technical content. The series achieves a 104-line net reduction through careful deduplication, completing a multi-iteration effort to make rebase documentation more approachable without sacrificing completeness.

**git-history RFC documentation polish**  
Jean-Noël Avila provided detailed documentation feedback on Patrick Steinhardt's `git-history` RFC series, focusing on parameter naming consistency (`<commit>` vs `<revision>`) and option syntax standardization across the `drop`, `reorder`, and `split` subcommands. The reviews maintain Git's documentation conventions while accepting the core functionality, suggesting the series is nearing readiness for implementation review after these presentational refinements.

**In brief**  
**MyFirstContribution guide update** -- Daniele Sassoli's v2 simplifies remote repository instructions by presenting both `git/git` and `gitgitgadget/git` options neutrally with workflow comparison links.  

**git refs exists subcommand** -- Meet Soni's v2 series adds test infrastructure for the new subcommand, sharing tests with `git show-ref --exists` via a common library after rebasing onto merged prerequisites.  

**Commit untracked files proposal** -- Isaac Oscar Gariano suggested letting `git commit` handle untracked files directly; Brian m. carlson recommended an explicit `--include-untracked` flag for safety.  

**Compat object format warning** -- A patch documented `extensions.compatobjectformat` as experimental due to fundamental limitations with packed objects and submodules.  

**What's cooking** -- Junio's status update noted 30+ topics merged to master/next including ort improvements, push error handling fixes, and Bloom filter optimizations.  

**On the radar**  
**NonStop platform support** -- The Rust debate has hardened into an explicit tradeoff between modernization and platform compatibility, with no technical compromise currently viable.  

**Progress meter redesign** -- Johannes Sixt's signal-free approach appears poised to replace the current implementation pending CI validation and performance measurements.