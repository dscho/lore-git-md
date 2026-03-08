# Git Mailing List Digest - 2025/11/24 (Monday)

**The day in brief.** A moderately busy Monday with 43 emails across 13 threads, featuring significant progress on the hook subsystem refactoring, documentation standardization, and several bugfix discussions. The standout developments include the completion of a major hook API modernization series and Junio's "What's cooking" report highlighting upcoming changes.

## Notable threads

### Hook subsystem modernization reaches completion

Adrian Ratiu and Emily Shaffer's 10-part series refactoring Git's hook subsystem reached its final iteration today, converting the last remaining hooks (receive-pack's update and post-update hooks) to use the new structured API. The series introduces:

1. A callback-based parallel execution API with stdin streaming capabilities
2. Output capture functionality for server-side hooks
3. Standardized hook execution patterns across all hook types
4. Memory safety improvements identified in earlier reviews

The implementation maintains backward compatibility while laying groundwork for future features like config-based hooks and parallel execution. Junio's positive review of the pre-push hook conversion suggests this major infrastructure change is ready for integration after addressing his pipe closure timing question.

### Documentation standardization completes for transport commands

Jean-Noël Avila's series standardizing the documentation for `git fetch`, `git pull`, and `git push` to consistent AsciiDoc synopsis style was finalized today with a v2 that fixed a minor typo. The changes:

- Convert all three commands' man pages to use `_<placeholder>_` syntax
- Standardize option presentation with monospace formatting
- Improve complex option descriptions via bulleted lists
- Update cross-references to match new style

Junio confirmed the corrected version is ready for merging, completing this phase of the multi-year documentation standardization effort. The purely presentational changes span 10 files and over 800 lines while maintaining all existing content.

### Cache-tree parsing safety discussion evolves

Jeff King and Junio Hamano continued their discussion about cache-tree parsing safety in the ASan hardening series, with Junio proposing a more fundamental solution - introducing a cache-tree version 2 format that would avoid text parsing entirely. The current patch (4/9 in the series) addresses memory safety but leaves some edge cases unhandled, prompting Junio to suggest:

1. A binary format (network byte order or varint) for future-proofing
2. Better validation of the current text format as an interim solution

The discussion reveals tension between immediate safety fixes and longer-term architectural improvements, with Junio leaning toward the latter despite acknowledging no major deficiencies in the current format beyond parsing edge cases.

### `git clone` config parsing regression sparks debate

A reported regression in `git clone`'s config parsing (removing whitespace-trimming around keys) evolved into a broader discussion about intended behavior. Jeff King provided historical context showing the removed behavior:

- Originated in 2010 code
- Was already partially broken by 2021 changes
- Conflicts with Git's config key format rules

Junio noted the documentation specifies `-c` should take `<key>=<value>` as a separate argument, suggesting the "regression" might actually correct undocumented behavior. The thread is converging toward accepting the change as a bugfix rather than regression, though documentation inconsistencies remain to be addressed.

## In brief

**Hook subsystem refactoring v3** -- Adrian Ratiu and Emily Shaffer's 10-part series converting all remaining hooks to the new API addresses memory leaks and review feedback, with successful CI runs indicating readiness for final review.

**ASan hardening series** -- Jeff King considers Junio Hamano's more thorough cache-tree parsing validation approach while weighing performance and strict_string_checks compatibility.

**GIT_REF_URI implementation** -- Karthik Nayak addresses review feedback by clarifying documentation and explaining why repository format tracking isn't updated for temporary overrides.

**`git config set` error fix** -- Corrects misleading suggestions about `--add` in multi-value config error messages, properly recommending `--value`, `--append`, or `--all`.

**Optional path segfault fix** -- Junio agrees to backport D. Ben Knoble's fix for crashes when handling `:(optional)` paths to the 2.52.1 maintenance release.

**`git instaweb` path handling** -- Fixes `@` character interpolation in Perl config generation while raising broader security concerns about executable config files.

**`git config unset` help text** -- René Scharfe corrects misleading descriptions of `--all` and `--value` flags, with Junio queuing the straightforward documentation fix.

## On the radar

**`git whatchanged` deprecation** -- Kristoffer Haugsbakk reinforced the project's stance on requiring `--i-still-use-this` despite user objections, suggesting this thread may conclude unless new arguments emerge.

**Secret management workflows** -- The discussion about Git filter limitations for secret restoration continues, with Junio providing additional technical context about why smudge filters can't access working tree files.