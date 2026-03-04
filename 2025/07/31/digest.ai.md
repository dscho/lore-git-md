Here's the daily digest for July 31, 2025:

## The day in brief

A busy day with 113 emails across 18 threads, featuring significant progressions in several major series. The promisor-remote protocol extension nears completion after addressing final terminology concerns, the `git last-modified` command receives final approval after six iterations, and a major performance optimization for remote ref operations shows dramatic speed improvements. Meanwhile, multiple refactoring efforts modernize string handling across the codebase.

## Notable threads

### Promisor-remote protocol extension reaches v7

Christian Couder's series extending the promisor-remote protocol with configurable field validation has reached v7 with all technical issues resolved. The implementation now includes:

- Server-side advertisement of additional fields via `promisor.sendFields`
- Client validation through `promisor.checkFields`
- Case-sensitive protocol comparisons with case-insensitive config handling
- Comprehensive test coverage and documentation

The only remaining discussion point is whether to use "field" or "configuration variable" terminology in the interface. Junio Hamano has approved the parsing improvements while leaving the terminology question open for final resolution.

### `git last-modified` approved for merging

After six iterations, Toon Claes' `git last-modified` command has received final approval from Junio Hamano. The feature provides tree-level path tracking with Bloom filter optimization, showing 50.6% speedup for top-level checks. The series has been simplified by:
- Dropping the controversial `--format` option
- Consolidating the callback architecture
- Adapting to recent Bloom filter API changes

The implementation includes 19 test scenarios and performance benchmarks, with all major technical questions now resolved.

### Remote ref renaming sees massive performance gains

Patrick Steinhardt's optimization series for remote ref operations demonstrates dramatic improvements:
- Files backend: 238s → 2s (113x faster)
- Reftable backend: 8.6s → 1.2s (7x faster)

The v2 series restructures the transaction mechanism to use atomic operations and adds prefix-limited iteration. While introducing a minor regression for single refs with large reflogs (5.5ms→476ms), the overall wins are deemed substantial enough for inclusion.

## In brief

**String-list API refactoring** -- Junio Hamano's 11-part series modernizes string handling by replacing `strbuf_split*()` with `string_list_split*()`, reducing code while maintaining behavior across multiple subsystems.

**`core.commentChar=auto` deprecation** -- Phillip Wood's series to remove this problematic setting is complete, with Git 3.0 will error out while current versions show warnings and migration advice.

**`git repo info` format improvements** -- Lucas Seiki Oshiro's series adds NUL-terminated output support and refines the `references.format` field handling based on review feedback.

**Windows CRLF handling fix** -- Johannes Sixt corrects line-ending processing in interactive commands like `git reset -p` that showed corrupted output on Windows.

**Git-gui macOS modernization** -- The v2 series removes deprecated bundle infrastructure and improves askpass helper resolution using argv0-based path derivation.

**JSON output for Git commands** -- Ron Ziroby Romero explores implementation approaches for `git log --json`, noting Patrick Steinhardt's similar work on `git status --json`.

## On the radar

**Rust xdiff optimizations** -- Ezekiel Newren's series continues discussion about FFI type alignment benefits, showing how matched types enable zero-copy interop in the xxhash implementation.

**`git refs list` unification** -- The v4 series factors out common logic between `for-each-ref` and the new `refs list` command into shared infrastructure, awaiting final review.

**Git-secure proposal** -- An RFC for version 2 of this security wrapper expands command coverage and adds token validation, seeking feedback on integration approaches.