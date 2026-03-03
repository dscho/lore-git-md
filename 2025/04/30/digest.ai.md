# Git Mailing List Digest - 2025/04/30

**The day in brief.** A moderately busy Wednesday with 42 emails across 16 threads, featuring significant progress on maintenance task decomposition, architectural discussions around Scalar, and several bugfix investigations. The standout developments include Patrick Steinhardt completing the technical prerequisites for replacing `git gc` with granular maintenance tasks and Junio Hamano proposing documentation style unification for negated options.

## Notable threads

### Maintenance task decomposition completed

Patrick Steinhardt's v2 series ([1](https://lore.kernel.org/git/20250430102504.12345-1-pks@example.com/)) implementing the final standalone maintenance tasks needed to fully replace `git gc` received approval from Derrick Stolee. The series adds worktree pruning and rerere garbage collection as configurable maintenance tasks, completing the technical foundation for eventually deprecating `git gc`. Key improvements in v2 include configurable auto-execution thresholds (`maintenance.worktree-prune.auto` and `maintenance.rerere-gc.auto`) and thorough test coverage. Junio Hamano raised a performance consideration about the rerere collection phase, though this didn't block approval.

### Scalar maintenance control design debate

A series adding `--no-maintenance` flags to Scalar commands ([2](https://lore.kernel.org/git/20250430102438.12345-1-scalar@example.com/)) sparked architectural discussion from Junio Hamano. While agreeing with the feature's motivation (preventing maintenance.lock conflicts in automated setups), he questioned whether maintenance toggling belongs at the command level rather than being passed down to `register_dir()`. This feedback suggests the patches may need restructuring to better separate concerns before integration, despite being functionally complete.

### `git mv` assertion fixes finalized

Patrick Steinhardt's two-patch series addressing assertion failures in `git mv` when moving parent/child directories ([3](https://lore.kernel.org/git/20250430124456.12345-1-pks@example.com/)) received final polish from Junio Hamano. The changes replace an `assert()` with `BUG()` (following project norms) and add hashmap-based detection of invalid parent/child moves. Junio suggested additional test verification that the index and working tree remain unchanged after rejected moves, marking the series as nearly ready for inclusion.

### Documentation style proposal

Junio Hamano proposed standardizing how negated options are documented across Git's manpages ([4](https://lore.kernel.org/git/xmqq4jz7u9i8.fsf@gitster.g/)), responding to a thread about `--[no-]single-branch` discoverability. The suggested format would list positive and negative forms separately (improving grep-ability) while maintaining unified descriptions of all three states (enabled, disabled, default). This would resolve the immediate case while creating project-wide consistency for approximately 80 existing documentation entries.

## In brief

**Git Merge 2025 remote participation** -- Taylor Blau announced expanded virtual options for Git Merge 2025 ([5](https://lore.kernel.org/git/20250430214841.12345-1-taylor@example.com/)), addressing accessibility concerns with two-way A/V for the Contributor's Summit and free remote attendance.

**`git stash` submodule message bug** -- A user reported incorrect "On master:" messages in `git stash -m` when submodules are present ([6](https://lore.kernel.org/git/20250430140517.12345-1-user@example.com/)), specifically affecting Git for Windows 2.49.0.

**`git restore -m` conflict behavior** -- Junio explained the intentional design behind `git restore -m` resurrecting committed conflict markers ([7](https://lore.kernel.org/git/xmqqh6n3j4f9.fsf@gitster.g/)), dating back to 2009's conflict state tracking implementation.

**`git send-email` Outlook ID handling** -- A transitional patch in the Outlook Message-ID series ([8](https://lore.kernel.org/git/20250430083103.12345-1-email@example.com/)) introduced string-based internal control while maintaining boolean CLI compatibility.

**Nested replace refs discussion** -- Junio softened his opposition to nested replace refs ([9](https://lore.kernel.org/git/20250430001125.12345-1-tao@example.com/)) after Tao Klerks demonstrated existing duplicate-replacement safeguards.

## On the radar

**Interactive diff context configuration** -- Leon Michalak's proposal to inherit `diff.context` for interactive commands ([10](https://lore.kernel.org/git/20250430080446.12345-1-leon@example.com/)) is converging toward a hybrid approach combining config inheritance with flag overrides.

**`git commit <path>` staging behavior** -- Junio acknowledged the UX gap where path-limited commits include unstaged changes ([11](https://lore.kernel.org/git/xmqq4jz7u9i8.fsf@gitster.g/)), leaving the door open for a potential `--cached` mode addition.