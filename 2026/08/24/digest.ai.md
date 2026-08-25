# Git mailing list daily digest for 2026/08/24

## The day in brief
The Git mailing list saw active discussion across several key areas today. The most consequential developments included: resolution of the case-sensitivity policy for `git history squash` autosquash markers (now requiring strict lowercase matching), progress on the geometric repacking race condition fix with architectural refinements, clarification of the AI-generated contributions policy debate, and multiple performance optimizations advancing toward integration. Several long-standing threads reached critical inflection points, particularly around ODB abstraction and pluggable backends.

## Notable threads

### Global config file handling inconsistency fix
**[2025/10/10/01-14-05]**
Junio C Hamano provided maintainer clarification confirming that Delilah Ashley Wu's three-patch series fixing the `--global` config listing inconsistency (where `git config list --global` only showed `$HOME/.gitconfig` despite Git reading both that and `$XDG_CONFIG_HOME/git/config`) aligns with Git's original migration-friendly design. The write path intentionally writes to only one file, but the read-side discrepancy was unintentional. This confirmation reinforces the series' motivation and clears the path for integration.

The series modifies `config_sequence()` in `config.c` to handle global scope by reading both config files through `do_git_config_sequence()`, adding flags to restrict config reading to global scope, and introducing `success_count` tracking and `require_successful_config` logic. Windows path normalization is limited to XDG config paths, and error handling prevents regressions with unreadable files.

### `git history squash` autosquash marker resolution policy
**[2026/06/14/19-25-39]**
The long-running debate over case-sensitivity in autosquash markers (`fixup!`, `squash!`, `amend!`) reached resolution today. Phillip Wood initially suggested accepting all-uppercase or all-lowercase OIDs while rejecting mixed-case ones to match `git rebase --autosquash` behavior. However, Junio C Hamano clarified that Git has never emitted uppercase hexadecimal OIDs (citing brian m. carlson's `bc/restrict-hex-to-lowercase` topic) and that `starts_with()` is sufficient, implying strict case-sensitive matching is the correct policy.

This decision requires Harald Nordgren to reroll patch 7/8 of the v14 series to implement strict case-sensitive OID matching and add test coverage for uppercase OID rejection. The series remains functionally complete except for this change and is otherwise ready for integration, with all other feedback addressed and Junio's "Will replace" sign-off from v7.

### Geometric repacking race condition fix
**[2026/08/18/22-34-04]**
The architectural discussion around Elijah Newren's two-patch series fixing a race condition in geometric repacking advanced significantly today. Jeff King (Peff) initially challenged the necessity of the fix for `OBJECT_INFO_QUICK` callers, but Patrick Steinhardt clarified that the bug affects *all* object lookups when an object exists in multiple packs covered by a multi-pack-index (MIDX) and the preferred pack is removed.

The discussion then focused on implementation refinements:
- Peff proposed a tri-state return value from `fill_midx_entry()` to distinguish between "not found," "found but unavailable," and "found and available"
- Patrick refined this to scan only MIDX-covered packs when the preferred pack is missing
- Peff suggested tying the fallback to `SECOND_READ` to optimize performance for `QUICK` callers
- Derrick Stolee introduced the existing `multi-pack-index expire` mechanism as a complementary strategy to reduce race frequency

The fix's necessity is now widely accepted, but the implementation details remain under active discussion. The series targets a crash in `git replay` and missing-object errors in other server-side operations.

### AI-generated contributions policy debate
**[2026/08/23/14-19-11]**
The policy discussion around AI-generated contributions reached a critical juncture today. Junio C Hamano initially suggested a middle ground allowing AI-assisted contributions from trusted contributors who disclose their use of tools and demonstrate sufficient due diligence. However, Oswald Buddenhagen forcefully rejected this approach, arguing that AI tools inherently undermine the ability to verify code provenance—even for trusted contributors—due to risks of license contamination or copyright infringement from training data.

This reply narrows the discussion to a binary choice: maintain the strict policy (blocking Alexey Samsonov's `utimensat()` series and similar contributions) or accept the legal risks of allowing AI-generated code. The Project Leadership Committee or broader community must now clarify whether the policy will be amended to reflect a more nuanced stance or upheld as-is.

Weijie Yuan raised an additional concern about whether `Assisted-by` trailers create liability by demonstrating "knowing" distribution of potentially infringing code, framing the discussion as a trade-off between legal risk mitigation and maintainer awareness.

### ODB abstraction and pluggable backends
**[2026/08/18/07-55-55]**
The `report` hook feature series advanced toward integration today. Junio C Hamano queued the v3 series in `seen` and manually resolved a conflict with Justin Tobler's concurrent "pluggable writes" series. However, he objected to the hook name `report` as overly generic and suggested `receive-report` as a more specific alternative. Patrick Steinhardt agreed with this naming concern.

The series adds a new hook that runs after all ref updates are committed but before the status report is sent to the client. The hook receives the pkt-line encoded report on stdin and replaces it with its stdout; stderr is forwarded to the client via the sideband channel. A non-zero exit rewrites all ref status lines to "report hook failed" and allows `receive-pack` to exit cleanly, aligning with the `pre-receive` hook behavior.

Karthik Nayak confirmed the series is effectively complete with all prior feedback addressed, including exit status semantics, pkt-line format documentation, and preparatory refactoring.

### `git pull --hard`/`--reset` design debate
**[2026/08/18/11-34-33]**
The design discussion around Artur Bieniek's proposal to add a `--hard` mode to `git pull` reached a potential conclusion today. Junio C Hamano firmly rejected both the original `git pull --hard` proposal and Artur's alternative `git reset --pull --hard` design, suggesting the functionality belongs in a user-configured alias rather than a built-in command.

Junio's position frames this as a workflow preference rather than a missing capability: users who frequently need this behavior can define an alias (e.g., `git config --global alias.sync '!git fetch && git reset --hard @{u}'`), while core commands remain focused on their existing responsibilities. This stance aligns with his prior objections about `git pull`'s scope and the principle that force-pushed or rebased upstream branches should require explicit user intervention.

Artur Bieniek had earlier clarified that the core use case is updating the *working tree* to match a force-pushed or rebased upstream branch, not just the branch pointer, and floated the `reset --pull` alternative to move the destructive operation to `git reset` while adding network I/O.

## In brief
- **[2026/08/08/08-21-41]** Yoichi Nakayama agreed to implement a follow-up fix for the one-argument `git worktree add` latent bug, where the command should error out when ambiguity exists rather than silently creating a local branch from HEAD.
- **[2026/08/10/19-04-48]** Phillip Wood raised forward-looking questions about the Windows-specific ODB deadlock fix, suggesting (1) passing `-c gc.auto=false` to `git commit` when spawned from the sequencer, and (2) automatically closing the ODB whenever any Git command is spawned. Junio C Hamano endorsed exploring the global ODB-release mechanism as a "great approach to study" for long-term refactoring.
- **[2026/08/13/21-12-33]** Nikolaus Schuetz proposed updating `git-symbolic-ref.adoc` to document the current behavior (exit code 128 without `--quiet`, exit code 1 with `--quiet`) rather than aligning the exit codes or dropping the tests, reaching a practical compromise in the design debate.
- **[2026/08/19/13-19-36]** Oswald Buddenhagen doubled down on preferring nesting subsystems under `lib/` (e.g., `lib/odb/`) to align with Patrick Steinhardt's `libgit.a` effort, and clarified that the primary issue with include style is the use of double quotes instead of angle brackets for cross-subsystem includes.
- **[2026/08/21/06-55-51]** Patrick Steinhardt proposed enabling `push.negotiate` by default (or conditionally for shallow clones) via a new `push.negotiate=shallow` mode, which would automatically enable negotiation for shallow clones while avoiding latency overhead for full clones.
- **[2026/08/24/13-01-10]** Junio C Hamano's "What's cooking" report confirmed five ODB/performance topics are ready for `next`: `ps/odb-geometric-repack-loose-threshold`, `ps/odb-pluggable-pack-generation`, `jt/receive-pack-pluggable-writes`, `ps/odb-eagerly-load-alternates`, and `vm/complete-history`. Patrick Steinhardt endorsed these and plans to review `dk/use-nsec-runtime` soon.
- **[2026/08/24/20-35-41]** Junio C Hamano raised a usability concern about Kristoffer Haugsbakk's `--[no-]range-diff-notes` feature for `git format-patch`, noting that `--no-notes --range-diff-notes` would suppress notes in both contexts rather than enabling them only for the range-diff.
- **[2026/08/24/21-45-40]** Junio C Hamano fixed a minor typo in `Documentation/technical/rerere.adoc` ("a file with line A its early part" → "a file with line A in its early part").