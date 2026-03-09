# Git Mailing List Digest — 2025/01/06 -- 2025/01/12

## The week in brief

A busy week with 437 emails across 141 threads saw significant progress toward Git 3.0 preparations, multiple security fixes, and important refactorings. Key developments include Patrick Steinhardt's breaking changes infrastructure for Git 3.0, comprehensive security hardening for `git blame`, and the completion of major refactorings in combine-diff and hash algorithm handling. The week also saw the release of Git v2.48.0 with new features like `remote.<name>.serverOption` and continued memory safety improvements.

## Key developments

### **Breaking changes infrastructure for Git 3.0**

Patrick Steinhardt's series establishing patterns for handling breaking changes in Git 3.0 saw extensive discussion. The patches introduce a `WITH_BREAKING_CHANGES` build option to gate removal of long-deprecated features like `.git/branches/` directories (deprecated since 2005) and the `pack-redundant` command (deprecated since 2011). Junio Hamano provided important historical context about the `.git/branches/` mechanism, noting it wasn't strictly replaced by newer systems but rather coexisted due to its simplicity for certain workflows. The deprecation warning strategy debate revealed philosophical differences between temporary silencing (Steinhardt) and persistent warnings to ensure migration (Hamano), with Junio ultimately favoring the latter approach based on painful experience from the Git 1.6 transition.

### **Security hardening for `git blame` and credential handling**

Patrick Steinhardt's v3 series comprehensively addressed three security issues in `git blame`'s object ID handling: an out-of-bounds read with large `--abbrev` values, a buffer overflow in boundary commit handling, and a pre-existing vulnerability in blank spacing for UNINTERESTING commits. The fixes maintain compatibility with both SHA-1 and SHA-256 repositories while adding extensive test coverage. 

Credential security saw multiple improvements including documentation warnings against using `git-credential-cache` for personal access tokens (PATs) and Jeff King's proposal to change `transfer.credentialsInUrl` default from "allow" to "warn". The PAT storage discussion revealed tension between security hardening and maintaining Git's service-agnostic design, with no clear consensus yet on the best path forward.

### **Major refactorings land**

Jeff King completed his 14-part combine-diff refactoring, methodically improving memory management and code clarity while eliminating marginal optimizations in favor of cleaner code. The changes introduce `combine_diff_path_new()` to centralize tricky allocation logic and simplify path handling by moving string construction to callers.

brian m. carlson finalized an 8-part series to eliminate separate "unsafe" variants of hash operations, introducing `unsafe_hash_algo()` that provides wrapped algorithm variants. The refactoring simplifies Git's hash algorithm interface while maintaining functionality, reducing maintenance burden and potential confusion between operation paths.

### **Reftable entropy handling debate**

Patrick Steinhardt proposed a fix for reftable failures on ia64 systems where OpenSSL's PRNGD exhausts entropy during concurrent operations, introducing a `CSPRNG_BYTES_INSECURE` flag for fallback to less secure random bytes when cryptographic strength isn't required. The discussion expanded into broader architectural concerns about reftable depending on Git's CSPRNG layer and platform support requirements versus codebase purity. Randall Becker confirmed the fix addresses real-world issues on NonStop systems, while Junio Hamano questioned whether the entropy exhaustion scenario is over-optimized given modern OpenSSL behavior.

## In brief

**Git v2.48.0 released** with 605 non-merge commits from 93 contributors, including new `remote.<name>.serverOption` configuration and enhanced `git rebase --rebase-merges` behavior. **CI modernization** progressed with Patrick Steinhardt's series removing Azure Pipelines remnants and stabilizing tests including a complex solution for flaky SIGPIPE tests in submodules. **zlib-ng integration** showed ~25% speedup in object access operations through a clean abstraction layer while maintaining standard zlib compatibility. **The remote object-info protocol** implementation matured, allowing efficient metadata queries from v2 protocol servers via `git cat-file --batch-command`. **Documentation standardization** continued with Jean-Noël Avila leading conversion of man pages to consistent AsciiDoc synopsis style. **A regression in bare repositories** with multiple remotes was identified and fixed, where `git describe` failed after adding a second remote due to incorrect HEAD handling.

## Looking ahead

The **submodule path validation** discussion continues, debating whether to add a `submodule.validate` option to disable symlink checks while balancing security against backward compatibility. The **OS version reporting** design is converging on a minimal initial implementation through `get_uname_info()` without configuration knobs. **Linux FSMonitor support** may see revived effort as Alexander Shopov announced plans to rebase and update previously submitted patches from 2022 and 2024. The **Rustification effort** remains a point of contention, particularly regarding platform support concerns raised by Randall Becker for NonStop systems lacking Rust tooling.