# Git Mailing List Digest — 2025/01/08

**The day in brief.** A busy Wednesday with 72 emails across 19 threads, featuring significant progress on multiple fronts. The remote object-info protocol implementation reached a mature state, while hash algorithm handling saw a major refactoring series land. Notable discussions included platform-specific RNG failures and a revived effort for Linux FSMonitor support. Two bugfixes stood out: a `git diff` segfault fix and Windows-specific reftable test fixes that were fast-tracked for the upcoming release.

## Notable threads

### Remote object-info protocol implementation matures

Eric Ju's 9-part series implementing client-side support for remote object-info functionality reached completion today. The final patches add `remote-object-info` support to `git cat-file --batch-command`, allowing efficient querying of object metadata (currently just sizes) from v2 protocol servers. The implementation includes comprehensive tests covering multiple transport protocols and error cases, building on earlier server-side work. Review feedback from Calvin Wan and Christian Couder was incorporated, addressing protocol version handling and error message clarity. This represents a significant step forward in Git's ability to query remote repositories without transferring full objects.

### Hash algorithm refactoring complete

brian m. carlson's 8-part series to eliminate separate "unsafe" variants of hash operations was finalized today. The changes introduce a new `unsafe_hash_algo()` function that provides wrapped algorithm variants, replacing the previous pattern of `unsafe_` prefixed functions. The series includes test infrastructure updates, hashfile checkpoint hardening, and cleanup of now-unused scaffolding. Of particular note is patch 7, which addresses crash safety in hashfile checkpoint initialization. This refactoring simplifies Git's hash algorithm interface while maintaining the same functionality, reducing maintenance burden and eliminating potential confusion between safe and unsafe operation paths.

### Platform RNG failures spark broader discussion

A thread about random number generation in reftable expanded into a system-wide discussion of OpenSSL CSPRNG reliability after Randall Becker reported failures on ia64 systems with OpenSSL 3.0.11/PRNGD. The conversation revealed that Git's `git_rand()` function has correctness dependencies in reftable (for temporary file safety) while being unreliable on some platforms. Patrick Steinhardt and Junio Hamano debated solutions ranging from using system `rand()` to complete removal of OpenSSL backend support. The discussion highlighted tensions between security requirements and platform compatibility, particularly for NonStop systems that depend on OpenSSL. No resolution was reached, but the thread surfaced important architectural considerations for Git's RNG system.

### Linux FSMonitor support revived

Alexander Shopov announced plans to rebase and update Linux filesystem monitoring implementation patches that were previously submitted in 2022 and 2024. This would provide native filesystem monitoring on Linux similar to macOS's existing FSMonitor support, potentially speeding up status operations. The proposal revives discussions from PRs #1352 and #1667, with the author seeking initial feedback before proceeding with rebase work. While technical details weren't included in today's email, this signals renewed interest in a feature that could significantly improve performance on Linux systems.

### Critical bugfixes

Two important bugfixes stood out today. First, a fix for a `git diff` segfault when encountering invalid diff status, which adds validation to detect uninitialized status fields and exit gracefully. While straightforward, the patch would benefit from test coverage to verify both the crash scenario and new error path.

Second, Johannes Schindelin identified and fixed Windows-specific heap corruption in reftable tests caused by allocator switching. The solution - passing NULL to `reftable_set_alloc()` rather than explicit malloc/free pointers - was fast-tracked to master after review from René Scharfe and will be included in Git for Windows v2.48.0-rc2. This addresses `STATUS_HEAP_CORRUPTION` errors that manifested as exit code 127 in test runs.

## In brief

Justin Tobler's `git rev-list --missing` extension to report object types evolved into a more general attribute reporting system after Junio Hamano suggested outputting all available metadata (type, path) rather than implementing selective filtering. The thread settled on recording attributes only on first insertion into the missing objects table, matching `--objects` behavior for path handling.

A meson build system fix from Sam James added missing `git_version_file` dependencies in Perl-related build files, resolving parallel build race conditions. The straightforward change was fast-tracked to master while maintaining meson's "work in progress" status in release notes.

Scott Chacon proposed changing `help.autocorrect=1` to trigger immediate correction rather than a 0.1s delay, fixing common user confusion where "1" was mistaken for a boolean enable flag. Kristoffer Haugsbakk suggested documentation updates to accompany the behavior change.

## On the radar

The submodule path validation discussion continues, with Junio Hamano and Vadim Zeitlin debating whether to add a `submodule.validate` option to disable symlink checks. The conversation centers on balancing security validation against backward compatibility, particularly regarding configuration during clone operations. While Zeitlin argues the current workaround (manual initialization) suffices for this niche use case, the thread remains open for naming suggestions and consideration of future validation needs.