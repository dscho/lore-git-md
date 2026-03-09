# Git Mailing List Digest — 2025/12/25

**The day in brief.** A quiet Christmas day with 10 emails across 6 threads, mostly wrapping up existing discussions. The most notable activity centers on final polish for Harald Nordgren's configurable branch comparison in `git status` (now at v8) and Junio's detailed review of PID-based lockfile debugging. A performance report about `git pack-refs` on large repositories may warrant attention from the refs backend experts.

## Notable threads

### PID-based lockfile debugging review

Junio Hamano provided a thorough review of the v3 patch for PID-based lockfile debugging, raising several implementation concerns. The core feature — using `~pid.lock` files to track processes holding Git locks — remains sound, but Junio identified inconsistencies in documentation (`.pid.lock` vs `~pid.lock`), questioned the parameter design, and highlighted error handling edge cases. This continues the pattern of Junio's careful implementation scrutiny after earlier threads resolved the broader design questions. The remaining issues appear addressable in a follow-up revision.

### Configurable branch comparison reaches v8

Harald Nordgren's long-running series to add configurable branch comparison to `git status` progressed through three iterations today (v6-v8), responding to Junio's feedback about patch organization and commit messages. The feature now uses `status.goalBranch` to show divergence from a secondary branch (like "upstream/main") alongside normal tracking info. The implementation appears technically complete with comprehensive test coverage across various branch states and edge cases. Junio's v6 feedback focused on presentation quality — recommending squashed commits and consistent subject lines — suggesting the technical approach is now satisfactory.

## In brief

**Refs debug output fix** — Karthik Nayak confirmed Patrick Steinhardt's earlier review of Greg Funni's fix for boolean display in refs tracing output, resolving a simple `*required` vs `required` dereference issue.

**`git branch -v` behavior inquiry** — Sergey Organov questioned why `-v` doesn't work with `--show-current`, noting the inconsistency with `--list` behavior. No patches proposed yet.

**Slow `pack-refs` performance** — Martin Fick reported `git pack-refs --all` taking 5+ minutes on a Gerrit server with 3M refs. Brian m. carlson identified object existence verification as the likely culprit, with NFS exacerbating the overhead.