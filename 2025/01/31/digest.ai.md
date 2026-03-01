# Git Mailing List Digest — 2025/01/31

## The day in brief

A busy Friday with 78 emails across 18 threads saw significant progress on several fronts. The standout developments include final approval for Derrick Stolee's pack-objects name hashing optimization, resolution of a long-standing atomic push exit code issue, and Junio Hamano's approval of the Clar test framework conversion series. Meanwhile, design discussions continue around OS version reporting in the protocol and credential storage warnings.

## Notable threads

### **Atomic push exit code propagation finalized**

Patrick Steinhardt and Jiang Xin's series fixing atomic push exit code propagation reached completion after four iterations. The final version (v4) introduces `ERROR_SEND_PACK_BAD_REF_STATUS` to consistently handle ref status failures across protocols while maintaining HTTP behavior. The comprehensive solution includes new test coverage in t5543 and t5548, verifying both regular and porcelain output formats. Taylor Blau gave his final approval, noting all technical concerns were addressed through style fixes, version validation consolidation, and removal of a less useful hash algorithm. The changes ensure atomic push failures properly propagate back to the pusher rather than silently succeeding - a subtle but important correctness fix.

### **Worktree bare detection bug resolved**

Olga Pilipenco's bugfix for bare repository detection in worktrees with worktree-specific configs reached its final form after multiple review rounds with Eric Sunshine and Junio Hamano. The v3 patch introduces `is_main_worktree_bare()` to properly check the main worktree's bare status when called from secondary worktrees, fixing cases where worktree-specific configs would incorrectly report the main worktree as non-bare. The solution includes comprehensive test coverage in t3200-branch.sh and addresses subtle design considerations around worktree state tracking. Junio requested one final clarification about the `!worktree->is_current` optimization check before merging.

### **New `--revision` option for git clone**

Toon Claes proposed a new `--revision` option for `git clone` that allows cloning a single specific ref or commit hash without creating remote-tracking branches. The 6-patch series includes significant preparatory refactoring of `builtin/clone.c` before introducing the feature, which is particularly useful for CI systems needing minimal clones. Junio raised several technical concerns in the final review, questioning the clearing of `option_branch` and noting a Sparse warning from type mismatch. The implementation otherwise appears thorough with comprehensive tests in t5621-clone-revision.sh covering branch/tag/HEAD refs, raw hashes, and various option combinations.

### **Packed-refs validation series progresses**

shejialuo's packed-refs validation series saw extensive discussion about header checking and sortedness verification. Junio clarified that fsck should accept headerless files (for pre-v1.5.0 compatibility) and only flag malformed headers, not unknown optional traits. The sortedness checking discussion revealed the runtime handles both sorted and unsorted files differently based on the header's "sorted" trait, leading to consensus that validation should mirror this conditional behavior. The series continues to balance strict validation with backward compatibility as it moves toward completion.

## In brief

Taylor Blau gave final approval to Derrick Stolee's configurable name hashing for pack-objects delta compression, noting the v4 changes addressed all concerns through style fixes and version validation consolidation. Seyi Kuforiji's Clar test framework conversion series (hashmap, decorate, strbuf, and strcmp-offset tests) received Junio's approval after incorporating Patrick Steinhardt's feedback on test isolation and function naming. Patrick Steinhardt proposed generic hash context wrappers to simplify hash operations across 19 files, with Junio noting the changes appear sensible pending coordination with parallel pack-writing work. A GitLab CI fix corrected whitespace/style check base commit determination by properly handling null variables in parameter expansion.

## On the radar

The OS version capability discussion remains unresolved between Christian Couder's preference for a separate parseable field and Junio Hamano's insistence on using the existing opaque agent string. An RFC patch proposing warnings for insecure credential storage sparked debate about helper identification and warning timing, with Junio suggesting the advice mechanism might be more appropriate than warning(). The GSoC 2025 microproject documentation system hit a final impasse over whether retirement should be mechanical (Junio's position) or require patch review (Patrick Steinhardt's preference).