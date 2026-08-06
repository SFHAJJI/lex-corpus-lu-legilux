# lex-corpus-lu-legilux

[![License](https://img.shields.io/badge/license-CC--BY--4.0-blue?style=flat-square)](LICENSE)
[![Source](https://img.shields.io/badge/source-Legilux-e0705f?style=flat-square)](https://legilux.public.lu)
[![Corpus](https://img.shields.io/badge/corpus-live%20coverage-brightgreen?style=flat-square)](https://law.soufien.lu/coverage)
[![Evidence](https://img.shields.io/badge/evidence-verbatim%20%C2%B7%20append--only%20%C2%B7%20sha256-24292f?style=flat-square)](https://law.soufien.lu/verify)
[![Live](https://img.shields.io/badge/live-law.soufien.lu-6f42c1?style=flat-square)](https://law.soufien.lu)

**Every version of Luxembourg's consolidated law, as data, with its dates.**
Regulators publish the current rule; this repository keeps every consolidated
state Legilux has published, so *"what applied on 15 March 2022?"* is a file
read, not an archaeology project.

The generated [manifest](manifest.json) and the
[live coverage page](https://law.soufien.lu/coverage) are the source of truth for
counts and date ranges.
Honest coverage claim: *dense and reliable from 2017 onward; real but sparse
before; isolated historical and future-dated snapshots exist.* Acts that Legilux
never places in its consolidated collection are **not** in this repository; that
gap is stated explicitly rather than folded into a completeness claim. The
[measured catalogue boundary and proposed normative-act increment](https://github.com/SFHAJJI/lex/blob/main/docs/luxembourg-scope.md)
explain why expanding beyond consolidations requires document-class semantics,
not a blind import of every resource labelled `Act`.

**[Per-article dataset](https://github.com/SFHAJJI/lex-articles)** ·
**[Live demo](https://law.soufien.lu)** ·
**[MCP endpoint](https://law.soufien.lu/mcp)** ·
**[Engine](https://github.com/SFHAJJI/lex)**

## Why a git repository?

Because a clone is a complete, tamper-evident copy that nobody can silently
edit: every byte is covered by a sha256 recorded inside the content itself. No
API, no account, no database server, and the format outlives any website built
on top of it.

What git is **not** here is the timeline. A commit records when this corpus
observed a version, never when the law applied, and publishers routinely
backfill older consolidations and issue texts dated years ahead. So both time
axes live inside the hashed content, in each version's `meta.json`, and never
in mutable commit metadata. The tree is laid out by work and by validity date,
so it reads as a history without tooling, while `git log` reads as an ingest
log, which is a different and much less interesting thing.

The full argument, and what that choice cost, is written up at
[law.soufien.lu/decisions](https://law.soufien.lu/decisions).

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
first-sighting timestamp (append-only event chain, transaction time lives
*inside* hashed content, never in mutable commit metadata), a record sha256,
and the link to the official text. **The tree reads as a legislative history
without tooling; `git log` reads as the ingest log.**

## Full text, from the official channel

Each version carries the **verbatim Akoma Ntoso 3.0 XML** as published by the
Service central de législation at its official manifestation endpoint
(`legilux.public.lu/filestore/…`, the same files the official site serves,
on the robots-permitted host). The publisher licenses these content files
**CC-BY-4.0** (its own documentation: « les fichiers de contenu … licence
CC-BY … utilisations commerciales ou non », plus a machine-readable
`dct:license` triple on every manifestation). Bodies are stored **byte-verbatim
and append-only**, the sha256 in each `meta.json` covers the exact retrieved
file; no text is ever altered or overwritten. Where a body is not yet fetched,
the version says so explicitly (`text.available: false`) and links out.
Every language expression in the consolidated collection is retained.

## The six intake answers (spec §1.5)

1. **What does it publish?** Lois, règlements grand-ducaux, codes, recueils,
   the Constitution, and their consolidated versions, as named by the
   publisher. https://legilux.public.lu
2. **Authority of each type?** Binding instruments of Luxembourg law, published
   in the Journal officiel. Asserted by the publisher's own publication acts.
3. **Retrievable mechanically?** Yes: the official SPARQL endpoint
   (https://data.legilux.public.lu/sparqlendpoint), published as an open-data
   resource. robots.txt permits the endpoint; body paths are disallowed and are
   not fetched.
4. **May we republish the text?** **Yes**, the publisher's own documentation
   licenses the content files under CC-BY-4.0 (commercial reuse included), and
   each manifestation carries a machine-readable CC-BY-4.0 licence triple; a
   CC-BY grant covers the sui generis database right as well. Act text
   additionally carries no copyright (loi du 18.04.2001, art. 10, 8°).
5. **May we republish the metadata?** Yes: CC-BY grant on the Legilux open-data
   dataset (data.public.lu, dataset `62c83bfd9794ec8e47b5bc68`), attribution
   below.
6. **Does the publisher retain superseded versions?** Yes, Tier A: validity
   intervals are publisher-supplied (`jolux:dateApplicability`), 100% coverage
   of the versioned corpus.

## Attribution

Data: **Ministère d'État, Service central de législation, Grand-Duché de
Luxembourg** (Legilux open data, CC-BY). Metadata converted from source RDF to
JSON; bodies stored verbatim as retrieved; no text altered. See
[NOTICE](NOTICE), attribution obligations survive into forks.

## Consume it

- Browse: any `works/<slug>/versions/<date>/meta.json`
- The lex-index/3 release contract uses a canonical whole-artifact manifest.
  It binds every released file by hash and size and is signed with a
  non-exportable Azure Key Vault P-256 key. The embedded index stamp remains
  public provenance, not the runtime trust root. The live rollout state is
  reported at [law.soufien.lu/verify](https://law.soufien.lu/verify).
- MCP server + web demo: https://github.com/SFHAJJI/lex

## How this repository stays current

One scheduled job runs nightly and drives every publisher in the fleet. There is
no manual publication step. GitHub Actions uses OIDC and short-lived Azure
authorization; the signing private key never leaves Key Vault.

1. **Ingest**, ask the publisher which versions exist, download any not seen
   before, write them *verbatim*. An existing body file is never reopened for
   writing: the evidence layer is append-only by construction.
2. **Anomaly gate**, if the work count drops by more than 5%, assume a partial
   upstream response, discard everything and **commit nothing**. A bad night
   leaves yesterday's good data in place rather than corrupting history.
3. **Derive**, regenerate the per-article layer in
   [lex-articles](https://github.com/SFHAJJI/lex-articles).
4. **Determinism guard**, if derived output changed while no source file did,
   the extractor is non-deterministic: fail loudly, **commit nothing**.
5. **Index & sign**, build lex-index/3 and the benchmark evidence, sign the
   canonical artifact manifests, and verify every file before release.
6. **Deploy safely**, build an immutable image, start a zero-traffic Container
   App revision, exercise health, MCP, Luxembourg and EU search, then promote it.
7. **Report**, write a three-state outcome per publisher
   (`ran_committed` / `ran_no_change` / `failed_*`) and open an issue on failure.

Every index carries a provenance stamp recording **when it was built and from
which corpus commit**, and every MCP tool response returns it. Runtime trust is
reported separately from the pinned whole-artifact manifest. Live status:
<https://law.soufien.lu/built>

## Support

This is free and open, and it stays that way whatever you decide. It is also not free to run:
the live site, the nightly jobs and the storage sit on Azure infrastructure I pay for out of
pocket, and I maintain it on my own time.

If it saved you an afternoon, you can [buy me a coffee ☕](https://buymeacoffee.com/shajji)
and put it towards the hosting bill. Starring the repo helps just as much, and costs nothing.
