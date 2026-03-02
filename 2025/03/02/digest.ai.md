# Git Mailing List Digest — 2025/03/02

**The day in brief.** A moderately active Sunday with 20 emails across 13 threads, featuring progress on several technical fronts. The most notable developments include continued refinement of the `show-branch` refactoring, a growing consensus around replacing the problematic git-mentoring Google Group, and an intriguing proposal for distributed code comments. The bash completion fixes sparked a performance optimization discussion, while several small but useful patches addressed documentation builds and editor configurations.

## Notable threads

**`show-branch` refactoring advances**  
Ghanshyam Thakkar built on Junio Hamano and Jeff King's earlier discussion about replacing `show-branch.c`'s static `MAX_REVS` limit with dynamic commit-slab storage. The thread has progressed from problem identification to implementation planning, with consensus around using the existing but underutilized `commit_slab_with_stride` infrastructure. Thakkar's contribution identifies two key requirements for full removal of the limit: converting static arrays to heap allocation and implementing dynamic bit storage, suggesting either a custom bitset or reuse of `ewok.h`'s bitmap implementation. This technical discussion exemplifies Git's careful approach to architectural changes, leveraging dormant capabilities that were designed for exactly this class of problem.

**Mentoring list migration gains support**  
Christian Couder joined the growing consensus to retire the spam-plagued git-mentoring Google Group, supporting Johannes Schindelin's Discord proposal while also suggesting IRC and the main mailing list as additional help channels. The discussion has moved from whether to migrate to how best to document and maintain the replacement options. With Junio Hamano already supportive of closing the Google Group, the remaining questions center on documentation location and maintenance responsibilities for the new support channels. This transition reflects the project's pragmatic approach to infrastructure when original solutions become unsustainable.

**Distributed code comments proposed**  
ZheNing Hu's proposal for Git-native inline code commenting continued to evolve, with Randall Becker suggesting an implementation using ancillary data structures tied to commits and line ranges. The discussion highlights both the potential value (distributed knowledge sharing without platform lock-in) and challenges (synchronization mechanics, non-core server support) of such a feature. Becker's technical suggestions around signature handling and merge behavior provide concrete implementation considerations, though the thread remains in exploratory stages. This proposal touches on a genuine workflow gap between Git's current tools and platform-specific features, though its path to core integration remains uncertain.

**Bash completion fixes prompt performance debate**  
David Mandelberg's series fixing bash completion for remote names with slashes drew performance concerns from Phillip Wood about scaling with many remotes. The discussion now weighs tradeoffs between correctness (proper slash escaping) and efficiency (avoiding per-remote `for-each-ref` calls), with platform compatibility and edge cases adding complexity. This technical debate exemplifies Git community's attention to both correctness and real-world performance, even for seemingly small quality-of-life improvements. The extensive test coverage (180+ lines added) provides a solid foundation for evaluating alternative implementations.

## In brief

Nikolay Shustov submitted a feature-rich git-p4 patch series adding configuration options for error encoding, changelist creation, shelving workflows, and editor control, plus a bugfix for file handling after reverts. A new contributor proposed adding Verilog language support to Git's diff patterns via `userdiff.c`. The Meson build system received a straightforward fix for documentation generation after the `.txt` to `.adoc` transition. A minimal `.editorconfig` update extended shell script formatting rules to `.bash` files. Git Rev News edition #120 was announced with its usual contributor acknowledgments.

## On the radar

The developer documentation inquiry thread may spark broader discussion about onboarding resources, with one response advocating `git log -L` as a code archaeology tool. A bug report highlighted PKCS#11 URI escaping issues in recent Git versions, though no analysis or fix has yet been proposed. The incomplete reftable patch submission awaits proper content for evaluation.