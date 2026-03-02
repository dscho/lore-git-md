# Git Mailing List Digest — 2025/03/02

**The day in brief.** A moderately active Sunday with 20 emails across 13 threads, featuring continued discussion on several technical fronts. The most notable developments include progress on removing `MAX_REVS` limitations in `show-branch`, a proposed migration of the mentoring list from Google Groups to Discord, and an intriguing proposal for distributed inline code comments. Several smaller fixes and improvements rounded out the day's traffic.

## Notable threads

### Refactoring show-branch's commit flag storage

The ongoing effort to refactor `builtin/show-branch.c` saw further technical discussion today. Ghanshyam Thakkar built on Junio Hamano's earlier response, identifying two key requirements for fully removing the `MAX_REVS` constraint: converting static arrays to heap allocation and implementing dynamic bit storage. The thread has reached consensus on using the existing `commit_slab_with_stride` infrastructure for bit storage, with today's discussion focusing on practical execution details. Thakkar suggested either a custom bitset API or reuse of the existing `ewok.h` bitmap implementation, maintaining alignment with the previously agreed solution path while exploring implementation options.

### Mentoring list migration to Discord

Christian Couder joined the discussion about replacing the problematic git-mentoring@googlegroups.com infrastructure, supporting the proposed migration to Discord while also suggesting the #git-devel IRC channel and main Git mailing list as additional alternatives. The thread shows growing consensus that the Google Groups list should be retired due to its maintenance issues, with the remaining questions focusing on which channels to formally endorse and how to document them. This community infrastructure discussion appears to be moving toward implementation with broad agreement on the shutdown but remaining details to resolve.

### Distributed inline code comments proposal

ZheNing Hu's proposal for adding GitHub/GitLab-style inline code comments to Git while maintaining its distributed nature generated thoughtful responses. Randall Becker suggested implementing this as an ancillary data structure tied to commits and line ranges, with content blobs that could be signed. He noted the system would need push/fetch mechanisms similar to `--notes` and highlighted challenges with non-core Git server support and potential conflicts with existing concepts in GitLab/GitHub. The discussion continues to explore implementation feasibility, particularly around distributed synchronization and compatibility with existing Git infrastructure.

## In brief

Nikolay Shustov proposed several new configuration options and command-line flags for git-p4, along with a bugfix for file handling after revert operations. The changes improve workflow around shelving operations and workspace management.

A new contributor submitted a patch adding Verilog language support to Git's built-in diff patterns, modifying `userdiff.c` to better handle Verilog source code files.

David Mandelberg submitted a bugfix series for bash completion issues with remote names containing slashes, though performance concerns were raised about the current implementation's scaling with many remotes.

A build system fix corrected an oversight in the Meson configuration where documentation file extensions weren't updated when the project switched from `.txt` to `.adoc`.

A minimal housekeeping patch extended `.editorconfig` to include `.bash` files, ensuring consistent formatting for Git's Bash completion script.

## On the radar

A bug report surfaced about PKCS#11 authentication failing due to URI escaping issues in Git 2.48.1, with the reporter noting this worked correctly in earlier versions. The thorough report provides clear reproduction steps but no proposed fix yet.

The question about comprehensive developer documentation for Git's internals may spark further discussion about onboarding resources, especially with Summer of Code participation mentioned as motivation.