# Git Mailing List Digest - 2025/10/05

**The day in brief.** A moderately busy Sunday with 36 emails across 16 threads, featuring ongoing discussions about licensing concerns for Rust components, interactive add navigation improvements, and several documentation standardization patches. The most notable developments include a new licensing question about xdiff's Rust implementation and René Scharfe's v2 series refining `git add -p` behavior.

## Notable threads

**Licensing concerns for Rust xdiff implementation**  
Yee Cheng Chin raised an important licensing question regarding the new Rust components in xdiff, noting the original LGPL-licensed xdiff code now lacks explicit license headers in its Rust implementation. This could inadvertently cause the components to inherit Git's GPL license, potentially breaking compatibility with projects like libgit2 and Vim/Neovim that rely on xdiff's LGPL status. The discussion connects to Phillip Wood's earlier concerns about license mixing when converting C to Rust, adding new complexity to the ongoing Rustification effort. This appears to be a substantive issue requiring input from both legal experts and xdiff maintainers.

**Interactive add navigation improvements**  
René Scharfe's v2 patch series (5 parts) significantly refines hunk navigation behavior in `git add -p`, addressing both immediate bugs and broader interface consistency. The changes make all navigation commands (j/J/k/K/y/n/e) handle hunk roll-over uniformly, add comprehensive test coverage, and fix state management issues in the `permitted` bitmask. Junio Hamano provided review feedback focusing on navigation symmetry and error message clarity, though the core design appears settled. This series represents a thoughtful response to earlier discussion about making hunk navigation more predictable.

**git clean pathspec handling bug**  
A concerning bug report revealed `git clean -X` incorrectly handles negated pathspecs with directories, potentially leading to unintended data loss. The issue occurs when excluded directories (like `.idea/dictionaries`) are still processed for deletion despite explicit exclusion patterns. The problem appears to stem from fundamental flaws in `clean.c`'s directory traversal logic where pathspec constraints are improperly applied. This represents a serious issue given the data loss implications, though no workaround was yet proposed in the discussion.

## In brief

**String-list API documentation polish** -- shejialuo addressed post-merge feedback on their string-list refactoring series, planning commit message improvements regarding type safety considerations between `int` and `size_t` usage.

**git worktree documentation formatting** -- Jean-Noël Avila provided AsciiDoc syntax feedback on Michal Suchánek's worktree manual page improvements, focusing on cross-reference syntax and section anchor placement.

**Symbolic reference validation** -- Junio Hamano reviewed a files-backend patch for symref validation, questioning whether the current `refname_is_safe()` approach sufficiently matches the project's broader validation patterns.

**git whatchanged deprecation** -- Kristoffer Haugsbakk clarified replacement instructions (`git log --no-merges --raw`) as the community moves forward with removing this legacy command.

**gitweb configuration path fix** -- A patch addressed Perl `@INC` path issues causing gitweb to fail finding its config file, with solutions involving explicit path prefixes in the installation.

**gitk UI theming merge** -- Johannes Sixt merged Mark Levedahl's theme system modernization into gitk, combining ttk widget updates with macOS compatibility fixes.

**Documentation standardization** -- Jean-Noël Avila sent a 3-patch series converting `git-stash`, `git-tag`, and `git-worktree` man pages to use Asciidoc's synopsis block format for consistent styling.