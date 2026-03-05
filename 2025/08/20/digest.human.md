Here's the daily digest for August 20, 2025:

## The day in brief

A moderately busy day with 90 emails across 24 threads, featuring significant progress on several fronts. Key highlights include the completion of a major documentation standardization effort, ongoing refactoring of Git's packfile subsystem, and a new bug report about color handling in interactive commands. Junio Hamano also published a comprehensive "What's cooking" status update.

## Notable threads

### Documentation standardization finalized

Jean-Noël Avila's long-running effort to standardize Git's documentation format reached completion today with the final patch addressing Asciidoctor compatibility in `pretty-formats.adoc`. The fix resolves rendering differences between Asciidoc.py and Asciidoctor 2.0.16 by adjusting verbatim span markers. Junio Hamano requested the technical explanation for the changes be included in the commit message, marking the conclusion of this 14-patch series that has already been approved for merging into 'next'.

### Packfile store refactoring continues

Patrick Steinhardt's 16-part series to move packfile management into a dedicated `struct packfile_store` saw extensive review discussion today. Key points included:
- Clarification of the MRU (most-recently-used) list implementation from Jeff King
- Style discussions around boolean fields and bit fields
- Questions about data structure choices and edge case handling
- Mechanical refactoring of packfile-related functions

The series is receiving thorough technical review while maintaining its architectural direction, with several patches now in the polishing stage.

### Interactive command color handling bug

A new thread emerged reporting inconsistent color handling in Git's interactive patch mode across different subcommands. Jeff King identified the root cause as a 2021 regression from the Perl-to-C conversion, where some commands fail to properly handle `color.ui` settings. The thread includes:
- Detailed reproduction steps
- Technical analysis of the color initialization chain
- A proposed partial fix
- Workaround using `interactive.diffFilter`

The discussion also touched on related diff configuration limitations that are intentionally maintained for patching requirements.

### `git whatchanged` deprecation discussions

Multiple threads continued discussing the potential deprecation of `git whatchanged`, with:
- Confirmation that `git log --raw --no-merges` is the intended replacement
- Verification that timestamp extraction (`--pretty=%at`) works equivalently
- UX refinements to the deprecation warning message
- Discussion of command aliasing limitations in older Git versions

The consensus appears to be maintaining the command as legacy functionality while improving migration guidance.

## In brief

**Submodule gitdir path encoding** -- Adrian Ratiu's series received review feedback on test coverage gaps and platform-specific issues in path length validation, particularly on Windows and macOS.

**Dangling symref fix finalized** -- Jeff King's series preventing fetch from clobbering dangling symrefs reached completion after addressing Patrick Steinhardt's implementation questions.

**Bulk-checkin refactoring** -- A new 3-part series from René Scharfe eliminates global state in the bulk-checkin subsystem as part of the ongoing `the_repository` removal effort.

**gitk documentation** -- Michael Rappazzo proposed adding a comprehensive README.md for gitk, with discussion focusing on contribution workflow details and patch submission best practices.

**`git repo info` enhancements** -- A GSoC contributor submitted a 2-part series adding a `-z` flag and object format information to `git repo info`.

## On the radar

**Hash algorithm transition** -- Simon Richter raised new concerns about build script compatibility during hash algorithm transitions, suggesting future discussion may need to address migration utilities.

**Grafts removal** -- Questions emerged about whether shallow clone operations depend on grafts mechanism, indicating potential documentation needs for Git 3.0.