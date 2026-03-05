# Git Mailing List Digest - 2025/09/08

**The day in brief.** A busy Monday with 143 emails across 29 threads, dominated by major discussions around Rust infrastructure adoption and enterprise compatibility concerns. Key highlights include Patrick Steinhardt's v3 Rust RFC series, ongoing enterprise LTS policy debates, and Christian Couder's promisor-remote protocol extension reaching technical completion. Several performance optimizations and bugfix series also saw significant progress.

## Notable threads

### Rust Infrastructure RFC v3

Patrick Steinhardt submitted the third iteration of his RFC series introducing Rust infrastructure to Git core, now expanded to 8 patches. The series establishes build system support (meson and Makefile), implements a varint subsystem conversion as proof-of-concept, and documents a phased transition timeline targeting mandatory Rust in Git 3.0. Key changes in v3 include:

- Makefile improvements for proper dependency tracking
- Varint safety enhancements with overflow tests
- CI fixes for AlmaLinux and Rust toolchain setup
- Clarified 2-year LTS support window for C-only branch

The discussion revealed ongoing tensions around transition timelines and platform support, with Junio Hamano and others questioning the "test balloon" characterization given known platform incompatibilities. Technical debates emerged about type safety at the Rust/C boundary and build system design tradeoffs, particularly around Meson vs Cargo workflows.

### Enterprise LTS Policy Debate

The Rust transition discussion bifurcated into a parallel thread focused on enterprise impact, where Randall Becker detailed strict compliance requirements for regulated industries. The key sticking point is maintaining an official C-only LTS branch that meets auditor validation needs during the Rust transition. Proposals include:

- Junio designating official LTS maintainers (similar to Linux stable)
- 1.5-2 year support window for security backports
- Repository-hosted branch to satisfy audit requirements

The discussion remains at an impasse between enterprise needs (30-day CVE SLAs, 5-year support expectations) and project maintenance realities, with Becker insisting compliance frameworks mandate official channels rather than community-maintained forks.

### Promisor-Remote Protocol Extension

Christian Couder's 8-part series extending the promisor-remote protocol to support configurable validation of remote attributes reached technical completion in v8. The implementation:

- Introduces `struct promisor_info` to replace strvec arrays
- Adds server-side field transmission (`promisor.sendFields`)
- Implements client validation (`promisor.checkFields`)
- Modernizes string handling throughout

Junio Hamano acknowledged the series is ready for merging pending only a terminology decision between "field" vs "configuration variable" for the new options. This represents a major step forward in making promisor remotes more configurable and secure.

### Submodule Path Encoding

Adrian Ratiu's v2 series addressing filesystem conflicts in submodule paths through URL-style encoding saw thorough review. The 10-patch set:

- Moves gitdirs to `.git/submodules/` from `.git/modules/`
- Implements RFC 3986 encoding with case handling
- Adds config override mechanism
- Includes comprehensive test coverage

Reviewers identified minor issues around path length validation and version compatibility markers, with Adrian agreeing to add an `extensions.submoduleEncoding` config key in v3 to prevent older Git versions from misinterpreting the new path structure.

## In brief

**HTTP clone connectivity resolved** -- Ellie confirmed GitHub-side fixes for HTTP/2 clone failures that caused "RPC failed; curl errors", though the thread highlighted broader needs for resumable transfers.

**git-new-workdir removal impacts** -- Gabriel Scherer reported workflow disruption from the script's removal, as `git worktree` doesn't fully replace its ability to check out the same branch multiple times. Kristoffer Haugsbakk noted `--force` can bypass some restrictions but Junio suggested detached HEAD as an alternative.

**xdiff performance optimizations** -- A 4-part series from Alexander Monakov optimized hash calculation in xdiff, showing 8% speedups on `git log --oneline --shortstat` through function splitting, branch prediction hints, and parallel djb2 hashing.

**Color handling fixes** -- Jeff King's v2 series fixed regressions in interactive patch mode color output, properly separating `color.diff` and `color.interactive` handling while adding thorough test coverage.

**Batched ref updates on case-insensitive FS** -- Karthik Nayak's series fixed F/D conflict handling in batched transactions, adding new `REF_TRANSACTION_ERROR_CASE_CONFLICT` and improving error recovery.

**Command deprecation framework** -- A v3 series enhanced Git's command deprecation system with alias shadowing support and improved messaging, now awaiting final flag-based implementation refinement per Junio's feedback.

**git replay --update-refs** -- Siddharth Asthana proposed atomic ref updates for `git replay`, sparking design discussion about performance implications and flag simplification (combining `--update-refs` and `--batch`).

## On the radar

**SHA-256 interoperability** -- brian m. carlson's status update revealed substantial but incomplete progress on SHA-1/SHA-256 interoperability, with submodule handling and shallow operations remaining challenges that could impact Git 3.0 timing.

**WebDAV protocol extension** -- Milan Hauth's proposal to support PROPFIND in dumb HTTP met resistance from Junio Hamano who views it as contrary to the protocol's intentionally minimal design, despite demonstrated use cases with nginx.