# Git Mailing List Digest - 2026/02/07

**The day in brief.** A busy Friday with 54 emails across 15 threads, dominated by major Rust infrastructure work and security discussions. The standout items are brian m. carlson's SHA-1/SHA-256 interoperability series reaching technical completion and Phillip Wood's security-focused enhancements to the commit-msg hook. Junio's "What's cooking" report provides a comprehensive snapshot of the project's current state.

## Notable threads

### Rust infrastructure for SHA-1/SHA-256 interoperability

brian m. carlson's 16-patch series introducing Git's first major Rust component has reached technical completion, marking a significant milestone in the Rustification effort. The work enables SHA-1 and SHA-256 repositories to exchange objects through a new Rust-based loose object map format, building on prior hash-algorithm-agnostic work. Key components include Rust abstractions for object IDs and hash algorithms, FFI bindings for Git's hash context, and a new binary object map format with comprehensive unit testing. The series has evolved through three iterations addressing Windows CI failures and incorporating documentation improvements. While technically complete, it awaits final CI verification before merging, with Ezekiel Newren handling complementary CI infrastructure work.

### Security enhancements for commit-msg hook

Phillip Wood proposed a series enhancing the sample `commit-msg` hook to detect and reject commit messages containing unindented diffs or separator lines ("---") that could confuse `git am`. This addresses a long-standing security vulnerability where embedded diffs might be misinterpreted as patches to apply. The implementation includes pattern matching for dangerous content while maintaining backward compatibility and supporting configurable comment characters. Junio Hamano raised workflow concerns about legitimate uses of separator lines, prompting discussion of optional checks via environment variables. The layered approach provides practical mitigation while deeper parser solutions continue being debated.

### `the_repository` removal in wt-status.c

Shreyansh Paliwal completed a 3-patch series removing `the_repository` and `the_hash_algo` usage from `wt-status.c`, part of the ongoing effort to eliminate global variables. The changes systematically replace global state with repository-specific instances, carefully handling NULL worktree cases that previously caused crashes. The implementation demonstrates safe repository access patterns while maintaining existing functionality, though two global dependencies remain preventing full removal of `USE_THE_REPOSITORY_VARIABLE`. The series has incorporated feedback from Karthik Nayak and Phillip Wood about NULL safety and interface improvements.

### Documentation standardization continues

Jean-Noël Avila's long-running effort to standardize Git man pages to consistent AsciiDoc synopsis-style formatting saw another iteration reviewed and approved by Kristoffer Haugsbakk. The changes cover `git-submodule.adoc`, `git-show.adoc`, and related pages, maintaining accuracy while improving presentation. Kristoffer confirmed all technical content remains intact while achieving markup consistency, giving the series his Reviewed-by tag. This represents the typical conclusion of these documentation patches - thorough review cycles leading to consensus that the changes maintain accuracy while improving presentation.

### `git merge-file` config handling fix

Yannik Tausch finalized a patch making `git merge-file` respect `merge.conflictStyle` configuration consistently whether run inside or outside a repository. The implementation uses `repo_config()` infrastructure directly as suggested by Junio, maintaining the existing precedence order (command-line flags override config). A new test case verifies the behavior works as expected outside repository contexts. The solution is now complete with no remaining open questions, serving both to fix the behavior and document it properly while also providing a reference example of proper patch submission format.

## In brief

**CI workaround for uutils coreutils** -- Colin Stagner proposed a targeted fix replacing only the problematic `dirname` command with GNU coreutils' implementation while keeping the rolling release CI environment intact, which Junio accepted with minor implementation feedback.

**Shallow clone boundary fix** -- Samo Pogačnik confirmed addressing Junio's feedback about inconsistent terminology in his patch fixing edge cases in `--shallow-since` behavior, with the patch now awaiting domain-expert review.

**`git clean --keep-ignored` proposal** -- Sylvain Rabot suggested adding an option to make `git clean` respect ignored files to prevent accidental deletion of valuable configurations, sparking discussion about implementation approaches.

**`git patch-id` documentation** -- Kristoffer Haugsbakk improved the man page with explanations about multi-patch processing efficiency and added a concrete example script showing how to use patch IDs to correlate commits.

**Bash completion updates** -- D. Ben Knoble extended bash completion to support newer `git stash import` and `export` subcommands, implementing basic option and argument completion for these operations.

## On the radar

**ODB transaction API changes** -- Junio's "What's cooking" report highlights ongoing work by Josh Tobler to make the object database transaction API per-object-source, part of broader object storage improvements.

**Incremental MIDX repacking** -- Taylor Blau continues work on geometric repacking optimizations, building on his prior contributions to pack storage efficiency.

**`git history` command** -- Patrick Steinhardt's proposed new command for history rewriting remains in development, representing a significant new capability for manipulating commit graphs.