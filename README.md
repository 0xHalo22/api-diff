# api-diff

vendor API documentation changes, tracked in git.

most API vendors don't publish good changelogs. they silently edit their
docs, and the first time you know something changed is when your production
code breaks at 2am. this repo is a modest attempt at fixing that: a daily
scraper that fetches a list of public documentation pages, normalizes them
to markdown, and commits them to git. the **git history IS the changelog**.

> want to know what changed in the Stripe `charges` API in the last 90 days?
> ```bash
> git log docs/stripe/charges.md
> git diff HEAD~30 -- docs/stripe/charges.md
> ```

## what it tracks

see [`sources.toml`](./sources.toml) for the full list. the current mix is
deliberately diverse to test the scraper:

- Python PEPs (stable reference material, good for verifying the scraper)
- RFCs (same)
- MDN Web Docs
- Stripe, GitHub REST, Anthropic API references

adding a new source is a 4-line diff to `sources.toml`.

## browse

every tracked doc lives under [`docs/`](./docs), mirroring the path specified
in `sources.toml`. for example:

- [`docs/python/pep-0008.md`](./docs/python/pep-0008.md)
- [`docs/stripe/charges.md`](./docs/stripe/charges.md)

each file has a header comment with the source URL and last-fetched timestamp.

## how it works

a [github actions workflow](.github/workflows/scrape.yml) runs once a day:

1. read `sources.toml`
2. for each source: fetch the URL, extract main content with a CSS selector,
   convert HTML to markdown, normalize whitespace
3. compare against the previously-committed version (ignoring the timestamp)
4. commit only the files that actually changed

pages that fail to extract (often due to heavy client-side rendering) emit a
warning but don't block the run — they just get skipped that day.

## limitations

- **no JavaScript rendering.** pages that require a headless browser to see
  content won't work yet. we detect this (content too short) and warn.
- **HTML structure changes can create false-positive diffs.** if a vendor
  redesigns their docs site, every line will look "changed" even if the
  content is identical. the normalization pipeline tries to minimize this
  but it's not perfect.
- **markdown conversion is lossy.** complex tables, interactive widgets,
  and syntax-highlighted code blocks may not round-trip perfectly.

PRs welcome, especially for new sources.

## credits

all source content is copyright of its respective owner. this repo fetches
only public documentation for the purpose of tracking changes, similar in
spirit to the [Wayback Machine](https://web.archive.org) but scoped to a
handful of developer-relevant API references.
