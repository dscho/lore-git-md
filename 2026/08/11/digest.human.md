# Git mailing list daily digest for 2026/08/11

## The day in brief

The Git project saw significant progress on several fronts today, including the completion of a long-running optimization series, a major ODB transaction extension for `git receive-pack`, and a heated debate over root-directory reorganization. A Windows-specific textconv bug and a new security feature for OCSP validation also drew attention.

## Notable threads

### Root-directory reorganization debate intensifies
Patrick Steinhardt refined his proposal to reduce root-directory clutter, emphasizing **discoverability for newcomers** over code organization. He argued that moving library files to a `lib/` directory makes entry points like `README.md` and `Makefile` more visible, even if the `lib/` directory itself lacks clear structure. Junio C Hamano countered that the `lib/` directory is an **arbitrary intermediate layer** that fails to deliver meaningful organization, contrasting it with past subsystem moves (e.g., `builtin/`, `refs/`) that were tied to substantial updates. Junio warned the patch may *worsen* navigation by forcing contributors to re-learn the structure later. Steinhardt doubled down on the newcomer-focused rationale, conceding the change is subjective but maintaining that a cleaner root directory improves onboarding.

### `paint_down_to_common()` optimization series completes
Kristofer Karlsson announced that v8 of the `paint_down_to_common()` optimization series is **technically complete, fully reviewed, and queued in Junio’s tree**. The series is now procedurally unblocked and ready for advancement.

### ODB transactions extended to `git receive-pack`
Justin Tobler posted v3 of a nine-patch series extending ODB transactions to `git receive-pack`, addressing all feedback from v2. Key improvements include:
- Fixing a critical lockfile lifecycle flaw in the "files" backend via explicit `struct odb_source *` tracking.
- Introducing `odb_transaction_finalize()` and `odb_transaction_commit_and_finalize_or_die()` to enforce proper transaction lifecycle management.
- Removing reliance on global variables (e.g., `alt_shallow_file`, unpack limit caching) by introducing a `struct unpack_opts` and explicit file descriptor passing.
- Adding `odb_transaction_write_pack()`, the generic interface completing the ODB abstraction for `git receive-pack`.

### FSEvents queue regression explained
Tamir Duberstein provided a detailed technical explanation for the FSEvents queue behavior, tracing the regression to the April 2026 timeout change (56cef9cb1a). He cited Apple’s documentation, which states that FSEvents queue drainage is non-deterministic, and linked the original 2022 fsmonitor cookie protocol commit (b05880d357), which warned about this behavior. Patrick Steinhardt pushed back on Tamir’s use of an AI-generated explanation but acknowledged the flush call is empirically required, citing precedent from Watchman.

### Textconv filter misconfiguration on Windows
Skybuck Flying reported a misconfigured textconv filter on Windows causing silent corruption of `git diff` output. The root cause was isolated to MSYS argument parsing mishandling the `\r` escape sequence in `sed` commands, turning `s/\r//` into `s/r//`. Jeff King confirmed the issue as a shell-quoting problem in the textconv filter configuration, attributing it to user misconfiguration rather than a Git bug. Skybuck provided a corrected configuration command for Windows users: `` `git config --global diff.lfclean.textconv "sed -e s/\\r//"` ``.

## In brief
- **Reflog API debate**: Patrick Steinhardt countered Junio’s proposal for a new reflog API (`refs_reflog_edit_in_bulk()`) with an alternative: extending Git’s reference transactions to handle reflog edits natively.
- **`--drop-filtered` repack series**: Junio clarified that the three deferred items (remote verification, recency-based culling, drop log) are **intended as follow-up work**, confirming the series is feature-complete for its stated scope.
- **Trace2 instrumentation**: Patrick Steinhardt reviewed the Trace2 instrumentation patch for `git fetch-pack`, approving the test extension despite a minor test design quirk.
- **Memory-safety fix**: René Scharfe proposed a minimal rewrite of `worktree_basename()` to eliminate a pointer underflow causing a memory-safety issue.
- **Coccinelle role refined**: Patrick Steinhardt distinguished between **temporary API-migration rules** (which he questions) and **permanent antipattern-enforcement rules** (which he supports), proposing periodic audits to clean up accumulated cruft.
- **TUI proposal refined**: Steinhardt refined his TUI proposal to focus on *targeted* UI improvements (e.g., hierarchical hunk selection) rather than building a "full UI."
- **Geometric repacking threshold**: Steinhardt posted a bugfix adjusting the geometric repacking threshold for loose objects from 100 to 6700 to match `git gc`'s behavior. Justin Tobler endorsed the patch as "trivially correct."
- **Segmentation fault fix**: Christian Couder posted a bugfix preventing a segmentation fault when `git --shallow-file` is invoked without a required argument. Junio raised procedural concerns about the patch’s scope and test coverage.
- **`http.sslVerifyHost` debated**: Patrick Steinhardt and brian m. carlson opposed a new `http.sslVerifyHost` option, arguing it oversells its security benefits and is vulnerable to active MITM attacks.
- **Reflink support**: Vlad Petric posted a patch adding reflink support for local clones, revising the fallback order to hardlink → reflink → byte-for-byte copy after Junio’s feedback.
- **OCSP validation**: graysongordon-gl posted a patch introducing `http.sslVerifyStatus`, enabling Git to validate stapled OCSP responses during TLS handshakes. Junio requested the removal of an in-code comment explaining the default.
- **Git for Windows security update**: Johannes Schindelin announced Git for Windows 2.55.0(4), fixing CVE-2026-62960, a Windows-specific vulnerability in bundle URI handling.
- **Worktree lifecycle hooks**: Alexander G. Riccio described a real-world use case for worktree lifecycle hooks, citing gigabytes of orphaned build cache data left behind by Xcode.
- **Proxy authentication bug**: Tom Noonan II reported a bug in `git-remote-https` where the HTTP transport layer fails to extract and use credentials from proxy URLs.
- **Bash completion fix**: Junio posted a two-patch series fixing Bash completion for `git checkout -C <dir>` to only suggest tracked paths.