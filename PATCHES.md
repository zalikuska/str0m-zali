# Zali patches to str0m 0.25.0

Fork of [str0m](https://github.com/algesten/str0m) 0.25.0 used by the Zali Messenger SFU.
Only `src/`, `Cargo.toml` (without `[[example]]`/`[[bench]]`/`[[test]]` tables, whose files are
not copied), `README.md`, `LICENSE-MIT.txt` and the unit-test fixture `docs/chrome-sdp.json`
are included. License: MIT OR Apache-2.0, same as upstream. Every change is marked `ZALI PATCH`.

The crate's own tests pass with the patch (873 lib tests,
`cargo test --lib --no-default-features --features openssl,vendored`); one test hardcoded the
old buffer size and now derives it; one pacer test queued more than the shorter queue limit
allows and now queues less.

| Constant | File | Upstream | Here | Why |
|---|---|---|---|---|
| `MAX_MISORDER` | `src/streams/register_nack.rs` | 100 | 1000 | NACK window in packets. A 1080p screen layer is 1000–1500 packets/s and a keyframe is hundreds of packets at once, so a lost packet left the window before its retransmission arrived and the retransmission was dropped. Every such gap meant a freeze plus a new keyframe, which broke again. 1000 = libwebrtc `kMaxNackPackets`. |
| `NACK_MIN_INTERVAL` | `src/session.rs` | 33 ms | 10 ms | How often NACKs may be sent. On a LAN, 33 ms before the first request is ~50 packets of a 1080p layer. |
| `DEFAULT_QUEUE_LIMIT` | `src/pacer/leaky.rs` | 2 s | 250 ms | How long the average packet may wait in the pacer before it is forced out. The pacer paces at 1.1 × the delay-based estimate; when a Wi-Fi hiccup dips the estimate, an SFU keeps forwarding the same layer (it can only switch layers), so the layer queued for up to 2 s and then burst out: seconds of lag, then a fast-forward. |

Found in a live screen share over Wi-Fi: the 1080p layer had 1–8 unrecoverable gaps per
10 s, while 960×540 and 480×270 over the same link had none.

## Upgrading

Check whether these are configurable upstream by then. If not, copy the new release's `src/`,
`Cargo.toml`, `README.md` and `LICENSE-MIT.txt` here, drop the three table kinds from
`Cargo.toml`, and re-apply all three changes.

## Using

```toml
[patch.crates-io]
str0m = { git = "https://github.com/zalikuska/str0m-zali", rev = "<commit>" }
```
