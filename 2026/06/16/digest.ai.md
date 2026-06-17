# The Git Project Mailing List Daily Digest

**2026/06/16 in brief**
A busy day with 94 emails across 26 threads. The standout developments: Junio Hamano acked Johannes Schindelin's Windows large-object series, clearing the path for merging; Patrick Steinhardt's `git refs` consolidation effort faced substantive design feedback from Junio; and the `git replay --linearize` series was merged to 'next'. Several long-running efforts reached key milestones, while new architectural discussions emerged around reference manipulation and AI attribution.

---

## Notable threads

### Windows large-object support reaches merge readiness
**Windows: large object support (LLP64)** -- Johannes Schindelin's 7-patch series converting object size handling from `unsigned long` to `size_t` received Junio Hamano's ack for the delta-handling patch (2/7). The series, which has been production-validated in Git for Windows for years, systematically addresses 32-bit truncation issues on Windows where `size_t` is 64-bit but `unsigned long` is 32-bit. Junio's "Makes sense" ack signals the series is on track for merging, with only routine integration adjustments remaining.

The implementation touches 72 files across core functionality (object handling, packfiles, deltas, ODB layer) and includes comprehensive test coverage for 5GB+ objects. Patrick Steinhardt's earlier review identified cleanup opportunities in `cat-file.c`, which were incorporated in v2. The series represents the culmination of Philip Oakley's original LLP64 compatibility work and complements ongoing large-file handling improvements.

---

### Reference command consolidation faces design scrutiny
**refs: add delete, update, and rename subcommands to git refs** -- Patrick Steinhardt's 4-patch series adding reference-writing capabilities to `git refs` received substantive design feedback from Junio Hamano. The maintainer raised high-weight concerns about the lack of atomic "create if not exists" semantics in the `update` subcommand, suggesting either sentinel values (e.g., `""` for `<oldoid>`) or a dedicated `create` subcommand.

Junio also flagged naming inconsistencies (`<oldvalue>` vs `<old-value>`), error message formatting (quoted vs unquoted object IDs), and edge-case behavior for `rename` (reflog migration when the source lacks a reflog). The series aims to consolidate reference manipulation functionality previously scattered across `git-update-ref`, `git-symbolic-ref`, and other commands, but Junio's feedback suggests a v2 iteration will be needed to address the atomicity and consistency concerns.

---

### Rebase squash feature debates architectural direction
**rebase: add --squash option** -- Harald Nordgren's proposal to add a `--squash` option to `git rebase` faced decisive architectural challenges. Phillip Wood identified efficiency concerns with the current implementation (picking commits one-by-one risks repeated conflict resolution) and proposed a three-way merge approach for the entire commit range. Meanwhile, Patrick Steinhardt endorsed D. Ben Knoble's alternative of implementing `git history squash`, which would offer more flexible range selection and fit naturally into the `git history` roadmap.

The thread highlights the tension between targeted optimizations and general solutions, with the `git history` approach gaining momentum as a more extensible foundation. The rebase implementation's future now hinges on whether it can address Phillip's efficiency concerns or whether the feature will migrate to the `git history` command.

---

### SHA-1/SHA-256 interoperability reaches milestone
**SHA-1/SHA-256 interoperability** -- brian m. carlson announced that the core functionality of the SHA-1/SHA-256 interoperability work is now working with a passing test suite. The branch implements protocol extensions for shallow clones, submodule handling, and partial clones, with recursive delta resolution in `git index-pack` reworked iteratively. Junio Hamano raised security concerns about packfile URIs (which deliver multiple packs that are only complete in aggregate) and large-object promisors (requiring the server to possess the entire history for object mappings).

While the branch is not targeted for Git 3.0 due to its 200+ patch volume and testing requirements, the milestone represents significant progress on a long-running effort. The work remains available for testing but unstable, with no `Reviewed-by` or `Acked-by` tags yet.

---

## In brief

**`git cat-file --batch-command` security series** -- Pablo Sabater's 12-patch GSoC series implementing dynamic capability-based validation for remote object metadata queries received final technical clarification. Junio Hamano identified and then retracted a type-safety warning after discovering the problematic `size_t *sizep` field was introduced by a separate in-flight topic (Dscho's work), which will be resolved as an evil merge.

**`git branch --delete-merged` series** -- Harald Nordgren's 15-iteration series implementing safe automated branch cleanup faced final review feedback from Phillip Wood. The review focused on API consistency (replacing a boolean `dry_run` parameter with flags) and test design improvements (repository reuse, consolidated assertions). The series is functionally complete but requires a v16 iteration to address these polish items.

**`git replay --linearize` series** -- Toon Claes's 3-patch series implementing `--linearize` for `git replay` was merged to 'next' after incorporating all review feedback. The feature flattens merge commits into linear history by walking only first parents, providing server-side functionality equivalent to `git rebase --no-rebase-merges`. The series includes comprehensive test coverage for root commits, `--linearize`/`--advance` combinations, and patch preservation verification.

**`git history` path formatting series** -- K Jayatheerth's GSoC series standardizing path formatting in `git repo info` reached final design resolution. Phillip Wood's concern about the `PATH_FORMAT_DEFAULT` API wart was addressed by removing it from the shared API and handling the backward-compatibility quirk locally in `rev-parse.c`. The series is now ready for merging.

**`git fetch.followRemoteHEAD` series** -- Matt Hunter's 7-patch series implementing a global `fetch.followRemoteHEAD` configuration received final review from Junio Hamano. The maintainer raised a concern about configuration value validation (silently ignoring invalid values) but did not block merging, suggesting a follow-up patch to improve error handling consistency.

**ODB abstraction series** -- Patrick Steinhardt's 17-patch ODB abstraction series received surface-level mechanical reviews from Justin Tobler. The series eliminates global state in setup.c and converts packed object storage to `struct odb_source`, but remains blocked by fundamental refdb issues requiring an 11-patch fix series.

**Documentation updates** -- Kristoffer Haugsbakk's 6-patch series updating `SubmittingPatches` with trailer conventions faced ongoing debate about non-identity trailers. Patrick Steinhardt requested concrete examples, while Kristoffer argued such trailers are rare in Git's history and mostly relevant to downstream projects. The series also addressed trailer ordering (chronological sequence) and DCO sign-off requirements for `Based-on-patch-by:`.

---

## On the radar

**AI attribution trailers** -- Marius Spix's proposal to adopt an `Assisted-by:` trailer for AI-assisted commits faced pushback from Kristoffer Haugsbakk. The discussion highlighted Git's historical resistance to hardcoding project-specific trailers (citing a 2013 `Fixes:` patch rejection) and questioned whether `Assisted-by:` should be treated as a "taint" marker. No consensus has emerged, but the thread reflects growing community interest in formalizing AI attribution practices.

**Rebase abort hints** -- Harald Nordgren's patch adding `--abort` hints to rebase failure and status messages faced workflow concerns from Phillip Wood. The reviewer argued that exec failures (typically minor issues like test failures) should not suggest aborting, as continuing is usually preferable. Junio Hamano framed the hint as a harmless "escape hatch" for new users, leaving the discussion unresolved.

**Notes handling during rebase** -- Uwe Kleine-König's bugfix patch addressing incorrect note copying during rebase received initial review. The patch adds a `dropped_commit` output parameter to `do_pick_commit()` to skip note copying when commits are dropped. Phillip Wood suggested the approach, but no substantive review has occurred yet.