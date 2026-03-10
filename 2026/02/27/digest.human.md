Here's the daily digest for 2026/02/27:

## The day in brief

A busy Friday with 107 emails across 27 threads, featuring significant progressions in several major efforts. The fsmonitor Linux implementation nears completion with final timeout handling discussions, while the configurable cover letter formatting series reaches consensus on syntax. Notable new contributions include performance optimization for high-concurrency upload-pack operations and a serious bug report about shallow fetch behavior.

## Notable threads

**Linux fsmonitor timeout handling finalized**  
Paul Tarjan and Junio Hamano concluded discussion on the fsmonitor daemon's timeout behavior in the Linux inotify implementation. The 1-second timeout for cookie synchronization only triggers when filesystems fail to deliver expected events (like overlayfs in containers), not on truly idle systems. The daemon's two-phase notification approach ensures the timeout acts as a safety net without affecting normal operation where events arrive in sub-millisecond times. This marks the last major review point before merging the Linux fsmonitor support.

**Cover letter formatting syntax settled**  
After five iterations, Mirko Faina's configurable cover letter formatting series reached consensus on requiring explicit "log:" prefixes for format strings in both command-line and configuration. Junio Hamano approved the final design, which maintains consistency between interfaces while allowing future format additions. The implementation now includes comprehensive test coverage and documentation for the new `--cover-letter-format` option and `format.commitListFormat` config.

**Upload-pack performance optimization**  
Patrick Steinhardt proposed reducing lock contention in `git-upload-pack` during high-concurrency scenarios by buffering more data before writing. The change cuts write syscalls by ~67% when cloning large repositories. While Junio approved the approach, brian m. carlson and Jeff King raised concerns about keepalive interference during slow transfers, prompting discussion of alternative optimizations like increasing `csum-file.c`'s buffer size or using `writev()`.

**Shallow fetch boundary bug**  
A serious correctness issue was reported in `git fetch --shallow-since`, where merge commits straddling the cutoff date can produce invalid shallow repositories. The bug causes boundary commits to be marked in `.git/shallow` without being included in the packfile, violating repository invariants. This affects date-based shallow fetches (but not depth-based ones) and could impact any repository with merges across the cutoff date.

## In brief

**Submodule remote handling finalized** -- The v4 patch fixing submodule remote name assumptions is now complete with all test portability issues addressed and ready for integration.

**Duplicate option detection** -- René Scharfe implemented systematic validation for duplicate long options in Git's parser, addressing cases Junio identified while avoiding performance regressions.

**gitk link color customization** -- Wang Zichong's patch to make gitk's hyperlink colors configurable was approved with minor wording tweaks from Johannes Sixt and Junio.

**Oidmap cleanup API** -- A 5-part series refactored oidmap memory management to use explicit callbacks instead of boolean flags, improving type safety across rev-list, odb, and sequencer code.

**Test suite improvements** -- Multiple threads addressed test reliability: fixing exit code handling in t3700 (Siddharth Shrimali) and modernizing assertions in t3310 (Francesco Paparatto).

## On the radar

**The_repository removal GSoC proposal** -- Tian Yuchen's proposal to reduce global state continues evolving through community feedback, now addressing configuration parsing distinctions raised by Phillip Wood.

**Repository path reporting** -- Eslam Reda Ragheb's series expanded scope to include metrics functionality while reviewers focus on stabilizing the core path interface design.

**Bundle URI redirect handling** -- A bug report highlighted that `git clone --bundle-uri` fails to follow HTTP redirects, affecting real-world use cases like Linux kernel clones.