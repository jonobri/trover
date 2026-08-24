# trover 📰

**A Claude Code skill for searching Trove, the National Library of Australia's digitised newspaper archive — without an API key.**

*Trover*: the old common-law action for recovering property that ended up in someone else's hands. Which is roughly the job.

[Trove](https://trove.nla.gov.au) holds millions of digitised Australian newspaper pages, and it is the best primary source there is for dating an Australian word, checking a claim about colonial politics, or reading what people actually said about an event at the time.

It is also a JavaScript application with a search interface that quietly lies to you. `curl` gets an empty shell. Three of its URL parameters silently return the wrong result set. The v2 API is retired. This skill is what is left after working all of that out.

## What it covers

- **Searching without an API key** — Trove returns an empty shell to plain fetches, so searches go through [`shuck`](https://github.com/jonobri/shuck) with a minimum `--wait 25`.
- **⚠️ The parameters that lie.** `sortby` returns "No results" for a query that otherwise returns hundreds. `l-year` is silently ignored and hands back the *unfiltered* total, so the count looks plausible. `l-decade` combined with `l-year` returned "No results" for a year with a known article in it. `n=100` breaks the page entirely. Only `l-artType` and `l-decade` are reliable.
- **Article text** — the plain-text OCR rendition at `/newspaper/rendition/nla.news-articleNNNNNNN.txt` returns 200 with no key. The `api.trove.nla.gov.au/v2/` endpoint is retired (410).
- **Getting article IDs** even when link extraction reports zero links.
- **OCR discipline** — what nineteenth-century OCR reliably gets wrong, and why figures must be checked against the page image rather than the text layer.
- **Coverage limits** — Trove is a *historical* corpus. The modern mastheads are not in it, and *The Age* is not in it at all. For anything after about 1955 this is the wrong tool.
- **The antedating trap** — how to avoid announcing a "discovery" that a published paper already made, earlier.

## What it can't do

Contemporary media. Trove's newspaper digitisation runs to the mid-1950s for most titles, with a handful going later (*The Canberra Times* to 1995). If you need ABC, Guardian Australia or a metropolitan daily from the last thirty years, you need a media database, not Trove.

## Install

```sh
git clone https://github.com/jonobri/trover.git ~/Code/trover
ln -s ~/Code/trover/skills/trover ~/.claude/skills/trover
```

Requires [`shuck`](https://github.com/jonobri/shuck) on your `PATH` for the search step. Article retrieval needs only `curl`.

## Why a skill and not a tool

There is nothing to install beyond `shuck` and `curl` — the difficulty is entirely in knowing which URL parameters can be trusted and which quietly hand you the wrong answer. That is knowledge, not code, so it ships as a skill.

## Licence

MIT. The archive itself belongs to the National Library of Australia; check Trove's [terms of use](https://trove.nla.gov.au/about/create-something/using-api) before redistributing anything you retrieve.
