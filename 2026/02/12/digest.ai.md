Here's the daily digest for February 12, 2026:

**The day in brief.** A moderately active day with 87 emails across 23 threads, featuring significant progress on several major efforts including promisor remote enhancements, ODB abstraction cleanups, and UI improvements to `git history`. The most active discussions centered around Christian Couder's LOP series nearing completion and Patrick Steinhardt's ODB flag cleanups.

**Notable threads**

**LOP (Large Object Promisors) series finalization** -- Christian Couder's 9-patch v3 series implementing `promisor.storeFields` and `--filter=auto` for promisor remotes appears ready for merging after addressing all review feedback. The series enhances partial clone workflows by allowing dynamic filter updates from servers and secure storage of remote fields. Patrick Steinhardt provided extensive technical review throughout the series, with final refinements today focusing on API cleanliness and error handling.

**ODB flag type safety finalized** -- Patrick Steinhardt confirmed he'll proceed with Junio's recommendation to keep unsigned integers for ODB API flag parameters rather than converting to enum types. This resolves the last open design question in the object database abstraction effort after extensive discussion about compiler limitations and type safety trade-offs. The decision maintains consistency with existing code while avoiding what Junio characterized as "a cleanup that requires further cleanup."

**Parallel hooks design debate** -- Phillip Wood and Patrick Steinhardt raised important considerations about the parallel hooks implementation, questioning whether compile-time settings should override user configuration and suggesting an alternative design where each hook declares its parallel-safety via config. The discussion revealed tension between safety (conservative defaults) and flexibility (user control), with Adrian Ratiu open to alternatives that preserve safety while allowing selective parallelization.

**git am security documentation finalized** -- Kristoffer Haugsbakk's documentation warnings about patch roundtrip hazards were queued by Junio after final wording refinements. The patch adds explicit caveats to `format-patch-caveats.adoc` about how embedded diffs in commit messages can be misinterpreted during `git am` processing, complementing technical solutions under discussion.

**git history UI improvements** -- Patrick Steinhardt sent a 4-patch series improving the new `git history` command's user experience by moving validation earlier and renaming technical options (`--ref-action` becomes `--update-refs` with separate `--dry-run`). The changes prevent wasted user input when validations fail and make the interface more intuitive. Junio and D. Ben Knoble provided detailed reviews focusing on test behavior and documentation accuracy.

**In brief**

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Diffcore-break dangling pointer fix** -- Han Young addresses a segfault in blobless clones by fixing a dangling pointer issue in diffcore-break machinery.

**Pack-objects performance optimization** -- Phillip Wood's patch prevents unnecessary object fetches during geometric repacking of promisor packs, showing 2-16% speedups.

**String list sorting standardization** -- Amisha Chhajed's patches replace separate sort-and-deduplicate operations with `string_list_sort_u()` in sparse-checkout and help command output.

**Test modernization** -- Ashwani Kumar Kamal updates t9812 to use `test_path_is_file()` instead of raw `test -f` calls.

**On the radar**

**the_repository removal effort** -- Phillip Wood and Junio discussed config access validation during repository initialization, revealing architectural tensions around submodule operations that legitimately need to read config from non-`the_repository` instances.

**UTF-8 alias support** -- Jonatan Holmgren discovered a shell completion edge case with malformed `<alias>.command` syntax that will be addressed in v5, though the core functionality remains approved.

**git allow proposal** -- A new `.gitallow` file concept was floated to control which Git commands can run in a repository, though it's at an early exploratory stage.