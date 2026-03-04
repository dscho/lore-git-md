# Git Mailing List Digest - 2025/06/20

**The day in brief.** A busy Friday with 79 emails across 15 threads, dominated by major technical discussions around the SHA-256 transition and IMAP improvements. Key highlights include the SHA-256 default hash algorithm series nearing completion, the long-running `imap-send` overhaul receiving maintainer approval, and ongoing debates about code formatting policies.

## Notable threads

### SHA-256 default hash algorithm transition

brian m. carlson's 10-part series to make SHA-256 the default hash algorithm when built with `WITH_BREAKING_CHANGES` reached final review stages today. The changes introduce new constants (`GIT_HASH_DEFAULT` and `GIT_HASH_ORIGINAL`) to distinguish between legacy SHA-1 requirements and the new default, with extensive test suite updates to handle both algorithms. Junio Hamano raised thoughtful questions about constant naming and backward compatibility testing, leading to productive discussion about repository initialization edge cases. The series appears technically sound after addressing these concerns, with only minor question of whether to rename `GIT_HASH_ORIGINAL` to something more explicit like `GIT_HASH_LEGACY_SHA1` remaining open.

### IMAP-send overhaul complete

After 19 iterations, Aditya Garg's comprehensive `imap-send` improvement series received Junio's approval for merging. The changes fix critical configuration parsing bugs, add OAuth2.0 support, improve folder management, and enhance error handling. Today's final version addressed the last remaining string safety concerns by switching to `xstrfmt()` and `strbuf` for credential encoding. A parallel thread from Jörg Thalheim improved IMAP configuration error messages, now using Git's standard `advise()` API to provide actionable guidance when required settings are missing. These changes make `imap-send` both more functional and user-friendly after years of relative neglect.

### Batched reference update fixes

Karthik Nayak submitted final fixes for production issues with batched reference updates, addressing a segfault in the files backend and directory/file conflict handling in `git receive-pack`. The changes implement phase-based transaction processing (deletions first, then other operations) to maintain consistency with pre-batched behavior. Junio noted these v5 patches have already been merged but suggested future work could rework the base topic directly rather than applying fixes on top. The thread highlights Git's careful approach to performance optimizations - the batched updates provide significant speedups but required multiple iterations to handle edge cases correctly.

### Repo-info command design discussions

The new `git repo-info` command series saw extensive design feedback today, particularly around its output format handling and `--allow-empty` flag justification. Junio Hamano suggested inverting the flag's polarity to `--all` for clearer semantics and proposed architectural improvements to the field handling code. Karthik Nayak repeatedly emphasized the need for better documentation, noting the lack of man pages is causing `make check-docs` failures. The thread shows a healthy mentor-mentee dynamic, with experienced contributors guiding the GSoC student through Git's design principles while allowing room for learning.

### Code formatting policy debate

The ongoing discussion about Git's `.clang-format` configuration and style enforcement continued today with philosophical differences emerging. Junio Hamano argued for consistent mechanical improvements in new code, while Christian Couder warned against "bikeshedding" through excessive style checker suggestions. Brian m. carlson advocated for fire-and-forget automation similar to Go's `gofmt`, but Junio pushed back until the tool can avoid objectively worse formatting. The thread reveals tensions between consistency and readability, with no clear resolution yet on how aggressively to enforce style rules.

## In brief

**Symref usage patterns** -- Kristoffer Haugsbakk shared practical experiences with symbolic references, detailing a stable workflow using `refs/H` and `refs/O` symrefs after encountering pitfalls with other approaches.

**Netrc credential helper port validation** -- Maxim Cournoyer's patch to support symbolic port names received RFC6335-based validation rules from Andreas Schwab, resolving earlier questions about allowed characters.

**Submodule remote lookup regression** -- Junio reported a segfault in Jacob Keller's submodule series caused by removal of `branch->merge_name`, requiring a fix before the otherwise-approved changes can proceed.

**Clang-format rule limitations** -- Junio responded to Karthik Nayak's example where the `RemoveBracesLLVM` rule produces questionable output, suggesting it may be too blunt for Git's style needs.

## On the radar

**Rustification effort** -- While not active today, Ezekiel Newren's work to introduce Rust code remains a contentious topic, particularly regarding platform support concerns raised by Randall S. Becker.

**Documentation conversion** -- Jean-Noël Avila's synopsis-style man page conversion continues in the background, with periodic updates expected as more files are converted.

**ODB abstraction** -- Patrick Steinhardt's object database refactoring work is nearing completion but may require follow-up patches as the SHA-256 changes land.