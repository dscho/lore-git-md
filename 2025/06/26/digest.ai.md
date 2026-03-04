# Git Mailing List Digest - 2025/06/26

**The day in brief.** A moderately active day with 53 emails across 14 threads, dominated by an extensive technical discussion about signal handling in git-daemon. Other notable topics include bloom filter optimizations, line length style debates, and several bugfixes. The signal handling thread saw multiple architectural approaches proposed and debated, while other discussions focused on polishing near-final patch series.

## Notable threads

### Git daemon signal handling debate intensifies

The ongoing discussion about signal handling in git-daemon saw significant activity today, with multiple approaches proposed and debated. Carlo Marcelo Arenas Belón submitted a v3 series implementing SA_RESTART toggling to address zombie process cleanup on OpenBSD and crashes on AIX. Phillip Wood countered with arguments for a simpler approach that relies on existing EINTR handling, which Junio Hamano initially endorsed. Carlo then pushed back, citing concerns about historical changes invalidating EINTR assumptions, and later proposed an RFC patch implementing a self-pipe mechanism as an alternative solution.

The discussion revealed deep technical considerations around platform-specific signal behavior, with particular focus on Windows compatibility. The thread now presents three distinct architectural paths forward: SA_RESTART manipulation (Carlo's v3), documentation-only minimalism (Phillip's suggestion), and the self-pipe approach (Carlo's RFC). The debate remains collegial but highlights fundamental questions about how Git should handle platform-specific edge cases in core infrastructure.

### Bloom filter optimization nears completion

Lidong Yan's series optimizing pathspec bloom filters moved closer to completion after addressing Junio Hamano's review feedback. The discussion focused on test coverage and implementation details of the `struct bloom_keyvec` changes. Junio confirmed that existing tests already verify correctness by comparing bloom-filtered results with non-bloom results, but encouraged additional test cases to specifically guard against false negatives. Lidong acknowledged all feedback and committed to improving the cover letter wording and adding more test coverage in v2.

This series represents a performance optimization for revision traversal with multiple pathspec elements, and the technical discussion today focused on final verification and documentation improvements before potential merging. The core approach remains unchanged, with the conversation now centered on ensuring robust test coverage for the optimization.

### Line length style debate continues

The discussion about enforcing line length limits in Git's code style saw continued debate today. Karthik Nayak's proposal to set a 120-character limit in .editorconfig faced pushback from multiple contributors. Junio Hamano reframed the discussion around balancing the documented 80-column guideline with practical readability needs, while Justin Tobler raised concerns about how the setting affects vim's format operator behavior.

The conversation has evolved from a simple number debate to a more nuanced discussion about the philosophy of line length enforcement, including how tooling should interact with developer judgment. No consensus was reached, but the discussion revealed important considerations about how style guidelines affect real-world workflows across different editors and development environments.

## In brief

**Fast-export signature handling** -- Elijah Newren provided detailed technical analysis of multiple signature support across hash algorithms in response to Christian Couder's v4 patch, clarifying signature representation requirements.

**JavaScript diff driver tests** -- Johannes Sixt identified several test file formatting issues in the JavaScript pattern matching series, including incorrect marker placement and missing newlines.

**bswap.h typo fixes** -- Kristoffer Haugsbakk corrected three documentation typos in Sebastian Andrzej Siewior's byte-order handling series.

**git-credential-netrc follow-up** -- Maxim Cournoyer acknowledged a minor threading irregularity in their v4 submission, with Junio confirming the technical content matches what was already merged.

**Interactive rebase worktree bug** -- Denilson Sá Maia reported persistent issues with `git rebase -i` failing in secondary worktrees with false "local changes" errors, with discussion ongoing about potential hook interactions.

**Commit message comment character fix** -- Ayush Chandekar's v2 patch addressing comment character selection during rebase conflicts received positive reviews, with test improvements suggested by Kristoffer Haugsbakk.

**Git daemon socket handling** -- Carlo Marcelo Arenas Belón submitted a v2 patch fixing EINTR handling during accept() calls, adding retry logic to prevent unnecessary socket advancement.

**git log bisect ref decoration** -- Jeff King identified commit 92156291ca as changing the default decoration filtering behavior that now excludes bisect refs, suggesting a config workaround.

## On the radar

**Rustification effort** -- While not discussed today, Ezekiel Newren's ongoing work to introduce Rust code into Git remains a significant long-term effort that may resurface soon.

**ODB abstraction** -- Patrick Steinhardt's object database abstraction work continues in the background, with potential for new patches in coming days.

**Documentation synopsis conversion** -- Jean-Noël Avila's effort to standardize man page formatting remains active, though no new patches appeared today.