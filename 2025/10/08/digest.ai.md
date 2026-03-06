# Git Mailing List Digest - 2025/10/08

## The day in brief
A busy day with 158 emails across 33 threads, dominated by policy discussions around AI-generated contributions, technical refinements to `git replay`'s atomic ref updates, and a security proposal for repository trust verification. The AI policy debate reached new complexity with concrete examples of past AI-assisted contributions, while the `git replay` interface design converge on an extensible approach. A security RFC proposing token-based repository verification sparked lively discussion about submodule handling and alternative designs.

## Notable threads

### AI contribution policy debate intensifies
The ongoing discussion about AI-generated contributions reached new complexity as Elijah Newren identified six concrete use cases that would be prohibited under Junio Hamano's proposed QEMU-style policy but appear legally unproblematic. Christian Couder argued existing review mechanisms can handle AI-assisted contributions without special prohibitions, pointing to Newren's already-merged AI-assisted documentation series as evidence. Michal Suchánek raised new concerns about MIT license attribution requirements that even open-source-trained LLMs typically fail to meet. The thread now grapples with whether to grandfather existing contributions, create exceptions for low-risk cases, or maintain a blanket prohibition.

### Atomic ref updates for git replay` finalized
After extensive discussion, the interface design for `git replay`'s atomic ref updates converged on an extensible `--update-refs=<mode>` approach over simpler boolean flags. Siddharth Asthana's series will use `replay.defaultAction` config with modes like `update-refs`, `show-commands`, and future `format` support. The thread resolved remaining questions about hook integration (deferred for now) and edge case handling while maintaining the atomic-by-default behavior. Junio Hamano's concerns about future extensibility were addressed by adopting the enum-based design favored by Phillip Wood and Christian Couder.

### Security proposal for repository trust verification
Michael Lohmann proposed a token-based system to prevent arbitrary code execution from untrusted repositories, where Git would verify a secret token before executing hooks or config commands. The RFC sparked debate about submodule handling (each would need its own token), container compatibility, and token copying vulnerabilities. Brian m. carlson suggested an alternative config allowlist approach using `[safe]` sections, while Taylor Blau raised concerns about nested bare repositories. The discussion continues to weigh security guarantees against usability and deployment constraints.

### String-list API refactoring concludes
The string-list API refactoring series marked for inclusion in 'next' after final discussions about signed vs unsigned integer types. Collin Funk advocated for signed `ptrdiff_t` with overflow detection patterns from GNU Coreutils, but the series maintained its security-focused `size_t` approach. Junio Hamano humorously characterized the preference for `size_t` as a "disease or superstition" while acknowledging it represents the project consensus. The thread shows how even finalized technical work can prompt ongoing conceptual discussions about programming patterns.

### Git 3.0 and SHA-256 interoperability planning
The Git 3.0 planning discussion expanded to include additional ecosystem components like gitweb and cgit in compatibility assessments. SZEDER Gábor added Codeberg.org to the list of forges needing coordination, while Taylor Blau and Randall Becker debated JGit's architectural dependencies on core Git functionality. The thread revealed differing perspectives on release timeline flexibility, with Patrick Steinhardt favoring firm deadlines and Taylor advocating tolerance for reasonable slips to accommodate ecosystem readiness.

## In brief

**Documenting `--committer-date-is-author-date` pitfalls** -- Julia Evans added warnings to `git-am` and `git-rebase` man pages about timestamp ordering violations this option can cause, with follow-up discussion about user motivations for manipulating commit dates.

**`git add -p` hunk splitting finalized** -- Phillip Wood confirmed the approved behavior changes are ready for merging, making split hunks always require explicit user selection.

**Rust CI infrastructure improvements** -- Patrick Steinhardt's v2 series added formatting checks, Clippy linting, MSRV verification, and Windows support while leaving line width policy open for discussion.

**`git-pull` documentation overhaul** -- Julia Evans' v2 series restructured parameter documentation and integration strategy explanations, with Junio Hamano suggesting further refinements to technical accuracy.

**Refs modernization series expanded** -- Patrick Steinhardt added a safety patch to fully reset iterator state between advances in the tag peeling and ref iteration series, maintaining its 13% performance improvement.

**`git bundle` clone behavior clarified** -- Junio Hamano explained the conservative ref fetching behavior is intentional, with `--mirror` recommended for complete repository copies.

**`git whatchanged` deprecation impacts** -- Multiple users reported issues with Jenkins integrations, with solutions available in Jenkins' issue tracker for migrating to `git log` equivalents.

**New whitespace check proposed** -- Junio Hamano suggested adding EOF newline detection as a substantial project, with Brian m. carlson supporting its value for developer education.

## On the radar

**Outreachy applications** -- Three applicants expressed interest in the `the_repository` removal project, beginning their community onboarding under Usman Akinyemi's mentorship.

**Documentation formatting fixes** -- Ramsay Jones and Kristoffer Haugsbakk continued mechanical AsciiDoc corrections while revealing gaps in meson build coverage.

**Fast-import/export signature parity** -- Christian Couder's series implementing consistent signed tag handling awaits final approval after addressing minor documentation nits.