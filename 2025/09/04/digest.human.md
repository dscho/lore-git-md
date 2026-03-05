# Git Mailing List Digest - 2025/09/04

**The day in brief.** A high-volume day with 101 emails across 21 threads, dominated by major architectural discussions about Rust integration and commit graph handling, alongside significant feature work on history editing commands and the long-planned default branch name change. The Rust adoption debate reached a critical juncture with Patrick Steinhardt's RFC series, while Junio Hamano raised fundamental objections to the commit graph refactoring approach.

## Notable threads

**Rust infrastructure RFC lands** -- Patrick Steinhardt kicked off a major architectural discussion with a 3-patch RFC series introducing Rust infrastructure to Git core ([1](https://lore.kernel.org/git/20250904142642.12345-1-pksh@example.com)). The proposal includes meson build system support, a varint.c replacement as a test balloon, and documentation declaring Rust will become mandatory in Git 3.0 (targeting 2H 2026). Immediate pushback focused on build system requirements (meson 1.9.0 being too recent) and the silent fallback behavior that may mask platform compatibility issues. Ezekiel Newren countered with a Cargo-based alternative approach, while brian m. carlson emphasized practical packaging concerns. Junio Hamano's brief responses acknowledged the technical implementation but avoided endorsing the broader mandatory adoption plan.

**Commit graph architecture debate** -- Patrick Steinhardt's 6-part series moving commit graph handling from object_database to odb_source ([1](https://lore.kernel.org/git/20250904124954.12345-1-pksh@example.com)) sparked a fundamental design challenge from Junio Hamano. While praising the technical execution, Hamano argued commit graphs logically span the entire object store rather than individual sources, presenting scenarios where history ranges cross source boundaries. This critique questions the series' core premise and may require architectural reconsideration despite the clean implementation. The discussion continues with Hamano scrutinizing even minor implementation details like redundant NULL checks in bloom filter code.

**git-history command matures** -- Patrick Steinhardt posted v3 of his ambitious `git-history` series ([1](https://lore.kernel.org/git/20250904142747.12345-1-pksh@example.com)), now spanning 18 patches implementing Jujutsu-inspired history editing with `drop`, `reorder`, `split`, and `reword` subcommands. The implementation builds on extensive sequencer infrastructure changes and includes comprehensive test coverage. Junio engaged with technical specifics, suggesting `patch-delta.c` as a more meaningful test case than the minimal `varint.c` conversion. A reported diff visibility issue in the `split` subcommand remains under investigation, but the series appears well-structured for eventual merging.

**Default branch name change finalized** -- Phillip Wood's 4-patch series switching Git's default branch name from "master" to "main" (when built with WITH_BREAKING_CHANGES) received final approval ([1](https://lore.kernel.org/git/20250904132128.12345-1-phillip.wood@example.com)). The changes maintain backward compatibility while updating tests and documentation, culminating a multi-year effort since the 2020 deprecation warning. Junio suggested potential follow-ups around advice messaging for users following outdated tutorials, but the core implementation is now queued for merging.

**Slab allocator API cleanup completes** -- A 5-iteration series fixing dangling pointer issues in Git's slab allocator concluded with Junio's approval of the final NULL handling approach ([1](https://lore.kernel.org/git/20250904174416.12345-1-flare@example.com)). The changes introduce `alloc_state_free_and_null()` to atomically clean up allocator state while following Git's philosophy of failing visibly on programming errors. Jeff King suggested a middle ground for NULL pointer handling that was incorporated into the final version.

## In brief

**Partial clone workflow improvements** -- Dilyan Palauzov proposed making blobless clones the default with new commands for fetching missing history ([1](https://lore.kernel.org/git/20250904093308.12345-1-dilyan@example.com)). Derrick Stolee noted similar functionality exists in `scalar clone` and `git backfill`.

**combine-diff recursive flag fix** -- A bugfix addressed unexpected recursive behavior in `diff-tree -c` that was causing issues for the new `git last-modified` command ([1](https://lore.kernel.org/git/20250904115340.12345-1-tc@example.com)). Junio raised concerns about backward compatibility with output that has been stable since 2006.

**upload-pack ACK deduplication** -- Patrick Steinhardt fixed a protocol v2 regression where parent commits of already-seen commits weren't properly marked ([1](https://lore.kernel.org/git/20250904124233.12345-1-pksh@example.com)). Junio acknowledged the solution after discussing edge cases with non-chronological commit sending.

**rebase hook environment variables** -- A bug report identified that environment variables and `-c` config options aren't passed to prepare-commit-msg hooks during `rebase --continue` ([1](https://lore.kernel.org/git/20250904124613.12345-1-user@example.com)).

**repo-info enhancements** -- Lucas Seiki Oshiro's series adding `-z` shorthand and object format reporting to `git repo info` was approved after style fixes ([1](https://lore.kernel.org/git/20250904134015.12345-1-lucas@example.com)).

## On the radar

**Rust adoption timeline** -- The mandatory Rust requirement proposed for Git 3.0 (2H 2026) remains controversial, with platform support and build system requirements as open questions. The discussion is likely to continue as distributors weigh in.

**xdiff Rust implementation** -- Ezekiel Newren indicated his cbindgen-based FFI approach for the xdiff port is progressing but not yet ready for submission ([1](https://lore.kernel.org/git/20250904225558.12345-1-e@example.com)).

**khash vs oidset performance** -- Jeff King conceded to René Scharfe's security arguments in the ongoing data structure optimization discussion, moving closer to resolution ([1](https://lore.kernel.org/git/20250904111619.12345-1-peff@example.com)).