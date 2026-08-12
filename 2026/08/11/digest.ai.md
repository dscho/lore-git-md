# Git mailing list daily digest for 2026/08/11

## The day in brief

Patrick Steinhardt and Junio C Hamano debated the merits of a root-directory reorganization to reduce clutter, while Justin Tobler posted a major revision of the ODB transaction series for `git receive-pack`. A critical Windows textconv bug was diagnosed, and Git for Windows 2.55.0(4) was released to address a security vulnerability.

## Notable threads

### Root-directory reorganization for discoverability
Patrick Steinhardt refined the motivation for moving library code into a `lib/` directory, emphasizing **reducing root-directory clutter** to improve newcomer discoverability of non-library files like `README.md`, `SECURITY.md`, and subsystem directories (`refs/`, `builtin/`). Steinhardt acknowledged Taylor Blau’s critique that `lib/` does little to clarify project structure but argued the primary benefit is **filtering out noise** rather than organizing library code. Steinhardt explicitly stated the goal is **discoverability for newcomers**, not "prettiness" or code hygiene, conceding the change is subjective but maintaining a cleaner root directory makes entry points more explicit.

Junio C Hamano critiqued the `lib/` directory as an **arbitrary intermediate layer** that fails to deliver meaningful organization, contrasting it with past subsystem moves (e.g., `builtin/`, `refs/`, `odb/`) that were tied to *substantial updates* and created clear, intuitive groupings. Hamano argued the patch may *worsen* navigation by forcing contributors to re-learn the structure later when further reorganization becomes necessary. Steinhardt doubled down on the newcomer-focused rationale, arguing a cleaner root directory—even without further classification—improves discoverability by making entry points more explicit, but did not engage with Hamano’s incremental reorganization alternative.

### ODB transactions for `git receive-pack` (v3)
Justin Tobler posted v3 of the nine-patch series extending ODB transactions to `git receive-pack`, addressing all feedback from v2. Key fixes include:
- A critical lockfile lifecycle flaw in the "files" backend, ensuring lockfile paths remain valid post-commit via explicit `struct odb_source *` tracking in `index_pack_lockfile()`.
- Introduction of `odb_transaction_finalize()` and `odb_transaction_commit_and_finalize_or_die()` to enforce proper transaction lifecycle management.
- Removal of the static caching of the unpack limit, replacing it with a lazy, per-repository lookup via `repo_config_get_uint()`.
- Refactoring `unpack()` to eliminate reliance on global variables by introducing a `struct unpack_opts` that explicitly passes all previously implicit state.
- Replacement of the implicit stdin assumption in `unpack()` with an explicit file descriptor parameter (`int pack_fd`) for backend-agnostic use.
- Addition of explicit ODB source tracking to avoid fragile reliance on source list ordering.
- Introduction of `odb_transaction_write_pack()`, the generic interface completing the ODB abstraction for `git receive-pack`.

### Windows textconv filter misconfiguration
Skybuck Flying reported a misconfigured textconv filter on Windows causing silent corruption of `git diff` output, tracing the root cause to MSYS argument parsing mishandling the `\r` escape sequence in `sed` commands. Jeff King confirmed the issue as a shell-quoting problem, noting the filter `sed -e s/\r//` is interpreted as `s/r//`, stripping all 'r' characters instead of carriage returns. Skybuck provided a corrected configuration command for Windows users: `` `git config --global diff.lfclean.textconv "sed -e s/\\r//"` ``. Theodore Tso dismissed the issue as a "Windows problem" rather than a Git usability concern.

### Geometric repacking threshold adjustment
Patrick Steinhardt posted a bugfix patch adjusting the geometric repacking threshold for loose objects from 100 to 6700 to match `git gc`'s behavior and reduce race conditions in high-concurrency repositories. Justin Tobler reviewed the patch, confirming the mechanism behind the overly aggressive repacking and endorsing the patch as "trivially correct," though questioning the specific choice of 6700 in the commit message.

### `http.sslVerifyHost` security debate
Patrick Steinhardt reviewed the `http.sslVerifyHost` security-adjacent configuration option, arguing it oversells its security benefits and should explicitly acknowledge its vulnerability to active MITM attacks. Brian M. Carlson strongly opposed the option, asserting it provides no meaningful security, creates a false sense of security, and is unnecessary given modern alternatives (ACME, DANE). Carlson highlighted ARP spoofing as a realistic MITM vector on unsecured networks and noted that CNs are obsolete in TLS verification.

### Reflink support for local clones
Vlad Petric via GitGitGadget posted a patch adding reflink support for local clones and file copies on Linux filesystems that support copy-on-write reflinks (e.g., Btrfs, XFS, ZFS). Junio C Hamano reviewed the patch, questioning why reflinks are attempted *before* hardlinks, given that object files are immutable and hardlinks are universally supported on local filesystems. Petric revised the patch to change the fallback order to hardlink → reflink → byte-for-byte copy.

### Git for Windows 2.55.0(4) security release
Johannes Schindelin announced Git for Windows 2.55.0(4), which fixes CVE-2026-62960, a Windows-specific vulnerability in bundle URI handling that could disclose the user’s NTLMv2 hash via attacker-controlled bundle URIs pointing to network shares.

## In brief

- **`paint_down_to_common()` optimization (v8)**: Kristofer Karlsson via GitGitGadget posted v8 of the series, now **technically complete, fully reviewed, and queued in Junio’s tree**.
- **Reflog API vs. reference transactions**: Patrick Steinhardt countered Junio’s proposal for a new reflog API (`refs_reflog_edit_in_bulk()`) with an alternative: extending Git’s reference transactions to handle reflog edits natively.
- **`--drop-filtered` repack series**: Junio C Hamano clarified that three deferred items (remote verification, recency-based culling, drop log) are **intended as follow-up work, not blocking requirements for v5 and later revisions**.
- **Trace2 instrumentation for `git fetch-pack`**: Patrick Steinhardt reviewed the patch, confirming the instrumentation is useful and approving the test extension.
- **FSEvents queue behavior**: Tamir Duberstein provided a detailed technical explanation for the regression’s origin, tracing it to the April 2026 timeout change (56cef9cb1a) and citing Apple’s documentation. Patrick Steinhardt pushed back on Tamir’s use of an AI-generated explanation, insisting the author demonstrate understanding of the behavior.
- **`worktree_basename()` memory-safety fix**: René Scharfe proposed a minimal rewrite to eliminate the pointer underflow, replacing the original loop with an integer-index loop that stops at the string boundary.
- **Coccinelle’s role in Git**: Patrick Steinhardt refined his position, distinguishing between **temporary API-migration rules** (which he questions) and **permanent antipattern-enforcement rules** (which he supports).
- **TUI proposal refinement**: Patrick Steinhardt refined his earlier proposal to focus on *targeted* UI improvements (e.g., hierarchical, navigable hunk selection) as a pragmatic middle ground.
- **`git switch` advice message (v4)**: Yoichi Nakayama via GitGitGadget posted v4 of the patch, addressing Junio’s feedback on the commit message and advice message’s usability. Ben Knoble endorsed signaling ambiguity via warning or advice when no deterministic resolution policy exists.
- **Editor integration via `%f` substitution**: Johannes Sixt proposed introducing the feature in **small, reviewable steps**, starting with a minimal use case (e.g., "open an editor at this file/line" using only `%f` substitution).
- **Geometric repacking edge case**: Patrick Steinhardt suggested triggering an all-into-one repack when objects exceed a certain percentage of the repository size. Taylor Blau acknowledged the theoretical risk but argued the scenario is unlikely in practice.
- **ODB alternates refactoring**: Justin Tobler provided substantive reviews of the first three patches, confirming the motivation and implementation, and raised a clarifying question about the fourth patch’s handling of `GIT_ALTERNATE_OBJECT_DIRECTORIES`.
- **Sequencer resource safety**: Junio C Hamano raised a concern about the safety of releasing resources mid-operation in the sequencer, suggesting the commit message clarify that the ODB will be reacquired in a "refreshed state" after auto-maintenance.
- **Bash completion for `git checkout -C <dir>`**: Junio C Hamano posted a two-patch series fixing completion to only suggest tracked paths (those in the index) rather than falling back to untracked paths.
- **`--shallow-file` segmentation fault**: Christian Couder posted a bugfix patch preventing a segmentation fault when `git --shallow-file` is invoked without a required argument. Junio C Hamano reviewed the patch, raising procedural concerns about scope and test coverage.
- **`http.sslVerifyStatus`**: Grayson Gordon posted a patch introducing `http.sslVerifyStatus`, enabling Git to validate stapled OCSP responses during TLS handshakes. Junio C Hamano requested removal of an in-code comment explaining the default, arguing design rationale belongs in the commit message.
- **Worktree lifecycle hooks**: Alexander G. Riccio provided a real-world use case for post-remove and post-prune hooks, describing how Xcode and other build systems leave behind gigabytes of orphaned build cache data.
- **`git-remote-https` proxy authentication bug**: Tom Noonan II reported a bug where the HTTP transport layer does not automatically extract and use the username:password pair from a proxy URL when the proxy responds with a 407 challenge.