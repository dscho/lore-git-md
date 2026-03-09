Here's the Git mailing list digest for January 8, 2026:

---

### The day in brief
A moderately busy day with 50 emails across 29 threads, featuring several notable developments. The submodule gitdir path encoding series reached final approval, PID file debugging landed, and multiple documentation improvements progressed. Platform-specific fixes for macOS iconv behavior emerged as a recurring theme.

---

### Notable threads

**Submodule gitdir path encoding finalized**  
Adrian Ratiu's long-running series to implement runtime-configurable submodule gitdir paths through `extensions.submodulePathConfig` has received final approval from both Patrick Steinhardt and Junio Hamano. The v8 iteration addresses all technical concerns, including a four-tier fallback system (plain/encoded/numbered/hashed paths), case-folding protection via RFC3986 encoding, and comprehensive test coverage. Josh Steadmon confirmed successful testing against Google's downstream implementation, clearing the last hurdle before merging.

**PID file debugging ready for merge**  
Paulo Casaretto's feature to help diagnose stale locks via `~pid.lock` files is now production-ready after addressing Junio Hamano's final documentation feedback. The implementation includes Windows support via mingw_kill(), automatic cleanup through tempfile registration, and a 139-line test suite. While PID reuse remains a theoretical limitation (noted by D. Ben Knoble), the consensus is this represents an acceptable tradeoff for the debugging benefits.

**xdiff refactoring for Rust interop**  
Ezekiel Newren's 10-part series to refactor xdiff internals received detailed technical review from Phillip Wood, particularly around the new `ivec` (interoperable vector) type. The feedback identified several areas needing attention - struct naming conventions, potential undefined behavior in pointer casting, and missing error handling - while generally approving the direction. This foundational work for future Rust integration will likely see another iteration addressing these concerns.

**Reftable iterator optimization proposal**  
Tsahi Elkayam outlined a more efficient approach for reftable's object-to-ref lookups, currently hampered by full block scans instead of proper index utilization. The RFC suggests maintaining iterator state to read only relevant ref blocks, potentially improving performance in this key area of Patrick Steinhardt and Karthik Nayak's reftable backend work.

**MacOS iconv workarounds**  
Torsten Bögershausen and René Scharfe proposed a platform-specific fix for macOS 14/15's buggy iconv implementation, which corrupts ISO-2022-JP to UTF-8 conversions. The two-patch series introduces `ICONV_RESTART_RESET` to handle state loss during chunked conversions, with build system logic to enable it only when needed. This continues Git's pattern of accommodating platform quirks while maintaining correct behavior.

---

### In brief

**`git status` push tracking status** -- Harald Nordgren followed up on his 17-iteration series to show push divergence in `git status`, confirming all feedback was addressed and asking about merge timing.

**Test modernization** -- Andrew Chitester's v2 patch for `t1420-lost-found.sh` replaces fragile line counting with explicit directory comparisons per Junio's suggestion, exemplifying Git's evolving test best practices.

**Documentation standardization** -- Michael Lyons completed the `git-blame` man page conversion to synopsis-style format across two patches, earning Jean-Noël Avila's approval ("LGTM") for this mechanical but important documentation update.

**Whitespace checking** -- Adrian Ratiu's whitespace validation patch series reached final iteration, though Johannes Sixt questioned its utility given legitimate tab usage in table formatting scenarios.

**Patch-ID documentation** -- Kristoffer Haugsbakk's 6-part series polishing `git-patch-id` docs received positive review from Pushkar Singh, particularly appreciating the clarified `--verbatim`/`--stable` interaction.

**`git add -p` decision tracking** -- A 7th iteration improved the UI for tracking previous hunk decisions during interactive staging, implementing Junio's original suggestion with refined prompt formatting.

---

### On the radar

**Delta islands question** -- Martin Fick surfaced an apparent limitation where delta islands ignore objects not covered by bitmaps, seeking clarification on this potentially undocumented constraint affecting head isolation.

**Rust text handling** -- Emerging from the `.gitignore` encoding thread, Collin Funk noted Rust runtime behaviors (SIGPIPE handling and FD management) that may need consideration for future Rust integration.

**`git init --config` proposal** -- David Miguel Susano Pinto requested `--config` support matching `git clone`'s functionality, though brian m. carlson suggested existing alternatives (shell aliases, `includeIf`) may suffice.

**Rust diff headers** -- Benno Lossin's proposal for better trait context in Rust diff hunk headers sparked technical discussion about multi-line header implementation challenges.