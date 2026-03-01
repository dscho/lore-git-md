# Git Mailing List Digest — 2025/01/09

**The day in brief.** A busy Thursday with 65 emails across 20 threads saw significant activity in several areas: Jeff King's major combine-diff refactoring series reached completion, Patrick Steinhardt addressed multiple memory safety issues, and discussions continued about deprecation warnings and configuration handling. The Git for Windows 2.48.0-rc2 release candidate also dropped, bringing platform-specific fixes.

## Notable threads

### Combine-diff refactoring complete

Jeff King's 14-part series to improve Git's combine-diff implementation reached completion today with Junio Hamano's review and merge. The systematic refactoring ([initiated yesterday](https://lore.kernel.org/git/20240108135148.GA28754@coredump.intra.peff.net/)) touched nearly every aspect of the combine-diff path handling, with highlights including:

- Introduction of `combine_diff_path_new()` constructor to centralize tricky allocation logic
- Removal of marginal optimizations that complicated the code for minimal benefit
- Simplified list management using pointer-to-pointer techniques
- Better documentation of struct fields and invariants

The changes netted a 184-line reduction while making the code more maintainable. Junio noted some underlying complexity remains ("one ugly hack") in the memory layout, but approved the improvements to clarity and safety.

### Memory safety in blame and credential-cache

Patrick Steinhardt addressed two subtle memory safety issues:

1. **Blame OID output**: Fixed an out-of-bounds read when using absurdly large `--abbrev` values (like 9000) that stemmed from switching from `printf` to `fwrite`. After [Johannes Schindelin identified the issue](https://lore.kernel.org/git/nycvar.QRO.7.76.6.2501091049430.55@tvgsbejvaqbjf.bet/), the solution reverted to `printf` with length truncation while preserving special formatting for boundary commits (marked with ^ prefix).

2. **Credential-cache daemon**: Patched the helper to only include authtype and credential data in responses when explicitly requested via `CREDENTIAL_OP_RESPONSE` capability, fixing a case where "credential fill" could print incomplete credentials. This v5 iteration [Junio's earlier feedback](https://lore.kernel.org/git/xmqqh6jjy6zl.fsf@gitster.g/) about the behavioral change's impact.

### Deprecation warning strategy debate

An ongoing discussion about legacy reference naming deprecations ([started in December](https://lore.kernel.org/git/4e2a4b0f-1e1e-4f4e-b3e3-5e5e5e5e5e5e@example.com/)) saw new input today on how to handle warning messages. The thread revealed philosophical differences:

- **Junio Hamano** favored mandatory warnings as migration incentives, citing painful transitions from Git 1.6
- **Patrick Steinhardt** proposed an environment variable escape hatch (`GIT_ALLOW_DEPRECATED_REMOTES`) for smoother transitions
- **Robert Coup** suggested a more general deprecation suppression mechanism

Junio ultimately [expressed reservations](https://lore.kernel.org/git/xmqq5y0qj1p8.fsf@gitster.g/) about making warnings too easy to ignore, though remained open to solutions that don't "make the Git 3.0 transition overly painful."

### Git for Windows 2.48.0-rc2 released

Johannes Schindelin announced [Git for Windows 2.48.0-rc2](https://lore.kernel.org/git/nycvar.QRO.7.76.6.2501091756070.55@tvgsbejvaqbjf.bet/), featuring:

- Updated components (Git v2.48.0-rc2, cURL 8.11.1, MinTTY 3.7.7)
- New default of native Windows HTTPS transport backend
- Installer now properly prevents installation on unsupported Windows versions
- Fixed cache helper's "unknown error" messages

The release candidate status and explicit call for testing suggests this contains substantial changes from previous versions.

## In brief

**Test modernization**: Seyi Kuforiji's conversion of hash algorithm tests to the clar framework received final approvals from both Patrick Steinhardt and Junio Hamano, completing the move from t-hash.c to u-hash.c with full test coverage preserved.

**Configuration fixes**: Scott Chacon revised the `help.autocorrect` patch per Junio's feedback to handle boolean values ("true"/"false") more comprehensively while maintaining backward compatibility with numeric delays.

**Documentation**: Adam Johnson fixed misplaced trailer configuration docs by moving them from git-interpret-trailers.txt to a new config/trailer.txt, while maintaining includes from both locations.

**Build system**: Johannes Schindelin addressed a race condition in parallel documentation builds by using PID-based temporary filenames in GIT-VERSION-GEN, also incidentally fixing a Windows filename character issue.

**New bug report**: Danila Manturov reported a regression where shallow fetches (`--depth=1`) no longer retrieve tags pointing to the shallow boundary commit, a behavior that worked in Git 2.47.1 but broke in 2.48.0-rc0.

## On the radar

**OS version capability**: The discussion about adding OS version reporting to Git's protocol continues to evolve, with Usman Akinyemi now proposing predefined format options ("full", "short", "medium") instead of flexible placeholders, while Junio advocates for an even simpler boolean control.