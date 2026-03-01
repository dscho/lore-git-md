# Git Mailing List Digest — 2025/01/08

**The day in brief.** A busy Wednesday with 72 emails across 19 threads, featuring significant progress on multiple fronts. Key developments include the completion of a hash algorithm refactoring series, new remote object-info capabilities in `git cat-file`, and extended missing object reporting in `git rev-list`. Platform-specific issues around random number generation and test infrastructure also drew attention.

## Notable threads

### Hash algorithm refactoring complete

brian m. carlson's 8-part series to eliminate separate "unsafe" variants of hash operations reached completion today. The final patches ([1](https://lore.kernel.org/git/20250108191454.1234567-1-sandals@crustytoothpaste.net/)) introduce a new `unsafe_hash_algo()` function that provides wrapped access to hash implementations, removing the risk of accidentally mixing safe and unsafe operations. The series includes thorough test infrastructure updates and hardening for hashfile checkpoints, particularly addressing crash safety in `fast-import` and `bulk-checkin`. This foundational work simplifies Git's hash algorithm handling while maintaining backward compatibility.

### Remote object-info in cat-file

Eric Ju's 9-part series implementing client-side support for remote object-info functionality culminated today with](https://lore.kernel.org/git/20250108183739.1234567-1-eju@google.com/) a new `remote-object-info` command for `git cat-file --batch-command`. The feature allows querying object metadata (currently just sizes) from v2 protocol servers without downloading full objects. The implementation includes comprehensive tests covering multiple transport protocols and error cases, building on earlier transport layer refactoring. This completes the client-side counterpart to server-side support added in a2ba162cda.

### Rev-list gains missing object attributes

Justin Tobler's work on extending `git rev-list --missing` ([thread](https://lore.kernel.org/git/20250108221835.1234567-1-jtobler@gitlab.com/)) evolved significantly through today's discussion. What began as a simple `print-type` option has grown into a more extensible attribute reporting system after feedback from Junio Hamano and Christian Couder. The consensus now favors outputting all available attributes (type, path) using a `?<oid> [<token>=<value>...]` format with C-style quoting, maintaining backward compatibility while allowing future expansion. The implementation will use an `oidmap` to track first-seen attributes.

### Random number generation challenges

A wide-ranging discussion ([thread](https://lore.kernel.org/git/20250108174058.1234567-1-gitster@pobox.com/)) about random number generation in reftable expanded to address system-wide concerns after Randall Becker reported OpenSSL CSPRNG failures on ia64 systems. The conversation revealed that reftable's temporary file handling actually depends on suffix uniqueness for crash safety, contrary to initial assumptions. Patrick Steinhardt and Junio Hamano debated solutions balancing platform constraints (particularly NonStop's OpenSSL dependency) against reliability requirements, with brian m. carlson suggesting potential workarounds like libbsd fallbacks.

## In brief

A segfault fix for `git diff` when processing invalid status values ([patch](https://lore.kernel.org/git/20250108060150.1234567-1-example@contributor.org/)) adds validation for uninitialized `diff_filepair->status` fields. The meson build system gained Perl component dependency fixes ([patch](https://lore.kernel.org/git/20250108034237.1234567-1-sam@gentoo.org/)) to resolve parallel build races. Johannes Schindelin fixed Windows-specific heap corruption in reftable tests ([patch](https://lore.kernel.org/git/20250108160005.1234567-1-dscho@microsoft.com/)) by properly handling allocator consistency. Scott Chacon improved `help.autocorrect` UX ([patch](https://lore.kernel.org/git/20250108193146.1234567-1-schacon@gmail.com/)) by making `=1` trigger immediate correction as users expect.

## On the radar

Alexander Shopov announced plans to revive Linux FSMonitor support ([email](https://lore.kernel.org/git/20250108152710.1234567-1-ashopov@microsoft.com/)), rebasing previous work from PRs #1352 and #1667. The submodule path validation discussion continues ([thread](https://lore.kernel.org/git/20250108190005.1234567-1-vadim@example.org/)) with Junio Hamano seeking input on configuration design for symlink checks. The reference deprecation warnings thread saw maintainer pushback ([email](https://lore.kernel.org/git/20250108170928.1234567-1-gitster@pobox.com/)) against Patrick Steinhardt's proposed environment variable for suppressing "branches/" and "remotes/" prefix warnings.