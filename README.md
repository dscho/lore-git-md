# The Git mailing list, distilled.

The [git@vger.kernel.org](https://lore.kernel.org/git/) mailing list is where
Git is built: patches, reviews, design debates, bug reports, flame wars about
whitespace. It is also a firehose. Keeping up with it is essentially a
full-time job.

This repository is the **digestif** -- a strong drink to help you digest a
heavy meal of emails. Think of it as a sommelier for your inbox, except the
sommelier is a language model and the wine is patch review.

**Browse the digests at [git-digestif.github.io](https://git-digestif.github.io/).**

Every message from the [lore.kernel.org/git](https://lore.kernel.org/git/)
archive is converted to clean Markdown, organized by date, threaded via
symlinks, and augmented with AI-generated per-email summaries and daily,
weekly, and monthly digests. A [scheduled
workflow](.github/workflows/daily-digest.yml) runs every three hours to keep
things fresh.

The toolkit that produces all of this lives at
[git-digestif/lore-git-md-helper](https://github.com/git-digestif/lore-git-md-helper).

## License

The email content is from the public
[lore.kernel.org](https://lore.kernel.org/git/) archive. The tooling and
AI-generated summaries are [MIT OR
Apache-2.0](https://github.com/git-digestif/lore-git-md-helper).
