# Git Mailing List Digest - December 12, 2025

**The day in brief.** A moderately busy Friday with 54 emails across 19 threads, featuring significant progress on several fronts. The Scalar configuration documentation series reached its final form, Justin Tobler's `repo structure` enhancements expanded with comprehensive size reporting, and discussions continued about macOS build issues and C99 adoption. Junio's "What's cooking" report provided a comprehensive snapshot of the project's current state.

## Notable threads

**Scalar configuration documentation finalized** -- Derrick Stolee's series to document and clean up Scalar's configuration settings reached its fourth iteration, now with comprehensive documentation of all recommended settings and their rationales. The series has evolved through thorough review, removing obsolete configurations (like core.fscache and credential.validate), correcting an unintentional regression in index.skipHash, and adding "# set by scalar" annotations. Junio Hamano provided final editorial tweaks to the documentation phrasing, suggesting this well-reviewed series is nearing integration.

**Repository structure reporting enhanced** -- Justin Tobler's series adding object size reporting to `git repo structure` progressed significantly, with a 7-patch v2 that introduces both inflated content sizes and on-disk storage metrics. The implementation carefully handles human-readable formatting (using SI units for counts and IEC for sizes) while maintaining machine-readable keyvalue/nul formats. The series includes extensive test coverage and addresses platform-specific variations in disk size reporting. Discussion continues about test methodology, particularly around verifying disk usage output against `git cat-file --batch-check`.

**macOS build system challenges** -- The thread about macOS iconv workarounds expanded with detailed analysis of package manager paths (Homebrew's architecture-specific locations vs. MacPorts/Fink conventions) and confirmation that NO_ICONV is non-viable on macOS due to precompose_utf8.c's hard dependency. René Scharfe and Carlo Marcelo Arenas Belón provided technical clarifications about path conventions, while Junio Hamano guided the discussion toward practical solutions that accommodate Homebrew's standard installation patterns without overcomplicating the build system.

**C99 flexible array adoption debate** -- The discussion about requiring C99 flexible array member syntax took an interesting turn as Junio Hamano proposed a more aggressive approach - completely removing the legacy FLEX_ARRAY compatibility layer rather than just documenting it as a Git 3.0 breaking change. This "weather balloon" strategy would test real-world platform compatibility by forcing the change, with the understanding it could be reverted if problems emerge. The minimal code impact (31 lines removed) makes this a low-risk test of Git's C99 readiness.

## In brief

**HTTP 429 rate limiting refinements** -- Vaidas Pilkauskas responded to Taylor Blau's review of the HTTP 429 series, agreeing to make sleep_for_retry() non-blocking and improve code organization in the next version.

**MIDX optimization integration** -- Patrick Steinhardt confirmed Junio Hamano's resolution merging his MIDX rewrite avoidance work with Taylor Blau's incremental MIDX changes is technically sound, with only a minor documentation gap noted.

**Memory leak fixes acked** -- Justin Tobler reviewed Patrick Steinhardt's memory management fixes, confirming the v2 changes (minus the deferred geometric repack work) look good with improved test coverage.

**SSH authentication diagnostics** -- D. Ben Knoble questioned technical assumptions in a proposal to improve SSH failure messages, while also objecting to the original message's accusatory tone.

**`reset --hard` safety discussion stalls** -- After Junio Hamano expressed skepticism about changing `--hard`'s behavior, the thread shifted toward recovery documentation, with D. Ben Knoble sharing a blog post about using `git fsck --lost-found`.

**Rust build system portability** -- A macOS build failure was identified in Rust support due to GNU-specific sed usage, with Eric Sunshine confirming a more portable `-n`/`p` flag combination would work.

**`git replay` hash handling** -- A discussion emerged about `git replay`'s behavior with raw commit hashes, which currently creates objects without updating refs or providing output.

## On the radar

**Rename detection inconsistency** -- Luca Balsanelli reported a case where `git diff` and `git merge` handle file renames differently despite using the same similarity threshold, suggesting either a bug or undocumented behavior difference.

**`pretty=reference` formatting question** -- A brief bug report about multi-line message handling was retracted, leaving open whether there's a genuine issue with the output.