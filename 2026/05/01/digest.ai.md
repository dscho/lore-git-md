# Git Mailing List Digest - 2026/05/01 (Friday)

**The day in brief.** A moderately active Friday with 50 emails across 13 threads, featuring significant progress on several fronts. The standout developments include a v2 of the `git url-parse` plumbing command, substantive reviews of the `git format-rev` series, and a new proposal for automatic branch pruning. Security-conscious fixes for HTTP proxy handling and Unix domain socket tests also moved forward.

## Notable threads

### **New `git url-parse` plumbing command (v2)**  
The second iteration of a feature to expose Git's URL parsing logic as a reusable command has landed, addressing feedback from v1. The 8-patch series refactors internal URL handling (renaming `enum protocol` to `enum url_scheme` for RFC compliance) before implementing the new command. Key capabilities include parsing all Git URL formats (including SCP-style and IPv6), extracting specific components (scheme, host, path etc.), and thorough test coverage. The implementation carefully handles edge cases like bracketed hostnames and `~user` paths while maintaining compatibility with Git's existing URL handling. Reviewers Torsten Bögershausen and Ghanshyam Thakkar contributed substantive feedback on SCP syntax and malformed URL handling that shaped the final design.

### **`git format-rev` stream processing concerns**  
Phillip Wood's detailed review of Kristoffer Haugsbakk's experimental `git format-rev` command identified critical gaps in its stream handling design. The discussion revealed that the command lacks proper delimiters between input messages and their responses, which could cause problems in single-process use cases with variable-length output. Kristoffer acknowledged the issues (including limited concurrency experience affecting initial design) and plans revisions to add null-terminator modes for robust stream processing. This exchange exemplifies Git's rigorous review culture, with experienced contributors ensuring new features meet robustness standards before graduating from experimental status.

### **Automatic branch pruning proposal**  
Harald Nordgren introduced a new `fetch.pruneBranches` configuration option to automate local branch deletion when upstream branches are pruned. The carefully designed feature offers three modes: `false` (default/no change), `safe` (delete only if branch tip is reachable from upstream), and `force` (unconditional deletion). The implementation preserves checked-out branches regardless of mode and includes thorough documentation and test coverage. This first iteration appears well-considered, balancing automation with safety nets for unpushed work. The design follows Git's configuration patterns and could significantly streamline branch cleanup workflows if adopted.

## In brief

**HTTP proxy security hardening** -- A patch now rejects proxy URLs with invalid schemes (like "htpp://") rather than silently stripping the scheme. The change centralizes validation in a new `set_curl_proxy_type()` helper and adds explicit error messages.

**Worktree function rename** -- Phillip Wood renamed `get_worktree_from_repository()` to `get_current_worktree()` to better reflect its behavior after a previous change made it always return the current worktree, as suggested by Junio.

**FreeBSD commit failure diagnosis** -- Jeff King suggested a misconfigured pager (`pager.commit`) might explain silent failures in `git commit --verbose` on FreeBSD, though the theory awaits confirmation from the original reporter.

**Ruby rename detection behavior** -- Phillip Wood explained why moving Ruby classes between namespaces triggers delete/add rather than renames (indentation changes reduce similarity below Git's threshold), suggesting a workaround via split commits.

**Windows large object fixes** -- Discussion continued on test ergonomics for Johannes Schindelin's >4GB object handling series, with Jeff King and Derrick Stolee debating how to manage the tests' significant resource requirements (160s runtime, 4-8GB disk space).

## On the radar

**git-gui worktree safety** -- The debate over launching git-gui from .git directories intensified with Mark Levedahl demonstrating data corruption risks in Johannes Sixt's compatibility-preserving proposal, favoring Shroom Moo's stricter v4 approach. Maintainer input is now needed to resolve the safety vs. compatibility tradeoff.