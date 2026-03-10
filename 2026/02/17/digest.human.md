Here's the daily digest for February 17, 2026:

**The day in brief.** A busy day with 98 emails across 23 threads, dominated by final polishing of several major series - ref backend selection, HTTP 429 retry support, and worktree API refactoring. Key developments include Junio Hamano's security analysis of push certificate changes and consensus reached on multiple long-running efforts.

**Notable threads**

**Ref backend selection nears completion**  
Karthik Nayak's series enabling zero-downtime migrations between ref backends (files<->reftable) is in final polishing after maintainer approval. Today's patches (v6) addressed environment variable support, stub management centralization, and path safety confirmation. Patrick Steinhardt and Karthik discussed test verification details, with only minor question remaining about whether to add backend-specific assertions for alternate reference directories. The series implements critical functionality for GitLab's large repository needs through multiple configuration mechanisms (config, URI, environment).

**HTTP 429 retry support finalized**  
Vaidas Pilkauskas's v3 series implementing RFC-compliant handling of rate-limited HTTP responses is complete. The simplified blocking sleep() approach replaces v2's non-blocking timers and includes curl-native Retry-After parsing (curl >=7.66.0), test portability improvements using test-tool date, and comprehensive trace2 instrumentation. The implementation provides three new options (http.maxRetries, http.retryAfter, http.maxRetryTime) to configure retry behavior when servers return 429 status codes.

**Worktree API refactoring progresses**  
The ongoing effort to standardize worktree handling saw extensive discussion about NULL parameter semantics and architectural direction. Phillip Wood proposed `get_current_worktree()` to construct worktree structs from repository data, while Shreyansh Paliwal worked in parallel on wt-status.c cleanup. Junio Hamano provided key guidance on repository/worktree relationships, questioning whether `repo->worktree` sufficiently defines the association. The thread demonstrates careful coordination between multiple contributors on this complex refactoring.

**Push certificate security analysis**  
Junio Hamano raised significant security concerns about Lorenz Leutgeb's proposal to modify push certificate "pushee" headers for privacy. Junio identified potential attack vectors if certificates become repository-agnostic, emphasizing their role in binding ref updates to specific repositories. Lorenz later clarified a legitimate middleman use case involving remote helpers, but the discussion revealed deeper design questions about what push certificates should certify beyond cryptographic signatures.

**In brief**

**Config-list.h dependency tracking** -- D. Ben Knoble finalized Meson build system handling of `config-list.h` dependencies with proper path escaping and explicit script dependencies, fixing real-world bisection issues.

**German translation fix** -- Matthias Schwarzott corrected a misspelling in the German translation of "allow-indentation-change" that was causing error message loops.

**Format-patch --from fix** -- Mirko Faina's patch ensuring `--from` applies to cover letters reached v4 with all review feedback addressed, including documentation updates to clarify the behavior.

**Git apply directory handling** -- Patrick Steinhardt suggested using `strbuf_normalize_path()` for comprehensive path handling in `--directory` rather than just fixing the `./` prefix case.

**On the radar**

**the_repository removal** -- Tian Yuchen inquired about GSoC project scope for continuing René Scharfe's long-running effort to eliminate global state, with Karthik Nayak providing initial guidance on subsystem selection.

**Core.trustctime migration** -- Junio Hamano raised concerns about delayed error detection in Ayush Jha's RFC patch moving the config to `struct repo_settings`, suggesting early validation may be preferable to lazy loading.