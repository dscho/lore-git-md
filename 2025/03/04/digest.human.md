# Git Mailing List Digest - 2025/03/04

## The day in brief

A busy day with 98 emails across 27 threads, featuring the Git 2.49.0-rc1 release announcement, continued progress on the `the_repository` removal series, and several test modernization efforts. Key highlights include a resolved discussion about `git add -p` hunk splitting behavior and a new bug report about `git apply` handling BSD-style patch markers.

## Notable threads

### Git 2.49.0-rc1 released

Junio Hamano announced the first release candidate for Git 2.49.0, containing 367 non-merge commits from 68 contributors. The release introduces the new `git backfill` command for blobless clones, improves shallow clone support, and includes various performance optimizations. Todd Zullinger reported a test failure on s390x architecture related to the new backfill command's batch fetching behavior, which the maintainers are investigating.

### `git add -p` hunk splitting finalized

Phillip Wood confirmed he'll update his patch series to use the existing `WITH_BREAKING_CHANGES` compile-time switch for modifying hunk states during splitting. The change makes split hunks transition from "selected" to "undecided" state, addressing a long-standing UX issue. Two follow-up improvements (hunk state visibility and quit prompts) were discussed but will be handled separately in future work.

### `the_repository` removal progresses

Patrick Steinhardt's 12-part series to eliminate the `the_repository` global variable saw significant review activity today. Justin Tobler provided feedback on several patches, including the conversion of pack-writing functions (03/12), core.bigFileThreshold config (04/12), and object conversion logic (08/12). The series is methodically converting subsystems to explicit repository parameters while maintaining existing functionality.

### `git apply` BSD patch marker bug

A new bug report highlighted `git apply` incorrectly handling BSD-style "No newline at end of file" markers in patch files. Unlike GNU diff which only includes these markers at file endings, BSD diff includes them per hunk, causing `git apply` to incorrectly strip newlines from each marked line rather than just at file endings. The issue affects interoperability when applying patches generated on BSD/MacOS systems.

### Test modernization efforts

Multiple test modernization threads saw progress:
- Seyi Kuforiji submitted a 2-patch series converting trailer and URL normalization tests to the Clar framework
- Mahendra Dani's patch to modernize t1403's `remove_object` helper concluded after extensive discussion about `rm -f` behavior and test design
- Jeff King and Patrick Steinhardt continued discussing environment isolation for unit tests, debating the right balance between comprehensive control and implementation complexity

## In brief

Jeff King's zlib inflation series saw final stylistic discussions about error handling in `unpack_loose_rest()`, with Junio agreeing to maintain the `--literally` flag while rejecting unknown object types. The ref consistency checks series may defer its `git fsck` integration to next release due to Patrick Steinhardt's concerns about insufficient real-world testing. A memory leak fix for the reftable reader was refined to handle NULL pointer edge cases. Documentation discussions converged on using `WITH_BREAKING_CHANGES` (rather than the double-negative `!without-breaking-changes`) for conditional documentation of deprecated features.

## On the radar

The `git format-patch` subject prefix discussion continues exploring how to best indicate mentoring program participation, with Junio advocating for reusing existing `--subject-prefix` rather than adding new flags. A feature request proposes relaxing `.gitignore` pattern rules to allow simpler syntax for excluding directories while including specific subpaths. The meson build system discussion identified a fix for template directory path issues when runtime prefix is enabled. New contributor Hisham Hasan began onboarding for GSoC 2025, receiving guidance about microprojects and environment setup.