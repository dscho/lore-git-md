Here's the Git mailing list digest for January 7, 2025:

---

### The day in brief
A busy day with 105 emails across 34 threads, dominated by technical discussions around reftable entropy handling, CI modernization, and documentation standardization. Key developments include maintainer approval for the `the_repository` refactoring series and a new proposal for symlinked submodules.

---

### Notable threads

**Reftable entropy handling debate intensifies**  
Patrick Steinhardt proposed a solution for reftable failures on ia64 systems where OpenSSL's PRNGD exhausts entropy during concurrent operations. The patch series introduces a `CSPRNG_BYTES_INSECURE` flag to allow fallback to less secure random bytes when cryptographic strength isn't required. While Randall Becker confirmed the fix addresses real-world issues on NonStop systems, Junio Hamano raised architectural concerns about reftable depending on Git's CSPRNG layer. The discussion evolved into a broader debate about platform support requirements versus codebase purity, with Brian m. carlson later questioning whether the entropy exhaustion scenario is over-optimized given modern OpenSSL behavior.

**CI modernization series reaches v3**  
Patrick Steinhardt's comprehensive CI overhaul progressed to its third version, now reduced to 10 patches. The series removes Azure Pipelines remnants, stabilizes tests (including a complex solution for flaky SIGPIPE tests in submodules), and standardizes containerized execution across GitHub Actions and GitLab CI. Notable changes include rootless container support, i386 testing additions, and Ubuntu version updates. The series appears ready for integration after extensive review, particularly around the submodule test reliability solution which uses 2500 artificial submodules to force buffer saturation.

**`the_repository` removal advances**  
Junio Hamano approved Patrick Steinhardt's 15-commit series removing `the_repository` usage from various subsystems including match-trees, graph rendering, and credential handling. The maintainer endorsed the incremental "bubble up" strategy that maintains bug-for-bug compatibility during the transition. This marks significant progress in the long-term effort to eliminate the global repository variable, with reviewers Toon Claes and Karthik Nayak having previously validated the approach.

**Symlinked submodule proposal**  
Vadim Zeitlin proposed adding a configuration option to disable Git's validation of submodule paths being directories rather than symlinks. The security check (introduced in e8d0608944) prevents various attacks but breaks legitimate workflows where symlinked submodules save disk space. Brian m. carlson and Junio Hamano engaged constructively, discussing implementation details like config option naming (`safe.submodules` or `submodule.validate`) and bootstrapping strategies. The discussion balances security hardening against practical workflow needs in trusted environments.

**Object-info protocol nears completion**  
The client-side implementation of the object-info protocol (allowing metadata queries without full object downloads) received final polish in its 8th iteration. Calvin Wan and Eric Ju suggested simplifications to response parsing logic in patch 5/6, while the final patch added `remote-object-info` support to `git cat-file --batch-command`. The feature appears ready for inclusion after thorough review, currently supporting only the "size" attribute but designed for future expansion.

---

### In brief
- Jeff King submitted post-merge refinements to the LSan test infrastructure, improving leak detection logic and documentation after the core technical solution was merged.
- Jean-Noël Avila continued the documentation standardization effort, addressing review feedback on manpage conversions for `git-commit` and `git-notes`.
- A bug was reported and confirmed fixed (since v2.41.0) where `git restore --staged -p` would crash during merge conflicts when encountering "Unmerged path" lines.
- Justin Tobler's patch to make `fsck.skipList` configuration parsing more strict (rejecting empty/invalid paths) gained maintainer approval.
- Kristoffer Haugsbakk's typo fixes for the 2.48.0 release notes were accepted, improving phrasing and correcting idioms like "thundering herds".

---

### On the radar
- The `prune-remote-refs` maintenance task series hit a roadblock when Junio Hamano rejected Shubham Kanodia's suggestion to merge prefetch behavior changes without proper justification, emphasizing that modifications to existing commands must stand on their own merits.
- Matěj Cepl's `git-request-get` proposal (a companion to `git-request-pull` for fetching and emailing patches) received initial maintainer feedback about proper Perl script conventions and testing requirements, awaiting a revised submission.
- An SMTP bearer authentication issue was reported where long OAuth tokens (>220 chars) fail in `git send-email` due to apparent Net::SMTP command corruption, with reproduction details provided but no solution yet proposed.

--- 

Today's traffic showed the project balancing architectural evolution (reftable, `the_repository` removal) with careful attention to edge cases and documentation quality. The reftable entropy discussion particularly highlighted Git's ongoing challenges in maintaining both security and platform compatibility.