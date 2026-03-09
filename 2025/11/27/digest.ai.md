# Git Mailing List Digest - 2025/11/27

**The day in brief.** A moderately busy Thursday with 41 emails across 13 threads, featuring significant progress on several fronts. Key developments include resolution of the Lisp userdiff driver series, architectural alignment on reference backend URIs, and the debut of a major Rust infrastructure RFC. The `git replay` command continues to evolve with new revert capabilities while Scalar documentation reaches completion.

## Notable threads

### Rust infrastructure refactoring proposal

Ezekiel Newren kicked off a 13-patch RFC series proposing a major restructuring of Git's Rust infrastructure from a single-crate to a multi-crate Cargo workspace. The ambitious redesign aims to improve modularity and compilation efficiency by splitting functionality into separate crates (gitcore, link-with-c, generate-headers) with proper FFI boundaries. The series is explicitly marked as incomplete and seeking feedback, with known issues including broken Windows Meson builds and missing test integration. This represents a significant architectural shift from Patrick Steinhardt's prior single-crate approach, potentially setting the stage for broader Rust adoption if the design proves sound.

### Reference backend URI design converges

The reference backend URI discussion reached consensus on using environment variable overrides (`GIT_REF_URI`) for temporary reference store redirection during migration workflows. Junio Hamano initially proposed explicit API calls but ultimately accepted the environment variable approach as fitting Git's existing patterns for temporary configuration overrides. The thread now focuses on finalizing the URI format syntax (`://` vs `:`) and filesystem path handling, with Karthik Nayak maintaining the current implementation while thoughtfully engaging with Junio's design feedback.

### Lisp userdiff driver finalized

Scott Burson's unified Lisp dialect support for Git's diff highlighting achieved technical consensus in its v2 iteration. The implementation combines Scheme and Common Lisp/Emacs Lisp patterns through disjunctive regexps that handle case sensitivity, constrained whitespace, and vertical bar syntax while preserving backward compatibility. Johannes Sixt provided final review confirming the patterns' correctness while suggesting minor presentation improvements. The series now awaits only procedural adjustments to patch submission format before integration, marking a successful conclusion to the multi-iteration discussion.

### `git replay` gains revert capabilities

The experimental `git replay` command saw continued refinement of its new `--revert` functionality. Siddharth Asthana demonstrated how the implementation already supports reverting non-contiguous commits through Git's standard revision range syntax, addressing Junio Hamano's workflow requirements. The team also agreed on an interface change to require explicit target branch specification (`git replay --revert <branch> <range>`), maintaining consistency with other operation modes. A separate patch introduced empty commit dropping behavior matching `git rebase --empty=drop`, further aligning the command with established Git conventions.

### Scalar configuration documentation complete

Derrick Stolee's Scalar configuration documentation series reached its conclusion with Junio Hamano's approval of the final patch. The comprehensive documentation covers all remaining config settings with detailed rationales, following earlier patches that removed stale values and reorganized the handling. Only minor typo fixes remain before the series can be merged, capping off a thorough effort to clarify Scalar's configuration landscape.

## In brief

**Patience diff optimization** -- Yee Cheng Chin's v2 patch improves the LCS calculation in the patience diff algorithm, reducing unnecessary binary searches for 8-14% speedups in `git log --patience` across multiple codebases.

**Submodule non-branch references** -- Alon Bar-Lev inquired about implementing support for non-branch references (like Gerrit's refs/changes/) in submodule checkouts, seeking guidance on whether such a patch would be accepted.

**Localization inconsistencies** -- brian m. carlson highlighted discrepancies in message localization between `git push` (English-only) and `git pull` (properly localized), surfacing tensions between plumbing stability and user-facing i18n needs.

**German translation accuracy** -- A separate l10n thread identified potential inaccuracy in German error messages where "Commit" is used where "Revision" would be more correct for non-commit objects.

**Bash completion debate** -- SZEDER Gábor and Junio Hamano questioned the value of completing Git's cryptic short flags (-p, -P) without explanatory context, stalling the previously approved feature.

## On the radar

**Hook subsystem modernization** -- Adrian Ratiu's parallel process infrastructure changes for hook refactoring continue to receive polish from Junio Hamano's feedback, with the pre-push hook conversion now addressing pipe closure timing.

**Documentation warning consensus** -- The thread about `--committer-date-is-author-date` reached quiet resolution with final approvals of documentation that strongly warns against the option's problematic nature while preserving availability.