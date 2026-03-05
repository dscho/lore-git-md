# Git Mailing List Digest - August 17, 2025

**The day in brief.** A moderately active Saturday with 21 emails across 15 threads, featuring several notable developments: the completion of the `git repo` command series, ongoing discussions about JSON output for `git log`, and a major localization update for Git 2.51.0. Documentation refinements and technical edge cases dominated much of the day's discussion.

## Notable threads

### `git repo` command series approved

Junio C Hamano gave final approval to Lucas Seiki Oshiro's `git repo` command series after extensive review, marking the successful completion of this GSoC project. The maintainer confirmed the documentation formatting for the `--format` option follows Asciidoctor best practices, using `:::` for description lists. This v11 series, which introduces a new command hierarchy for repository management, is now cleared to move to the 'next' branch.

### JSON output for `git log` proposal

Ron Ziroby Romero proposed adding JSON output support to `git log` via a `--pretty=json` flag, sparking a detailed technical discussion. The thread quickly moved from high-level design to concrete implementation challenges, particularly around handling non-UTF-8 data in Git objects. Brian m. carlson suggested leveraging Git's existing `json-writer.h` with URL-encoded byte strings, while Junio Hamano probed the proposal's scope regarding diff outputs and special commit headers like GPG signatures.

### Rust dependency and Windows version support

Junio Hamano clarified policy around Windows version support in the ongoing Rust dependency series, acknowledging Git for Windows' documented support policies (currently Windows 8.1+) while emphasizing the need for conscious version support decisions. The discussion revealed tensions between security best practices and backward compatibility requirements, particularly regarding Rust build target selection and documentation of platform support decisions.

### Documentation rendering issues in Asciidoctor

SZEDER Gábor reported rendering issues in the `pretty-formats.adoc` conversion when built with Asciidoctor 2.0.16, highlighting ongoing challenges in maintaining consistent rendering across different AsciiDoc processors. The problems include unwanted `+` characters in default values and misaligned example formatting. While the series has already been approved for merging, this serves as important context for future documentation standardization work.

### Submodule gitdir path encoding test failures

Adrian Ratiu reported test failures in the GitHub CI pipeline for the completed 9-patch series on submodule gitdir path encoding. The v1 series uses URL-style encoding (RFC 3986) with case encoding for filesystem compatibility, but platform-specific issues on Windows and macOS will require fixes in v2. The comprehensive test coverage (24 files) suggests these are likely edge cases rather than fundamental design problems.

## In brief

**Localization updates for 2.51.0** -- Jiang Xin coordinated updates for 10 languages, with substantial changes to Catalan (3k+ lines), Vietnamese (1.5k+), and Swedish (1.1k+). Junio Hamano merged the updates after Git 2.51-rc2.

**CMake build system fix** -- Johannes Schindelin's CMake fix for reftable test discovery was acknowledged by Junio Hamano after being missed in the 2.51.0 release cycle.

**`git stash pop --unstaged` discussion** -- Junio Hamano questioned whether Phillip Wood's proposed `--unstaged` option should be enabled by default rather than requiring a new flag, continuing the thread's exploration of stash workflow improvements.

**Index file checksum inspection** -- Jeff King provided shell commands for viewing the SHA-1 checksum at the end of Git index files, noting there's no built-in Git command for this debugging need.

**`git-add` documentation refinements** -- Jean-Noël Avila and Junio Hamano discussed balancing completeness and beginner-friendliness in documenting `git add`/`git commit` interactions, favoring simple initial explanations with advanced details later.

## On the radar

**Rustification platform support** -- The ongoing discussion highlights unresolved questions about how Rust dependencies will affect platform support policies, particularly for Windows versions and NonStop systems.

**Documentation standardization** -- The Asciidoctor rendering issues and `git-add` pedagogical discussions suggest continued attention needed to documentation consistency and presentation.