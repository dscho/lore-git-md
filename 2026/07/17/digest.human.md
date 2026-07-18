## The day in brief

A busy day on the Git mailing list (101 emails, 22 active threads) saw **two major series land in `next`**, a **contentious design debate stall a high-profile feature**, and **multiple bugfixes and optimizations advance**. The standout developments:

- **`git cat-file --batch-command` remote object info** (Pablo Sabater) reached v19 and is **queued for `next`**, completing a GSoC project with robust security and protocol design.
- **`the_repository` removal for `trust_executable_bit`/`has_symlinks`** (Tian Yuchen) hit v7 and is **ready for inclusion**, resolving all prior feedback.
- **`git replay --linearize`** (Toon Claes) remains **on hold** after Junio and Elijah’s design debate over multi-branch behavior reached an impasse.
- **`git last-modified` Bloom filter optimization** (Toon Claes) posted v1, with **correctness bugs and test gaps** identified by Taylor Blau.
- **`git show-branch` slab conversion** (Gatla Vishweshwar Reddy) saw **cultural pushback** from Junio over AI-generated contributions, despite technical readiness.

---

## Notable threads

### `git cat-file --batch-command` remote object info (GSoC) **queued for `next`**
**Headline**: Pablo Sabater’s **19th iteration** of the `remote-object-info` series is now **technically complete** and **queued for `next`**, marking the culmination of a GSoC project that enables secure, metadata-only queries of remote objects (e.g., size) without downloading full objects.

### What’s new

- **v19** adds a **new commit** ensuring bare OID requests trigger an unconditional existence check on the server, avoiding lazy-fetching in partial clones while maintaining robustness.
- **Dynamic placeholder handling**: The client now silently omits unsupported placeholders (e.g., `%(objecttype)`) based on the server’s advertised capabilities, matching `for-each-ref` behavior.
- **Memory leak fix**: A leak in `transport-helper.c` was plugged, and the session-state bug (where default formats could leak into subsequent commands) was resolved.
- **Security**: Strict input validation (10,000-object batch limit, 8KB URL length cap) and bounds-checking in response parsing.

### Key technical details

- **Protocol**: The `object-info` capability (v2) advertises supported attributes (`size`, `type`) when `transfer.advertiseobjectinfo` is enabled.
- **Client-side**: The `remote-object-info` command in `git cat-file --batch-command` batches multiple OIDs into a single request, minimizing round trips.
- **Transport layer**: New `fetch_object_info` vtable entry implemented for smart transports and transport helpers.

### Why it matters

This series **closes a long-standing gap** in Git’s remote query capabilities, enabling tools like GitLab and Forgejo to fetch object metadata (e.g., size) without downloading full objects. The design is **extensible** (future support for `type` and other attributes) and **secure** (strict validation, dynamic allow-lists).

### Next steps

- **Minor nits** (documentation, translation context, alphabetical ordering) may be addressed in a final polish before merging to `master`.
- **Adoption**: GitLab and Forgejo are likely early adopters, given their use of `last-modified` and similar queries.

---

### `the_repository` removal for `trust_executable_bit`/`has_symlinks` **ready for inclusion**
**Headline**: Tian Yuchen’s **v7** of the series migrating `trust_executable_bit` and `has_symlinks` into `struct repo_config_values` is **uncontested and ready for inclusion**, completing the `the_repository` removal effort for these core index-related settings.

### What’s new

- **v7** keeps `ce_mode_from_stat()` as a `static inline` function in `read-cache.h` (addressing Junio’s design concern from v6) while adding a `struct repository *` parameter.
- **Mingw compatibility**: Refactored to use a compile-time macro (`platform_has_symlinks()`) and static function (`mingw_platform_has_symlinks()`), eliminating mutable global state.
- **Getter functions**: `repo_trust_executable_bit()` and `repo_has_symlinks()` now include repository pointer validation (though Junio’s v7 feedback questions whether this is necessary).

### Key technical details

- **Files touched**: `read-cache.c`, `environment.c/h`, `apply.c`, `compat/mingw.c`, and related headers.
- **Behavior**: No changes; this is a **mechanical refactoring** preserving existing functionality.
- **Performance**: `ce_mode_from_stat()` remains `static inline` to preserve potential benefits in hot paths.

### Why it matters

This series **demonstrates incremental progress** in the `the_repository` removal effort, establishing a **unified getter pattern** that may inform future configuration migrations. The mingw adjustments are particularly noteworthy, as they **eliminate mutable global state** while preserving native symlink detection via environment variables.

### Next steps

- **Junio’s stylistic feedback** (formatting, defensive programming) on patch 3/4 is **low-weight** and may be addressed in a follow-up.
- **Merge to `next`** is expected soon, with no substantive objections remaining.

---

### `git replay --linearize` **on hold** after design debate
**Headline**: Toon Claes’s **v7** of the `--linearize` option for `git replay` remains **on hold** after Junio and Elijah’s **design debate** over multi-branch behavior reached an impasse. The series is **technically complete** but blocked by a **philosophical disagreement** about user intent.

### The debate

- **Junio’s position**: The current behavior is **intentional and reasonable**. The user explicitly requests a linear history (`--linearize`) and updates multiple branches, so concatenating them into a single chain is the **direct result of combining these features**. The escape hatch (invoking `git replay` multiple times) is sufficient.
- **Elijah’s position**: The current behavior **violates the principle of least astonishment**, creates a **recoverability asymmetry** (untangling concatenated branches is harder than concatenating independent ones), and is an **emergent, unintended behavior** that neither feature implies. He proposes a **minimal fix**: restrict `--linearize` to a single positive ref.

### Key technical details

- **Behavior**: `--linearize` flattens all specified branches into a single linear history, concatenating them in an order determined by the revision walk (not the user’s command-line order).
- **Edge case**: Combining `--linearize` and `--revert` on a partial merge range (e.g., a range that includes a merge commit and one side of the merge but excludes the other) may not be handled reliably.

### Why it matters

This debate **highlights a tension** between flexibility and predictability in Git’s design. The outcome will set a precedent for how future commands handle **multi-branch operations** and whether **emergent behaviors** are acceptable if they align with user intent.

### Next steps

- **Toon must respond** to both positions, either:
  1. **Defending the current behavior** (aligning with Junio).
  2. **Adopting Elijah’s minimal fix** (restrict `--linearize` to a single positive ref).
  3. **Proposing a third alternative** (e.g., one of Junio’s syntax suggestions).
- Until then, the series remains **on hold in `next`**.

---

### `git last-modified` Bloom filter optimization **v1 posted, bugs identified**
**Headline**: Toon Claes posted **v1** of a four-patch series optimizing `git last-modified` by reusing Bloom filters, but **Taylor Blau identified correctness bugs and test gaps** that may invalidate the cover-letter benchmarks.

### What’s new

- **Patch 1/4**: Refactors `revision.c` to deduplicate a Bloom-key precondition check.
- **Patch 2/4**: Exposes a new helper function, `revs_maybe_changed_in_bloom`, to avoid redundant filter lookups.
- **Patch 3/4**: Wires the helper into `git last-modified`, cutting runtime by **3.7×** (52 ms → 14 ms) on the ziglang/zig repository.
- **Patch 4/4**: Extends the optimization to wildcard pathspecs (e.g., `*.c`).

### Bugs identified by Taylor Blau

1. **Patch 2/4**: Unintentionally changes Bloom filter statistics reporting (incrementing `count_bloom_filter_not_present` even when no key vectors exist).
2. **Patch 3/4**: **Correctness bug** when `--show-trees` is used. The pathspec does not cover all entries in `lm->paths` (which includes ancestor tree entries), so a Bloom filter that says “no changes” for the pathspec may still miss changes to those ancestor trees.
3. **Patch 4/4**: **Test coverage gap**. The existing test suite (`t8020`) does not exercise the Bloom filter code path because it never writes a commit-graph with `--changed-paths`, and the test cases use unquoted wildcards (e.g., `a/*`), which the shell expands into literal pathspecs.

### Why it matters

`git last-modified` is used by **GitLab and Forgejo** to batch-query last-modified commits for directory entries. The current implementation is **inefficient** (checking every path against every commit’s Bloom filter), and this series **aligns its performance with `git log`**. However, the **correctness bugs** must be fixed before the optimization can land.

### Next steps

- **Toon must address the bugs** in patches 2/4 and 3/4 and add test coverage for patch 4/4.
- **Follow-up**: The series may need a v2 to fix the issues before further review.

---

### `git show-branch` slab conversion **cultural pushback**
**Headline**: Gatla Vishweshwar Reddy’s **v9** of the `git show-branch` slab conversion is **technically ready**, but Junio **escalated a cultural critique** about AI-generated contributions, delaying integration.

### What’s new

- **v9** fixes a **NULL pointer dereference** in the reflog path and clarifies the intentional use of a `{}` block in `join_revs()` for C99 scoping.
- **Junio’s critique**: The patch exhibits signs of being **generated or submitted without proper human review**, citing two inconsistencies:
  1. The commit message claims to have removed the `max_revs` limit entirely, yet the documentation still mentions a 64-branch limit.
  2. The patch removes the local `UNINTERESTING` definition while leaving a comment advising future migration to the shared definition.

### Key technical details

- **Refactoring**: Replaces `object.flags` with a commit-slab (`commit_rev_flags`) using runtime-computed stride, removing the 27-branch limit.
- **Files touched**: `builtin/show-branch.c`, `Documentation/git-show-branch.adoc`, `t/t3202-show-branch.sh`.

### Why it matters

Junio’s critique **sets a cultural precedent** for contributions generated with AI assistance. The project **accepts AI tools for proofreading or drafting** but **rejects submissions where the human lacks domain knowledge** to critically assess the output. This thread may prompt **clearer guidelines** for AI-assisted contributions.

### Next steps

- **Gatla must demonstrate human understanding** of the code paths and fixes in the patch.
- **Junio’s technical concerns** (the two inconsistencies) are already addressed in v9, but the **cultural expectation** remains unresolved.

---

## In brief

- **`git stash rename`** (Emin Özata): **Stalled** after Junio questioned the practicality of rewording 211 stashes and suggested broader stash redesign (e.g., branch-tied stashes). The author must address both the **immediate justification** and the **maintainer’s design critique**.
- **`git bisect --auto-reset`** (Harald Nordgren): **v2 posted**, implementing the parameterized `--auto-reset=<where>` option (values `original` or `found`). The series is **ready for review**, with no substantive objections remaining.
- **`git history` signing** (Souma): **v2 posted**, teaching `git history` to sign rewritten commits. The series addresses Patrick Steinhardt’s v1 feedback (preparatory refactoring, documentation, tests) and is **under active review**.
- **Contributor documentation** (Junio): **v1 under review**, clarifying patch submission, review, and integration processes. Junio accepted Patrick Steinhardt’s suggestion to acknowledge **reviewer disinterest** as a possible reason for silence.
- **`gitweb` bugfix** (Travor Liu): **v2 posted**, fixing object ID rendering in commitdiff index lines. The series is **ready for integration**, with no open questions.
- **Trace2 timestamp bugfix** (Derrick Stolee): **Under review**, with Taylor Blau proposing an alternative fix targeting the `xsnprintf()`/`libintl` boundary. The discussion highlights a **platform-specific edge case** (Git for Windows).
- **`git repo info` path keys** (K Jayatheerth): **v2 posted**, adding seven new path-related keys (e.g., `path.toplevel.absolute`, `path.git-prefix`). The series is **mechanically clean** and awaits **substantive review**.
- **`git repack --drop-filtered`** (Siddharth Shrimali): **RFC v1 complete**, but Junio identified a **blocking logical flaw** in the `--drop-filtered` vs `--write-bitmap-index` validation. The author plans a v2 to **explicitly track command-line vs config-driven bitmaps**.
- **`wt-status` performance** (Sahitya Chandra): **v2 posted**, replacing O(n²) `string_list_insert()` calls with O(n log n) append-and-sort. The patch is **ready for integration**, with no substantive objections.
- **`git history fixup` worktree bug** (Toon Claes): **New bug report**, demonstrating that `git history fixup` corrupts the index in other worktrees. This is a **serious correctness issue** for multi-worktree setups.
- **Loose object backend refactoring** (Patrick Steinhardt): **9-patch series posted**, moving loose-object writing logic into the ODB abstraction layer. The series is **under review** and part of the **broader ODB effort**.
- **Swift userdiff driver** (Shlok Kulshreshtha): **Queued in `seen`**, awaiting validation from Swift users before progressing to `next`.
- **`git rebase -i` fixup edge cases** (Phillip Wood): **v1 posted**, fixing two edge cases with skipped fixup/squash commands. The series is **ready for review**, with thorough test coverage.

---

## On the radar

- **`git replay --linearize`**: The **design debate** between Junio and Elijah remains unresolved. Toon must respond to both positions before the series can progress.
- **`git last-modified` Bloom filter optimization**: **Correctness bugs** in patches 2/4 and 3/4 must be fixed, and **test coverage** for patch 4/4 must be added.
- **`git show-branch` slab conversion**: **Cultural pushback** from Junio may delay integration until the author demonstrates deeper understanding of the changes.
- **`git stash rename`**: The **justification and design critique** from Junio must be addressed before the patch can advance.