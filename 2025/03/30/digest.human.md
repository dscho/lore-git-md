# Git Mailing List Digest — 2025/03/30

**The day in brief.** A moderately active Sunday with 23 emails across 10 threads sees several long-running threads reach their final polish stages. Key developments include the completion of Bash function detection improvements, resolution of Windows build issues for reftable decoupling, and final refinements to `git blame` porcelain output. Documentation modernization continues with a new series converting command synopses.

## Notable threads

**Bash function detection finalized**  
Moumita Dhar's GSoC project to improve shell script recognition in Git's diff machinery reaches completion with v4 patch addressing the last maintainer feedback. The series enhances Bash function detection to handle multiline definitions and line continuations while expanding coverage of shell syntax elements like parameter expansions. After multiple rounds of technical implementation, today's update focuses on test file formatting - ensuring proper newlines where shell script concatenation might occur. The changes maintain backward compatibility and include comprehensive test coverage, appearing ready for final maintainer approval.

**Reftable decoupling clears Windows hurdles**  
Patrick Steinhardt's effort to remove Git-specific dependencies from the reftable library (`ps/reftable-sans-compat-util`) receives final confirmation from Johannes Schindelin that Windows build issues are resolved. The 18-commit series systematically eliminates Git-specific utilities (BUG(), COPY_ARRAY(), POSIX wrappers) to improve portability, a key architectural step for ref backend abstraction. With CI failures addressed through workarounds for MIMALLOC code, Junio marks the topic for promotion to 'next', clearing its path through the integration pipeline.

**`git blame` porcelain output refined**  
The long-running thread about marking ignored/unblamable lines in `git blame` porcelain output reaches its final polishing stage. Karthik Nayak's implementation adds proper metadata lines ("ignored"/"unblamable") while maintaining backward compatibility. Today's discussion catches a minor output formatting issue (redundant newlines from `puts()`) and improves test robustness by switching from line counting to full output verification with `test_cmp`. All major design decisions are settled, with this final version addressing the last identified issues before merging.

**Revision walker edge case fixed**  
Matt Hunter's patch addressing `--left-only`/`--right-only` behavior with unrelated histories evolves through test improvements. The core fix (setting `revs->limited = 1` in revision.c) remains stable while the verification methodology improves - replacing Bash process substitution with portable temporary files and shifting from line counting to semantic validation of disjoint commit sets. The changes mirror a similar solution for `--cherry-pick` and demonstrate Git's attention to both correctness and cross-platform testing.

## In brief

Documentation modernization continues with a 5-patch series converting `git-reset`, `git-rm`, and `git-mv` man pages to use modern AsciiDoc synopsis blocks and standardized formatting. The changes are purely presentational, applying mechanical updates to command synopses and option references while maintaining all technical accuracy.

Nikolay Shustov follows up on his `git p4` error message encoding fix, clarifying the failure scenario (non-UTF-8 Perforce messages during clone) and defending the class-based refactoring approach against review feedback. The thread continues discussion about Python 2/3 compatibility challenges in the git-p4 bridge.

A security-related question surfaces about git-daemon configuration post-CVE, with a public read-only server encountering "dubious ownership" errors for relative paths despite workarounds for absolute paths. The thread may yield configuration guidance for maintaining public access under new ownership verification requirements.

Alexander Shpilkin inquires about extending stash functionality to create snapshot objects including untracked files without modifying the working tree, identifying a gap for build system integration use cases. The thread may prompt discussion of stash internals or alternative approaches.

## On the radar

Git Rev News edition 121 is in draft review, celebrating 10 years of the community newsletter. Christian Couder invites contributions and proofreading before the April 1 publication, noting some anniversary statistics remain in progress.