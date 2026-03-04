# Git Mailing List Digest - 2025/06/24

**The day in brief.** A moderately busy day with 48 emails across 11 threads, featuring progress on several technical fronts. Key developments include resolution of the symbolic port names debate in git-credential-netrc, ongoing optimization work for MIDX and cruft packs, and a security report about shell injection in Bash completion. Junio's "What's cooking" report provides a snapshot of the current development landscape.

## Notable threads

### Symbolic port names in git-credential-netrc reaches resolution

Maxim Cournoyer's v3 series implementing helper-side resolution of symbolic port names appears headed for merging after extensive discussion. The final version incorporates all requested changes, including renaming the validation function to `port_num()` and sharing logic with `git-send-email`. While the credential protocol documentation mentions only numeric ports, Junio Hamano's focus on submission mechanics rather than technical concerns suggests acceptance of this pragmatic solution. The implementation now robustly handles both numeric and symbolic ports across multiple components while maintaining backward compatibility.

### MIDX and cruft pack optimizations progress

Taylor Blau's performance optimization series for MIDX and cruft pack interaction saw detailed review discussion today. Jeff King identified an issue with incremental MIDX handling that Taylor addressed by properly traversing MIDX chains via `m->base_midx`. The thread revealed deeper architectural questions about whether MIDX's incremental representation should be abstracted from callers, though the immediate bug fix takes a practical approach. Junio also caught minor style issues in several patches, all of which Taylor promptly acknowledged. With substantive issues resolved, the series appears ready to progress.

### Security concern: Shell injection in Bash completion

Ondrej Pohorelsky reported a concerning interaction between Git's branch switching and Bash completion where branch names containing shell metacharacters (like `>`) could be interpreted as redirection commands during tab completion. While the branch creation itself works (creating unusual but valid refs), the completion phase in Bash's parsing introduces a security vector. The issue doesn't affect Zsh, which properly escapes special characters. The report lacks specific location of the vulnerability in Git's code but provides clear reproduction steps, marking this as an issue needing prompt attention.

### `git repo-info` command design evolves

Lucas Seiki Oshiro's GSoC project to create a `git repo-info` command saw continued design discussion today. Phillip Wood advocated for NUL-terminated plaintext output (matching Git's conventions in `git config -z`) and raised important questions about JSON encoding of non-UTF8 paths. Junio Hamano endorsed the NUL-terminated approach, noting keys can be guaranteed LF-free. The thread also debated whether to include path resolution functionality currently in `git rev-parse --git-path`, with Phillip arguing for a unified interface while Lucas expressed concern about scope creep. Test improvements dominated the latter part of the discussion, with Phillip and Junio both contributing refinements to the test suite.

## In brief

**Git daemon signal handling** -- Carlo Marcelo Arenas Belón submitted a series to fix zombie process cleanup on OpenBSD and crashes on AIX by modernizing signal handling with `sigaction()`. The discussion revealed complex platform-specific behaviors and POSIX deprecation concerns.

**Test infrastructure improvements** -- Jeff King's `test_seq -f` format option series saw final polishing, with documentation added and debate about format string validation. Performance measurements showed the shell implementation outperforms awk for typical test case sizes.

**Commit-graph bloom filter optimization** -- Lidong Yan and Junio Hamano confirmed the current bloom filter implementation enforces AND semantics for multiple pathspecs when OR semantics are needed, outlining necessary architectural changes.

**Fetch-prune optimization** -- Jeff King reviewed Phil Hord's O(N^2) to O(N log N) optimization for dangling ref reporting, suggesting potential interface improvements for warning output formatting.

**gitk external diff fix** -- A v6 patch fixing rename detection in gitk's external diff functionality appears ready after addressing all technical feedback through six iterations.

## On the radar

**Batched reference updates** -- Karthik Nayak's `kn/fetch-push-bulk-ref-update` series, temporarily removed from 'next' for fixes, now reports all feedback addressed and awaits confirmation before re-submission.

**SHA-256 by default** -- brian m. carlson's `bc/use-sha256-by-default-in-3.0` continues cooking in Junio's integration branches, preparing for Git 3.0's breaking changes.