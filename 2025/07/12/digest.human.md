# Git Mailing List Digest - 2025/07/12

**The day in brief.** A moderately busy Saturday with 30 emails across 11 threads. The standout development is the completion of Lidong Yan's bloom filter optimization series, now ready for merging after multiple review rounds. Platform compatibility issues dominated discussion, with RHEL 6 compilation problems and Windows line ending behavior generating the most traffic. The `core.commentChar=auto` deprecation reached final consensus, while a new submodule configuration proposal emerged.

## Notable threads

### Bloom filter optimization completes

Lidong Yan's six-part series optimizing pathspec handling with bloom filters has reached its final form, with all substantive feedback addressed. The patches introduce `struct bloom_keyvec` to support multiple literal pathspec queries while maintaining the strict "all must match" requirement. Benchmark results show significant speedups - 1.33x in Git's own repository and 7.5x in the LLVM repository for multi-pathspec cases. The implementation carefully preserves fallback behavior for wildcard pathspecs while optimizing the literal case. Derrick Stolee and Junio Hamano have signed off, indicating this is likely to merge soon.

### RHEL 6 compilation woes

Азат Усманов reported a cascade of compilation failures when building Git 2.50+ on RHEL 6 systems, despite using the same GCC 13.4.0 compiler that works on RHEL 8. Initial missing `random.h` errors gave way to deeper `sane-ctype.h` macro syntax problems, and finally linker errors for `getrandom()`. Phillip Wood pointed to an existing patch for the header issues, while Ramsay Jones noted RHEL 6 may be too old for official support and suggested using `CSPRNG_METHOD=` as a workaround. The thread highlights the challenges of maintaining compatibility with enterprise Linux distributions that have decade-long support cycles.

### Windows line ending debates

Jason Cho's report about unexpected CRLF endings despite `core.eol=lf` settings sparked detailed discussion about Git's line ending handling hierarchy. Torsten Bögershausen diagnosed the file was committed with CRLFs originally, while Johannes Sixt suggested configuring `core.whitespace` to suppress `^M` markers in diffs. Jason later clarified their actual need is consistent LF endings for text analysis tools comparing content across files. The thread reveals tension between Git's attribute-based design and users wanting simpler global control, with no clear resolution yet for Jason's specialized workflow.

### GPG path expansion semantics

An ongoing discussion about `gpg.program` configuration took an interesting turn when Junio Hamano acknowledged his earlier claim about command-line argument support was incorrect. Andreas Schwab demonstrated the current implementation fails both as a path (no tilde expansion) and as a command (can't handle arguments). This leaves the setting's intended semantics unclear, with the thread now questioning whether to properly implement command parsing or document the limitations of the current path-like behavior.

## In brief

**`core.commentChar=auto deprecation finalized` --** Oswald Buddenhagen and Junio Hamano agreed to simplify the deprecation advice by recommending `git config list --show-scope` to locate `auto` settings, rather than providing specific unset commands. The fail-safe approach of erroring out on `auto` after Git 3.0 was confirmed.

**Meson PCRE2 default change** -- A patch proposes disabling PCRE2 by default in meson builds to match Makefile behavior, primarily addressing macOS's broken system PCRE2. Brian m. carlson questioned whether this should be macOS-specific rather than global.

**Submodule configuration proposal** -- A new thread suggests defaulting `push.recurseSubmodules` to "check" and adding `clone.recurseSubmodules` to automatically initialize submodules, aiming to prevent common novice pitfalls while maintaining backward compatibility.

## On the radar

**Rustification effort** -- While not active today, Ezekiel Newren's work to introduce Rust code into Git remains a contentious topic, particularly regarding platform support concerns raised by Randall S. Becker for NonStop systems. The discussion may resurface as Git 3.0 approaches.