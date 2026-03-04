# Git Mailing List Digest - 2025/07/14

**The day in brief.** A moderately busy Monday with 38 emails across 13 threads, featuring significant architectural work on repository metadata commands, final refinements to several merged features, and ongoing discussions about Git's regex architecture. The standout items include a major revision of the `git repo-info` command hierarchy and Junio's "What's cooking" report highlighting upcoming changes for Git 3.0.

## Notable threads

### New `git repo` command hierarchy

Lucas Seiki Oshiro presents a v4 series rebranding `git repo-info` as the first subcommand in a new `git repo` namespace. This complete rewrite introduces a table-driven callback system for repository metadata fields, initially supporting reference format detection (`references.format`), bare repository status (`layout.bare`), and shallow status (`layout.shallow`). The output format has been simplified to null-terminated key-value pairs, dropping JSON support in favor of Git's conventional machine-readable format. The series establishes infrastructure for future subcommands like `git repo stats` while maintaining thorough test coverage and documentation. This represents a significant architectural shift in how Git exposes repository metadata, consolidating functionality currently scattered across commands like `git rev-parse`.

### Post-merge refinements to pagination feature

Several emails discuss final polish for the recently merged `--start-after` pagination in `git for-each-ref`. Christian Couder suggests extracting a goto block into a function and removing an unnecessary else clause, while Junio raises deeper concerns about prefix handling during iterator seeks. The discussion reveals subtle edge cases in how the ref iterator API maintains prefix bounds during paginated iteration, potentially requiring follow-up fixes despite the feature being already merged. Junio also questions whether pagination belongs in porcelain commands like `git branch`, highlighting Git's distinction between scripting and interactive use cases.

### PCRE2 integration debate expands

The meson build system discussion about PCRE2 defaults on macOS has evolved into a broader architectural debate. Junio proposes unifying Git's regex handling under PCRE2 by exposing `grep_pat` throughout the codebase, while Carlo Arenas advocates isolating PCRE2 into a standalone `git-grep` binary. The exchange reveals fundamental tensions between comprehensive architectural change and incremental modularization, with implications for Git's build system and internal APIs. This discussion now extends far beyond the original macOS build issue, touching on long-term direction for Git's text matching capabilities.

### C99 `bool` standardization finalized

Phillip Wood's series to formalize `bool` usage for predicate functions concludes with Junio's approval. After an 18-month experimental period that began in December 2023, the CodingGuidelines now explicitly permit `stdbool.h`, aligning with Git's gradual adoption of C99 features. The mechanical conversions of string utility and strbuf functions to use `bool` returns proceed with only minor documentation nits around hyphenation in commit messages. This represents the completion of a long-planned modernization effort that saw no issues in production use.

## In brief

**Signature handling test style** -- Elijah Newren and Junio agree on refining test redirection scoping in the merged fast-import signature format changes, though the current state is acceptable.

**SHA256 support for gitk** -- Johannes Sixt queues Takashi Iwai's patch with minor code reorganization, moving hash length initialization for better readability.

**Git daemon signal handling** -- Junio approves the v4 series while noting Windows errno handling could be improved later when actual callers emerge.

**Pathspec bloom optimization** -- Derrick Stolee confirms the v6 series is ready after clarifying commit message wording about DEVELOPER=1 fixes.

**Ref-cache cleanup** -- Junio removes unused `find_ref_entry()` function that's been obsolete since 2017, calling it "long overdue".

**GPG path expansion** -- A one-line change enables tilde expansion for `gpg.program` paths, with Junio suggesting adding regression tests.

**Compound literals experiment** -- Phillip Wood proposes using C99 compound literals in `strbuf_init()`, prompting discussion about maintainability tradeoffs.

## On the radar

**SHA-256 default preparation** -- Junio's "What's cooking" notes ongoing work to make SHA-256 the default hash algorithm in breaking changes mode.

**Reftable default backend** -- The reftable backend will become default in Git 3.0 when experimental features are enabled, per Junio's status update.

**`the_repository` removal** -- Significant progress continues on removing this global variable, particularly in the object-file subsystem.