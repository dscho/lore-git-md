Here's the Git mailing list digest for October 16, 2025:

## The day in brief

October 16 saw lively discussion across multiple fronts, with 110 emails spanning 24 threads. The day was dominated by technical debates around Message-ID tracking in commits and deprecation of `git diff X..Y` syntax, while also seeing significant progress on security patches and maintenance strategy improvements. Key developments include Junio Hamano's approval of Taylor Blau's 49-part repack refactoring series and a contentious discussion about commit metadata storage approaches.

## Notable threads

**Message-ID tracking proposal sparks debate**  
James Bottomley proposed storing email Message-IDs in commit headers to better link commits to mailing list discussions, particularly for kernel workflow needs. The 3-patch series modifies mailinfo.c to collect Message-IDs and stores them as commit headers. Junio Hamano raised concerns about header maintenance during rebases and cherry-picks, while brian m. carlson highlighted security risks of automatic Message-ID inclusion. Alternatives like Git notes and special headers (`X-Git-Original-Message-ID`) were discussed, with the thread revealing deep tensions between kernel workflow requirements and general Git history operation robustness.

**Deprecating confusing diff syntax**  
Martin von Zweigbergk's proposal to remove potentially confusing `git diff X..Y` and `X...Y` syntax continued with extensive discussion about compatibility and alternatives. Junio Hamano emphasized cross-command consistency needs, particularly for `git checkout`'s fork-point operations. D. Ben Knoble proposed `git diff --merge-base X Y` as an existing alternative, while Kristoffer Haugsbakk suggested following Git's typical deprecation process rather than immediate removal in Git 3.0. The thread highlighted challenges in changing established behaviors that span multiple commands.

**Security patches near completion**  
Michael Lohmann's security-focused series addressing arbitrary hook execution in untrusted repositories reached its final review stages. The patches introduce both temporary (`--allow-unsafe`) and persistent safety controls while preventing repositories from marking themselves as safe via local config. Junio Hamano questioned the necessity of new environment variables given existing alternatives, prompting discussion about the right balance between security and usability. The series appears ready for merging after addressing minor implementation feedback.

**Geometric maintenance strategy advances**  
Patrick Steinhardt and Justin Tobler progressed work on a new "geometric" maintenance strategy that combines geometric repacking with cruft pack fallbacks for better monorepo scaling. The 8-part series introduces configurable strategies applicable to both manual and scheduled maintenance, with thorough test coverage. Junio Hamano approved several preparatory refactorings, signaling the architectural changes are sound. This represents a significant enhancement to Git's large repository maintenance capabilities.

**In brief**  

**Repack refactoring approved** -- Junio Hamano gave final approval to Taylor Blau's 49-part repack refactoring series after positive reviews from Jeff King and Patrick Steinhardt, completing major architectural improvements to `builtin/repack.c`.

**GPG/SSH line ending fix** -- Okhuomon Ajayi addressed CR/LF handling in signature verification, renaming `remove_cr_after()` to `trim_cr_before_lf()` to properly preserve standalone CR characters while still normalizing Windows line endings.

**Documentation formatting fixes** -- Ramsay Jones submitted v3 of a series fixing asciidoc warnings in technical docs (`remembering-renames.adoc`, `sparse-checkout.adoc`, `commit-graph.adoc`), with all changes now applied to the integration branches.

**GitHub Actions updates** -- Johannes Schindelin adapted Dependabot's version bump suggestions for Git's CI workflows, updating `actions/checkout`, `actions/setup-python` and other dependencies across multiple workflow files.

**Signed-off-by legal discussion** -- D. Ben Knoble initiated a thread clarifying that Git's Signed-off-by tag represents DCO certification per project docs, contrasting with GNU's more formal copyright assignment process.

**On the radar**  

**Git 3.0 planning** -- Brian m. carlson's timeline discussion for SHA-256 transition continues, with current focus on gathering ecosystem readiness reports while targeting October 2026 for Git 3.0.