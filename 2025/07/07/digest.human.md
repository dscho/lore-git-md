# Git Mailing List Digest - 2025/07/07

**The day in brief.** A moderately active day with 83 emails across 29 threads, featuring several notable developments: final approvals for multiple patch series (string-list modernization, `git apply --intent-to-add` fixes, and SSH signing memory leak), continued discussion on metadata preservation in email workflows, and a new Windows regression report. The day saw revived discussions on Perl test conversions and BSD memory detection, while documentation standardization efforts reached completion.

## Notable threads

**Perl test conversion follow-up fixes**  
Patrick Steinhardt's Perl removal series, merged earlier, required follow-up fixes for edge cases in test scripts converted to shell/awk/sed. The discussion revealed two issues in t5333-pseudo-merge-bitmaps.sh: an incomplete sed conversion for creating refs (missing pipe terminator) and problematic pattern extraction. While the test passed incorrectly due to empty output matching nothing, the fixes ensure proper reference creation and pattern matching. This demonstrates the importance of rigorous output validation when converting test scripts, even after thorough pre-merge review.

**Promisor-remote capability extension ready**  
Junio Hamano is preparing to merge Christian Couder's v5 series extending the promisor-remote protocol to support configurable validation of remote attributes (`partialCloneFilter` and `token`). The changes since v4 are minimal - expanded commit message explanations, fixed typos, and added code comments. The implementation provides both server-side advertisement and client-side verification while maintaining backward compatibility. With CI passing and all feedback addressed, this completes a series that adds security through strict field handling between servers and clients.

**String-list modernization finalized**  
Junio C Hamano confirmed he will merge shejialuo's v3 series modernizing string-list tests and implementation after Patrick Steinhardt's final approval. The 8-patch series converts all functional tests from shell to properly isolated C unit tests while improving the core implementation with warning fixes, parameter removal, and logic simplifications. The only remaining open item was an optional parameter reorganization deemed non-blocking, marking successful completion of this test modernization effort that builds on the author's previous string-list work.

**Metadata preservation in email workflows**  
Junio Hamano proposed a generic `--extra-headers` mechanism for `git format-patch` as an alternative to the specific X-Change-ID solution, sparking extensive discussion about commit metadata preservation. The thread explored legal considerations (OFAC sanctions, license enforcement), technical approaches (headers vs trailers), and email client behaviors. While consensus favors allowing pseudonyms, the discussion revealed complex tradeoffs between inclusivity, legal compliance, and technical implementation that will require careful policy wording.

**Windows 11 regression report**  
Fabio Frumento reported Git for Windows 2.50.0 failing to launch on Windows 11 Pro 64-bit, with the terminal window appearing then closing immediately. The issue is reproducible and reverting to 2.49.0 resolves it, though the report lacks diagnostic details. This appears to be a Windows-specific regression that maintainers will need to investigate, particularly given the clear reproduction case (upgrade breaks, downgrade fixes).

## In brief

**Reftable as default for Git 3.0** -- Junio Hamano acked Patrick Steinhardt's v2 series to establish reftable as the default ref storage format in Git 3.0, maintaining the versioned default transition pattern with build-time guard.

**BSD memory detection fix** -- Carlo Marcelo Arenas Belón's v5 patch fixing `git gc --auto` memory detection on BSD systems was approved, simplifying from v4's endian-specific handling while maintaining key functionality improvements.

**`git fetch --prune` optimization** -- A v4 series addressing O(N^2) performance in dangling ref reporting awaits final review, showing 410s->1s improvement in test cases through sorted prune lists with binary search.

**`for-each-ref` pagination naming** -- Consensus formed on using `--start-after` for the new pagination option after extensive discussion of semantics, with the implementation providing O(log N) seeking across all ref backends.

**DCO/pseudonym policy** -- Discussion continued on relaxing real-name requirements for Signed-off-by trailers, with legal considerations (OFAC sanctions, license enforcement) emerging as key factors alongside inclusivity goals.

**On the radar**

**Git Mini Summit announcement** -- Patrick Steinhardt announced a half-day community event alongside Open Source Summit Europe in Amsterdam on August 28th, with talk proposals due July 28th.