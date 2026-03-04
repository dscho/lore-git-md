# Git Mailing List Digest - 2025/07/31

**The day in brief.** A busy Thursday with 113 emails across 18 threads saw significant progress on several fronts. Key developments include final approval of the `git last-modified` command, major performance improvements for remote ref operations, and ongoing refinement of the promisor-remote protocol extensions. The string-list API also saw extensive refactoring while JSON output support and Rust integration discussions continued.

## Notable threads

### `git last-modified` approved for merging

After six iterations, Junio Hamano has approved Toon Claes' `git last-modified` series for merging. The new command provides tree-level path tracking with Bloom filter optimization, showing 50.6% speedup for top-level checks. The final version drops the controversial `--format` option while maintaining comprehensive test coverage (19 scenarios) and SHA-256 compatibility. Patrick Steinhardt's suggestion about Bloom filter API usage was noted but didn't block merging, with Junio accepting the current `prepare_commit_graph()` export approach.

### Remote ref renaming performance overhaul

Patrick Steinhardt's series optimizing remote ref operations shows dramatic improvements - from 238 seconds down to 2 seconds (113x faster) for the files backend. The v2 implementation uses atomic transactions and prefix-limited iteration to eliminate quadratic behavior while properly handling edge cases like dangling symrefs. The changes affect core ref handling code across multiple commands, with the reftable backend maintaining its performance advantage (8.6s to 1.2s). Junio indicates this will be included in the next integration batch.

### Promisor-remote protocol extension refinements

Christian Couder's promisor-remote extension series reached v7 with all technical implementation complete, pending final terminology decisions. The changes allow servers to advertise additional remote attributes (like `partialCloneFilter` and `token`) through new `promisor.sendFields`/`checkFields` configs. Junio's review focused on parsing improvements, praising the switch to `string_list_split_in_place()` while suggesting minor string handling refinements. The thread's remaining discussion centers on whether to use "field" or "configuration variable" terminology in documentation.

### JSON output format discussions progress

Ron Ziroby Romero advanced the JSON output proposal by addressing concrete implementation questions for `git log`. The thread has moved from philosophical debate about encoding issues to technical design, with Patrick Steinhardt's `git status --json` work providing a precedent for flag naming (`--json` vs `--pretty=json`). Romero is reconsidering whether to modify `pretty.c` directly or create a separate `json-log.c` implementation, indicating active development on this long-discussed feature.

### `git-secure` v2 proposal

An RFC for version 2 of `git-secure` was proposed, expanding this security wrapper to cover `pull`, `fetch`, and submodule operations. The new version adds token validation and improved messaging while exploring deeper Git integration options. The author seeks feedback on making this a native feature, with working prototype available on GitHub.

## In brief

**String-list API refactoring** -- Junio Hamano's 11-part series replaces `strbuf_split*()` with `string_list_split*()` across multiple subsystems, reducing code by 12 lines net while improving memory safety.

**`core.commentChar=auto` deprecation** -- Phillip Wood's series to remove this problematic option is complete, with Git 3.0 behavior previewed via `WITH_BREAKING_CHANGES`. The changes include clear migration advice for users.

**Windows CRLF handling fix** -- Johannes Sixt corrected line-ending processing in interactive commands like `git reset -p`, replacing `strbuf_getline_lf()` with `strbuf_getline()` to properly handle CRLF on Windows.

**`git refs list` implementation** -- A v4 series introduces this new subcommand as a thin wrapper around `for-each-ref` functionality, sharing core logic and documentation via new infrastructure.

**Git-gui macOS modernization** -- v2 patches remove deprecated app bundle infrastructure and improve askpass helper resolution using argv0-based path derivation.

**Documentation typo fixes** -- Multiple threads addressed minor documentation issues, including a clone.adoc formatting correction and string-list API commit message typos.

## On the radar

**Rust xdiff optimization** -- Ezekiel Newren's series continues with discussions about FFI type alignment benefits and whitespace handling tradeoffs between Rust and C implementations.

**`git repo info` command** -- Lucas Seiki Oshiro's GSoC project progresses with refinements to field handling and output formatting based on review feedback.

**Meson build integration** -- git-gui and gitk relocation to subprojects/ remains the final blocker for full meson build system integration.