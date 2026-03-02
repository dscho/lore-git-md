Here's the daily digest for April 22, 2025:

**The day in brief.** A moderately active day with 98 emails across 24 threads, featuring significant progress on several fronts. Key developments include resolution of the long-running `pull.ff` configuration discussion, completion of Meson benchmark integration, and a new RFC for tree-level blame functionality. The Change-ID format debate continues with technical depth, while multiple email-related patches aim to improve Microsoft ecosystem compatibility.

**Notable threads**

**Configuration precedence for pull.ff and rebase settings**  
D. Ben Knoble and Junio Hamano concluded the months-long discussion about interaction between `pull.ff=only` and branch-specific rebase settings. Through careful examination of test cases in `t5520-pull.sh`, they determined existing configuration combinations (`pull.rebase=true` with `branch.<name>.rebase=false`) already provide the desired behavior - enforcing fast-forward-only for main branches while allowing rebasing for topic branches. This thread reached consensus that no code changes are needed, though documentation improvements could make these interactions more discoverable.

**Meson benchmark integration finalized**  
Patrick Steinhardt's v3 series integrating Git's performance benchmarks with Meson received final approvals after addressing all technical concerns. The 5-patch set handles repository format compatibility, Perl path portability, out-of-tree builds, core benchmark wiring, and configuration options. Christian Couder confirmed his earlier concerns were addressed, leaving only minor questions about error handling when the `time` command is missing. This completes a key piece of Git's build system modernization.

**Change-ID format debate continues**  
The technical discussion about standardizing Change-IDs for cross-forge compatibility evolved with new perspectives on uniqueness requirements. Junio Hamano raised concerns about Jujutsu's random generation approach lacking cryptographic guarantees, proposing instead to use commit object names. Remo Senekowitsch countered that 128-bit random IDs provide sufficient collision resistance in practice, citing Jujutsu's production experience. The thread now weighs theoretical guarantees against practical implementation considerations, with semantic purity emerging as a key requirement for tracking change evolution.

**git send-email improvements for Microsoft ecosystem**  
Multiple patch series (v2 and v3) from Aditya Garg and Julian Swagemakers added OAuth2 authentication support to `git send-email`, addressing Microsoft's mandatory authentication requirements. Companion patches fix Outlook's proprietary Message-ID handling to maintain thread continuity. A new `--smtp-passeval` option enables dynamic token generation. Junio Hamano provided detailed feedback on documentation accuracy and implementation clarity, particularly around when Message-ID modifications take effect.

**git-blame-tree RFC**  
A new 5-patch RFC series introduced `git-blame-tree`, a command to track changes across directory trees rather than individual files. The implementation includes core functionality, Bloom filter optimizations (yielding 5-52x speedups), and comprehensive performance tests. The series builds on work from Jeff King, Ævar Arnfjörð Bjarmason, and Derrick Stolee, positioning it as a production-ready feature seeking design feedback before finalization.

**In brief**  

**Performance test fix** -- Jeff King corrected a test case in `p5332-multi-pack-reuse.sh` that was failing due to obsolete "+" prefix syntax from a dropped feature.

**Windows/ARM64 compatibility** -- Johannes Schindelin addressed stack depth limitations for Windows/ARM64 builds while acknowledging nedmalloc's unmaintained status may warrant broader removal.

**String-list test modernization** -- A 5-patch series converted string-list tests from shell to C unit tests while fixing sign comparison warnings in the implementation.

**Wildcard pathspec fix** -- A bugfix addressed incorrect wildcard handling when literal wildcard-named files exist, with Junio providing test structure feedback.

**Dead code removal** -- Lucas Seiki Oshiro removed an unused `log_reencode` field from `struct rev_info`, completing a straightforward cleanup.

**On the radar**  

**Memory allocator strategy** -- The Windows/ARM64 thread surfaced broader questions about Git's allocator choices, with Johannes Schindelin suggesting mimalloc as a potential nedmalloc replacement.

**Build system modernization** -- Eli Schwartz's Meson function check standardization series awaits final decisions on Windows-specific handling of function checks.