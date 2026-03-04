Here's the Git mailing list digest for July 7, 2025:

## The day in brief
A moderately busy day with 83 emails across 29 threads, featuring several patch series reaching final approval, ongoing design discussions, and a Windows regression report. Key developments include resolution of the BSD memory detection fix, progress on the `for-each-ref` pagination feature, and substantive policy discussions around contributor identity requirements.

## Notable threads

**Perl test conversion follow-ups**  
Patrick Steinhardt's team addressed edge cases in the Perl-to-shell test conversion effort, fixing an awk warning in t4150-am.sh and correcting a sed command in t5333-pseudo-merge-bitmaps.sh that was causing silent test passes. These minor but important fixes demonstrate the ongoing refinement needed after large-scale test infrastructure changes.

**Reftable default preparation**  
Junio Hamano acked Patrick Steinhardt's v2 series to make reftable the default ref storage format in Git 3.0, confirming the thread has progressed to v3 with expanded documentation and test coverage. The changes introduce a versioned default transition with build-time guard while allowing runtime opt-in via `feature.experimental`.

**BSD memory detection finalized**  
After multiple iterations addressing type safety and platform compatibility, Carlo Marcelo Arenas Belón's v5 patch for BSD system memory detection in `git gc --auto` was approved. The solution simplifies from v4's endian-specific handling to a more maintainable approach using `uint64_t`/`uint32_t` fallbacks while resolving FreeBSD build issues.

**`for-each-ref` pagination naming**  
The thread reached consensus on using `--start-after` for the new pagination option after extensive discussion of alternatives (`--skip-until`, `--start-with`). The implementation provides O(log N) seeking across all ref backends with comprehensive test coverage, now awaiting final merge with only minor wording fixes remaining.

**Contributor identity policy**  
A substantive discussion emerged around relaxing real-name requirements for Signed-off-by trailers, with Junio Hamano and others weighing legal considerations (OFAC sanctions, license enforcement) against inclusivity goals. The thread shows movement toward allowing pseudonyms while maintaining traceability, with Linux kernel's "known identity" policy serving as precedent.

## In brief

**Bitmap memory leak fixes** -- Junio called for final confirmation on Lidong Yan's series fixing memory leaks in pack-bitmap.c, which has undergone six iterations with Taylor Blau and Jeff King's review.

**Submodule config improvements** -- K Jayatheerth's v8 series addressing submodule configuration management is ready for final review, with patch 1/2 already approved and patch 2/2 handling redundant active entries.

**SSH signing tempfile fix** -- redoste's v3 patch fixing a tempfile leak in SSH signing operations was queued after incorporating test refinements from Patrick Steinhardt and Phillip Wood.

**`git apply --intent-to-add`** -- Raymond Pasco's bugfix series restoring correct behavior for this long-broken feature was approved with expanded commit messages explaining the index handling changes.

**Windows 11 regression** -- Fabio Frumento reported Git for Windows 2.50.0 failing to launch on Windows 11 Pro, with version 2.49.0 working correctly, marking a potential issue for Dscho to investigate.

## On the radar

**Git Mini Summit** -- Patrick Steinhardt announced a half-day community event alongside Open Source Summit Europe in Amsterdam on August 28th, with talk proposals due July 28th.

**Signature handling in fast-export** -- Christian Couder's series remains stalled since June 19th, with Junio considering marking it "Stalled" pending response to design questions about signature format representation.

**Remote naming validation** -- Jeff King's patch to prevent ambiguous remote names during `git remote add` is nearing completion after addressing parameter naming and slash handling feedback.