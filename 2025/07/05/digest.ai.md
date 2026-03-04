# Git Mailing List Digest - 2025/07/05

**The day in brief.** A moderately active day with 9 emails across 5 threads, featuring substantive discussions around remote naming collisions, SSH signing fixes, and contributor identity policies. The standout items are Jeff King's pragmatic solution to remote refspec collisions and an emerging discussion about pseudonymous contributions in the context of a tempfile leak fix.

## Notable threads

**Remote naming collision prevention** -- Jeff King follows up on Per Cederqvist's report about slash-containing remote names causing refname collisions, expanding the discussion to show this is fundamentally a refspec collision issue. His patch ([PATCH]) adds validation in `builtin/remote.c` to detect when new remote names would create ambiguous ref updates (like "outer/inner" vs "outer"), while still allowing manual config edits. The solution focuses on `git remote` as the natural place for preventative checks, avoiding performance impacts from broader refspec validation. This middle-ground approach addresses the most common foot-gun scenarios while respecting legitimate advanced use cases.

**SSH signing tempfile leak and contributor identity** -- What began as a straightforward bugfix for tempfile cleanup in SSH signing has evolved into a broader discussion about contributor identity policies. After redoste submitted a fix for leaked signing key files, Jeff King raised a question about DCO compliance when contributors use pseudonyms. brian m. carlson advocated for flexibility, citing cases like gender transition and safety concerns. Redoste later provided a test case for their original fix while sharing personal context about their pseudonym use. The thread now spans both technical improvements to signing key cleanup and important policy questions about contributor inclusion.

**Conflict marker comment character edge case** -- A new thread addresses edge cases in Git's automatic comment character selection when processing commit messages with conflict markers. The patch proposes solutions for scenarios where `core.commentChar=auto` interacts poorly with non-standard conflict comments. Two approaches are presented: always using "#" in auto mode when conflicts exist (Phillip Wood's suggestion) versus skipping auto-selection entirely during conflicts. The well-motivated analysis builds on prior work and presents concrete implementation options for reviewers to consider.

## In brief

**SHA-256 transition discussion** -- Aditya Garg engages with brian m. carlson's technical explanation about the hash conversion process, confirming they'll review the implementation while noting constraints around attending the Git Merge talk.

**Merge documentation fixes** -- Timur Sultanaev corrects an ASCII diagram in the git-merge man page, fixing branch labeling and standardizing whitespace formatting for consistent display.