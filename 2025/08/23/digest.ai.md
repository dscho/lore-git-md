# Git Mailing List Digest - 2025/08/23

**The day in brief.** A busy Saturday with 60 emails across 10 threads, dominated by the ongoing debate about Rust adoption in Git's xdiff infrastructure and its impact on platform compatibility. The Rust discussion escalated into a policy debate about whether to make Rust a mandatory dependency, while Johannes Sixt proposed an alternative approach to progress meter signaling that could avoid the issue entirely. Elsewhere, Julia Evans' git-rebase documentation improvements reached final approval, and Meet Soni's `git refs exists` series saw test infrastructure refinements.

## Notable threads

**Rust infrastructure debate escalates** -- Ezekiel Newren's RFC series introducing Rust infrastructure for xdiff (now at v3) triggered a critical policy discussion when Randall Becker objected that making Rust mandatory would exclude NonStop platforms where Rust isn't available. The thread saw heated exchanges about platform support tradeoffs, with Becker citing "tens of thousands" of affected users while Newren and Brian Carlson emphasized security benefits. Technical solutions like mrustc were proposed but deemed unworkable, leaving the project facing a binary choice between modernization and backward compatibility. Junio Hamano meanwhile focused on final technical details like whitespace issues, suggesting the series is otherwise merge-ready pending resolution of the platform policy question.

**Progress meter signaling alternatives** -- Carlo Marcelo Arenas Belón proposed replacing `setitimer()` with `alarm()` for progress updates, but Johannes Sixt countered with a more radical signal-free approach using periodic `getnanotime()` checks instead. Junio Hamano initially expressed reservations about timer overhead but later acknowledged modern OS optimizations may make it viable. Sixt is now polishing his implementation, which would completely eliminate signal handling from the progress meter while maintaining Windows compatibility - potentially offering a cleaner solution than the original signal-based approaches.

**git-rebase documentation finalized** -- Julia Evans' long-running effort to improve the git-rebase man page reached completion with v9 implementing Junio Hamano's final structural feedback. The series reduces the document by 104 lines while improving clarity through better organization - notably moving the `--onto` explanation earlier with a proper section heading. The changes exemplify Git's documentation philosophy: leading with concrete examples, introducing core concepts first, and eliminating redundancy while preserving technical accuracy. With all review feedback addressed, this series appears ready for merging.

**git-history subcommand docs refined** -- Jean-Noël Avila provided detailed documentation feedback on Patrick Steinhardt's RFC series introducing the new `git-history` command. The reviews focused on standardizing parameter names (`<commit>` vs `<revision>`), improving option syntax presentation, and ensuring consistency with Git's documentation conventions. These polish changes don't affect the implementation but help align the new command's docs with established practices, particularly important for a user-facing feature inspired by Jujutsu's history editing capabilities.

**git refs exists test infrastructure** -- Meet Soni's GSoC project to add a `git refs exists` subcommand saw test infrastructure refinements in v2, implementing Patrick Steinhardt's suggestion to share tests between the new command and the existing `git show-ref --exists` functionality. The series demonstrates careful attention to test organization - extracting common test cases into a shared library while maintaining full coverage of ref existence scenarios (regular refs, HEAD, symrefs, special refs). This continues Git's trend of consolidating ref-related commands under the `git refs` namespace with robust test coverage.

## In brief

**Bugreport template formatting** -- Kristoffer Haugsbawk submitted v3 of his bugreport template improvements, addressing Junio Hamano's feedback by removing unnecessary `>` prefixes and adding spacing between questions.

**First contribution guide update** -- Daniele Sassoli's v2 revision clarifies that both `git/git` and `gitgitgadget/git` can be used as remotes when submitting PRs, with a link comparing their differences.

**Commit untracked files proposal** -- Isaac Oscar Gariano suggested extending `git commit` to handle untracked files directly, with Brian m. carlson recommending a new `--include-untracked` flag for backward compatibility.

**Compat object format warning** -- A documentation patch added warnings that `extensions.compatobjectformat` is experimental and incomplete, particularly for packed objects and push/fetch operations.

## On the radar

**What's cooking report** -- Junio Hamano's regular status update noted 30+ topics merged to integration branches, including ort merge improvements and Bloom filter optimizations, while flagging several topics needing rerolls before the 2.51 release.