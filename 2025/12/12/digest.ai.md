# Git Development Digest - December 12, 2025

**The day in brief.** A moderately busy Friday with 54 emails across 19 threads, featuring significant progress on Scalar configuration documentation, macOS build system fixes, and repository structure reporting. The most notable developments include the v3 Scalar config series nearing completion and Justin Tobler's comprehensive extension of `git repo-structure` with size reporting capabilities.

## Notable threads

**Scalar configuration documentation finalized** -- Derrick Stolee's series to document and clean up Scalar's configuration settings reached its third iteration, now with comprehensive documentation of all recommended settings and their rationales. The series removes obsolete configurations (like core.fscache and credential.validate), corrects an unintentional regression in index.skipHash, and adds "# set by scalar" annotations to modified config values. Junio Hamano provided final review with only a minor grammatical suggestion, indicating this is likely the last round before integration.

**Repository structure reporting enhanced** -- Justin Tobler extended Git's `repo structure` command with both inflated content sizes and on-disk storage metrics in v2 of his series. The implementation now handles human-readable formatting with proper alignment (using decimal units for counts and binary units for sizes) while maintaining machine-readable keyvalue/nul formats. The series includes robust test verification using `git cat-file --batch-check` and addresses feedback about code reuse from Junio and Patrick Steinhardt. This represents a significant expansion of Git's repository analysis capabilities.

**macOS build system fixes** -- The ongoing thread about macOS iconv workarounds saw detailed discussion of package manager path handling, particularly for Homebrew installations on Apple Silicon (/opt/homebrew) versus Intel (/usr/local). René Scharfe clarified that NO_ICONV is non-viable on macOS due to precompose_utf8.c's hard dependency on iconv functionality, narrowing the solution space to proper ICONVDIR configuration. The thread also surfaced outdated Fink package manager path references that may need updating.

**C99 flexible array adoption** -- Junio Hamano proposed an aggressive approach to require C99 flexible array member syntax by removing Git's legacy FLEX_ARRAY compatibility layer entirely, rather than just documenting it as a Git 3.0 breaking change. This "weather balloon" test would force the issue to see if any platforms still need the pre-C99 support, with the option to revert if problems emerge. The change is motivated by future plans to adopt Linux kernel-style fortified memcpy features.

## In brief

**HTTP 429 rate limiting** -- Vaidas Pilkauskas responded to Taylor Blau's review of the HTTP 429 handling series, agreeing to make sleep_for_retry() non-blocking and refactor header parsing in the next version.

**Memory leak fixes** -- Justin Tobler acked Patrick Steinhardt's v2 series fixing commit-graph and submodule ODB memory leaks, noting the geometric repack patch was correctly deferred for promisor pack handling.

**MIDX optimization integration** -- Patrick Steinhardt confirmed Junio Hamano's merge resolution between his MIDX rewrite avoidance work and Taylor Blau's incremental MIDX changes works as intended.

**`git reset --hard` safety** -- After Junio's firm rejection of behavior changes, D. Ben Knoble shared recovery techniques for lost work, shifting focus to mitigation rather than prevention.

**Rust build system fix** -- Eric Sunshine confirmed the GNU sed dependency in Rust builds is unnecessary, proposing a portable `-n`/`p` solution for macOS compatibility.

**On the radar**

**`git replay` with raw hashes** -- A new thread questions whether the command should handle raw commit hashes more gracefully, either by erroring clearly or providing output when no ref updates occur.