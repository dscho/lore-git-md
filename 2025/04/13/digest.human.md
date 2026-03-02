# Git Mailing List Digest — 2025/04/13

## The day in brief  

A quiet Sunday with 11 emails across 6 threads**,** dominated by technical discussions about patch signing workflows and a bug report about `git difftool` behavior. The most notable developments include a detailed exploration of GPG signing limitations in Git's email workflow and confirmation of a counterintuitive `git difftool` edge case.

## Notable threads  

### GPG signing limitations in patch workflows  

Klaus Frank initiated a discussion about the inability to GPG-sign patches when using `git send-email`, expressing concerns about patch authenticity. Matt Hunter and brian m. carlson provided detailed technical explanations of why commit signatures break during patch application (due to committer identity changes) and why PGP/MIME signing presents implementation challenges. The thread revealed Git's historical prioritization of patch quality over transport authentication, though brian m. carlson suggested potential solutions involving PGP/MIME or custom headers to preserve metadata. This discussion highlights a long-standing gap in Git's email workflow that periodically resurfaces in the community.

### `git difftool` loses copied files with meld  

A user reported that files copied during a `git difftool --tool=meld -d` session disappear when the tool exits, contrary to expectations. Jayatheerth confirmed this is intentional behavior - the command operates on temporary directories that Git cleans up after the session. While acknowledging the UX confusion, the response maintained this as a safety feature and recommended using standard Git commands (`git restore`/`git checkout`) for file restoration instead of relying on the diff tool's file operations. The exchange serves as useful documentation of this non-obvious behavior for future users.

## In brief  

Philippe Blain's performance test fixes gained a third commit correcting documentation in p7821-grep-perl.sh, completing the series that had already been approved for merging. The multi-remote synchronization discussion saw D. Ben Knoble reiterating their local-repository-as-source approach despite CI environment constraints. Koji Nakamaru provided a clear solution for pre-loading GitHub PATs using `git credential approve` with a here-document format example.  

## On the radar  

The GPG signing discussion shows signs of evolving from problem identification to potential solution exploration, with brian m. carlson hinting at possible prototyping work. This bears watching as it touches fundamental aspects of Git's email-based contribution workflow.