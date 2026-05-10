# Cloudflare

Forward-looking notes for extending xs to run on Cloudflare. Nothing here is
implemented yet -- this file exists to capture constraints, questions, and
candidate paths so the work can be picked up once the local tooling, code,
and examples are all in a good state.

## Goal

Let xs run on Cloudflare in addition to its current local-first deployment, so
the same event-sourced primitives can back a service exposed via Workers
without giving up the local-first story.

## Where xs lives today

- Rust binary (`xs serve`) -- tokio-based, embeds nu, talks HTTP/Hyper,
  supports TLS, Unix sockets, and iroh peer-to-peer.
- Storage: `fjall` (embedded LSM) + `cacache` content-addressed blobs on the
  local filesystem.
- Single-process, long-lived; clients stream frames over HTTP or UDS.

The local-first design is non-negotiable. Cloudflare support is additive.

## Candidate Cloudflare paths

1. **Workers + Durable Objects (DO) -- native rewrite**
   - Replace fjall with DO storage (SQLite-backed) for the stream log, R2 for
     blob payloads, KV for indexes if needed.
   - Each "stream" becomes a DO instance; HTTP entry runs in a Worker that
     routes to the right DO by stream id.
   - Pros: scales, cheap, idiomatic CF.
   - Cons: largest re-implementation -- xs internals are tokio + filesystem,
     none of which exists in Workers. Effectively a port, not a deploy.

2. **Hybrid -- CF Workers as edge gateway, xs upstream**
   - Workers terminate TLS/HTTP at the edge, authenticate, and proxy to an
     xs instance running elsewhere (self-hosted).
   - Pros: gets edge + auth benefits without porting xs.
   - Cons: not really "xs on Cloudflare"; doesn't help users who only have
     a CF account.

CF Containers are explicitly out of scope.

## Open questions

- Does the iroh transport survive any of these (Workers can't bind raw UDP)?
  Probably reserved for self-hosted only, with CF deployments restricted to
  HTTP.
- How do we model the embedded nu engine in Workers? It's a heavy dep that
  almost certainly doesn't compile to `wasm32-unknown-unknown` cleanly --
  spike-test before committing to path #1.
- What's the auth story? xs today assumes a trusted local socket; on CF we
  need a real token model. Look at how `joeblew999/auth-service` does it.
- Streaming: Workers support streaming responses, but per-request CPU and
  duration limits matter. SSE/long-poll patterns may need DO sockets.

## Tooling

When work starts:
- Add Cloudflare-specific entries to `mise.toml` `[tools]` (e.g.
  `"npm:wrangler"`, `"cargo:worker-build"`).
- Use `joeblew999/.github` reusable workflows for any CF deploy CI.
- See `joeblew999/plat-trunk` and `joeblew999/authz-core` for working
  patterns (Doppler secrets -> GH Actions -> Wrangler deploy).

## Status

Not started. Revisit after:
1. mise + CI on the `joeblew999` branch are green.
2. Local examples (`docs/`, `examples/`) run cleanly end-to-end.
3. We have a clear answer to the nu-in-Workers question above.
