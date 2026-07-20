# aval-core

SDK-less Solana substrate for WebAssembly (wasm32-wasip2) agent-tool plugins.
This is the canonical source for the core that powers the **Aval** suite of
ZeroClaw plugins (`nonce-vault-init`, `durable-tx-build`, `approval-recheck`
— see [zeroclaw-plugins PR #98](https://github.com/zeroclaw-labs/zeroclaw-plugins/pull/98)).

The plugin registry validates each plugin as a fully self-contained crate, so
the suite vendors these modules verbatim into each plugin's `src/core/`.
Fix here first, then re-vendor; the vendored copies carry a header saying
exactly that.

## What it provides

- `codec` — hand-rolled base58 and base64 (no `bs58`, no `base64` crate)
- `pubkey` — 32-byte keys, well-known program ids, `createAccountWithSeed`
  derivation, PDA search with an ed25519 off-curve check, ATA derivation
- `instruction` — byte-exact encoders for the system-program instructions a
  durable-nonce payment rail needs, SPL `TransferChecked`, ATA
  `CreateIdempotent`, and memo
- `message` — compact-u16, legacy message compilation (dedupe + runtime
  account ordering), unsigned-transaction serialization, and the reverse
  parser used for approval-time verification
- `nonce` — the 80-byte durable nonce account layout
- `amount` — decimal-string ⇄ base-unit conversion in integer arithmetic
  (floating point never touches a money path)
- `rpc` — a five-method Solana JSON-RPC client behind an injectable
  `HttpPost` trait: host tests inject fixtures, wasm shims inject `waki`

## Why not solana-sdk

It does not compile for `wasm32-wasip2` inside a WIT component without a
fight, and this rail needs a page of byte layouts, not a runtime. The only
non-trivial dependencies are `sha2` and `curve25519-dalek`
(`default-features = false`) because PDA derivation genuinely requires an
off-curve check. Every layout is pinned by byte-exact tests so drift is loud.

`overflow-checks = true` in release: money code aborts on arithmetic
overflow rather than wrapping.

## License

MIT — see [LICENSE](LICENSE).
