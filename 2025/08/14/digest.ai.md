# Git Mailing List Digest - 2025/08/14

**The day in brief.** A moderately active day with 38 emails across 13 threads, featuring final refinements to several in-flight series, technical discussions about SHA-1/SHA-256 interoperability, and documentation improvements. Key developments include consensus on the `git repo info` output behavior and architectural discussions about hash algorithm transition storage formats.

## Notable threads

### `git repo info` output behavior finalized

The long-running discussion about the new `git repo info` command's output format reached consensus today, with multiple contributors confirming the decision to preserve input order exactly as requested without any sorting or deduplication. Phillip Wood reinforced that this approach best serves scripting needs, particularly for Windows users where process overhead makes multiple invocations expensive. The thread saw test refinements from Lucas Seiki Oshiro and Eric Sunshine, with Junio Hamano weighing in on future-proofing considerations regarding reference backends. The series is now in 'next' with all major design questions resolved.

### SHA-1/SHA-256 interoperability storage format

brian m. carlson initiated a technical discussion about improving the storage mechanism for hash mappings between SHA-1 and SHA-256 objects. The current `loose-object-idx` approach doesn't scale well for certain cases like shallow clones. Junio Hamano engaged in detailed design discussion, proposing separation between static mapping data and dynamic metadata. The conversation explored tradeoffs between various storage formats (pack index extensions, reftable-like approaches) and lifecycle management through `git gc`. While no final decision was reached, the thread made clear progress toward a solution that balances lookup efficiency with update performance.

### Documentation linting tools approved

Junio Hamano gave final approval to the documentation linting series after successful testing in the 'seen' branch. The 6-patch series introduces automated checks for man page formatting consistency, covering linkgit macros, section delimiters, definition lists, and option formatting. Jean-Noël Avila confirmed agreement with the merge decision, marking the conclusion of this development effort. The tools will help maintain consistency across 117 documentation files as part of the broader documentation modernization effort.

## In brief

**Submodule hash algorithm fix** -- Michael Schroeder proposed a fix for shallow submodule clones when superproject and submodule use different hash algorithms, with Junio Hamano and brian m. carlson providing review feedback that revealed deeper architectural questions about hash algorithm compatibility.

**Rebase documentation refinements** -- Junio Hamano provided technical corrections to Julia Evans' rebase internals description, clarifying that commit enumeration and filtering happen in a single operation rather than separate steps.

**`git-jump` space handling** -- The thread finalized a solution for handling filenames with spaces in `git-jump` diff output, settling on a simple regex fix that addresses the common case while explicitly not handling rare edge cases with control characters.

**`git-add` glossary updates** -- Junio Hamano proposed terminology improvements to the Git glossary, better defining "index", "staging area", and the verb "to stage" to match actual command usage.

**Abbreviation length fix merged** -- Junio Hamano incorporated the final refinements to fix object name abbreviation handling, removing hash-length comparisons in favor of NUL termination checks.

**`git-count-objects` documentation** -- Daniele Sassoli's patch documenting the "packs" line in verbose output completed review and was queued for merging after addressing style feedback.

## On the radar

**Hash algorithm storage format** -- The discussion about SHA-1/SHA-256 interoperability storage continues to evolve, with clear technical alternatives emerging but no final decision yet reached. This foundational work will affect Git's ability to handle repositories with mixed hash algorithms.

**Submodule hash handling** -- The submodule discussion took an unexpected turn when brian m. carlson pointed out the current approach only works in one direction, raising questions about whether mixed hash algorithm support is feasible at all without proper conversion.