# Git Mailing List Digest - 2025/08/18

## The day in brief

A busy Monday with 46 emails across 18 threads, featuring significant progress on several fronts. The standout developments include a major release (Git v2.51.0), substantial work on `git describe` edge cases, and continued discussion about Rust integration policies. Junio Hamano's "What's cooking" report and governance email provide valuable snapshots of the project's current state.

## Notable threads

### Git v2.51.0 released

Junio Hamano announced Git v2.51.0, marking a new stable version with 506 non-merge commits from 91 contributors. The release includes several user-facing improvements: the reftable backend is now mature enough to become default in Git 3.0, `git switch` and `git restore` shed their experimental labels, and new `--compact-summary` options improve merge/pull output. Internally, the release continues the project's architectural work with more `the_repository` removal patches and object store restructuring. The release notes document the beginning of the deprecation path for `git whatchanged`, which immediately prompted a notification from the Jenkins Git client plugin maintainer about their ongoing usage.

### Describe command safety overhaul

Jeff King led a comprehensive effort to harden `git describe` against edge cases, culminating in a 5-patch series that addresses several safety issues. The changes improve error handling for blob descriptions, unborn branches, and corrupted repository states. The series evolved from theoretical discussion to concrete implementation, with Junio approving the early patches while the more substantive changes continued review. The work demonstrates Git's careful approach to edge case handling - even for scenarios that require repository corruption to trigger.

### Rust integration policy discussion

The ongoing conversation about Rust as a hard dependency saw important maintainer input today. Junio Hamano responded to Ezekiel Newren's RFC series, emphasizing that initial Rust adoption should focus on framework establishment (style guidelines, FFI patterns) rather than performance claims. While accepting that Rust version requirements may vary across platforms, Junio suggested the xdiff optimizations might not be the ideal first showcase for Rust in Git. Ben Knoble later reinforced the emerging consensus around using default rustfmt configurations for code style. The thread reveals the project's careful approach to major architectural changes.

### Rebase documentation finalized

Phillip Wood and Junio Hamano finalized an 8-iteration documentation improvement for `git-rebase`, resolving the last open question about `--onto` option placement. The series, which reduces the man page by 104 lines while improving clarity, represents a significant documentation win. The final discussion centered on balancing visibility of commonly-used features with detailed examples - a compromise was reached to maintain the dedicated section while adding cross-references from basic usage descriptions. Julia Evans' original implementation work, guided by Junio and Phillip, demonstrates the project's collaborative documentation process.

### Line-log fixes for multiple ranges

SZEDER Gábor submitted a thorough fix for line-log functionality when processing multiple `-L` ranges, addressing an assertion failure, infinite loop, and output issues. The series corrected boundary condition handling that had been problematic since the feature's 2013 introduction, with test updates showing some expected outputs had been wrong all along. Junio acknowledged the comprehensive fix, which maintains behavior across both SHA-1 and SHA-256 repositories. The work highlights how even long-standing features continue to receive careful attention to edge cases.

## In brief

**Submodule hash algorithm mismatch** -- Michael Schroeder pushed back against Brian M. Carlson's assertion that mixed-hash submodules shouldn't work, arguing the behavior is already functional for non-shallow clones.

**Stash keep-index behavior** -- Phillip Wood and Ben Knoble discussed adding trailers to stash commits to enable automatic unstaged-change-only behavior when popping stashes created with `--keep-index`.

**Autosquash rebase cleanup** -- Phillip Wood identified inconsistent behavior between pick and fixup commands regarding commit.cleanup=strip when prepare-commit-msg hooks are used.

**Hostname-based config includes** -- Ayush Sharma revived discussion about adding `hostname:` conditions to `includeIf` sections, with Junio pointing to prior related threads.

**Patch line numbers proposal** -- Seyi Kuforiji suggested adding line numbers to `format-patch` output, though Junio raised concerns about breaking patch application.

**Nested interactive rebases** -- Isaac Oscar Gariano proposed allowing modification of earlier commits during an ongoing rebase, with Oswald Buddenhagen indicating related work in progress.

## On the radar

**Rust integration** -- The RFC series continues to generate discussion about appropriate use cases and policies, with performance claims needing refinement.

**Submodule mixed hashes** -- The debate continues about whether to fix edge cases or treat mixed-hash submodules as unsupported.

**Stash keep-index trailers** -- The proposed solution for better unstaged change handling awaits implementation feedback.