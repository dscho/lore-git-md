# Git Mailing List Digest - March 22, 2026

**The day in brief.** March 22 saw moderate traffic with 57 emails across 18 threads, featuring ongoing architectural discussions, test modernizations, and several patch series nearing completion. Key developments include continued refinement of the `--graph-lane-limit` feature, resolution of a remote-curl segfault issue, and progress on promisor file handling during repacks.

## Notable threads

### Graph column limiting reaches v3

Pablo Sabater's `--graph-lane-limit` patch series for constraining `git log --graph` output width has progressed to v3 with ongoing design discussion. The feature addresses terminal readability issues by truncating excess graph lanes while preserving commit markers. The latest iteration responds to Junio Hamano's feedback with an option rename (from `--max-columns`) and implicit `--graph` enablement. Johannes Sixt raised new concerns about the implicit behavior potentially complicating future configuration, reopening discussion about the UX implications. The series appears well-constructed but awaits resolution of these design questions before final review.

### Remote-curl segfault fix converges

A segfault in `git-remote-http` when parsing refspecs outside a repository context has been addressed through collaborative analysis. K Jayatheerth's v3 patch skips hash-algorithm checks when `the_repository` is NULL, following Jeff King's identification of the root cause. The solution emerged from discussion involving Junio Hamano and brian m. carlson about the architectural implications for hash algorithm handling in repository-less operations. The patch now includes a simplified test case using Git's `nongit` helper and appears ready for merging after addressing Junio's minor presentation feedback.

### Promisor file handling improvements

Lorenzo Pegorari's GSoC work on promisor file handling during repacks has progressed to v2 with significant implementation improvements. The series addresses a `NEEDSWORK` comment about preserving ref-to-packfile mappings, now using Git's strset for O(1) duplicate checking (replacing an O(n²) approach) and adding test coverage. Eric Sunshine's thorough review of v1 led to fixes for memory leaks and coding style violations. The changes demonstrate good responsiveness to feedback while maintaining focus on the core goal of properly handling debug information in `.promisor` files during repacks.

### Backfill argument validation refined

Siddharth Shrimali's patch to make `git backfill` reject unexpected arguments reached v3 after addressing Junio Hamano's feedback about error formatting and test coverage. The change brings the command in line with Git's standard behavior of erroring on invalid input. Phillip Wood suggested using `test_grep` for better failure diagnostics, which Junio endorsed. However, Derrick Stolee later raised concerns about potential conflicts with planned revision argument support, temporarily putting the change on hold pending design clarification.

### Strbuf optimization discussion concludes

The strbuf_getwholeline() optimization thread reached its conclusion with Junio Hamano and Jeff King agreeing that no further changes are needed beyond the original goal of NULL check removal. The discussion had explored performance improvements, memory safety, and behavioral consistency between getdelim() and non-getdelim() implementations. Junio clarified his participation was solely to enable a Coccinelle rule simplification, exemplifying Git's preference for targeted, minimally invasive changes to stable code.

## In brief

**Test modernization** -- Multiple contributors replaced raw `test -f` checks with `test_path_is_missing` across several test scripts, continuing the project's standardization effort. Jayesh Daga and others addressed incorrect helper usage while improving test framework setup.

**Diff-highlight improvements** -- Jeff King's series enhancing the Perl script saw review feedback about edge cases in config lookup optimization, with plans for a v4 addressing test chaining robustness.

**Branch flag validation** -- A patch rejecting `--show-current` with `-v` in `git branch` was put on hold after Junio questioned whether the flags could meaningfully work together rather than being incompatible.

**MacOS build fix** -- René Scharfe observed that a proposed REG_ENHANCED workaround may no longer be needed as CI began passing without it, suggesting the underlying platform issue resolved itself.

**STRBUF_INIT_CONST RFC** -- Mateo Patino inquired about reviving this stalled optimization, with Eric Sunshine providing historical context about fundamental API compatibility issues that previously blocked it.

## On the radar

**Geometric repacking fixes** -- Taylor Blau's series addressing MIDX bitmap generation with excluded-but-open packs awaits resolution of a subtle NULL pointer issue in the object-walking logic.

**The_repository removal** -- Shreyansh Paliwal confirmed their `git add -p` patch is queued, while Junio prefers the older version for its cleaner dependency graph despite a newer iteration existing.

**GSoC project coordination** -- Karthik Nayak redirected a contributor to join existing design discussions rather than submitting standalone patches, highlighting the program's structured onboarding process.