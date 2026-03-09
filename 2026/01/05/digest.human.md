# Git Mailing List Digest - 2026/01/05

**The day in brief.** A busy Monday with 89 emails across 36 threads, featuring significant progress on several fronts. The hook subsystem refactoring reached its final administrative approval, `git status` gained push tracking capabilities after 17 iterations, and Patrick Steinhardt advanced geometric repacking with promisor remotes. Documentation work continued with synopsis-style conversions and `git reset` man page refinements, while test modernization efforts saw steady progress.

## Notable threads

**Hook subsystem standardization complete**  
Adrian Ratiu's multi-year effort to standardize Git's hook infrastructure has cleared its final administrative step. The 13-part v6 series, which introduces structured stdin handling, output capture, and parallel execution capabilities while maintaining backward compatibility, received explicit approval from Junio Hamano ("I didn't see anything iffy") and Patrick Steinhardt. The work converts all major hooks (post-rewrite, pre-push, reference-transaction, and receive-pack) to use the new `hook.h` API, laying groundwork for future config-based hooks. With all CI passing and no outstanding objections, this architectural improvement is now ready for integration into `next`.

**`git status` push tracking merged**  
Harald Nordgren's 17-iteration series adding push remote tracking information to `git status` output was finalized today. The implementation shows divergence from both upstream and push tracking branches when they differ, with comprehensive test coverage (262 lines) verifying edge cases and custom refspec mappings. The change required careful refactoring of branch comparison formatting logic in `remote.c` to maintain existing behavior while adding the new functionality. Junio Hamano and Phillip Wood provided extensive review, with the final version addressing a subtle control flow regression in the refactoring that could incorrectly report divergence.

**Geometric repacking with promisor remotes**  
Patrick Steinhardt posted a 5-patch series fixing incompatibilities between geometric repacking and promisor remotes (used in partial clones). The changes allow combining `--stdin-packs` with `--exclude-promisor-objects` in `pack-objects`, refactor the repack geometry calculations for reuse, and implement proper promisor pack handling by maintaining separate geometric progressions. Test coverage in t5331 and t7703 verifies the new behavior, completing another piece of Steinhardt's ongoing repack machinery overhaul. The series appears well-constructed with thorough consideration of edge cases.

**`git reset` documentation refined**  
D. Ben Knoble shepherded the fourth iteration of documentation improvements for `git reset`, incorporating post-merge feedback from Jean-Noël Avila about formatting consistency. The v4 changes are purely stylistic - adding backticks around "HEAD" references and converting mode descriptions to imperative mood - building on substantive pedagogical improvements from earlier versions. The series originated with Julia Evans' work to make the command's behavior clearer, particularly around reset modes (`--soft`, `--mixed`, `--hard`) and pathspec handling, with Junio Hamano ensuring technical accuracy throughout.

**Ref storage backend selection advances**  
The ref backend abstraction effort progressed with discussion about worktree integration in Karthik Nayak's v3 patch series. Patrick Steinhardt noted that when a repository's main ref store location changes (e.g. to `/tmp/dir`), worktree ref stores should follow the same pattern. Nayak identified a chicken-and-egg problem with Git's directory detection logic that needs resolution before implementing this behavior. The thread has already established the config-based URI syntax via `extension.refStorage` with environment variable override capability, with worktree support being the final major piece before integration.

## In brief

**Reftable iterator fix** -- Tsahi Elkayam corrected undefined behavior in the indexed table reference iterator, with Patrick Steinhardt noting opportunities for future optimization in the object index design.

**French translation update** -- Jean-Noël Avila brought the French `.po` file up to date with current source strings.

**Test modernization** -- Multiple contributors replaced direct shell tests with helper functions in t1300, t1420, t2021, and t5403, improving failure diagnostics.

**Shallow clone ref-in-want fix** -- Matthew Dodd identified and fixed a protocol violation where upload-pack sent wanted-refs before shallow-info when `uploadpack.allowRefInWant` was enabled.

**`git subtree` regression analysis** -- Colin Stagner proposed removing a problematic heuristic in squash commit handling that caused failures with rebased histories.

**Linux fsmonitor meson support** -- Patrick Steinhardt added build configuration for the new Linux inotify backend to meson.build.

**`git add -p` initialization fix** -- SZEDER Gábor caught an uninitialized variable in the recently merged hunk decision display feature.

## On the radar

**`the_repository` removal considerations** -- Phillip Wood highlighted user experience regressions from lazy config parsing when moving "core.attributesFile" to repository settings, sparking discussion about command-specific validation needs.

**Bazel build proposal** -- Son Luong Ngoc floated adding Bazel support but met resistance from brian m. carlson over configurability concerns, suggesting the proposal needs refinement.

**`git whatchanged` deprecation** -- User reports confirmed `git log --raw` serves as an adequate replacement, validating the deprecation path.