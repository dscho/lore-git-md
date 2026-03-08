# Git Mailing List Digest - 2025/10/28

**The day in brief.** A busy Tuesday with 82 emails across 19 threads saw significant progress on multiple fronts. The atomic ref updates for `git replay` reached final approval, Rust integration discussions deepened, and several documentation improvements landed. Key highlights include the resolution of GPG signature stability issues and the introduction of diff algorithm configurability to `git blame`.

## Notable threads

### Atomic ref updates for `git replay` finalized

The long-running series to add atomic ref updates to `git replay` reached its conclusion today with v5 receiving maintainer approval. Siddharth Asthana incorporated final review feedback from Junio Hamano, Christian Couder, and Phillip Wood, addressing style nits and test improvements. The implementation now uses Git's ref transaction API (`ref_store_transaction_begin` etc.) for atomic behavior while maintaining backward compatibility through a `--ref-action=print` option. The series also adds a `replay.refAction` config variable for persistent mode setting. With all technical concerns resolved and comprehensive test coverage in t3650, this feature is now merge-ready after multiple iterations.

### Rust version policy debate emerges

A critical discussion unfolded around Git's minimum supported version policy for Rust, triggered by cbindgen integration requirements. Ezekiel Newren strongly advocated for setting Rust 1.63 (Debian oldstable) as the floor, noting that the current 1.49 target would make cbindgen integration impossible. This debate represents a key policy decision that must be resolved before finalizing Rust infrastructure work. Meanwhile, in the SHA-1/SHA-256 interoperability series, detailed technical discussions continued about FFI boundary design, particularly around hash algorithm type representation and safety considerations at the C/Rust interface.

### `git blame` gains diff algorithm configurability

Antonin Delpeuch's patch to make `git blame` respect `--diff-algorithm` and `diff.algorithm` config received maintainer approval in its third iteration. The implementation now handles all edge cases including interaction with the existing `--minimal` flag and proper command-line/config precedence. Test coverage in t8015 verifies the new functionality across various scenarios while maintaining Myers as the default algorithm. This long-requested feature allows users to select between minimal, patience, and histogram diff algorithms for blame operations.

### GPG signature stability resolved

Eric W. Biederman and Junio Hamano confirmed the solution to intermittent test failures in t1016-compatObjectFormat.sh, where GPG's SHA-1 signatures were producing different hashes under load. The fix modifies the GPG wrapper to properly freeze timestamps using `--faked-system-time=...!`, addressing the root cause rather than Junio's initial band-aid approach of disabling the flaky tests. This resolves a subtle compatibility testing issue that had persisted through multiple release cycles.

### Packfile store refactoring advances

A substantial 8-part series from Patrick Steinhardt restructures Git's packfile management, moving list handling into a dedicated `packfile_store` structure. The changes replace global linked lists with store-owned structures, introduce a `strmap` for pack name lookups, and ultimately replace the dual MRU/mtime tracking with a single MRU list. The final patch carefully handles an edge case where pack iteration could modify the list being iterated. This foundational work prepares for deeper integration with Git's object source layer while maintaining existing behavior.

## In brief

**Documentation markup fixes** -- Jean-Noël Avila corrects AsciiDoc formatting in git-checkout manpage, changing branch`_`name placeholders from backticks to underscores to avoid unintended HTML formatting.

**Git bisect documentation sync** -- Kristoffer Haugsbakk aligns `git bisect` usage strings between code and documentation, properly moving subcommand synopsis to the SYNOPSIS section and documenting the previously undocumented `next` subcommand.

**Fast-import/export translation work** -- Christian Couder's series marks 350+ error/warning messages for translation in fast-import and fast-export while cleaning up GPG interface code, with Junio Hamano reviewing the changes.

**Git for Windows 2.51.2 released** -- Johannes Schindelin announced this maintenance update, primarily fixing high-DPI display issues in the portable variant's credential helper while syncing with core Git 2.51.2.

**Whitespace bit documentation** -- Junio Hamano's v2 patch corrects and clarifies bit assignments in whitespace handling code, converting octal constants to bit-shift notation and properly documenting color-moved bits (17-19).

## On the radar

**Rust workspace structure proposal** -- Ezekiel Newren's suggestion to proactively adopt a Cargo workspace before further Rust expansion may influence upcoming infrastructure decisions, though not yet generating significant discussion.

**NonStop platform issues** -- Randall Becker's report of test failures on NonStop with OpenSSL 3.5, while likely a test suite rather than core functionality issue, warrants monitoring given the ongoing Rust compatibility concerns for this platform.