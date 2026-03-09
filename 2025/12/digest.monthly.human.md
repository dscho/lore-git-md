# Git Mailing List Digest - December 2025

**The month in brief.** December 2025 saw 1,325 emails across 399 threads, marking a busy end to the year with significant progress on multiple fronts. Key developments included the completion of Windows symlink support after a decade of work, major advancements in the ODB abstraction effort, and the finalization of Julia Evans' Git data model documentation. The month also featured heated security discussions around terminal escape sequences, substantial performance optimizations, and the completion of multi-year efforts like the hook subsystem modernization. While some debates (notably around `git-history`'s multi-branch handling) remained unresolved, the month closed with several foundational improvements poised to shape Git's architecture in 2026.

## Key developments

### Windows symlink support completed

After nearly a decade of development, Johannes Schindelin and Karsten Blees' comprehensive symlink support for Windows received final approval in mid-December. The 18-part series provides POSIX-compatible symlink operations through direct Win32 API calls, including proper `stat()`/`lstat()` behavior and directory symlink support. While defaulting to `core.symlinks=false` for safety, the implementation handles edge cases like phantom symlinks and MAX_PATH size reporting, representing a major milestone in Windows-POSIX compatibility. This work touches core filesystem operations and index handling while maintaining backward behavior, finally bringing Windows support to parity with Unix-like systems for this fundamental feature.

### ODB abstraction reaches critical milestone

Patrick Steinhardt's object database (ODB) abstraction effort saw two major advancements in December. First, an 8-part series restructured alternates handling through the ODB source abstraction, replacing direct filesystem operations with source-based interfaces. Then a 10-part follow-up shifted packfile storage tracking from the ODB level to individual ODB sources, ensuring each source (main repo, alternates, etc.) manages its own packfiles independently. Justin Tobler provided thorough review throughout, noting the changes align well with the project's direction. These foundational changes pave the way for pluggable object storage backends, though questions about alternates representation during write operations remain for future work.

### Git data model documentation finalized

Julia Evans' long-running effort to document Git's core data model reached completion in early December with Patrick Steinhardt's final review approval. The seven-iteration series introduces `gitdatamodel.adoc`, explaining objects, references, the index, and reflogs in accessible yet technically precise language. This fills a critical gap in Git's official materials, with reviewers praising its balance of accuracy and clarity. The standardized terminology (notably "file type" over "file mode") reflects careful consensus-building across multiple iterations, providing an authoritative reference that will benefit both new and experienced users.

### Terminal escape sequence security debate

A heated but technically grounded debate emerged about mitigating terminal escape sequence vulnerabilities in Git's sideband channel. Johannes Schindelin demonstrated concrete risks of sequences like `OSC P 1 0 ; ? ST`, while brian m. carlson showed SSH channel limitations. The philosophical divide centered on whether Git should attempt comprehensive sanitization or rely on terminal emulator hardening. While no final resolution was reached, Schindelin's proposal for a granular `sideband.allowControlCharacters` configuration gained traction as a potential compromise. This discussion highlighted ongoing tensions between security hardening and backward compatibility in Git's design.

### Hook subsystem modernization completed

Adrian Ratiu's multi-year effort to modernize Git's hook infrastructure reached completion in late December after six iterations. The 11-part series standardizes hook execution on a new `hook.h` API, introducing structured handling for stdin, output capture, and parallel execution while maintaining backward compatibility. Key improvements include batched stdin processing for receive hooks (500 lines per callback) and removal of direct `find_hook()` calls across the codebase. With sign-offs from Ævar Arnfjörð Bjarmason and Emily Shaffer, plus Junio Hamano's final approval, this foundational work modernizes one of Git's oldest subsystems and enables future hook management features.

## In brief

**Reference backend configuration** -- Karthik Nayak's series enabling backend selection via URI syntax reached consensus around extending `extension.refStorage` config rather than using environment variables alone.

**Submodule gitdir path encoding** -- Josh Steadmon's series implementing `extensions.submodulePathConfig` provides unified gitdir path resolution with a four-tier fallback system for conflicts.

**`git-history` command development** -- Patrick Steinhardt's experimental command saw extensive refinement, with `reword` and `split` subcommands built on replay infrastructure, though multi-branch handling remains debated.

**Performance optimizations** -- Taylor Blau's MIDX compaction series combined with Patrick Steinhardt's geometric repacking optimizations (31x speedup for no-op repacks) to dramatically improve large repository handling.

**Security hardening** -- René Scharfe eliminated insecure `mktemp(3)` usage while Johannes Schindelin addressed CVE-2024-32002 and CVE-2024-52005 with layered ANSI escape sequence defenses.

**`git repo structure` command** -- Justin Tobler extended this experimental command with detailed object size reporting showing both inflated and on-disk sizes with proper unit formatting.

**Documentation standardization** -- Jean-Noël Avila and Kristoffer Haugsbakk coordinated on converting manual pages to consistent synopsis formatting and AsciiDoc tables.

**MacOS build challenges** -- René Scharfe led discussions about macOS 15's broken stateful encoding conversion, resulting in new `NO_HOMEBREW` and component-specific build flags.

**Batched reference updates** -- A regression in detailed error messages was traced to transaction processing, with Karthik Nayak planning fixes to attach errors to individual failed ref updates.

**Promisor remote enhancements** -- A series introduced `promisor.storeFields` configuration and `--filter=auto` mode for dynamic filter combination from promisor remotes.

## Looking ahead

Several major topics will carry into 2026: the `git-history` command's multi-branch handling needs resolution, while the ODB abstraction effort is poised to introduce pluggable backends. The `status.goalBranch` design debate may see alternative proposals, and the Rustification effort remains contentious regarding platform support concerns. Performance work will likely continue focusing on MIDX handling and geometric repacking, while security discussions around terminal handling may yield concrete configuration options. With foundational improvements like Windows symlink support and hook modernization now complete, attention may shift to their follow-on benefits and integration with other subsystems.