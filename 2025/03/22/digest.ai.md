# Git Mailing List Digest — 2025/03/22

## The day in brief

A moderately active Saturday with 9 emails across 6 threads, dominated by a security-focused debate about git-shell command overrides. The most notable developments include a heated exchange about the safety of allowing script overrides for built-in commands, a bug report about `git diff -w`'s overly aggressive whitespace handling, and confirmation that a worktree-related issue has been resolved in recent Git versions.

## Notable threads

### Security debate over git-shell command overrides

Ayman Bagabas proposed an RFC to allow overriding git-shell built-in commands via scripts in the git-shell-commands directory, arguing this would enable use cases like access-controlled wrappers. The initial implementation moved directory checks earlier in the command dispatch flow. Elijah Newren raised significant security concerns, initially misunderstanding the scope but later clarifying objections specific to git-shell. The discussion revealed deep differences in how contributors assess risk — Bagabas maintains the security profile matches existing git-shell-commands behavior, while Newren sees built-in command interception as qualitatively riskier. The thread remains unresolved but has narrowed to git-shell-specific security analysis.

### `git diff -w` hides non-whitespace changes

Eugen Konkov reported unexpected behavior where `git diff -w` not only ignored whitespace changes (as intended) but also suppressed non-whitespace changes during a rebase conflict. The report included clear examples showing a variable rename (`name` to `qualified_name`) disappearing from diff output when `-w` was used. This appears to be a legitimate bug in the whitespace-ignoring logic that could mislead users during conflict resolution. No proposed fix was included, but the reproduction case is well-documented.

### Worktree rebase issue resolved

Andy Shevchenko confirmed that recent Git versions (exact version unspecified) have resolved an issue where `git rebase` would fail with "fatal: 'netboot' is already checked out" when the target branch was checked out in another worktree. The fix eliminates the need for workarounds involving separate checkout and rebase commands. This closes a thread that began with Jeff King identifying commit b5cabb4a967 as the original source of the check.

## In brief

Nikolay Shustov pinged their `git p4` patch improving error message encoding handling, which refactors transcoding logic into a new `MetadataTranscoder` class while maintaining Python 2/3 compatibility.

Marinos Nisiotis inquired about contributing Greek translations to the Git Book, seeking guidance on the localization workflow.

Eugen Konkov separately reported a merge conflict marker placement issue where Git marked unrelated code as conflicting while missing the actual semantic conflict in a `die` statement.