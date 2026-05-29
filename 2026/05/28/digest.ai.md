Here's the daily digest for May 28, 2026:

**The day in brief.** A moderately busy day with 48 emails across 20 threads, featuring several bugfixes nearing completion, platform-specific discussions, and some forward-looking feature proposals. Notable items include the resolution of Windows signal handling semantics and progress on `git rebase` symref support.

### Notable threads

**Windows signal handling alignment**  
Johannes Schindelin revealed that Git for Windows already implements nuanced signal handling that differentiates SIGTERM (allowing `atexit()` handlers) from SIGKILL (immediate termination), contrary to the native Windows API's simpler model. Junio C Hamano acknowledged this and deferred to Schindelin's plan to upstream the Git for Windows implementation, putting the original patch on hold. This resolves a long-standing divergence between platform behaviors while respecting Git's design constraints.

**Rebase symref handling fixed**  
A bugfix series addresses edge cases in `git rebase --update-refs` when dealing with branch symrefs (e.g., `main` pointing to `master`). The solution properly resolves symbolic references before queuing updates, skipping aliases of the current branch while maintaining all safety checks. Junio raised a broader question about potential overlap with recent ref filtering changes elsewhere (possibly in `git describe`), though this doesn't block the current fix.

**External notes command support**  
Siddh Raman Pant pinged their v3 series adding support for external commands to display Git notes content, which includes timeout handling and extensive test coverage. The feature allows delegating notes display via configurable external commands while maintaining security through input validation and timeout enforcement. The series appears ready for maintainer attention after multiple iterations.

**Line-log integration finalized**  
Michael Montalbo's series integrating `git log -L` with Git's standard diff output pipeline reached completion, with all feedback addressed and the patches merged to `next`. The final version includes support for metadata formats (`--raw`, `--name-status`) while intentionally excluding stat formats. A minor test helper syntax issue was identified and will be fixed separately.

**Config file locking timeouts**  
The discussion around Jörg Thalheim's config file locking timeout patch concluded with agreement on using `-fno-common` compiler flags for macOS/Xcode compatibility, avoiding source modifications. Harald Nordgren provided the specific implementation details for conditional flag application in config.mak.uname, which Junio accepted despite some lingering questions about version number patterns.

### In brief

**`git describe` pattern matching fix** -- Jacob Keller corrected `git describe --contains --all` to properly honor `--match` and `--exclude` patterns that were previously silently ignored.

**Memory leak in HTTP pack handling** -- Lorenzo Pegorari's v2 patch fixes both a memory leak and redundant tempfile cleanup in HTTP pack index verification error paths.

**Deprecated commit list functions removed** -- Kristoffer Haugsbakk completed the removal of long-deprecated functions from commit.h after verifying no remaining callers in active branches.

**Windows pathspec test fix** -- Kristofer Karlsson addressed Windows-specific test failures in `git ls-files` by skipping tests where backslash patterns are incorrectly normalized during pathspec parsing.

**Documentation typo fixes** -- The ongoing documentation typo cleanup series will be restructured to handle git-gui and translations separately from core fixes, per maintainer feedback.

**On the radar**

**Real-time collaboration proposals** -- Two threads explored adding real-time collaboration features, with Christian Couder suggesting integration with external tools like Jujutsu or Pijul rather than core Git changes. The discussion highlights ongoing tension between workflow desires and Git's distributed model.

**Secondary worktree hook bug** -- A new bug report demonstrates that hooks changing directories in secondary worktrees can cause `git status` to incorrectly report all tracked files as deleted. Reproduction cases are clear but no analysis from contributors yet.