# Git Mailing List Digest - December 19, 2025

**The day in brief.** A busy Friday with 65 emails across 22 threads, dominated by technical discussions around the proposed `git-history` command and a new `--no-indicators` diff option. Junio's "What's cooking" report provides a comprehensive snapshot of the current development landscape, while several documentation and test fixes move toward completion.

## Notable threads

### `git-history` command design evolution

Patrick Steinhardt's RFC for a new `git-history` command saw significant discussion today, particularly around its branch handling behavior during history rewriting operations. The thread reached a key compromise point where Steinhardt proposed implicitly defining the affected commit range as everything between the target commit and HEAD, similar to `git-replay`'s behavior. This represents a shift from his earlier "current-branch-only" stance toward the automatic multi-branch handling preferred by many reviewers.

However, Elijah Newren strongly challenged this approach, arguing it creates a "principle of least astonishment" violation by potentially leaving some dependent branches unmodified while others are rewritten. The discussion revealed fundamental philosophical differences - Steinhardt prioritizes UI simplicity while Newren emphasizes behavior consistency. Technical concerns about conflict resolution in complex branch topologies (raised by SZEDER Gábor) further complicate the design, suggesting the series may need additional refinement before reaching consensus.

### `--no-indicators` diff option debate

A proposal to add a `--no-indicators` flag to `git diff` and `git show` generated extensive discussion about whether the feature justifies core implementation. Harald Nordgren argued it would improve copy-paste workflows by removing +/- indicators while preserving colors, particularly for React/TypeScript development. However, multiple contributors (including Junio Hamano) questioned the necessity, suggesting alternatives like:

- Terminal column selection capabilities
- `git restore -p` for version recovery
- `git show` with temporary files
- Existing `--output-indicator-*` options
- Simple `sed` post-processing

The thread revealed a disconnect between the proposed solution and Git's existing capabilities, with the maintainer expressing skepticism about adding specialized features that can be composed from existing primitives. While Nordgren provided concrete use cases, they didn't address fundamental concerns about edge cases (newline markers, space handling) or engage substantively with the alternative workflows proposed.

### ANSI escape sequence security discussion

Junio Hamano weighed in on the ongoing security discussion about ANSI escape sequence handling in Git's sideband channel, proposing to move the configuration mechanism from the HTTP-specific layer to a more general transport layer. His suggestions for naming schemes like `sideband.allowEscapeSequences` reflect broader design considerations about where this security boundary should live in Git's architecture.

The email also revisited the security model debate, with Hamano challenging Johannes Schindelin's position that remote servers don't inherently become more trustworthy after initial interaction. This philosophical divide remains unresolved, with Hamano advocating for a trust-based graduated model while the current implementation favors strict defaults. Technical questions about ISO/IEC 2022 encoding and transport layer placement remain open but are considered low-priority given current usage patterns.

## In brief

**Windows symlink path length limits** -- Johannes Schindelin and Patrick Steinhardt agreed on a Windows-specific practical upper bound of 32,768 characters for symlink paths, balancing security against real-world usage needs.

**`git reset` documentation finalized** -- D. Ben Knoble shepherded Julia Evans' documentation improvements through final revisions, now merged with Junio Hamano's technical refinements.

**Bundle-URI validation fixes** -- Sam Bostock's v2 patch series now explicitly marks missing URIs in debug output while maintaining strict validation checks.

**Worktree documentation clarity** -- A documentation series standardized terminology around "missing" worktrees for the `--expire` option, incorporating Eric Sunshine's suggestion to use "prune" terminology.

**Curl 8.18.0 test compatibility** -- Jeff King confirmed test fixes for curl's stricter HTTP header handling are working with Daniel Stenberg's upstream changes.

**Rust build system fix** -- D. Ben Knoble's BSD/macOS compatibility fix for the Rust build system received final approvals.

**Duplicate free in path cache** -- Junio queued a straightforward fix for a duplicate `FREE_AND_NULL` line in repository path cache cleanup.

## On the radar

**Object handling performance regression** -- Junio's minimal fix is queued while Patrick Steinhardt prepares a more comprehensive rebased series addressing the root cause.

**Maintenance config file proposal** -- Matthew Hughes' RFC for `maintenance.configFile` received detailed technical review from Junio, questioning whether maintenance-specific config handling is the right approach.

**"What's cooking" highlights** -- Notable topics in Junio's status update include Windows symlink support, sideband control character handling, and ongoing MIDX incremental repacking work.