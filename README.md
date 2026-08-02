# lex-corpus-lu-legilux

**Every version of Luxembourg's consolidated law, as data — with its dates.**
Regulators publish the current rule; this repository keeps every consolidated
state Legilux has published, so *"what applied on 15 March 2022?"* is a file
read, not an archaeology project.

**1,399 works · 4,632 versions · 1849-03-14 → 2030-09-15.**
Honest coverage claim: *dense and reliable from 2017 onward; real but sparse
before; isolated snapshots back to 1849; forward to 2030.* Only ~6% of lois and
~4% of RGD are ever consolidated — the ≈24,579 never-consolidated acts are
**not** in this repository (their date coverage is unmeasured).

**[Per-article dataset](https://github.com/SFHAJJI/lex-articles)** ·
**[Live demo](https://law.soufien.lu)** ·
**[MCP endpoint](https://law.soufien.lu/mcp)** ·
**[Engine](https://github.com/SFHAJJI/lex)**

## Why a git repository?

Because the questions people actually ask about law are *time* questions, and
git answers them natively: `git log` is the legislative history, `git diff` is
"what changed between these two states", and a clone is a complete,
tamper-evident copy nobody can silently edit — every byte is covered by a
sha256 recorded inside the content itself. No API, no account, no database
server: the format outlives any website built on top of it.

## What a work looks like

```
works/rgd-1998-08-03-n4/            ← Nouveau Code de procédure civile
  meta.json
  versions/
    2015-09-01/meta.json            ← valid 2015-09-01 → 2017-05-26
    2017-05-27/meta.json            ← valid 2017-05-27 → 2018-05-22
    2018-05-23/meta.json            ← …and so on: 25 versions
```

Each `meta.json` carries the publisher-asserted validity interval, the
first-sighting timestamp (append-only event chain — transaction time lives
*inside* hashed content, never in mutable commit metadata), a record sha256,
and the link to the official text. **`git log` on this tree reads as a
legislative history without tooling.**

## Full text, from the official channel

Each version carries the **verbatim Akoma Ntoso 3.0 XML** as published by the
Service central de législation at its official manifestation endpoint
(`legilux.public.lu/filestore/…` — the same files the official site serves,
on the robots-permitted host). The publisher licenses these content files
**CC-BY-4.0** (its own documentation: « les fichiers de contenu … licence
CC-BY … utilisations commerciales ou non », plus a machine-readable
`dct:license` triple on every manifestation). Bodies are stored **byte-verbatim
and append-only** — the sha256 in each `meta.json` covers the exact retrieved
file; no text is ever altered or overwritten. Where a body is not yet fetched,
the version says so explicitly (`text.available: false`) and links out.
Languages: 4,501 French expressions plus the publisher's three singleton
de/en/lb expressions — all ingested.

## The six intake answers (spec §1.5)

1. **What does it publish?** Lois, règlements grand-ducaux, codes, recueils,
   the Constitution, and their consolidated versions — as named by the
   publisher. https://legilux.public.lu
2. **Authority of each type?** Binding instruments of Luxembourg law, published
   in the Journal officiel. Asserted by the publisher's own publication acts.
3. **Retrievable mechanically?** Yes: the official SPARQL endpoint
   (https://data.legilux.public.lu/sparqlendpoint), published as an open-data
   resource. robots.txt permits the endpoint; body paths are disallowed and are
   not fetched.
4. **May we republish the text?** **Yes** — the publisher's own documentation
   licenses the content files under CC-BY-4.0 (commercial reuse included), and
   each manifestation carries a machine-readable CC-BY-4.0 licence triple; a
   CC-BY grant covers the sui generis database right as well. Act text
   additionally carries no copyright (loi du 18.04.2001, art. 10, 8°).
5. **May we republish the metadata?** Yes: CC-BY grant on the Legilux open-data
   dataset (data.public.lu, dataset `62c83bfd9794ec8e47b5bc68`), attribution
   below.
6. **Does the publisher retain superseded versions?** Yes — Tier A: validity
   intervals are publisher-supplied (`jolux:dateApplicability`), 100% coverage
   of the versioned corpus.

## Attribution

Data: **Ministère d'État — Service central de législation, Grand-Duché de
Luxembourg** (Legilux open data, CC-BY). Metadata converted from source RDF to
JSON; bodies stored verbatim as retrieved; no text altered. See
[NOTICE](NOTICE) — attribution obligations survive into forks.

## Consume it

- Browse: any `works/<slug>/versions/<date>/meta.json`
- The signed SQLite index (`index-lu-legilux.db`) is published as a release
  asset — filter-first queries, FTS over titles, ECDSA-P256-signed stamp.
- MCP server + web demo: https://github.com/SFHAJJI/lex

## How this repository stays current

One scheduled job runs nightly and drives every publisher in the fleet. There is
no manual step, and deliberately one cron and one credential for all of them.

1. **Ingest** — ask the publisher which versions exist, download any not seen
   before, write them *verbatim*. An existing body file is never reopened for
   writing: the evidence layer is append-only by construction.
2. **Anomaly gate** — if the work count drops by more than 5%, assume a partial
   upstream response, discard everything and **commit nothing**. A bad night
   leaves yesterday's good data in place rather than corrupting history.
3. **Derive** — regenerate the per-article layer in
   [lex-articles](https://github.com/SFHAJJI/lex-articles).
4. **Determinism guard** — if derived output changed while no source file did,
   the extractor is non-deterministic: fail loudly, **commit nothing**.
5. **Index & publish** — rebuild the SQLite index, sign it (ECDSA P-256),
   attach it to a release, regenerate the JSONL/Parquet datasets.
6. **Report** — write a three-state outcome per publisher
   (`ran_committed` / `ran_no_change` / `failed_*`) and open an issue on failure.

Every index carries a signed stamp recording **when it was built and from which
corpus commit**, and every MCP tool response returns it — so freshness is a
property of the data, not a claim on a webpage. Live status:
<https://law.soufien.lu/built>
