Here's the daily digest for September 22, 2025:

## The day in brief
A busy day with focused technical discussions rather than major new developments. The Rust integration effort dominated traffic with coordination between Patrick Steinhardt and Ezekiel Newren, while several bugfix series reached completion. Notable threads included push optimization analysis and ongoing platform compatibility concerns for Rust adoption.

## Notable threads

**Rust integration coordination** -- Patrick Steinhardt and Ezekiel Newren aligned their parallel Rust integration efforts, agreeing that Steinhardt's foundational patches will form the base for Newren's follow-up work. They resolved key questions about crate naming (`gitcore`), type naming conventions (standard C-style types at boundaries), and documentation accuracy. The discussion also covered platform compatibility timeline concerns, with Junio Hamano expressing reservations about the aggressive 18-month schedule for making Rust mandatory.

**Push optimization analysis** -- A thread exploring inefficient object transfers during push operations identified a regression where amended commits resend unchanged blobs. Jeff King traced this to a 2014 change optimizing for performance over bandwidth, proposing several potential solutions including smarter tree marking and bitmap usage. Sainan confirmed bitmap repacking works as a workaround but noted the need to maintain bitmaps after pulls.

**Dangling symref deletion edge case** -- Toon Claes identified a workflow breakage where intentional null-to-null updates of dangling symrefs were blocked by Jeff King's recent protection against accidental clobbering. The discussion evolved into broader questions about `update-ref` API design, particularly whether `update` should handle symref deletion when `symref-delete` exists. Junio and Jeff debated the tradeoffs between API consistency and workflow support.

**In brief**

**Stash index configuration** -- A 4-patch series adding `stash.index` config to control index restoration during stash application completed review and is ready for merging. The feature allows preserving staged state by default when desired.

**Xdiff refactoring** -- Ezekiel Newren's 12-patch series simplifying xdiff internals for Rust compatibility reached v4, with final discussion about boolean type usage for the `changed` field (formerly `rchg`).

**Color config fixes** -- Patrick Steinhardt's 5-patch series fixing `git config get --type=color` behavior was finalized, addressing test modernization, error handling, empty key support, and pager behavior.

**Gitk macOS fixes** -- Johannes Sixt picked up a long-standing patch to make gitk's window focusing robust when `osascript` is unavailable, moving it toward inclusion after 7 years.

**On the radar**

**SHA-1/SHA-256 interoperability** -- Brian m. carlson's work on hash algorithm conversion continues with tag signature validation rules being refined. The effort remains closely tied to Rust adoption timelines.

**Scalar config discoverability** -- Patrick Steinhardt and Derrick Stolee coordinated on documenting Scalar's configuration rationale while maintaining technical marking of config provenance.