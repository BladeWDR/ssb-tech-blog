+++
title = 'Meilisearch Dumpless Upgrades'
date = 2026-04-03T15:54:33-04:00
draft = false
+++

Short one today.

I learned that [Meilisearch](https://www.meilisearch.com) has added [experimental support for dumpless upgrades](https://meilisearch.notion.site/Dumpless-upgrade-fff4b06b651f81f1acafe24d4687b3f7).

So far it works well, though to be fair none of my Meilisearch databases are particularly important.

I have one for better searches in [Jellyfin](https://jellyfin.org) and another for my [Karakeep](https://karakeep.app) search index.

Losing either of them would be a minor inconvenience, whereas it's been a _massive_ inconvenience for me to have to dump and manually import every single time I patch Meilisearch.

You can either pass meilisearch the `--experimental-dumpless-upgrade` flag on the CLI, or set the `MEILI_EXPERIMENTAL_DUMPLESS_UPGRADE` environment variable.

In Docker you'd set it like this:

```yaml
environment:
  - MEILI_EXPERIMENTAL_DUMPLESS_UPGRADE=true
```
