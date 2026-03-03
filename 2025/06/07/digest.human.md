# Git Mailing List Digest - 2025/06/07

**The day in brief.** A moderately active Saturday with 16 emails across 9 threads, featuring platform compatibility updates, ongoing refactoring work, and Junio's latest "What's cooking" report. The standout items are Phillip Wood's finalized `git stash -p` fixes and a Windows build regression in the byte-order refactoring work.

##Notable threads

**`git stash -p` pathspec fixes reach v3** -- Phillip Wood's series addressing a 2020 regression in `git stash -p` option parsing has reached its third iteration with test improvements from Martin Ågren. The patches fix both the immediate regression (where `git stash -p <pathspec>` stopped working) and extend support to allow more flexible option ordering like `git stash -m <message> --patch`. The v3 updates strengthen test coverage of interactive patch selection behavior, with Martin suggesting further refinements to catch edge cases in pathspec handling. The series appears ready for merging after this final round of test polishing.

**Windows build failures halt byte-order refactoring** -- Sebastian Siewior's `bswap.h` refactoring, previously discussed for its big-endian system impacts, has been reverted from the integration tree after causing Windows build failures. The regression emerged despite extensive prior review focused on s390x behavior, highlighting the challenges of cross-platform compatibility work. Junio pulled the changes pending investigation, shifting the thread's focus from macro symmetry concerns to Windows-specific toolchain interactions.

**Solaris build configuration updated** -- Brad Smith's straightforward patch modernizing Solaris build flags received positive feedback from Jonathan Nieder and Junio. The changes remove unnecessary compatibility flags for Solaris 11+ while maintaining support for older versions. The discussion branched into broader questions about Solaris version support lifecycle management, with Junio noting version-specific sections allow clean removal of EOLed versions (Solaris 5.6 reached EOL in 2006) while suggesting potential consolidation of truly version-independent flags.

##In brief

**IMAP send authentication fixes** -- Aditya Garg's series gains a patch making CRAM-MD5 fail gracefully without OpenSSL, though Junio notes it would fit better earlier in the series.

**Submodule hardcoding discussion** -- Patrick Steinhardt concurs with Junio's approach to modifying deprecated functions in `remote.c` as part of `the_repository` removal efforts.

**Fetch prefetch edge cases** -- M Hickford raises questions about handling interrupted prefetches in poor network conditions, though no code changes are proposed yet.

**cURL type safety resolved** -- Peter Michael Green's reported type mismatches in curl interactions were already fixed in master by Jeff King's recent series.

##On the radar

**"What's cooking" snapshot** -- Junio's latest status update shows active development across 50+ topics, with memory leak fixes, API cleanups, and performance features progressing alongside platform work. The reftable test migration to `clar` and path-based delta compression stand out as notable ongoing efforts.