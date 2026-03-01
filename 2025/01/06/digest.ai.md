# Git Mailing List Digest - 2025/01/06

**The day in brief.** A busy Monday with 87 emails across 27 threads saw significant progress on Git 3.0 preparation, including infrastructure for breaking changes and deprecation of legacy remote configuration methods. Notable discussions included protocol capability proposals, type safety improvements for MSVC, and a new OS version reporting feature. Junio released Git v2.48.0-rc2 while maintaining active review engagement across multiple threads.

## Notable threads

### Breaking changes infrastructure for Git 3.0

Patrick Steinhardt's series to remove long-deprecated features saw extensive discussion today. The v2 patches introduce a `WITH_BREAKING_CHANGES` build option to control removal of obsolete functionality, starting with the `branches/` and `remotes/` directory-based remote configuration methods deprecated since 2005 and 2011 respectively. The series restructures CI to test breaking changes and removes the `pack-redundant` subcommand.

Junio Hamano provided historical context, noting the `.git/branches/` approach had unique workflow advantages for frequently-changing remotes that may not be fully replicated by modern equivalents. Christian Couder suggested enhancing the deprecation warnings to mirror the more aggressive approach used for `pack-redundant` (which required explicit `--i-still-use-this` opt-in), which Junio strongly endorsed. The discussion shows careful consideration of user experience during breaking changes while maintaining the technical direction toward removal.

### Protocol capability for OS version reporting

An Outreachy project introduced a new 'os-version' capability for Git's protocol v2, allowing clients and servers to exchange operating system information with privacy controls (`transfer.advertiseOSVersion` defaults to true but can be disabled). The 4-patch series includes configurable command-based version string generation via `osversion.command`.

Junio Hamano and Eric Sunshine provided detailed reviews, questioning placeholder variable usage in documentation and Windows test handling (where `uname(2)` behavior differs). The discussion highlighted the need for standardized OS name guidelines while maintaining the privacy-focused design. The series represents Git's continued protocol evolution while addressing real-world needs for environment identification.

### Remote ref namespace collision discussion

Matthias Braun reported a bug where deleted remote branches could cause fetch failures when replaced with slash-containing names (e.g., 'mystuff' → 'mystuff/branch_in_subdir'). Brian M. Carlson clarified this is expected behavior - Git preserves deleted remote tracking branches by default for historical context, and the solution is explicit pruning via `git fetch --prune` or configuration options.

Junio Hamano and Theodore Ts'o expanded the discussion into protocol-level considerations for triangular workflows, suggesting server-advertised push targets or custom capabilities as more robust solutions than branch naming conventions. The thread evolved into a specific feature proposal for commit reachability verification before expensive operations like VM launches in kernel testing workflows.

## In brief

**Bugfixes:** Elijah Newren addressed a regression in caret-based tag peeling (`^0` suffix handling) that affected Git's release process. M Hickford fixed credential-cache capability handling after protocol compliance issues were identified. Sören Krecker's type safety series resolved MSVC warnings across multiple subsystems, with edge case fixes for date parsing.

**CI Modernization:** Patrick Steinhardt's 10-patch series to modernize CI infrastructure neared completion, with fixes for flaky tests and containerized job standardization. The changes showed negligible performance impact (within ±1 minute across jobs) while improving reliability.

**Documentation:** Jean-Noël Avila's synopsis-style conversion work continued with reviews of git-notes and git-restore manpages. Feedback focused on commit message style and markup consistency in the ongoing standardization effort.

**Releases:** Junio announced Git v2.48.0-rc2 with 584 non-merge commits from 81 contributors. Randall Becker noted delayed NonStop platform testing due to maintenance on primary build systems.

## On the radar

The `the_repository` removal effort saw review feedback from Toon Claes, who suggested more targeted use of available repository pointers rather than passing `the_repository` through progress APIs. This architectural discussion may influence the direction of this long-running effort to eliminate global state.