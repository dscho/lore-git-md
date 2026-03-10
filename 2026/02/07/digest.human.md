# Git Mailing List Digest - 2026/02/07

**The day in brief.** A busy day with 54 emails across 15 threads, dominated by major Rust infrastructure work for SHA-1/SHA-256 interoperability and security discussions around `git am` vulnerabilities. Key developments include brian m. carlson's completed Rust series for object storage, new security-focused commit message hooks from Phillip Wood, and Junio's "What's cooking" report highlighting ongoing architectural changes.

## Notable threads

### Rust infrastructure for SHA-1/SHA-256 interoperability

brian m. carlson completed the second part of their Rust infrastructure series (16 patches) enabling SHA-1 and SHA-256 repositories to exchange objects through new Rust-based storage. This marks Git's first major production Rust component, introducing:

- Required Rust dependency for interoperability features
- `ObjectID` Rust struct mirroring Git's C `struct object_id`
- FFI bindings for hash contexts and file operations
- New binary object map format with comprehensive testing
- Windows/MSVC build fixes coordinated with Ezekiel Newren

The series has evolved through three iterations addressing cross-platform concerns and CI integration. While technically complete, it awaits final CI verification before merging. This represents a significant milestone in Git's Rustification effort while maintaining strict performance and correctness requirements.

### Security enhancements for commit message handling

Phillip Wood proposed a 3-patch series enhancing the sample `commit-msg` hook to prevent security issues where `git am` could misinterpret diffs embedded in commit messages. The changes:

- Detect and reject unindented diffs above the scissors line
- Flag standalone "---" separators that could truncate messages
- Handle custom comment characters and edge cases
- Provide clear error messages guiding users to indent problematic content

Junio Hamano raised workflow concerns about legitimate uses of separator lines, prompting discussion of optional checks via environment variables. The thread balances security against backward compatibility for established workflows while the deeper parser ambiguity remains unresolved.

### `the_repository` removal in wt-status.c

Shreyansh Paliwal completed a 3-patch series removing `the_repository` and `the_hash_algo` usage from `wt-status.c`, part of the ongoing global variable elimination effort. The changes:

- Replace global state with explicit repository parameters
- Handle NULL worktree cases safely
- Maintain behavior while reducing implicit dependencies
- Leave two remaining globals (`comment_line_str`, `DEFAULT_ABBREV`) for future work

The series demonstrates patterns for safely passing repository context through complex call chains, validated through multiple review iterations with Karthik Nayak and Phillip Wood.

## In brief

**Documentation standardization** -- Jean-Noël Avila's v4 series converting `git-submodule.adoc` and `git-show.adoc` to synopsis-style formatting received Kristoffer Haugsbakk's Reviewed-by after addressing all feedback.

**Git clean with ignored files** -- Sylvain Rabot proposed a `--keep-ignored` option for `git clean` to prevent accidental deletion of valuable configuration files, sparking discussion about implementation approaches.

**Patch ID documentation** -- Kristoffer Haugsbakk improved `git-patch-id` docs with performance data and a practical script example showing how to map commits across branches using patch IDs.

**CI workaround for uutils coreutils** -- Colin Stagner implemented a targeted fix for Ubuntu rolling release CI failures by symlinking GNU coreutils' `dirname` while keeping other tests on cutting-edge dependencies.

**Fuzz test regression** -- Junio reported a `BUG()` trigger in repository initialization checks during fuzz testing, revealing an interaction with Olamide Caleb Bello's `the_repository` removal work.

**Bash script updates** -- D. Ben Knoble added completion support for `git stash import/export` and fixed Meson build dependency tracking for generated headers.

## On the radar

**Shallow clone edge cases** -- Samo Pogačnik's v3 fix for `--shallow-since` boundary commit handling awaits domain expert review after addressing all of Junio's feedback.

**Repository-aware attributes** -- Ayush Jha's RFC to make `read_attr()` repository-specific sparked performance discussions, with Junio suggesting broader refactoring of `is_bare_repository()`.