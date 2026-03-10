Here's the daily digest for February 21, 2026:

## The day in brief

A moderately busy day with 45 emails across 21 threads, featuring several long-running efforts reaching completion. The most notable developments include the finalization of configurable branch comparisons in `git status`, progress on Linux fsmonitor support, and design discussions around cover letter formatting. Junio's "What's cooking" report provides a comprehensive snapshot of the project's current state.

## Notable threads

**Configurable branch comparisons ready for merge**  
Harald Nordgren's 28-iteration effort to add `status.compareBranches` configuration is finally ready for merging. The feature allows showing comparisons against multiple branches in `git status` output, initially supporting only `@{upstream}` and `@{push}` syntax. Jeff King provided foundational changes to `repo_dwim_ref()`, and the implementation includes extensive test coverage (337 lines in t6040-tracking-info.sh). Junio had initially missed the topic in his integration branches but now solicits final confirmation of real-world usefulness before merging.

**Linux fsmonitor nears completion**  
Paul Tarjan's inotify-based Linux fsmonitor implementation (now at v4) has proven stable in production use for two months. The patch addresses the last remaining issues - a 512-byte startup leak and 40-byte IPC leak - with Junio indicating these may not block merging. Patrick Steinhardt's meson build support will be split into a separate patch to unblock the core functionality. This completes fsmonitor backend support across all major platforms after multiple iterations addressing GPL concerns and memory safety.

**Symbolic ref namespace security debate**  
The thread about fixing NULL dereferences when handling symbolic refs crossing namespace boundaries has reached an impasse. While Troels Thomsen's fix works technically, Junio Hamano argues it violates namespace isolation principles by allowing cross-namespace symrefs. Junio now suggests explicitly rejecting such updates as security violations, representing a policy shift from the permissive approach in the original patch. The discussion highlights fundamental tensions in Git's namespace design without yet proposing concrete implementation changes.

**Cover letter format design evolves**  
Mirko Faina's RFC for improving `git format-patch` cover letters has evolved from a simple numbered list proposal to a flexible configuration-based approach. Junio suggests a `format.commitListFormat` setting with printf-style templates, automatically prepending `[N/M]` series position markers. While agreeing on the configurable approach, they debate command-line override syntax, with Junio emphasizing the need for temporary format changes. The thread moves toward implementation with only interface details remaining unsettled.

**Repository abstraction advances**  
Elijah Newren's series removing `the_repository` from merge-ort and replay reaches v3, now properly handling prefetch logic with `opt->repo` instead of falling back to the global. The six-patch series methodically eliminates repository globals, adding compile-time enforcement via `DO_NOT_USE_THE_REPOSITORY` macros. Patrick Steinhardt's feedback has been fully addressed, making this a significant step in the ongoing repository parameterization effort.

## In brief

**Histogram diff edge case fix** -- Yee Cheng Chin confirms the v2 patch addressing shifted change groups in XDF_HISTOGRAM_DIFF is nearly ready, with improved test cases coming soon.

**Path normalization refactor** -- Pushkar Singh's v3 patch extracting `skip_slashes()` from `normalize_path_copy_len()` is merge-ready after commit message clarification.

**Config-based hooks progress** -- Adrian Ratiu confirms parallel hook execution will be handled in a separate series, keeping the current config hooks work focused on core functionality.

**Patch-id documentation** -- Kristoffer Haugsbakk's improved `git-patch-id` man page additions are confirmed ready for merge after thorough review from D. Ben Knoble.

**Build system fixes** -- D. Ben Knoble's v6 patch for `config-list.h` dependency tracking in both Makefile and Meson builds is technically complete, with only a minor `DEPFILE` conditional question remaining.

**Gitweb mobile improvements** -- Rito Rhymes' mobile CSS and viewport tag changes are merge-ready after addressing Eric Sunshine's feedback about organization.

**String list standardization** -- Elijah Newren and Amisha Chhajed's patches standardizing `string_list_sort_u()` usage are complete, though a test case had to be dropped due to build system limitations.

**Interactive patch navigation** -- Samuel Abraham's `--[no-]auto-advance` for interactive patch commands receives final ack from Junio, marking successful GSoC microproject completion.

**Send-email encoding prompt** -- Shreyansh Paliwal's first contribution changes the 8-bit encoding prompt to "[default: UTF-8]" to prevent misinterpretation as a yes/no question.

**Pack-redundant leak fix** -- A small but important fix plugs a memory leak in `add_pack()` when `open_pack_index()` fails.

## On the radar

**Symbolic ref policy** -- The namespace isolation debate may require PLC input if technical and security perspectives remain divided.

**Parallel hooks** -- Adrian Ratiu's upcoming series on parallel hook execution will build on the config hooks foundation now nearing merge.

**Documentation conversion** -- Jean-Noël Avila's synopsis-style man page conversion continues progressing through the documentation files.