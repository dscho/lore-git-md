# Git mailing list daily digest for 2026/08/31

## The day in brief
The Git project saw significant progress on several fronts today. The `git replay --linearize` feature series reached v9, addressing all prior feedback and resolving the multi-branch ambiguity. The ODB abstraction effort continued with new pluggable fsck and alternates-at-creation topics queued in `next`. A security feature enabling OCSP staple validation in HTTPS connections graduated to `next`. Several long-standing topics, including the `USE_NSEC` runtime conversion, were marked ready for integration.

## Notable threads

### `git replay --linearize` reaches v9
**What changed**: Toon Claes posted v9 of the series introducing `--linearize` to `git replay`, which flattens merge commits into a linear history. The series is now technically complete, addressing all prior feedback including UX terminology refinements and the multi-branch ambiguity.

**Why it matters**: This feature provides a simpler alternative to Johannes Schindelin's earlier merge-replay implementation, offering predictable all-or-nothing flattening behavior. The v9 changes are non-technical, focusing on documentation clarity, reflecting the series' maturity.

### Key details

- `--linearize` cannot be used with multiple branches or `--contained`
- Merge commits are dropped, with subsequent commits reparented onto the last non-merge commit
- The series is ready for review but not yet in `next`

### ODB abstraction advances with new topics in `next`
**What changed**: Junio's "What's cooking" report shows two new ODB-related topics queued in `next`: `ps/odb-alternates-at-creation` (8 patches deferring alternates setup to ODB creation) and `ps/odb-pluggable-fsck` (10 patches moving fsck checks into backend-specific layers).

**Why it matters**: These topics represent key milestones in Patrick Steinhardt's ongoing ODB abstraction effort, which aims to make alternates and fsck checks implementation details of ODB backends.

### Key details

- `ps/odb-alternates-at-creation` removes the ad-hoc alternates writing API
- `ps/odb-pluggable-fsck` moves fsck logic into respective ODB backends
- Both topics are part of a multi-year effort to simplify the ODB interface

### OCSP staple validation graduates to `next`
**What changed**: Junio confirmed that the `http.sslVerifyStatus` feature, which enables OCSP staple validation for HTTPS connections, is ready for merge to `next`.

**Why it matters**: This security feature closes a gap where Git would accept revoked certificates from servers that don't provide OCSP staples, addressing a requirement for government and FIPS-compliant users.

### Key details

- New config key: `http.sslVerifyStatus` (boolean, default `false`)
- Fail-closed behavior: connections fail if staple missing or invalid
- Addresses security gap for OpenSSL-linked Git binaries in FIPS deployments

### `git checkout -m` autostash conflict handling
**What changed**: Harald Nordgren posted v2 of the series refining `git checkout -m` behavior when autostash is involved, focusing on the uncontroversial portion of separating autostash conflict advice from branch-switch confirmation messages.

**Why it matters**: This UX improvement addresses visual clutter without altering conflict resolution logic, making the output more readable during conflict scenarios.

### Key details

- Repurposed return value to encode three states: success (0), error (-1), conflict (1)
- Checkout logic prints blank line to stderr when conflicts detected
- Test updated to verify full output with `test_cmp`

## In brief

- **`git ident` redesign**: Andrew Pleeter posted v2 of the command (renamed from `git whoami`) with additive flags, machine-readable output, and support for X.509 signing keys, addressing all prior review concerns.
- **ODB alternates refactoring**: Patrick Steinhardt agreed to adopt the lockfile API for writing `info/alternates`, addressing TOCTOU race concerns in the alternates-at-creation series.
- **Trace2 hardening**: Derrick Stolee's series banning `die()`-triggering helpers from trace2 received final confirmation from Jeff King and Patrick Steinhardt, clearing it for `master`.
- **Branch name validation**: Bence Csókás reported an inconsistency where porcelain commands allow creating a branch named `@` despite `git check-ref-format` rejecting it.
- **`git worktree add` fixes**: René Scharfe's series fixing out-of-bounds reads and trailing-slash handling in `worktree_basename()` was merged to `next`.
- **`git fsck` pluggability**: Patrick Steinhardt's 10-patch series making fsck checks pluggable received final sign-off from Karthik Nayak, with all feedback addressed.
- **CI workflow cancellation**: Harald Nordgren's v3 patch for canceling stale GitHub Actions workflow runs was accepted, with Junio confirming the duplicate-run edge case is acceptable.
- **Typo fixes**: Several contributor housekeeping patches fixing spelling errors in comments and test descriptions were reviewed and either merged or awaiting final tweaks.