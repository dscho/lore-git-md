Here's the Git mailing list digest for July 10, 2025:

## The day in brief

July 10 saw active development across multiple fronts, with 79 emails across 21 threads. The day was dominated by technical refinements to ongoing series - particularly Patrick Steinhardt's MIDX refactoring and the bloom filter optimization work - along with several platform-specific fixes. Junio's "What's cooking" report provided a comprehensive snapshot of the project's current state as Git 3.0 preparations continue.

## Notable threads

**MIDX tracking moves to per-source storage**  
Patrick Steinhardt's 8-part series to move MIDX tracking from global state to per-source storage saw extensive review from Taylor Blau and Justin Tobler. The changes restructure how Git manages multi-pack indexes by attaching them to individual `struct odb_source` instances rather than maintaining a global linked list. Reviewers validated the technical approach while suggesting minor improvements to commit messages and code organization. The series completed its transition by removing the now-unused global infrastructure in the final patch.

**Bloom filter optimization finalizes**  
Lidong Yan's bloom filter optimization series reached its final stages, now showing significant performance improvements (1.33x-7.5x speedups) for multi-pathspec queries. Derrick Stolee provided additional benchmarking data and suggested final stylistic refinements to the `bloom_keyvec` abstraction. The changes enable bloom filter usage with multiple literal pathspecs while maintaining the fast path for single pathspec cases. Junio noted some unused variables that were cleaned up in response.

**Git daemon signal handling progresses**  
Carlo Marcelo Arenas Belón's v4 series for Windows-compatible signal handling in `git-daemon` received final typo fixes from Junio before being marked ready for merging. The changes transition from `signal()` to `sigaction()` while maintaining Windows compatibility, though deeper questions about MinGW's SIGCHLD behavior remain open for future investigation. The series preserves BSD-style signal semantics while explicitly deferring pipe-based process tracking to later work.

**Documentation policy on pseudonyms clarified**  
The thread about allowing pseudonyms in Signed-off-by trailers reached consensus on wording, with Jeff King and brian m. carlson agreeing to use Wikipedia-inspired language rather than CNCF-style phrasing. The policy will emphasize persistent, verifiable identities over legalistic "real name" requirements. Jacob Keller later clarified that abandoned pseudonyms would be handled similarly to other contributor changes - by excising bad-faith work or finding new maintainers for good-faith contributions.

**Release strategy proposal sparks debate**  
Junio Hamano proposed changing Git's release model to time-based monthly tags (e.g., "Git 2.50.202508") to encourage distros to update more frequently. brian m. carlson expressed skepticism, citing distros' reluctance to update due to potential behavior changes and citing examples from Chromium and Firefox. The discussion highlighted tensions between maintaining stability and encouraging security updates.

## In brief

**HTTP language tag compliance** -- Brian Carlson proposed filtering invalid "C" and "POSIX" locale values from Accept-Language headers to meet HTTP standards, with debate about whether this fixes actual issues or is purely preventative.

**Amazon Linux 2 build fix** -- Patrick Steinhardt addressed header inclusion conflicts causing build failures with GCC 7.3.1 by adjusting `sane-ctype.h` to prevent macro collisions with system headers.

**Reflog memory leak fix** -- Jeff King and Jacob Keller refined the timing of config cleanup in reflog expiration to properly handle both GC and multi-ref operations.

**Meson build path handling** -- Ramsay Jones and Patrick Steinhardt discussed solutions for `libexecdir` path resolution in the Meson build system, with Junio suggesting they collaborate on a joint approach.

**VS Code scissor-line issue** -- Investigation continued into why `git commit --verbose` fails to strip text below the scissor-line marker in VS Code, with testers unable to reproduce using default settings.

## On the radar

**SHA-256 default preparation** -- brian m. carlson's series to enable SHA-256 by default in Git 3.0's breaking changes mode is queued for 'next' after positive reviews.

**Cruft pack optimizations** -- Taylor Blau's performance improvements for pack-objects and MIDX handling around cruft packs received final approval and are production-validated at GitHub.