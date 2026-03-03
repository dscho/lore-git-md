# Git Mailing List Digest - 2025/05/20

**The day in brief.** A moderately busy day with 85 emails across 19 threads, featuring several notable technical discussions and patch series nearing completion. Key highlights include final refinements to the promisor-remote protocol handling, a security-focused series on exec-path handling, and ongoing debates about MPTCP support and `git stash` behavior changes. The day also saw several performance optimizations and documentation improvements moving toward consensus.

## Notable threads

### Promisor-remote protocol final polish

The promisor-remote protocol enhancement series reached its final polishing stage with a discussion about function naming in the refactored code. Karthik Nayak suggested renaming `remote_nick_find()` to better reflect its role in searching `promisor_info` structs. Junio Hamano weighed in with simpler alternatives like `promisor_info_find()`, while also raising a separate architectural question about potentially optimizing the linear search with a strmap. This thread shows the meticulous attention to detail in Git's review process, where even minor naming decisions receive careful consideration at the series conclusion.

### Exec-path security series

A security-focused series addressing how Git's exec-path is exposed to child processes generated significant discussion. The patches aim to prevent Git's internal binaries from being accidentally invoked by non-Git programs, particularly on Windows where this has caused real-world issues. The series includes preparatory refactors to modernize environment handling with strvecs, followed by the core change removing git_exec_path() from PATH for non-Git commands. Junio Hamano raised important concerns about the blanket PATH modification approach, suggesting a more surgical solution using a new `drop_git_exec_path` flag in the child_process struct. This thread highlights Git's careful balance between security and compatibility.

### MPTCP support debate continues

The Multipath TCP (MPTCP) support discussion advanced with technical clarifications from Linux kernel MPTCP maintainer Matthieu Baerts. He explained why Linux's implementation requires explicit application support through `IPPROTO_MPTCP` socket creation rather than following RFC 6897's transparency goals. The thread now focuses on configuration design questions, including whether MPTCP preference should be settable per-repository or per-host, and how this interacts with `git-daemon` deployments via inetd. Junio's probing questions reveal careful consideration of both technical feasibility and practical deployment scenarios for this network-level optimization.

### Stash option parsing fixes

Phillip Wood submitted v2 of a series fixing `git stash` option parsing regressions, splitting the changes into two logical parts as requested by Junio. The first patch restores pre-2020 behavior where "-p" implies "push", while the second enables more flexible option ordering like "-m <message> -p <pathspec>". The implementation uses `PARSE_OPT_STOP_AT_NON_OPTION` when push is assumed, maintaining rejection of malformed commands while allowing valid combinations. This thread demonstrates Git's review process ensuring both bug fixes and feature enhancements are properly separated and documented.

### Pathspec support for diff --no-index

Jacob Keller's series adding pathspec support to `git diff --no-index` reached advanced stages of review. The implementation introduces a `PATHSPEC_NO_REPOSITORY` flag and carefully handles directory comparisons outside Git repositories. Junio provided detailed feedback on edge cases and documentation, with particular attention to how pathspecs should interact with the two directory arguments. The discussion revealed consensus on the core approach while identifying minor improvements to test presentation and API design. This feature represents a significant enhancement to Git's diff capabilities in non-repository contexts.

## In brief

**Promisor-remote protocol naming** -- Christian Couder and Junio Hamano discuss final naming refinements for helper functions in the promisor-remote protocol series, considering both clarity and future optimization potential.

**Memory leak fixes in pack-bitmaps** -- Lidong Yan and Taylor Blau collaborate on a series fixing memory leaks during corrupted bitmap loading, now with added test coverage for the error paths.

**MIDX repack overflow fixes** -- Phillip Wood addresses integer overflow issues in multi-pack-index repack calculations, particularly important for 32-bit systems like Raspberry Pi.

**Batched reference updates optimization** -- Karthik Nayak's high-performance optimization for batched reference updates (22x speedup for reftable in fetch) receives final approval from Junio.

**Missing packfile optimization** -- Patrick Steinhardt's revised patch series reduces syscalls for missing packs in multi-pack-index scenarios while maintaining safety constraints.

**Submodule config safety** -- Junio and K Jayatheerth discuss final documentation and test coverage improvements for submodule configuration safety patches.

**Stash behavior change proposal** -- D. Ben Knoble's series to make `stash apply/pop` default to restoring the index when breaking changes are enabled awaits broader feedback on its ergonomic tradeoffs.

**Receive-pack connectivity check bypass** -- A finalized series implements a hidden `--skip-connectivity-check` option for GitLab's Gitaly service, with thorough test coverage and documentation.

**Documentation standardization** -- A 6-part series improves documentation around `--stdin` usage in git-notes and clarifies `core.commentChar` references throughout the docs.

**Test leak check cleanup** -- Obsolete leak-check variables are removed from test scripts now that leak checking is mandatory.

## On the radar

**Rustification effort** -- While not active today, Ezekiel Newren's work to introduce Rust code into Git remains a significant ongoing effort with periodic updates.

**the_repository removal** -- René Scharfe's long-running project to eliminate the `the_repository` global variable continues to generate mechanical patches across the codebase.

**Documentation synopsis conversion** -- Jean-Noël Avila's effort to convert all man pages to consistent "synopsis style" using AsciiDoc markup remains active with periodic updates.