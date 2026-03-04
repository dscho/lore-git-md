# Git Mailing List Digest — 2025/07/27

**The day in brief.** A moderately active Sunday with 22 emails across 8 threads, featuring significant progress on the new `git repo info` command and detailed reviews of several feature patches. The standout development is Eric Sunshine's thorough feedback on the `git repo info` series, particularly regarding output format stability, while Junio Hamano weighs in on multiple threads with maintainer guidance.

## Notable threads

### `git repo info` command approaches finalization

Lucas Seiki Oshiro's GSoC project to create a new `git repo info` command for repository metadata retrieval has reached v6, with all major technical questions resolved. The series migrates functionality from `git rev-parse` to a dedicated command with structured output formats (`keyvalue` and `nul`). Eric Sunshine's detailed reviews focus on forward compatibility, suggesting immediate implementation of proper value quoting even for current simple fields to prevent future breakage. The thread shows consensus on the phased approach but reveals differing perspectives on when to address quoting requirements — Lucas prefers deferring until needed, while Eric advocates establishing robust output conventions from the start. With test coverage now comprehensive and only minor documentation concerns remaining, this plumbing command appears ready for finalization pending resolution of the quoting question.

### Bash completion gains configurable ref sorting

Nelson Benítez León proposes adding `GIT_COMPLETION_REFS_SORT_BY_FIELDNAME` to customize how refs are sorted in bash completion (e.g., showing recently-used branches first). The implementation modifies eight call sites to respect the configured `for-each-ref` sort parameter while maintaining backward compatibility. Junio's response emphasizes Git's commit message conventions and the non-negotiable requirement for test coverage, though he raises no technical objections to the feature itself. The patch currently lacks tests due to the author's unfamiliarity with the framework, prompting Junio to suggest seeking testing assistance — a common onboarding challenge for new contributors touching the completion script.

### `core.commentChar=auto` deprecation nears submission

Phillip Wood provides a status update on his long-running effort to deprecate the problematic `core.commentChar=auto` setting. With Junio's approval of the implementation approach (which will make Git error out rather than silently fall back to `#`), the series is in final polishing before submission. This change, targeting Git 3.0, exemplifies the project's careful handling of backward-incompatible changes — thorough discussion ensuring consensus before implementation, followed by clear warnings before removal. The update confirms no outstanding technical issues remain, just final patch preparation.

## In brief

**Git-daemon path handling** -- Jeff King and Russell King analyze a Fedora-specific regression where tilde expansion fails, with SELinux policies and forgotten systemd overrides emerging as likely culprits in the path validation puzzle.

**`git grep` pathspec inconsistency** -- Junio confirms `git grep` mishandles the `:^:` exclude syntax that works in `git ls-files`, suggesting `:(exclude)` as the workaround while cautioning about potential side effects from fixing the shorthand.

**Help system behavior controversy** -- Junio strongly objects to a proposed change allowing `-h/--help-all` with additional arguments, citing dangerous edge cases with aliases and questioning the workflow's validity given Git's CLI guidelines.

## On the radar

**Windows workflow tools** -- Skybuck Flying's unconventional Git workflow proposal has progressed from concept to Windows executables, though with no community engagement yet and significant questions about integration and testing remaining unresolved. The parallel discussion of "laser development" metaphors suggests continued interest in exploring Git's capabilities for complex engineering scenarios.