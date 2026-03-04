# Git Mailing List Digest - 2025/07/14

**The day in brief.** A moderately busy Monday with 38 emails across 13 threads, featuring significant progress on several fronts. The standout developments include the formal adoption of C99 `bool` type in Git's codebase, a major architectural revision of the `git repo-info` command, and Junio's comprehensive "What's cooking" report outlining the project's current state. Technical discussions ranged from regex architecture to compound literals, while several patch series reached final approval stages.

## Notable threads

### C99 `bool` type officially adopted

Phillip Wood's series to standardize the use of C99's `bool` type for predicate functions has been formally accepted after an 18-month experimental period. The change updates Git's CodingGuidelines to permit `bool` usage and converts string utility functions in `git-compat-util.h` and `strbuf.[ch]` to return `bool` instead of `int`. Junio Hamano noted this follows Git's pattern of gradually adopting C99 features after proving them stable in production use. The series sparked a minor style discussion about hyphenation in commit messages ("case-dependent" vs "case independent"), with Brian m. carlson providing Chicago Manual of Style guidance.

### `git repo` command architecture takes shape

Lucas Seiki Oshiro submitted a major revision of the `git repo-info` proposal, now restructured as the first subcommand in a new `git repo` hierarchy. The v4 series introduces a table-driven field system for repository metadata, initially supporting three fields: `references.format`, `layout.bare`, and `layout.shallow`. The implementation uses a null-terminated key-value format and includes thorough documentation and build system integration. This represents a significant architectural shift from previous standalone command proposals, with clear separation between lightweight metadata (`repo info`) and future computational metrics (`repo stats`).

### Regex architecture debate expands

The discussion about PCRE2 integration in Git's build system has evolved into a broader debate about the project's regex architecture. Junio Hamano proposed unifying Git's text matching under PCRE2 by exposing the `grep_pat` struct and converting direct regex calls throughout the codebase. Carlo Arenas characterized this as "dreamware" and suggested a more modular approach - isolating PCRE2 functionality and potentially making `git-grep` a standalone binary. The exchange highlights tensions between comprehensive architectural overhaul and incremental improvements, with implications for Git's dependency management and internal design.

### Compound literals proposed

Phillip Wood floated a test balloon for using C99 compound literals in Git's codebase, demonstrating the technique with `strbuf_init()`. The proposal shows how compound literals could simplify struct initialization at call sites and reduce boilerplate code. Junio expressed cautious interest but raised concerns about maintainability, particularly around resource cleanup when structs gain members requiring release functions. This discussion may shape Git's approach to adopting more modern C idioms while maintaining its conservative stance on compiler requirements.

## In brief

**Signature handling finalized** -- Elijah Newren and Junio Hamano concluded discussion on Christian Couder's `fast-export`/`fast-import` signature format improvements, with only minor test formatting nits remaining.

**SHA256 support for gitk** -- Johannes Sixt reviewed Takashi Iwai's v2 patch for SHA256 repository support in gitk, suggesting some code organization improvements around hash length handling.

**Git daemon signal handling** -- Junio approved Phillip Wood's v4 series for Git daemon signal handling, leaving Windows errno behavior as a potential follow-up improvement.

**Pathspec bloom filters** -- Derrick Stolee and Junio finalized the pathspec bloom filter optimization series after addressing sign-off ordering and commit message precision in v6.

**Pagination API refinements** -- Christian Couder suggested small improvements to the recently merged `for-each-ref` pagination feature, including potential expansion to `git branch` and `git tag` (though Junio expressed skepticism about the latter).

**GPG path expansion** -- A bugfix enabling tilde expansion in `gpg.program` paths was acknowledged, with Junio suggesting adding regression tests for the new behavior.

## On the radar

**SHA-256 default preparation** -- Junio's "What's cooking" noted ongoing work to make SHA-256 the default hash algorithm when experimental features are enabled, signaling a major upcoming change.

**Reftable default backend** -- The reftable ref storage format will become the default in Git 3.0 for repositories with experimental features enabled, marking a significant architectural shift.

**Compound literals experiment** -- Phillip Wood's test balloon for C99 compound literals may lead to broader adoption if compiler compatibility and maintainability concerns can be addressed.