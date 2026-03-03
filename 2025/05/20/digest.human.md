# Git Mailing List Digest - 2025/05/20

**The day in brief.** A moderately busy day with 85 emails across 19 threads, featuring several notable discussions around performance optimizations, security improvements, and user experience changes. Key highlights include final refinements to the promisor-remote protocol, a security-focused series on exec-path handling, and continued debate about MPTCP support in Git.

## Notable threads

### Promisor-remote protocol final refinements

The promisor-remote protocol enhancement series reached its final polishing stage with a discussion about function naming. Christian Couder's v3 series, which refactors how Git handles server-advertised remote attributes, received minor naming suggestions from Karthik Nayak and Junio Hamano. The debate centered on whether to rename `remote_nick_find()` to better reflect its role in searching `promisor_info` structs. Junio suggested simpler alternatives like `promisor_info_find()` while also raising a separate architectural question about potentially using `strmap` for more efficient lookups. This thread shows the meticulous attention to detail in final reviews before series acceptance.

### Security: Removing exec-path from child process PATH

A security-focused series from Jeff Hostetler addresses potential issues when Git's internal binaries are exposed to non-Git child processes via PATH. The 4-patch series starts with test cleanups, modernizes editor environment handling to use `strvec`, refactors `prep_childenv()` for cross-platform consistency, and finally implements the core change to remove Git's exec-path from PATH for non-Git commands. Junio raised concerns about the blanket approach potentially breaking legitimate cases where child processes need Git binaries, suggesting a more granular control via a new `drop_git_exec_path` flag in `child_process`. This discussion highlights the balance between security and functionality in Git's process spawning.

### MPTCP support debate continues

The Multipath TCP (MPTCP) support discussion saw significant technical clarification from Linux kernel MPTCP maintainer Matthieu Baerts. He explained why Linux's implementation requires explicit application support through `IPPROTO_MPTCP` socket creation rather than following RFC 6897's transparency principle. The thread explored configuration granularity (global vs per-repository settings), performance tradeoffs, and deployment considerations - particularly around `git-daemon` usage via `inetd`. Junio questioned whether unconditional `IPPROTO_MPTCP` passing might harm functionality when MPTCP is unavailable, though Matthieu confirmed robust fallback to regular TCP is guaranteed. This thread represents ongoing architectural discussion about network protocol integration.

### Stash behavior changes for Git 3.0

D. Ben Knoble proposed making `stash apply/pop` default to restoring the index (equivalent to current `--index`) when breaking changes are enabled in Git 3.0. The 9-patch series includes test cleanups, documentation updates, and core behavior changes gated behind `GIT_BREAKING_CHANGES`. Junio expressed ergonomic concerns about the new default requiring more frequent `--no-index` typing, creating tension between fixing a known pitfall and practical usability. The series is at a critical decision point where technical merits conflict with workflow considerations, highlighting Git's careful approach to behavior changes.

### Pathspec support for diff --no-index

Jacob Keller's series adding pathspec filtering to `git diff --no-index` reached advanced stages of review. The implementation introduces `PATHSPEC_NO_REPOSITORY` flag handling and carefully considers edge cases in path prefix handling. Junio questioned whether `prefix_path_gently()` should accept prefixes without trailing slashes or require callers to ensure proper formatting, sparking discussion about API contracts and input validation. The thread shows Git's meticulous approach to pathspec handling, particularly when extending functionality to non-repository contexts.

## In brief

**Memory leak fixes in pack-bitmap** -- Lidong Yan and Taylor Blau collaborate on a series fixing memory leaks during corrupted bitmap loading, now with added test coverage that verifies the fixes by deliberately corrupting bitmap files.

**MIDX repack overflow fixes** -- Phillip Wood addresses integer overflow issues in multi-pack-index repack calculations, particularly affecting 32-bit systems like Raspberry Pi, with protections for both 32-bit and theoretical 64-bit overflow cases.

**Stash option parsing fixes** -- Phillip Wood splits his earlier patch into two parts: first fixing a regression where `git stash -p <pathspec>` stopped working, then extending support for more flexible option ordering like `-m <message> -p <pathspec>`.

**Submodule config safety** -- K Jayatheerth and Junio discuss final polishing of a submodule configuration safety patch, debating whether to split path safety and active flag optimization into separate commits and add more explicit test coverage.

**Documentation standardization** -- A 6-part series improves documentation around `--stdin` usage in git-notes and clarifies `core.commentChar` references, with Junio suggesting minor phrasing improvements to better show configuration hierarchy.

**Test cleanup** -- Karthik Nayak removes obsolete leak-check variables from test scripts now that leak checking is unconditional, affecting performance tests and path walk verification.

## On the radar

**Rustification effort** -- While not active today, Ezekiel Newren's ongoing work to introduce Rust code into Git remains a significant architectural discussion point, particularly around platform support concerns raised by Randall S. Becker.

**ODB abstraction** -- Patrick Steinhardt's object database abstraction work continues in the background, with recent activity around reftable and performance optimizations suggesting more patches may surface soon.

**Breaking changes for Git 3.0** -- Several threads reference planned breaking changes for Git 3.0, including the stash behavior modification discussed today, indicating more proposals may emerge as the release approaches.