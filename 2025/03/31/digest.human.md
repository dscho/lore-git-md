Here's the Git mailing list digest for March 31, 2025:

---

### The day in brief
March 31 saw significant activity across multiple fronts, with 91 emails spanning 28 threads. The day was dominated by major refactoring efforts - particularly Elijah Newren's landmark series removing the legacy merge-recursive backend in favor of merge-ort, and Patrick Steinhardt's extensive reftable API overhaul. Documentation fixes, build system improvements, and several GSoC proposal discussions rounded out a busy day of development.

---

### Notable threads

**Merge-recursive removal completed**  
Elijah Newren's 8-part series ([1-8/8]) marks the culmination of a multi-year effort to replace Git's legacy merge-recursive implementation with the modern merge-ort backend. The series systematically converts all remaining callers (checkout, merge, sequencer) to use merge-ort, adds final feature parity (diff algorithm support), then removes merge-recursive.[ch] and all associated test infrastructure. The changes delete over 5,000 lines of code while maintaining identical functionality through the new backend. This represents a major architectural simplification now that merge-ort has been the default since Git 2.33.

**Reftable API overhaul**

Patrick Steinhardt's 16-part series ([1-16/16]) refactors the reftable subsystem to improve external usability and enable new internal checks. The changes standardize naming (`reftable_reader` → `reftable_table`), expose block-level iteration for verification, and carefully separate public/private interfaces. The work enables future consistency checking in Git's reftable backend while making the library more suitable for projects like libgit2. The series shows meticulous attention to API boundaries, with comprehensive test coverage accompanying each change.

**Documentation formatting fixes**

A thread initiated by SZEDER Gábor identified Asciidoctor rendering issues with triple-dot syntax in man pages, where `{empty}` strings appeared in output. Jean-Noël Avila and others collaborated on fixes to `asciidoctor-extensions.rb.in`, improving ellipsis handling in command synopses like `git-mv <source>... <dest>`. While the core issue was resolved, some edge cases remain with 'diff-generate-patch.adoc' includes, suggesting future work may be needed on the documentation toolchain.

**Build system consolidation**

Patrick Steinhardt's v2 build system series ([1-5/5]) addressed several Meson integration gaps: fixing curl detection logic, improving gitweb JavaScript generation, adding contrib test support, and enabling cross-compilation. The series also updated CI configurations to properly use Visual Studio for Windows builds. One patch was dropped after Junio Hamano noted it duplicated prior work, keeping the focus on remaining build system inconsistencies.

**GSoC proposal: git metadata command**

K Jayatheerth proposed a new `git metadata` command to provide structured repository information (gitdir path, HEAD state, remote URLs) in JSON format. The well-scoped proposal addresses current automation pain points where scripts must parse human-readable output, and includes a detailed implementation plan with test coverage strategy. Mentors provided initial feedback suggesting the student demonstrate the approach through concrete examples before the formal proposal deadline.

---

### In brief

- **INI file diff support**: Lucas Seiki Oshiro's patch adding built-in userdiff support for INI files (gitconfig, systemd files) was approved after review, with comprehensive test coverage for section headers and comments.

- **Blame porcelain fixes**: Final polish was applied to the `git blame --porcelain` output formatting, addressing portability concerns in test scripts and optimizing sed invocations.

- **Windows lockfile races**: Consensus was reached on using `RtlGetLastNtStatus()` to properly handle pending-deletion race conditions in Windows file locking.

- **git-p4 encoding fixes**: Nikolay Shustov's series addressing non-UTF-8 Perforce message handling gained a reviewer, with testing planned against real Perforce installations.

- **Cygwin gitk fixes**: Mark Levedahl's patch restricting Windows security overrides to Windows-only operation was approved, fixing a regression in Cygwin environments.

- **Backfill batch size**: A platform-specific test failure on s390x was traced to integer type handling in `--min-batch-size`, with a fix using `OPT_MAGNITUDE` instead of `OPT_INTEGER`.

- **Performance benchmarks**: A 5-part series added Meson support for Git's performance test infrastructure while fixing repository format and Perl path handling issues.

---

### On the radar

- **git-daemon ownership checks**: Ongoing discussion about wildcard path handling in `safe.directory` configurations reveals edge cases when used with git-daemon, particularly around relative path resolution.

- **Stash untracked files**: A thread explored potential improvements to `git stash create --include-untracked` that would avoid modifying the working tree, though no immediate patches are planned.

- **SHA-1 backend reporting**: Justin Tobler's series to report SHA implementations in `git version --build-options` is converging on a symbol-based approach after review feedback about maintainability concerns.

The Git project continues its steady pace of architectural improvements and refinements, with particular focus on completing long-running modernization efforts like the merge-ort transition while maintaining stability across platforms.