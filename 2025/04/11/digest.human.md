# Git Mailing List Digest — 2025/04/11

**The day in brief.** A moderately busy day with 51 emails across 17 threads, featuring significant progress on several fronts. The object-file refactoring series reached completion, a new RFC for cruft pack optimization emerged, and multiple bug reports surfaced interesting edge cases. Junio's "What's cooking" report provides a comprehensive snapshot of the project's current state.

## Notable threads

### Object-file refactoring reaches completion

Patrick Steinhardt's 9-patch series reorganizing Git's object-file subsystem has been merged after thorough review. The changes establish clearer boundaries between object storage components by moving functions to more logical homes: directory utilities to dir.c, mmap functions to wrapper.c, platform-specific code to compat/open.c, index-related functions to read-cache.c, and object store management to the new object-store.c. The remaining object-file.c now focuses specifically on loose object handling. 

The series also removes global state for cached objects, moving their storage into the `raw_object_store` structure. Junio Hamano raised an architectural consideration about whether virtual objects storage belongs in the object store, particularly for submodule scenarios, but approved the current approach as safe while flagging it for future consideration. The changes represent a significant architectural improvement in Git's object storage code organization, touching 124 files with 2296 insertions and 2224 deletions.

### RFC: Cruft pack and MIDX optimization

Taylor Blau introduced an 8-part RFC series proposing changes to how Git handles cruft packs and MIDX during repacking. The core innovation is a new `--stdin-packs=follow` mode for `pack-objects` that creates reachability-closed packs by including objects from unspecified packs when they're reachable from included packs. This allows safely excluding cruft packs from MIDX while maintaining correctness, potentially improving performance by reducing MIDX size and complexity.

The series starts with preparatory refactoring of `pack-objects` code before introducing the new mode and its repack integration. Changes include standardizing option handling, improving delta compression for loose objects, and adding infrastructure to track MIDX-referenced packs. The final patch implements the core optimization, excluding cruft packs from MIDX when possible while using the new mode to ensure reachability closure.

### Bug reports surface interesting edge cases

Several bug reports highlighted subtle issues in Git's edge case handling:

1. **`git stash -k` with staged changes** incorrectly creates diffs between HEAD and unstaged changes rather than between staged and unstaged changes, causing unnecessary merge conflicts when popping. D. Ben Knoble pointed to a 2023 discussion addressing the same issue, suggesting this is a known limitation.

2. **`git add` wildcard matching** behaves unexpectedly when a literal file matching the wildcard pattern exists (e.g., a file named `f*` alongside `foo`). The first `git add 'f*'` only stages the literal `f*`, while subsequent runs correctly stage both files.

3. **Interrupted `git am` state detection** shows discrepancies between shell prompt detection (correctly showing AM/REBASE state) and `git status` behavior (incorrectly reporting REBASE). Phillip Wood proposed moving the state file creation earlier in the `am` process to fix this.

## In brief

Junio marked several topics ready for merging in his "What's cooking" report, including a performance optimization for bundle creation and meson build improvements. The `read-cache.c` bugfix from the static analysis series received final approval, establishing a precedent for handling tool-driven defensive programming changes. A `git clone --bundle-uri` user reported missing progress feedback during bundle processing, particularly problematic for large repositories. The `safe.directory` thread saw confirmation that environment variable clearing explains different wildcard matching behavior between single-process and multi-process operations. 

## On the radar

The discussion about adding a `--no-hooks` global option continues, with D. Ben Knoble sharing concrete npm/husky-related pain points where the feature would help. The multi-remote synchronization thread saw debate about whether to treat the local repository or a designated remote as the authoritative source, with Klaus Frank clarifying their CI/CD use case where no persistent local repository exists.