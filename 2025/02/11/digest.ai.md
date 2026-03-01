# Git Mailing List Digest — 2025/02/11

**The day in brief.** A moderately busy day with 66 emails across 26 threads, featuring significant progress on several fronts. Key developments include the finalization of a `--skip-reflog` optimization for reference migrations, a major revision of the pickaxe long-options series, and ongoing design discussions about `git clean` exclusions and rebase ref updates. Junio Hamano was particularly active today, weighing in on multiple threads with maintainer guidance.

## Notable threads

### Pickaxe long options reach v4

Illia Bobyr's series adding long option names (`--patch-grep` for `-G` and `--patch-modifies` for `-S`) saw significant restructuring in its fourth iteration. The 10-patch series now cleanly separates documentation fixes, core implementation, and test updates. Junio Hamano provided submission process feedback, noting the patches didn't apply cleanly to expected bases and emphasizing the importance of clear thread navigation. The technical implementation appears sound, with comprehensive test coverage verifying both short and long forms work identically. Documentation now consistently shows both variants, and bash completion support has been added.

### Rebase empty commit rewording fixed

Phillip Wood's v2 patch fixing interactive rebase's handling of empty commits during reword operations received maintainer approval. The fix ensures `--allow-empty` is properly passed when rewording fast-forwarded empty commits, with special handling for merge commits that don't require the flag. Junio Hamano approved the changes after the v2 iteration added requested documentation about the flags handling pattern. The patch now moves to the 'next' branch, resolving an edge case that could cause rewording to fail unnecessarily.

### Partial reference transactions face design critique

Phillip Wood raised fundamental concerns about Karthik Nayak's series introducing partial reference transactions. The critique challenges the series' atomicity claims and questions whether modifying transaction semantics is the right approach for performance optimization. Wood suggests a separate batching interface might be more architecturally sound than blending performance concerns with transaction guarantees. The discussion has shifted from implementation details to core design questions, potentially requiring significant reevaluation of the approach.

### `--skip-reflog` optimization finalized

Karthik Nayak's performance optimization for reference backend migrations reached its final form, adding a `--skip-reflog` flag to bypass costly reflog migration steps. The patch targets server repositories where reflogs exist but aren't needed, following multiple rounds of review that refined the message and test robustness. Junio Hamano provided final stylistic feedback about patch presentation and shell script details, but the technical implementation is sound and ready for inclusion after addressing all substantive concerns.

## In brief

**Test framework conversion:** Seyi Chamber provided a progress update on converting Git's test infrastructure to the clar unit test framework as part of an Outreachy internship, documenting experiences in a blog post.

**GSoC 2025 organization:** Kaartic Sivaraam updated documentation to reflect Patrick Steinhardt's preference for prioritizing ref consolidation over HTTP reftables support in the upcoming Google Summer of Code.

**Thunderbird patch script:** A brief discussion concluded about stylistic choices in the POSIX-compliant rewrite of `contrib/thunderbird-patch-inline/appp.sh`, with Junio Hamano preferring `printf '%s\n'` iteration over explicit formatting.

**Merge-recursive optimization:** Meet Soni proposed replacing O(n²) `string_list_insert()` calls with O(n log n) `string_list_append()` plus sort in merge-recursive's rename processing, receiving positive feedback from Elijah Newren with minor documentation suggestions.

**Boolean documentation:** Junio Hamano standardized boolean value syntax documentation across man pages, making git-config(1) the canonical reference, with Eric Sunshine providing a minor grammatical correction to the commit message.

## On the radar

**`git clean` exclusions:** The discussion about persistent exclusion patterns continues to evolve conceptually, with Junio Hamano pushing for clearer terminology and potentially broader "precious files" integration beyond just `git clean`. The thread shows no clear resolution yet on whether to use config-based storage or a separate file mechanism.

**Rebase `--update-refs` interactive mode:** Ivan Shapovalov's feature faces ongoing review, with Phillip Wood questioning the need for a CLI option and raising enum safety concerns. The documentation format is now settled, but core implementation decisions remain pending before testing can begin in earnest.