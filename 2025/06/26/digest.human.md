# Git Mailing List Digest - 2025/06/26

**The day in brief.** A moderately busy Thursday with 53 emails across 14 threads, dominated by an extensive technical discussion about signal handling in git-daemon. Key developments include a new RFC proposing pipe-based child process tracking as an alternative to signal handling, progress on the bloom filter optimization series, and several bugfixes for edge cases in rebase and commit message handling.

## Notable threads

### Git daemon signal handling debate intensifies

The ongoing discussion about git-daemon's signal handling saw significant activity today, with multiple approaches now on the table. Carlo Marcelo Arenas Belón submitted a v3 series implementing SA_RESTART toggling to address zombie process cleanup on OpenBSD and crashes on AIX, while also proposing an RFC alternative using self-pipes for child process tracking. Phillip Wood advocated for a simpler approach that relies on existing EINTR handling, arguing the SA_RESTART manipulation adds unnecessary complexity. Junio Hamano initially favored Phillip's minimal solution but Carlo pushed back, citing concerns about historical changes to EINTR assumptions. The thread also included detailed discussion of Windows-specific signal handling quirks, with Carlo suggesting to involve Johannes Schindelin for additional Windows expertise. This architectural debate appears to be reaching a critical decision point, with three distinct solutions now under consideration.

### Bloom filter optimization nears completion

Lidong Yan's series to enable bloom filter acceleration for multiple pathspec queries in revision traversal received positive feedback from Junio Hamano, who confirmed the existing tests already cover correctness by comparing bloom-filtered results with non-bloom results. Junio encouraged adding more test cases specifically to guard against false negatives, while Lidong agreed to implement several code style improvements suggested in review. The series appears technically sound and is now in its final polishing phase, addressing minor points about memory management patterns and test coverage before potential merging.

### Interactive rebase bug in worktrees

Denilson Sá Maia reported a long-standing issue where `git rebase -i` fails in secondary worktrees with spurious "local changes would be overwritten" errors, while non-interactive rebase works fine. The thread narrowed down potential causes, ruling out hook interference as the root issue. This appears to be a genuine worktree interaction bug rather than user error, though the intermittent nature suggests it may involve race conditions or improper state tracking during the rebase's temporary checkouts. The discussion continues to investigate this edge case that affects secondary worktrees while primary worktrees function correctly.

### Comment character selection fix

Ayush Chandekar's bugfix for auto-selecting comment characters during rebase conflicts progressed to v2 with positive reviews. The patch modifies `adjust_comment_line_char()` to skip conflict marker sections when choosing a comment character, preventing these markers from appearing in final commit messages. Junio Hamano approved the technical approach while Kristoffer Haugsbakk suggested test improvements, which Ayush incorporated in v2. The patch now appears ready for inclusion, having addressed all feedback about both implementation and test methodology.

## In brief

**Git daemon socket handling fix** -- Carlo Marcelo Arenas Belón submitted a v2 patch to correct EINTR handling during socket acceptance, adding retry logic that prevents the daemon from incorrectly advancing to the next socket when system calls are interrupted.

**Line length configuration debate** -- Discussion continued about setting max_line_length in .editorconfig, with Junio Hamano reframing the debate around balancing readability guidelines with practical coding needs rather than specific character counts.

**Commit message typo fixes** -- Kristoffer Haugsbakk pointed out three minor typos in Sebastian Andrzej Siewior's byte-order handling series ("implemenated", "wich", and "otimized"), which had already been deferred to Git 2.51.

**Git log decoration behavior** -- Jeff King identified commit 92156291ca as changing the default decoration filtering behavior that now excludes bisect refs, suggesting `log.initialDecorationSet=all` as a workaround to restore previous behavior.

**Obsolete FD_SETSIZE check removal** -- Carlo Marcelo Arenas Belón and Phillip Wood agreed on removing an obsolete file descriptor size check in git-daemon that became unnecessary after the 2005 switch from select() to poll().

## On the radar

**Fast-export signature handling** -- Elijah Newren's detailed analysis of signature representation across hash algorithms in fast-export/fast-import revealed deeper technical requirements than initially apparent, particularly around the interaction between signature handling and Git's hash transition architecture. The discussion is ongoing but hasn't seen new messages today.