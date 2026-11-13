# TorChord v0.2

A **private-access, experimental BitChord/TorBox add-on** built with a Cloudflare Worker. This is a technical preview, **not** a Nebula replacement or public multi-user service. It searches your existing TorBox downloads and can optionally query a separately configured Torznab indexer for files confirmed to be TorBox-cached.

**Status:** No successful live BitChord playback or TorBox account integration has been verified. BitChord request-attestation verification is not yet implemented. There is no uncached acquisition, prefetch, AirLock usage protection, rate limiting or public setup wizard.

## GitHub and Cloudflare deployment

Repository: https://github.com/chreid1973/Torchord

1. In Cloudflare, create an API token with **Workers Scripts: Edit** permission scoped to the relevant account. Find your Cloudflare Account ID in the dashboard. Never share the token in chat or commit it.
2. In GitHub, open **Settings → Secrets and variables → Actions**. Create the repository secrets `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID`.
3. Run **Actions → Deploy TorChord Worker → Run workflow**. The first deployment creates a Worker named `torchord-private`. Future commits to `main` automatically redeploy it.
4. In Cloudflare, go to **Workers & Pages → torchord-private → Settings → Variables and Secrets**. Add **encrypted secrets** `TORBOX_API_KEY` and `ACCESS_TOKEN`. Use a newly generated, random, URL-safe `ACCESS_TOKEN` at least 32 characters long. Make sure these variables are applied to the deployed Worker.
5. Optional: Add encrypted secrets `TORZNAB_URL` (an HTTPS Torznab API endpoint) and `TORZNAB_API_KEY`. Without these, only existing TorBox library files are searchable.
6. Use the **actual workers.dev URL displayed in Cloudflare**. Test `https://YOUR-WORKER.workers.dev/health`, then open `https://YOUR-WORKER.workers.dev/p/YOUR_ACCESS_TOKEN/manifest.json`. The BitChord add-on URL is that same private manifest link.

The repository is currently **public**, so keep all secrets and personal URLs out of source code, GitHub issues, commit messages and Actions logs. The `ACCESS_TOKEN` becomes part of your installation URL; treat that URL like a password. Anyone who learns it can query this prototype and may cause TorBox library actions.

**Important:** First pushes may trigger a failed GitHub Actions run until the Cloudflare deployment secrets are configured. Run the workflow again after adding the secrets. Do not publish your TorBox API key as a GitHub Actions secret; it belongs in Cloudflare Worker encrypted secrets.

## Local checks

Requires Node.js 22 or newer.

```sh
npm install
npm run check
```

The Worker can be deployed with `npm run deploy` after authenticating Wrangler, but the normal setup uses GitHub Actions.

## Smoke tests

```sh
curl -i 'https://YOUR-WORKER.workers.dev/health'
curl -i 'https://YOUR-WORKER.workers.dev/p/YOUR_ACCESS_TOKEN/manifest.json'
curl -i 'https://YOUR-WORKER.workers.dev/p/YOUR_ACCESS_TOKEN/search?q=example%20artist'
```

A successful manifest response only confirms routing; it does **not** confirm playback. Test with audio you are authorized to access. The Worker does not deliberately request uncached torrents. A cached torrent might still be added to your TorBox library when resolving a result, which may consume account resources.

## Limitations and troubleshooting

A `503 Configure Worker secrets first` response means `TORBOX_API_KEY` or `ACCESS_TOKEN` is missing. A 404 on the manifest usually means the token or path is wrong. Search results require matching library files or an appropriately configured indexer. TorBox endpoints and BitChord result formats have not been validated end to end, and BitChord may reject or mark the add-on outdated until request verification is implemented.

Keep audio discovery and retrieval limited to content you have rights to access.
