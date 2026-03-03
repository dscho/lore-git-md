# Git Mailing List Digest - 2025/06/07

**The day in brief.** A moderately active Saturday with 16 emails across 9 threads, featuring platform compatibility updates, ongoing refactoring discussions, and Junio's "What's cooking" status report. The most notable developments include Windows build failures from byte-order changes, Solaris build configuration updates, and type safety fixes for cURL interactions.

## Notable threads

### Windows build failures from bswap.h changes

Junio Hamano reported that Sebastian Siewior's byte-order handling refactoring has caused significant Windows build failures, prompting a revert from the integration tree. This comes after extensive discussion about big-endian system support, revealing unanticipated Windows compatibility issues. The thread's focus now shifts from s390x concerns to cross-platform testing requirements, with the changes needing broader validation before reconsideration.

### Solaris build configuration updates

Brad Smith submitted a patch updating build flags for Solaris 11+, removing unnecessary compatibility flags now that modern versions provide strcasestr(), memmem(), and mkdtemp(). Jonathan Nieder and Junio Hamano reviewed the changes positively, sparking a broader discussion about maintaining support for older Solaris versions (5.6-5.10) that reached EOL in 2006. The immediate patch is uncontroversial, but the thread may lead to future work on version support lifecycle management.

### cURL type safety issues resolved

Peter Michael Green reported type safety warnings in Git's cURL interactions when building against curl 8.14, affecting multiple components. Todd Zullinger noted these issues were already addressed by Jeff King's recent patch series (c80760403b), now in master and slated for v2.50.0-rc2. This quick resolution highlights the project's proactive handling of platform compatibility concerns.

## In brief

**Stash -p/--patch fixes** -- Phillip Wood's v3 series addressing option parsing regressions received additional test refinement suggestions from Martin Ågren, focusing on pathspec handling edge cases.

**IMAP send authentication** -- Aditya Garg's patch making CRAM-MD5 fail gracefully without OpenSSL was reviewed, with discussion about ideal series organization versus practical rebase considerations.

**Submodule hardcoding** -- Patrick Steinhardt endorsed Jacob Keller's approach to modifying deprecated functions in remote.c as part of the broader `the_repository` removal effort.

**Fetch optimization** -- M Hickford raised questions about handling interrupted prefetch operations in poor network conditions, potentially prompting future documentation or implementation refinements.

## On the radar

**What's cooking** -- Junio's status update highlighted 50+ active topics, with notable focus on memory leak fixes, API cleanups, and performance improvements like batched ref updates and path-based delta compression. Several topics are marked as expecting rerolls.