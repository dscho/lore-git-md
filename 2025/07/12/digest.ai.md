# Git Mailing List Digest - 2025/07/12

**The day in brief.** A moderately busy Saturday with 30 emails across 11 threads. The standout development is the completion of Lidong Yan's bloom filter optimization series, now ready for merging after multiple review rounds. Platform compatibility issues dominated discussion, with RHEL 6 compilation problems and Windows line ending behavior generating multiple threads. The `core.commentChar=auto` deprecation reached final consensus, while a new submodule configuration proposal emerged.

## Notable threads

**Bloom filter optimization completes**  
Lidong Yan's six-part series optimizing pathspec handling with bloom filters has reached its final form, ready for merging. The changes introduce a new `struct bloom_keyvec` to handle multiple pathspec queries more efficiently while maintaining strict "all must match" semantics. Benchmarks show significant speedups: 1.33x faster for Git's own repository (55.1ms vs 73.1ms) and 7.5x faster for the LLVM repository (263ms vs 1.974s) when processing multiple literal pathspecs. The series went through multiple review rounds with feedback from Derrick Stolee and Junio Hamano, addressing all technical concerns while maintaining backward compatibility. The final patches include comprehensive test coverage and clean separation of test infrastructure from production code.

**RHEL 6 compilation woes**  
Азат Усманов reported a cascade of compilation failures when building Git 2.50+ on RHEL 6 systems, despite using the same GCC 13.4.0 compiler that works on RHEL 8. Initial missing `random.h` errors gave way to deeper syntax problems in `sane-ctype.h` macros after manual fixes. Phillip Wood pointed to an existing patch addressing the `sane-ctype.h` issues, but subsequent linking failures revealed missing `getrandom()` support in RHEL 6's older libc. Ramsay Jones suggested this may exceed Git's platform support policy (RHEL 8 being the current minimum), offering `CSPRNG_METHOD=` as a workaround. The thread highlights the challenges of maintaining compatibility with legacy enterprise Linux distributions.

**Windows line ending behavior debate**  
Jason Cho's report about unexpected CRLF line endings despite `core.eol=lf` settings sparked a detailed discussion about Git's line handling hierarchy. Torsten Bögershausen diagnosed the file as having been committed with CRLFs originally, explaining why global settings weren't overriding the stored format. Johannes Sixt proposed configuring `core.whitespace` to suppress CR markers in diffs, while Jason clarified their actual need was consistent LF endings for text analysis tools comparing file contents. The thread reveals tension between Git's attribute-based design and users wanting simpler global control, with no perfect solution emerging for Jason's specialized workflow needs.

**`core.commentChar=auto` deprecation finalized**  
The long-running discussion about deprecating `core.commentChar=auto` reached consensus, with Junio Hamano endorsing Oswald Buddenhagen's simplified approach. Instead of detailed unset instructions, users will be directed to `git config list --show-scope` to identify where `auto` is configured. The fail-safe behavior (erroring out rather than silently falling back to `#`) was unanimously agreed upon. This cleanly concludes a migration path that began when `auto` was marked deprecated in Git 2.40, with the change slated for Git 3.0.

## In brief

**PCRE2 defaults in meson builds** -- A patch changes meson's default PCRE2 dependency from enabled to disabled to match Makefile behavior, specifically addressing macOS issues with broken system PCRE2 libraries. Brian m. carlson questioned whether this should be platform-specific rather than global.

**GPG program path expansion** -- Junio Hamano acknowledged being mistaken about `gpg.program` supporting command-line arguments, as Andreas Schwab demonstrated the current implementation fails to parse them. The discussion continues about whether to treat this as a path or command configuration.

**Submodule configuration proposal** -- A new thread suggests defaulting `push.recurseSubmodules` to "check" and adding `clone.recurseSubmodules` to automatically initialize submodules, aiming to reduce common pitfalls for novice users while maintaining backward compatibility.