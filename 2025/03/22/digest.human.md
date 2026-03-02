# Git Mailing List Digest — 2025/03/22

## The day in brief

A moderately active Saturday with 9 emails across 6 threads, dominated by a security-focused debate about git-shell command overrides. The most notable developments include a heated exchange about the safety of allowing script overrides for built-in commands, plus two well-documented bug reports about `git diff -w` behavior and merge conflict detection.

## Notable threads

### Security debate over git-shell command overrides

Ayman Bagabas proposed an RFC to allow overriding git-shell built-in commands via scripts in git-shell-commands, moving the command dispatch logic to check for script overrides before falling back to built-ins. The initial implementation appeared straightforward, modifying `shell.c` to execute matching scripts when present.

Elijah Newren raised significant security objections, initially misunderstanding the scope as affecting all Git commands before clarifying his concerns were git-shell-specific. The debate now centers on whether allowing built-in command overrides introduces new attack vectors compared to git-shell's existing ability to run arbitrary scripts. Bagabas maintains the security profile is equivalent, while Newren remains skeptical about unintended consequences. This architectural discussion appears far from resolved, with strong opinions on both sides.

### `git diff -w` hides non-whitespace changes

Eugen Konkov reported unexpected behavior in `git diff -w` where the whitespace-ignoring mode also suppressed legitimate non-whitespace changes during a rebase. The report includes clear examples showing a variable rename (`name` → `qualified_name`) disappearing from diff output when `-w` is used, contrary to the flag's documented purpose of only ignoring whitespace differences. This appears to be a genuine bug in the diff machinery's handling of the whitespace-ignoring mode.

### Merge conflict marker misplacement

In a separate report, Konkov documented another issue where Git's merge conflict detection incorrectly marked non-conflicting code while failing to properly highlight the actual conflicting change (again involving the `name`/`qualified_name` variable change). The detailed reproduction shows Git's hunk detection algorithm misidentifying the conflict boundaries in this Perl code scenario. Both bug reports provide excellent test cases for improving Git's diff and merge conflict resolution behaviors.

## In brief

Andy Shevchenko confirmed that recent Git versions have resolved his worktree-related `git rebase` issue, where operations would fail when the target branch was checked out elsewhere. The fix appears in an unspecified newer version, removing the need for his previous workaround.

Nikolay Shustov followed up on their `git p4` patch addressing error message encoding issues, refining the metadata transcoding logic to handle non-UTF-8 Perforce error streams consistently with existing configuration options.

Marinos Nisiotis inquired about contributing Greek translations to the Git Book, seeking guidance on the localization workflow for this new contributor.