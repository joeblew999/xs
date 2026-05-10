# Cloudflare

The CF design for the joint http-nu + xs effort lives in
[../http-nu/CLOUDFLARE.md](../http-nu/CLOUDFLARE.md) (joeblew999/http-nu,
`joeblew999` branch).

xs's role on CF is the storage/persistence backend (DO SQLite for the
LSM index, R2 for the CAS) -- a swap of `fjall` + `cacache` while
keeping `xs::store` / `xs::api` / `xs::processor` interfaces unchanged.
That work hasn't started; this file is a pointer so a fresh xs reader
finds the joint doc.

When CF work lands here it should:

- Add a feature flag (e.g. `cloudflare`) optional to default builds.
- Implement the storage backend behind the existing trait surface.
- Land additively under a new module (e.g. `src/cf/`) the way http-nu
  did, so upstream cablehead/xs merges stay clean.

Anything beyond that -- design rationale, candidate-path comparisons,
mise/wrangler tooling notes -- is in
[../http-nu/CLOUDFLARE.md](../http-nu/CLOUDFLARE.md).
