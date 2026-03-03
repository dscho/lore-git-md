# Git Mailing List Digest - 2025/05/18 (UTC)

## The day in brief

A moderately active Sunday with 31 emails across 11 threads, featuring documentation updates, test modernization, and ongoing technical discussions. Key developments include final polish on the MyFirstContribution tutorial updates, completion of the string-list test conversion series, and a proposal to add `--index` as an alternative to `git diff --cached`.

## Notable threads

### MyFirstContribution tutorial modernization finalized

Jayatheerth K's series updating Git's new contributor tutorial reached its final form today with v4 patches that:
- Remove references to the defunct git-mentoring mailing list
- Update the example command signature to include the now-required `struct repository *repo` parameter
- Replace `git_config()` calls with `repo_config()` to align with current practices

The changes maintain the tutorial's pedagogical approach while bringing it in line with modern Git development patterns. With thorough review already completed in earlier versions, this series appears ready for merging.

### String-list test conversion completed

shejialuo's 8-part series modernizing string-list tests and implementation reached completion today. The final patches:
- Convert remaining shell tests to C unit tests
- Remove the now-obsolete shell test script entirely
- Clean up sign comparison warnings in the core implementation
- Simplify the string-list insertion interface

The series maintains all test coverage while improving code quality, though the author chose to retain the performance test in shell despite reviewer suggestions to remove it. This represents a significant step in Git's ongoing test modernization effort.

### git diff --index proposal

A new thread proposes adding `--index` as an alternative to `git diff --cached`, arguing the new flag would be more intuitive since it directly names the index/staging area being referenced. The 20-line patch maintains backward compatibility while introducing the new option, with corresponding documentation and test updates. While technically straightforward, this may spark discussion about terminology churn versus the widespread existing use of `--cached` in documentation and tutorials.

## In brief

**Bash function recognition edge cases** -- Johannes Sixt and Junio Hamano discussed decimal number handling in shell script diffs, concluding the current behavior produces acceptable results even for filename patterns like "sample.3gp".

**Submodule configuration optimization** -- A v5 patch prevents redundant `.gitmodules` entries when adding submodules to paths covered by existing patterns, with final polish on commit message formatting.

**Email documentation reorganization** -- A series completed consolidating email provider credential helper documentation in git-send-email.adoc, removing redundant references from gitcredentials.adoc.

**Batched reference updates error handling** -- Karthik Nayak and Patrick Steinhardt discussed design choices around error handling in the batched transaction system, agreeing to keep the current transaction-aborting behavior for generic errors.

**Index optimization discussion** -- Elijah Newren questioned whether a proposed "two-file index" optimization duplicates existing split index functionality, prompting the original proposer to re-examine current capabilities.

## On the radar

**NFS bus error report** -- A new thread reports Git operations failing with bus errors on NFS-mounted directories, though minimal technical details were provided in the initial message.