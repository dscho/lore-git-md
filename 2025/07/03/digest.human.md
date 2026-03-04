# Git Mailing List Digest - 2025/07/03

**The day in brief.** A busy Thursday with 69 emails across 21 threads, dominated by technical refinements to major ongoing efforts like the reftable default transition and `--skip-until` pagination. Notable highlights include Patrick Steinhardt's reftable v2 series addressing feedback, a confirmed regression fix for `--diff-filter`, and productive discussions about Git's build system future.

## Notable threads

**Reftable as default format moves forward** -- Patrick Steinhardt's v2 series (6/8 patches today) advances the proposal to make reftable Git 3.0's default ref storage backend. The updated approach introduces a `REF_STORAGE_FORMAT_DEFAULT` macro in repository.h while maintaining files backend as default for stable releases. Documentation now better explains platform-specific benefits like case-insensitivity handling. The implementation discussion settled on setting defaults in `repository_format_configure()` rather than `REPOSITORY_FORMAT_INIT`, though future cleanup to use `_UNKNOWN` was identified. Karthik Nayak's review suggested additional documentation refinements around storage savings and implementation location.

**Pagination interface refinements** -- The recently merged `--skip-until` feature for `git for-each-ref` saw extensive post-merge discussion about interface clarity and race condition behavior. Patrick Steinhardt and Karthik Nayak refined the `ref_iterator_seek()` API to replace a `set_prefix` boolean with clearer flags. Junio Hamano raised then resolved concerns about race behavior, confirming the name-based approach correctly handles concurrent ref modifications by tracking sort position rather than existence. The thread also addressed documentation ambiguity about whether the skip marker is included in output.

**Build system roadmap clarified** -- Patrick Steinhardt provided authoritative guidance about Git's build system direction in response to FreeBSD compatibility questions. Meson remains experimental but targeted for completion this cycle, with both Makefile and Meson to be officially supported. Autotools will likely be removed after Meson reaches parity, though not immediately. Downstream packager Renato Botelho confirmed plans to test Meson in FreeBSD's ports system once feature-complete. Ramsay Jones later acknowledged a Meson-related regression fix for `GIT_EXEC_PATH` handling with custom libexecdir paths.

**Diff filter regression confirmed and fixed** -- What initially appeared to be expected behavior for `git log --diff-filter` with exclusion filters was identified as a regression by Jeff King, traced to a 2022 commit. The fix modifies `setup_revisions()` to properly consider both `filter` and `filter_not` fields when determining if diffs are needed. Original reporter Eric Salem confirmed the patch restores the expected behavior where exclusion filters (like `d` for deletions) work regardless of output format. The minimal one-line change with test coverage is now ready for merging.

**New repository inspection command takes shape** -- Lucas Seiki Oshiro's `git repo-info` series received detailed review from Patrick Steinhardt covering implementation quality and test rigor. Discussions touched command naming (potential `git repo` namespace), JSON output verification, and the unavoidable use of `the_repository` global in bare repo detection. The thread shows steady progress on this new metadata reporting tool, with Patrick providing thorough feedback on code organization and error handling patterns.

## In brief

**Memory leak fix in send-pack** -- Jacob Keller's patch fixing `extra_have` oid array handling saw review from Jeff King and Junio Hamano confirming the approach, with analysis of edge cases around `remote_refs` validity after `match_push_refs()` fails.

**Stash index lock error reporting** -- Version 2 of a patch improves `git stash` error messages for index lock failures while maintaining backward compatibility, now with explicit test verification of both old and new message formats.

**FreeBSD version support confirmed** -- Renato Botelho verified FreeBSD 13.5 as the oldest supported version (EOL April 2026), finalizing the thread about removing NO_MEMMEM and other FreeBSD 4.x-era conditionals.

**Bloom filter test cleanup** -- Lidong Yan proposed moving murmur3 test code into t/helper to avoid exposing the hash implementation in bloom.h, keeping with the series' focus on final API polish.

**Credential helper investigation** -- Michal Suchánek's HTTP-level tracing revealed the credential invalidation issue stems from a server incorrectly using HTTP 401 status codes, suggesting an upgrade path rather than Git changes.

## On the radar

**X-Change-ID headers** -- A proposal to add commit message change-id preservation through email workflows saw positive feedback from Jujutsu developers about compatibility. The author plans a v2 including git-am integration after successful manual testing.

**Submodule path handling** -- K Jayatheerth's month-old series improving renamed submodule handling remains in review limbo, with a polite nudge today seeking feedback on the helper logic implementation.