Here's the daily digest for February 6, 2026:

---

### The day in brief
February 6 saw active development across Git's core functionality, with 112 emails across 27 threads. The day was dominated by technical refinements to in-progress features, security discussions, and documentation improvements. Key highlights include progress on the `merge-ours` sparse-index integration, a security vulnerability report in `git am`, and multiple patch series nearing completion after thorough review cycles.

---

### Notable threads

**Security vulnerability in `git am` patch parsing**  
A serious security issue was reported where `git am` can inadvertently apply diffs embedded in commit messages, potentially introducing malicious changes. The thread revealed this behavior led to a real-world incident in a Debian package. Discussion centered on parsing challenges and backward compatibility, with Jeff King (Peff) providing historical context showing this is a long-standing known issue. While no immediate solution was proposed, the thread established consensus that this constitutes a real vulnerability needing attention.

**`merge-ours` sparse-index integration**  
Sam Bostock's two-patch series to add sparse-index support to the `merge-ours` strategy progressed through review. The changes make `git merge -s ours` properly respect sparse-checkout settings, preventing unnecessary index expansion. Junio Hamano and Patrick Steinhardt provided feedback on commit message clarity and repository parameter safety, with the series reaching v2 after incorporating all suggestions. This continues Git's ongoing effort to optimize operations for large repositories using sparse-checkout.

**Interactive hunk selection improvements**  
Abraham Samuel Adekunle's three-patch series adding `--rework-with-file` functionality for interactive commands (`add -p`, `checkout -p` etc.) underwent detailed review. The feature allows navigating between files during hunk selection rather than processing them sequentially. Junio Hamano focused on code organization and style refinements, with the series nearing readiness pending a rebase to resolve conflicts with other in-flight changes.

**Reference backend selection mechanism finalized**  
Karthik Nayak's series implementing configurable reference storage locations (allowing zero-downtime migrations between backends) received final review from Patrick Steinhardt. Only minor documentation nits remain in this v4 series that introduces both configuration (`extensions.refStorage`) and environment variable (`GIT_REFERENCE_BACKEND`) control methods. Junio Hamano signaled the technical approach looks good, suggesting this architectural improvement is ready for integration.

**Submodule `ignore=all` behavior consistency**  
Claus Schneider's v4 series aligning `git add` behavior with `git status` and `git diff` for submodules configured with `ignore=all` reached completion. The changes make `git add` skip such submodules by default unless forced with `--force`, addressing a long-standing inconsistency. The implementation includes comprehensive tests and documentation updates, with all technical feedback from earlier rounds addressed.

---

### In brief

**Meson build system updates** -- Patrick Steinhardt confirmed Windows compatibility for gitk/git-gui integration and fixed a missing `msgfmt` dependency in CI, with Junio handling the final subtree update.

**Documentation standardization** -- Jean-Noël Avila's v4 series for synopsis-style man page conversions received Kristoffer Haugsbakk's Reviewed-by after addressing all feedback on `git-submodule`, `git-show`, and related pages.

**Whitespace/symlink handling fix** -- Junio C Hamano finalized a patch correcting incorrect whitespace checks on symbolic links, with Patrick Steinhardt confirming the refined post-image handling approach.

**PID file debugging feature** -- Patrick Steinhardt approved the final version of Paulo Casaretto's PID file creation for lock debugging, noting only an optional test cleanup remains.

**Const-correctness updates** -- Collin Funk's series addressing C23 compiler warnings progressed with minor documentation tweaks from Ben Knoble after Jeff King's technical approval.

**HTTP authentication tests** -- Discussion clarified that Ashlesh Gawande's .netrc test additions primarily verify curl's behavior rather than Git's credential logic, though the tests remain technically sound.

---

### On the radar

**ODB enumeration API changes** -- Patrick Steinhardt's object database refactoring awaits resolution on mtime handling concerns raised by Taylor Blau, with Junio noting the discussion hasn't materialized yet.

**Configurable hooks proposal** -- Adrian Ratiu's series enabling hook configuration faces maintainer questions about motivating use cases, needing clearer justification for the architectural changes.

**AI-assisted translation tools** -- Jiang Xin's RFC for AI in l10n workflows saw documentation refinements but continues facing skepticism from some maintainers about quality risks.

**Graph log visualization** -- Toon Claes proposed a new `--untangle` option for `git log --graph` to simplify complex merge histories, with Junio requesting example output to evaluate the approach.

---

The day's activity reflects Git's steady progress on multiple fronts, with particular focus on security hardening, performance optimizations for large repositories, and interface improvements. Several significant features appear ready for integration pending final documentation polish or rebase work.