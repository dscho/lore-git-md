# Git Mailing List Digest - 2025/10/12

## The day in brief

A moderately active Sunday with 11 emails across 8 threads, featuring final polish on several documentation and test fixes alongside ongoing policy discussions. The day's most notable developments include resolution of a long-standing test configuration issue and continued debate about AI contribution guidelines.

## Notable threads

### Final touches on stash.index documentation

Ben Knoble's documentation patch for the `stash.index` feature has reached its final form after addressing maintainer feedback about commit message phrasing. The thread confirms consensus that merge commit references in documentation are unnecessary since the information can be queried directly. With all prior feedback addressed - including test isolation fixes and `--autostash` interaction documentation - the series remains ready for merging as-is. This represents the completion of a multi-iteration effort to properly document this stash behavior.

### AI policy retroactivity debate continues

Junio Hamano weighed in on the ongoing AI contribution guidelines discussion, arguing that the proposed AI policy should follow Git's established pattern for other guidelines - governing new contributions rather than requiring retroactive fixes. Responding to Elijah Newren's concern about grandfathering existing AI-assisted work, Junio drew parallels to coding style conventions where violations are only addressed when modifying existing code. This positions the AI prohibition as a forward-looking quality control rather than a mandate to rewrite history, though implementation details remain unresolved.

### Windows include path semantics questioned

Johannes Sixt introduced a new technical dimension to the Windows include path discussion, challenging Johannes Schindelin's patch converting relative includes to project-relative paths in `compat/mingw.c`. While earlier discussion focused on style consistency, Hannes argues the current relative paths (`#include "../foo.h"`) better communicate file relationships than either project-relative or system-style includes. This critique shifts the debate from settled style concerns to deeper questions about include path semantics and compiler behavior, potentially requiring additional Windows platform expertise to resolve.

## In brief

**GPG configuration test fix** -- Junio Hamano corrected a year-old regression in t1016-compatObjectFormat.sh where `test_config` was incorrectly used instead of `git config` for gpg.program setup, breaking test dependencies. Todd Zullinger confirmed the fix resolves failures in Fedora builds, though intermittent CI issues remain.

**Stash output behavior questions** -- Miroma raised final implementation questions about the `git stash` output behavior series, including `stash clear` reporting and `--quiet` flag consistency across subcommands, indicating thorough review before submission.

**Documentation patch status** -- Ben Knoble confirmed he'll address feedback on both his `stash.index` documentation and `:(optional)` pathname config patches, with Junio preferring incremental fixes over full rerolls for the latter.

**CI newline detection follow-up** -- Bello Olamide demonstrated improved understanding of detecting missing newlines at EOF in CI, now recognizing the need to compare against parent commits rather than just checking file states.