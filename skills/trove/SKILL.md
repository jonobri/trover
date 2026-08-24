---
name: trove
description: >-
  Search and retrieve from Trove, the National Library of Australia's digitised
  newspaper archive, without an API key. Covers the shuck-based search workaround
  (Trove is a JavaScript app that returns an empty shell to plain fetches), the
  plain-text article renditions, and the URL parameters that silently lie. Use
  whenever a task needs historical Australian newspapers — dating a word or
  phrase, finding contemporary coverage of an event before ~1955, tracing when a
  term changed meaning, or checking whether a claim about Australian history has
  primary-source support.
metadata:
  version: 1.0.0
---

# Trove — searching the NLA newspaper archive without an API key

Trove holds the National Library of Australia's digitised newspapers. Coverage is strong for colonial and provincial papers and runs to the mid-1950s for most titles, with some going later — *The Canberra Times* to 1995, *Tharunka* to 2010, various regional papers into the 2000s.

**What Trove cannot do:** contemporary media. The modern metropolitan mastheads — the ABC, Guardian Australia, the *Sydney Morning Herald* after 1954, and *The Age* at all — are not in it. For anything post-1955, Trove is the wrong tool; say so rather than searching and reporting an empty result as if it meant something.

## Searching

Trove is a JavaScript application. `curl`, `WebFetch` and `chrome --dump-dom` all return a shell with no results in it. Use `shuck`:

```sh
shuck "https://trove.nla.gov.au/search/category/newspapers?keyword=%22your%20phrase%22&l-artType=newspapers" \
  --wait 25 --dom out.html
```

`--wait 25` is the floor. A shorter wait renders the page chrome while the result list is still loading, which is indistinguishable from "no results".

Add `--all` to get the result links with their article IDs and correct mastheads:

```sh
shuck "<search url>" --wait 25 --all
```

**Prefer `--all` over parsing the rendered text.** Result entries in the text run together, so a regex pairing headlines with mastheads will silently pair each headline with the *next* entry's paper. `--all` gives one line per result with the URL attached.

## ⚠️ URL parameters that silently lie

These were established by testing, and each produced a page that looked like an answer and was not:

| Parameter | Behaviour |
|---|---|
| `l-artType=newspapers` | **Works.** Keep it on every query. |
| `l-decade=NNN` (e.g. `198`) | **Works.** The only reliable date filter. |
| `sortby=dateasc` / `datedesc` | **Returns "No results"** for a query that returns hundreds without it. Unusable. |
| `l-year=YYYY` | **Silently ignored** — returns the unfiltered total, so the count looks plausible. |
| `l-decade` + `l-year` together | **Unreliable** — returned "No results" for a year with a known article in it. |
| `n=100` (page size) | **Breaks the page.** No results parse at all. |

**Consequences.** You cannot sort. You cannot filter to a single year. You cannot enlarge the page. So finding the *earliest* instance means pulling a decade and reading the result page, and you will only see the first ~20 of however many there are. Say "earliest located in the first page of results for decade X", never "earliest".

**Verify counts by content, not by the number.** Two queries that both report the same total are probably the same unfiltered result set. Compare the article *years* on the rendered pages to confirm a filter actually applied:

```sh
# extract the years actually present in a rendered result page
grep -oE '\b(18|19|20)[0-9]{2}\b - Page' out.html | sort -u
```

## Retrieving article text

Once you have an article ID, the plain-text OCR rendition needs no key and no browser:

```sh
curl -sS "https://trove.nla.gov.au/newspaper/rendition/nla.news-articleNNNNNNN.txt"
```

Returns 200. Two endpoints that do **not** work and should not be retried: `api.trove.nla.gov.au/v2/...` returns 410 (retired), and `ndp/del/printArticleJpg/...` returns 404.

Article IDs appear in the raw DOM as `/newspaper/article/NNNNNNN` even when `shuck`'s link extraction reports zero links:

```sh
grep -oE "/newspaper/article/[0-9]+" out.html | sort -u
```

## OCR quality

Nineteenth-century OCR is decent on body text and unreliable on names, numbers and anything in small type. Observed in practice: `ms camp` for `his camp`, `Tim Chief Secretary` for `The`. Before quoting:

- Read the passage in context, not as a grep hit.
- Mark obvious slips with a bracketed correction, e.g. `taken all the tall poppies into ms [his] camp`.
- For any **figure or date**, check the page image rather than the text layer. OCR turns digits into other digits silently, and a plausible wrong number reads exactly like a right one.

## Before claiming an antedating

Trove makes it easy to find an early instance and easy to overstate what it means.

- **"Earliest located" is not "earliest existing."** Trove's coverage is uneven and its OCR misses words; the phrase may sit in an undigitised paper or an unrecognised scan.
- **Check the existing scholarship first.** Dictionary centres and historical linguists have usually been here already, and their citation files are larger than what they print. Finding a phrase in Trove and announcing it as a discovery, when a published paper already has an earlier instance, is the standard failure mode.
- **Distinguish the phrase from the sense.** An early instance of the *words* is not an early instance of the *meaning*. Track them separately and date them separately.

## Provenance

Trove search-result pages are generated, so store the **rendered DOM** (`--dom out.html`) alongside any count you cite, and store article text via the `.txt` rendition. Both get manifest rows like any other asset. Cite articles by masthead, full date, page and Trove article ID.
