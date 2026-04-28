# Here's the daily digest for 2026/04/27:

---

**The day in brief.** Monday saw moderate traffic with 64 emails across 17 threads, featuring several notable patch series nearing completion. Key developments include Christian Couder's promisor remote URL-based auto-configuration reaching merge readiness, Pablo Sabater's `git log --graph` improvements awaiting final review, and ongoing discussions about revision walk behavior and ref backend consolidation. The day also included routine documentation fixes and test improvements.

---

### Notable threads

### Promisor remote URL-based auto-configuration finalized

Christian Couder's series enhancing promisor remote functionality with URL-based auto-configuration is now complete. The v2 series introduces `promisor.acceptFromServerUrl` for pattern-based remote configuration, with strict security constraints for URL matching (exact scheme/port requirements, host boundary rules). The implementation includes comprehensive tests and documentation explaining the relationship with existing `acceptFromServer` functionality. Patrick Steinhardt and Junio C Hamano have reviewed the series positively, with all technical concerns addressed. This represents a significant usability improvement for organizations managing multiple promisor remotes.

### `git log --graph` improvements for root commits

Pablo Sabater's v3 patch series fixing misleading graph rendering for commits with excluded parents (including root commits) appears technically complete after addressing all prior feedback. The solution introduces a placeholder mechanism via an `is_placeholder` flag to properly indent unrelated commit lineages. While some visual quirks in diagonal line rendering remain for follow-up work, the core functionality is well-tested and documented. The series has been in "What's cooking" since early April with no outstanding technical objections, suggesting it may be awaiting final maintainer approval.

### Revision walk behavior debate continues

Mirko Faina's `--reverse=before` series for revision walks has sparked a broader discussion about the interaction between limiting and display options. While the technical implementation (including memory optimizations) is settled after four iterations, Junio Hamano and Johannes Sixt raised concerns about the intuitiveness of the `before/after` terminology. Chris Torek contributed a deep analysis of the underlying priority queue mechanics, suggesting the issue might be addressed more fundamentally by modifying `--max-count` behavior rather than overloading `--reverse`. The thread has shifted from implementation details to interface design philosophy.

### Ref backend consolidation progresses

Karthik Nayak's v3 series to consolidate object validation and peeling logic across ref backends is now complete. The final patches standardize how backends handle peeled object IDs, moving this functionality from packed and reftable backends into the shared refs layer. The changes maintain existing behavior while eliminating duplicate peeling operations. Patrick Steinhardt and Toon Claes have provided thorough review throughout the series' development, with all technical concerns addressed. This represents another step in the ongoing effort to reduce backend-specific code.

## In brief

**Reintegrate script error reporting fix** -- Corrects a redirection issue where warnings about malformed merge commits were incorrectly sent to stdout instead of stderr, which could silently corrupt output in maintainer workflows.

**Unix domain socket test fix** -- Johannes Schindelin modified t5564-http-proxy.sh to create sockets in $TMPDIR to avoid Linux's 108-character path length limit, following the pattern from t7528's SSH agent socket handling.

**Documentation formatting fixes** -- Kristoffer Haugsbakk corrected AsciiDoc syntax in the `git-log` man page's `--decorate` option description, standardizing delimiters and fixing block formatting.

**Post-merge hook behavior question** -- A user inquiry about why `post-merge` doesn't trigger after conflict resolution with `git merge --continue`, highlighting a potential gap between user expectations and current implementation.

## On the radar

**Partial clone optimizations** -- Elijah Newren's recently merged series optimizing `git cherry` and `git grep` for partial clones may see follow-up work to address Derrick Stolee's suggestion to centralize prefetch logic in `diff.c`.

**Alias regression fix** -- The fix for dotted alias handling appears ready pending final documentation polish about treating this as a legacy feature, with backporting to maint still under consideration.