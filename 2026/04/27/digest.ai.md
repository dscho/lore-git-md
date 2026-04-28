Here's the daily digest for 2026/04/27:

---

**The day in brief.** Monday saw moderate traffic with 64 emails across 17 threads, featuring several notable patch series nearing completion. Key developments include Christian Couder's promisor remote URL-based auto-configuration series reaching final form, Pablo Sabater's `git log --graph` improvements awaiting final review, and ongoing discussions about revision walk behavior and ref backend consolidation. Performance optimizations and documentation fixes rounded out the day's activity.

---

**Notable threads**

**Promisor remote URL auto-configuration finalized**  
Christian Couder's series enhancing promisor remote functionality with URL-based auto-configuration is now complete. The v2 series introduces `promisor.acceptFromServerUrl` for pattern-based remote configuration, with strict security rules for URL matching (exact scheme/port, host boundary constraints). The implementation includes comprehensive tests and documentation explaining the security model. The series builds on earlier LOP (Large Object Promisors) work and has received positive reviews from Patrick Steinhardt and Junio Hamano. With all feedback addressed, this represents a significant usability improvement for promisor remote workflows while maintaining strong security defaults.

**Remote group push implementation complete**  
Usman Akinyemi's feature adding remote group support to `git push` (mirroring existing `git fetch` functionality) has reached its final form in v4. The implementation now uses child processes via `run_command` for each remote, ensuring failures in one push don't affect others - following Junio Hamano's guidance. The series includes thorough test coverage and documentation about the behavior, particularly around the incompatibility with `--atomic` pushes. The shared group resolution logic between fetch and push commands has been refactored into `remote.c`, completing this long-running effort to bring push behavior in line with fetch's group capabilities.

**Graph rendering improvements for root commits**  
Pablo Sabater's v3 series fixing misleading `git log --graph` output for commits with excluded parents (including root commits) appears technically complete and awaits final review. The solution introduces a placeholder mechanism via an `is_placeholder` flag that keeps parentless commits "alive" for an extra row, forcing proper indentation of subsequent commits. The implementation handles both true root commits and commits with parents outside the displayed range through a "seems_root" concept. With all substantive feedback addressed and Junio indicating the implementation looks sound, this series may be stuck in review limbo despite being ready for merging.

**Revision walk behavior debate**  
Mirko Faina's series adding `--reverse=before` to revision walks has sparked discussion about the best user-facing terminology. While the technical implementation (including memory optimizations) is complete after four iterations, Junio Hamano and Johannes Sixt question whether `--reverse=before/after` will be intuitive, suggesting alternatives like `--reverse=oldest/newest` or even a separate `--max-count-oldest` option. Chris Torek contributed a deep analysis of the priority queue mechanics, suggesting the issue might be better solved at the queue level rather than through `--reverse`. The thread has shifted from implementation to interface design philosophy as the final UX polish is debated.

**Ref backend consolidation progresses**  
Karthik Nayak's v3 series consolidating object validation and peeling logic across Git's reference backends is now complete. The final patches modify the packed and reftable backends to use pre-peeled values from the `ref_update` structure rather than performing their own peeling operations. This eliminates duplicate work and ensures consistent behavior while simplifying backend implementations. The series has received thorough review from Patrick Steinhardt and Toon Claes, with all technical concerns addressed. This represents another step in the broader effort to move common ref operations into the shared layer.

---

**In brief**

**Commit signing regression fix** -- brian m. carlson's two-patch series fixes a regression where signatures for non-UTF-8 commit messages became invalid in Git 2.45.0+. The fix reorders operations in `commit_tree_extended()` to ensure UTF-8 validation happens before signing.

**Performance optimization for index-pack** -- Scott Bauersfeld's v3 patch increases I/O buffer sizes from 4KB to 128KB in `index-pack` and `unpack-objects`, showing 10-11% faster clones on FUSE filesystems. The change centralizes the buffer size definition as `DEFAULT_IO_BUFFER_SIZE`.

**Italian l10n fix** -- Matteo Beniamino's patch correcting misaligned "usage:" text in Italian translations is approved, though process questions about the Italian l10n team's status remain open.

**Test fix for Unix sockets** -- Johannes Schindelin fixed t5564-http-proxy.sh to handle Linux's 108-character path limit for Unix domain sockets by using $TMPDIR, following the pattern from t7528.

**Documentation formatting** -- Kristoffer Haugsbakk sent two patches fixing AsciiDoc formatting in the `git-log` man page's `--decorate` option description, standardizing delimiters and block syntax.

**Reintegrate script fix** -- A one-line patch corrects error message redirection in the Reintegrate script, ensuring warnings about malformed merge commits go to stderr rather than corrupting output files.

---

**On the radar**

**Alias regression fix** -- The discussion about fixing dotted alias handling has moved to documentation wording, with agreement to treat the feature as legacy while maintaining compatibility. Final wording pending Junio's review.

**Post-merge hook behavior** -- A new thread questions why `post-merge` doesn't run after conflict resolution, potentially revealing an oversight in Git's hook triggering logic.