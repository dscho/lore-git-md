# Git Mailing List Digest - 2025/11/26

**The day in brief.** A busy Wednesday with 67 emails across 18 threads saw significant activity around the `git replay` command's revert functionality, a new HTTP 429 retry mechanism, and ongoing discussions about reference backend selection. Junio's "What's cooking" report provided a comprehensive overview of the project's current state, while several documentation and bugfix patches rounded out the day's traffic.

## Notable threads

### `git replay` revert functionality design

The discussion around adding revert capability to `git replay` reached a critical design decision point today. Phillip Wood and Elijah Newren both argued that `--revert` should be a distinct operation mode rather than a flag that combines with `--onto` or `--advance`. Junio Hamano endorsed this perspective, noting it resolves questions about flag interactions. Siddharth Asthana, the feature's author, acknowledged the feedback and plans to restructure the interface accordingly in v2.

The thread also surfaced important questions about naming the operation modes, with Junio suggesting `--rebase`/`--cherry-pick` might be clearer than the current `--onto`/`--advance` names. Elijah Newren countered that the current names better reflect the actual operations given `git replay`'s explicit range specification requirements. This interface design discussion will significantly impact how intuitive the new command feels to users.

### HTTP 429 retry support

A new 3-patch series introduced automatic retry support for HTTP 429 (rate limited) responses. The implementation adds configurable retry behavior through three new http.* options: `maxRetries` (default 0), `retryAfter` (fallback delay), and `maxRetryTime` (300s ceiling). The series includes thorough test coverage of various Retry-After header formats and edge cases.

Notably, the second patch fixes a pre-existing memory leak in HTTP error handling that became more apparent with the new retry paths. The final patch adds comprehensive trace2 instrumentation to monitor retry operations in production. This appears to be a well-considered solution to a common pain point when interacting with rate-limited Git servers.

### Reference backend selection via URI

The discussion about selecting reference backends through a `GIT_REF_URI` environment variable continued with Junio Hamano raising several architectural concerns. He questioned whether the current URI terminology accurately reflects the implementation (which currently only handles directory paths) and proposed moving from environment variables to explicit API calls via a new `set_main_ref_store()` function.

The thread also saw debate about URI format details, particularly whether to require `file://` prefixes for filesystem paths or keep the simpler `reftable://path` syntax. This discussion ties into broader questions about how Git should handle pluggable storage backends, with Patrick Steinhardt's parallel work on object database abstraction providing relevant context.

## In brief

**Documenting `git-last-modified` options** -- Toon Claes sent a 3-patch series adding thorough documentation for previously undocumented features (-z NUL termination and --max-depth traversal control), with Junio suggesting implementation refinements for the NUL handling.

**Patience diff optimization** -- Yee Cheng Chin's simple 4-line change to avoid unnecessary binary searches in the patience diff algorithm showed 8-14% speedups in benchmarks, earning Junio's approval as a stupidly clever optimization.

**Scalar config documentation** -- A 5-patch series from Derrick Stolee added "# set by scalar" annotations, fixed a performance regression in index.skipHash, removed stale configs, and fully documented Scalar's configuration approach.

**gitk window persistence restored** -- Johannes Sixt controversially reverted a 2008 change that stopped gitk from remembering window positions between sessions, along with adding external diff support for renamed files.

**Cache-tree parsing safety** -- Jeff King and Junio debated approaches to hardening cache-tree parsing in Jeff's ASan series, considering both immediate fixes and a potential v2 format change.

## On the radar

**NonStop platform issues** -- Randall Becker continues investigating test failures in `git last-modified` on NonStop systems, now focusing on potential enum vs int bitwise operation differences on big-endian architectures.

**Plumbing/porcelain localization** -- Peter Krefting raised a long-standing inconsistency where `git push` shows English "Everything up-to-date" while `git pull` localizes its equivalent message, seeking architectural guidance on proper separation.