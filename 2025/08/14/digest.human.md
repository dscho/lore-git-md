# Git Mailing List Digest - 2025/08/14

**The day in brief.** A moderately busy Thursday with 38 emails across 13 threads, featuring final refinements to several in-flight series and substantive discussions about hash algorithm interoperability. Key developments include final approvals for the `git repo info` command and documentation linting tools, plus architectural debates about SHA-1/SHA-256 mapping storage and submodule handling.

## Notable threads

### `git repo info` output ordering finalized

The long-running discussion about the new `git repo info` command's output behavior reached consensus today, with Phillip Wood confirming the decision to preserve input order exactly (without sorting or deduplication) for scripting predictability. Lucas Seiki Oshiro and Eric Sunshine refined the test suite to verify this behavior, including handling duplicate field requests. Junio Hamano raised future-proofing considerations about ref backend assumptions in tests, but the series appears ready for merging after addressing these final details. This concludes weeks of discussion about replacing `git rev-parse` use cases with a more structured alternative.

### Documentation linting tools approved

Junio Hamano gave final approval to Jean-Noël Avila's documentation linting series after successful testing in the 'seen' branch. The 6-patch set introduces automated checks for man page formatting consistency across 117 files, enforcing rules for linkgit macros, section delimiters, and option formatting. Minor conflicts with in-flight topics were deemed manageable, marking the culmination of three iterations with feedback from multiple reviewers. This tooling will support Avila's ongoing synopsis conversion effort while preventing formatting regressions.

### SHA-1/SHA-256 storage format debate

brian m. carlson and Junio Hamano engaged in a detailed technical discussion about improving the storage format for hash algorithm interoperability mappings. The current `loose-object-idx` approach scales poorly for edge cases like shallow clones. Proposals include extending pack index v3 format versus creating a reftable-like solution, with Junio advocating for separating static mappings from dynamic metadata. The conversation revealed fundamental challenges in bidirectional lookup efficiency and update performance, with no immediate resolution but clear technical progress toward a solution.

### Submodule hash algorithm mismatch

Michael Schroeder proposed a fix for shallow submodule clones when superprojects and submodules use different hash algorithms, but the solution sparked deeper architectural questions. Brian M. Carlson identified that the patch only handles SHA-256→SHA-1 cases while risking data corruption in the reverse scenario. This shifted the discussion from implementation details to whether Git should support mixed hash algorithms in submodules at all, given tree object incompatibilities. Junio's review also noted terminology and implementation elegance concerns in the original patch.

## In brief

**xdiff optimization licensing clarified** -- Junio Hamano confirmed Alexander Monakov's C-based xdiff string hashing optimizations are welcome and properly licensed, removing two potential blockers for the performance work.

**`--max-depth` for diff-tree** -- Junio checked on the status of Toon Claes' series adding depth limiting to git-diff-tree, which appears ready for post-2.51 consideration after addressing review feedback.

**`git-jump` space handling** -- Final agreement reached on Jeff King's fix for space-containing filenames in git-jump, consciously excluding edge cases with quoted control characters for simplicity.

**Abbreviation length fix** -- Junio applied the final simplification to the object name abbreviation fix, removing hash-length comparisons entirely in favor of NUL termination checks.

**`git-add` documentation** -- Junio provided final review feedback on Julia Evans' glossary terminology patch, clarifying "index" vs "staging area" definitions to match actual command usage.

**`count-objects` docs** -- Daniele Sassoli's straightforward patch documenting the "packs" line in verbose output completed review and was queued for merging after style refinements.

## On the radar

**Rebase documentation** -- Julia Evans' rebase man page improvements are undergoing final technical accuracy reviews from Junio, particularly around commit selection internals.

**Conceptual model discussions** -- Emerging conversations about Git's index design and snapshot model, sparked by documentation work, may inform future explanatory content.