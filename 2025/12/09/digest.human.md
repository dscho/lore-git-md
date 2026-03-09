# Git Mailing List Digest - December 9, 2025

**The day in brief.** December 9 saw moderate traffic (73 emails across 20 threads), with significant discussions around the `git-history` command's multi-branch handling, ongoing MIDX compaction work, and macOS iconv compatibility issues. Notable developments include Junio's "What's cooking" report and resolution of the Windows symlink test suite preparation.

## Notable threads

**`git-history` multi-branch handling debate intensifies**  
The philosophical divide over how `git-history` should handle commits appearing in multiple branches continued with contributions from Martin von Zweigbergk and Kristoffer Haugsbakk. Both strongly supported Elijah Newren's position that the command should rewrite all branches by default, aligning with Jujutsu's behavior. Patrick Steinhardt acknowledged the workflow differences but questioned implementation efficiency, particularly around merge handling. The discussion revealed tensions between Git's traditional branch-centric approach and Jujutsu-inspired commit-oriented workflows.

**MIDX compaction series nears completion**  
Taylor Blau's 17-patch MIDX compaction series received extensive review from Patrick Steinhardt, with discussions covering API naming, backward compatibility, and bitmap handling. Key decisions included incrementing the MIDX version number to maintain compatibility with libgit2 and refining function names for checksum handling (`midx_get_checksum()` and `midx_get_checksum_hex()`). The review process demonstrated careful attention to both current functionality and future extensibility of the MIDX layer.

**macOS iconv workarounds proliferate**  
Multiple approaches emerged to handle macOS 15's broken stateful encoding conversion in `reencode_string_iconv()`. René Scharfe proposed both code-based workarounds (using `ICONV_BREAKS` flag) and build-system solutions (preferring Homebrew's libiconv). Technical discussions focused on proper state reset handling during buffer growth operations, with the thread revealing this as part of a broader pattern of iconv issues on macOS. The workaround options now include compile-time flags, runtime configuration, and alternative library paths.

**Windows symlink test prep finalized**  
Junio Hamano queued Johannes Schindelen's v2 series preparing Git's test suite for Windows symlink support after Patrick Steinhardt confirmed the refinements (grammar fixes, `cmp` usage, and expanded commit messages) addressed all review feedback. The changes ensure compatibility with MSYS2's upcoming default symlink support while maintaining cross-platform test behavior.

**"What's cooking" highlights active development areas**  
Junio's status report outlined numerous ongoing efforts including MIDX compaction, Windows symlink support, object database refactoring, and the new `git history` command. Notable mentions included Karthik Nayak's reference backend work (awaiting final review) and the HTTP 429 rate limiting series (under active discussion). The report provides a comprehensive snapshot of Git's development pipeline heading into year-end.

## In brief

**`the_repository` removal handoff** -- Bello Olamide confirmed taking over the attribute file handling patches from Ayush Chandekar as part of an Outreachy internship, with positive coordination between the contributors.

**HTTP authentication refinement** -- Ashlesh Gawande proposed a modified 403 response handling approach that preserves credentials while still allowing prompting when none exist, addressing prior concerns about workflow breakage.

**HTTP 429 rate limiting review** -- Taylor Blau provided detailed feedback on the implementation, suggesting improved error handling, function organization, and test structure while approving the overall approach.

**Shallow fetch depth fix** -- Samo Pogačnik submitted a series correcting relative-depth fetching in repositories with merged branches by unifying the absolute and relative depth calculation logic.

**Promisor object optimization** -- A v3 series extended the `PARSE_OBJECT_SKIP_HASH_CHECK` optimization to promisor object handling, showing dramatic speedups (76m→2m) in large pack scenarios.

**SSH client visibility proposal** -- Ryan Johnson suggested improving SSH authentication failures by showing which SSH binary Git is actually using, helping diagnose configuration mismatches.

**`--staged` vs `--cached` terminology** -- Lucas Seiki Oshiro proposed standardizing on `--staged` across commands, prompting Junio to question whether deprecating `--cached` might be premature given its established meaning.

**`repo structure` size reporting** -- A 6-patch series added both inflated and on-disk size metrics to `git repo structure` output, with careful attention to human-readable formatting and machine parsing.

## On the radar

**ODB alternates refactoring** -- Patrick Steinhardt's series to modernize alternates handling saw productive review from Justin Tobler, with discussions now focusing on source abstraction naming and representation in `struct object_database`.

**`git pull --rebase` data loss edge case** -- The thread exploring solutions for inconsistent remote-tracking states with multiple push URLs added a new proposal to make `git push` operations atomic with respect to remote-tracking updates.