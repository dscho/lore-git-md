# Git Mailing List Digest - 24 June 2025

**The day in brief.** A moderately busy Tuesday with 48 emails across 11 threads, featuring significant progress on several fronts. Key developments include the resolution of the symbolic port names debate in git-credential-netrc, ongoing optimization work for MIDX and cruft packs, and a security report about shell injection in Bash completion. Junio's "What's cooking" report provides a comprehensive snapshot of the project's current state.

## Notable threads

### Symbolic port names in git-credential-netrc reaches resolution

Maxim Cournoyer's v3 series implementing symbolic port name support in git-credential-netrc appears headed for merging after addressing all technical concerns. The implementation now shares validation logic between the credential helper and git-send-email through a new `port_num` function in perl/Git.pm. Junio Hamano's final feedback focused on submission formatting rather than technical issues, suggesting the design debate has concluded in favor of helper-side resolution despite the documentation's mention of numeric ports. The series demonstrates robust handling of both numeric and symbolic ports across platforms while maintaining backward compatibility.

### MIDX and cruft pack optimizations progress

Taylor Blau's performance optimization series for MIDX and cruft pack interaction saw detailed discussion about incremental MIDX handling. Jeff King identified a test failure when running with GIT_TEST_MULTI_PACK_INDEX_WRITE_INCREMENTAL=1, where cruft packs unexpectedly appeared in the MIDX. The fix modifies pack gathering in builtin/repack.c to properly traverse MIDX layers. While the immediate issue is resolved, Jeff raised broader questions about whether the MIDX API should abstract this complexity. The series also includes documentation for the new `test_seq -f` option and style cleanups, with all substantive technical issues now settled.

### Security concern: Shell injection in Bash completion

Ondrej Pohorelsky reported a concerning interaction between Git's branch switching and Bash completion where branch names containing special characters (like `>`) could be interpreted as shell redirections during tab completion. The issue demonstrates how a branch named `>/tmp/dangerfile` could create unintended files when tab-completed in Bash (though not in Zsh). While the branch creation itself works, the completion behavior represents a potential vector for unintended file operations. The report lacks specific location of the vulnerability in Git's code but provides clear reproduction steps affecting Git 2.49 on Fedora, suggesting this may require security-focused attention.

### Git daemon signal handling modernization

Carlo Marcelo Arenas Belón proposed a series to modernize signal handling in the Git daemon, addressing zombie process cleanup on OpenBSD and crashes on AIX. The patches replace legacy `signal()` calls with `sigaction()`, centralize SA_RESTART handling, and introduce platform-specific workarounds for systems lacking proper siginterrupt support. Junio Hamano questioned whether the approach overcomplicates the solution, prompting a detailed defense of using `sigaction()` given POSIX deprecation of `siginterrupt()`. Chris Torek contributed valuable historical context about Unix signal handling evolution that supports maintaining an empty handler function. The series remains in discussion about implementation aesthetics while the technical solution appears sound.

### `git repo-info` command design evolves

The design discussion for Lucas Seiki Oshiro's new `git repo-info` command continued with focus on output formats and functionality scope. Phillip Wood advocated for NUL-terminated plaintext output (`<key><LF><value><NUL>`) and raised concerns about JSON's handling of non-UTF8 paths, suggesting base64 encoding. Junio Hamano endorsed the NUL-terminated approach. The thread also saw extensive test improvements from Phillip and Junio, refining the reference format tests to follow Git's test suite conventions. The command's design is converging but still faces decisions about whether to incorporate path resolution functionality currently in `git rev-parse --git-path`.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Test infrastructure improvements** -- Jeff King's `test_seq -f` format option series saw discussion about performance tradeoffs between shell and awk implementations, with the shell version preferred for readability despite potential slowdowns with very large sequences.

**Batched reference updates** -- Karthik Nayak reported his `fetch-push-bulk-ref-update` series has addressed all review feedback and is ready for potential re-inclusion in 'next' after temporary removal for fixes.

**Documentation style conversion** -- Jean-Noël Avila continues converting man pages to synopsis style, with several documentation-only patches progressing through review.

## On the radar

**SHA-256 by default** -- brian m. carlson's series to make SHA-256 the default hash algorithm in Git 3.0's breaking changes mode continues cooking, with potential wide-ranging impacts.

**Object store abstraction** -- Patrick Steinhardt's large-scale effort to remove `the_repository` global progresses with preload-index changes now in master, representing steady incremental progress on this architectural change.

**Promisor remote capability** -- Christian Couder's enhancement to the promisor-remote capability mechanism remains in development, with recent activity suggesting it may graduate to 'next' soon.