**The day in brief**
23 July 2026 was a busy but routine day on the Git mailing list: 46 emails across 20 threads, with a mix of stalled topics nudged back to life, new bugfixes posted, and one long-running performance regression finally framed as a systemic alternates issue. The single headline is Junio’s “What’s cooking” report (#10), which graduated 11 topics to `master` and listed a dozen new ones under review—including Patrick Steinhardt’s `remote-object-info` batch-command, now effectively Ack’d by Karthik Nayak.

---

### Notable threads

**`git mv` misleading error messages**
Lucas Zamboni Orioli’s two-patch series fixing `git mv`’s error messages when the destination directory is missing reached v3 after Junio C Hamano reopened the symlink-handling debate. The first patch (improved error message) is already approved; the second (early directory existence check) now uses `stat()` to follow symlinks, which Junio rejects as unsafe in Git working trees. The thread is back in design limbo, with no clear resolution yet.

---

**`git repack --drop-filtered` for partial clones**
Siddharth Shrimali’s RFC series reclaiming disk space in partial clones by dropping promisor blobs exceeding a size threshold is now in its second design iteration. The blocking bitmap-validation flaw is settled, and Siddharth Asthana’s review elevates safety guards (merge/rebase checks, index validation) to non-negotiable prerequisites for v2. The drop log is implemented but still lacks documentation and test coverage for real drops. The series remains in active design, with no integration branch assigned.

---

**ODB performance regression in alternates**
Jeff King (Peff) reframed the `git receive-pack` connectivity-check slowdown as a **systemic alternates performance issue**, broadening the scope beyond the quarantine area to include `clone -s`, `--reference`, and large-scale hosting deployments. His critique of Patrick Steinhardt’s transaction-aware ODB write proposal—arguing that cross-process dependencies (e.g., `index-pack`/`rev-list` coordination) would force the solution to replicate existing “alternates juggling”—leaves the near-term fix (restoring the “pack-first, loose-second” lookup order across backends) as the most viable path forward. No one has yet volunteered to implement it.

---

**`git bisect --reset-when-found`**
Harald Nordgren’s GSoC-inspired feature that automatically resets the working tree after a successful bisect is now in `next`. Johannes Sixt’s surface-level review of v3 flagged minor test-script issues (shell safety, error-message boilerplate), but the series is otherwise complete. Junio’s musing about introducing non-fatal error helpers (`error_for_incompatible_optX()`) is purely exploratory and does not block integration.

---

**`git cat-file remote-object-info`**
Patrick Steinhardt’s 13-patch series adding a `remote-object-info` batch-command to `git cat-file` received a substantive review from Karthik Nayak, who considers it ready with only two minor nits that do not require a reroll. The series lets clients fetch object metadata (currently just size) from a remote server via protocol v2 without downloading the object itself, dynamically filtering format placeholders based on the server’s advertised capabilities. Pablo Sabater’s follow-up in the “What’s cooking” thread signals the topic is now effectively Ack’d and can proceed unless new issues surface.

---

### In brief

**Swift userdiff driver** -- Johannes Sixt acked Shlok Kulshreshtha’s v2 patch, closing the review loop. The driver is now queued in `next` and ready for integration testing.

**Documentation synopsis-style conversion** -- Jean-Noël Avila posted v2 of his four-patch series converting `git imap-send`, `git format-patch`, `git send-email`, and `git request-pull` to the new AsciiDoc synopsis style. The reroll addresses Junio’s feedback on backtick usage and dash count; the series is uncontroversial and likely to graduate soon.

**Submodule remote matching with `url.*.insteadOf`** -- Éric NICOLAS’s v2 patch fixing `git submodule update --remote` when URL aliases are in use is now ready for final review. The patch resolves Junio’s feedback on commit-message clarity and code simplification.

**`git clone --revision` segfault** -- Adrian Friedli’s one-line NULL check in `update_remote_refs()` is conceptually approved, but Junio has requested a regression test before merging.

**`http.connectTimeoutMS`** -- GalaxySnail’s feature patch adding millisecond-level control over HTTP connection timeouts needs to address Junio’s feedback on use-case justification, zero-value behavior, and test-script necessity.

**`git-p4` `# fixme` ambiguity** -- Hardik Kumar’s thread remains stalled on historical context; the author has explicitly asked Junio to clarify or remove the 2007 `# fixme` comment, but no response has been provided.

---

### On the radar

**`git replay` documentation standardization** -- Kristoffer Haugsbakk’s series synchronizing `replay.refAction` documentation between man pages has stalled administratively; Junio’s nudge gives the author until the next “What’s cooking” report to post v4 or risk discard.

**Shell completion dotfile hiding** -- Zakariyah Ali’s series hiding dotfiles by default in Git’s path completion is blocked on a structural refactoring of the AWK script in `__git_index_files`. Junio’s nudge signals the topic may be discarded if no v4 is submitted soon.

**`git-interpret-trailers` documentation** -- Kristoffer Haugsbakk’s 11-patch series is already merged, but Junio’s “What’s cooking” draft still lists it as stalled. The author has been nudged to confirm the topic’s status.