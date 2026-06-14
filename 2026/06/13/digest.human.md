# Here's the daily digest for 2026/06/13:

## The day in brief

A moderately busy Saturday with 45 emails across 16 threads, featuring significant discussions on security models, performance optimizations, and documentation improvements. The standout threads include a security proposal for hook execution in unzipped repositories, a major performance optimization for merge-base calculations, and ongoing debates about shallow clone configuration approaches.

## Notable threads

### Secure hook execution proposal faces security model concerns

Jamison Phillips' RFC for securing hook execution in unzipped repositories (via user confirmation and cryptographic verification) met substantive pushback from brian m. carlson, who argued Git's security model doesn't guarantee safety with untrusted working trees. The discussion revealed fundamental tensions between security and workflow needs, particularly around non-interactive contexts like CI/CD. The proposal now faces questions about whether it addresses the right problem space.

### Merge-base optimization shows 1000x speedups

Kristofer Karlsson provided concrete performance data showing 300-1000x speedups (4.85s → 6ms) for merge-base calculations in monorepo scenarios. The optimization introduces early termination when one commit branch is exhausted, with detailed analysis of STALE bit propagation and generation number handling. While the technical approach appears sound, some edge case testing (particularly with octopus merges) remains pending.

### Shallow clone configuration debate

Two parallel threads (from Matt Hunter and Hadrien Loge) proposed environment variables for shallow clone configuration, but maintainer Junio C Hamano strongly prefers a configuration-based approach in one thread, citing maintainability concerns. The discussion highlights tensions between automation needs and Git's configuration philosophy, with the environment variable approach now facing an uphill battle despite clear use cases in packaging workflows.

### Documenting patch review etiquette

Weijie Yuan's RFC to formalize patch review best practices in contributor documentation gained maintainer support. The series would explicitly document expectations around discussing feedback before rerolling and appropriate reroll cadences. Junio added valuable perspective about how slower iteration improves review quality and global participation, which the author will incorporate into the next version.

## In brief

**Tig 2.6.1 security release** -- Thomas Koutcher and team released tig 2.6.1 with fixes for a command injection vulnerability (#1432) plus 15 bugfixes and 4 improvements.

**Graph visualization improvements** -- Pablo Sabater's v5 series for `git log --graph` cascading indentation addresses CI failures by unsetting COMMIT_GRAPH in tests, with one known edge case remaining in revision.c.

**Compiler compatibility updates** -- Dominik Loidolt's v4 series modernizing version check macros in `compat/posix.h` received Junio's approval for integration into `next`.

**Test grep standardization** -- Michael Montalbo's v2 series adding `greplint.pl` to convert bare grep to `test_grep` continues despite maintainer concerns about automated conversion methodology.

**Global followRemoteHEAD ping** -- Harald Nordgren checks on status of his long-running `--track=fetch` series after mixed feedback, with the technically complete implementation awaiting a maintainer decision.

**NonStop Rust workaround** -- Randall S. Becker confirmed `NO_RUST=yes` successfully works around build issues, though Git 3.0's planned Rust requirement remains a future concern.

## On the radar

**No-op commit amend rejection** -- Junio rejected Harald Nordgren's proposal to optimize `--amend --no-edit` for unchanged commits, citing workflow compatibility concerns, though the discussion revealed some users rely on the current behavior for CI re-runs.

**Ref backend path handling** -- Jeff King's performance analysis showed 10x slowdowns with absolute paths in edge cases, though the tradeoff was deemed acceptable given modern ref storage formats.