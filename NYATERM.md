# NyaTerm fork notes

This branch is upstream `sspi` plus CI. It carries **no patch**.

- Fork: <https://github.com/nyakang/sspi-rs>
- Upstream: <https://github.com/Devolutions/sspi-rs>
- Base revision: `4172abefd5c7502d92ba4544b77bd71fd182041f`
  (`chore(release): prepare for publishing (#713)`, the `0.21.4` release content)
- Branch: `nyaterm`

It exists because 0.21.4 was never published — crates.io stops at 0.21.3 — while
NyaTerm wants what that commit carries: `847304f build(deps): move RustCrypto
crates to stable and update picky (#712)`, which pins `picky = "=7.0.0-rc.26"` and
removes the prerelease `p256`/`p384`/`p521`/zkcrypto/dalek pins that made this
crate unresolvable next to a released `aes-gcm`, plus
`4878c50 fix(auth_identity): accept `@` in down-level account names` and
`6d17708 fix(kerberos): remove unnecessary sequence number incrementation` on the
NLA path.

## Patches

None.

## Not carried here

Three patches were dropped over two rebases:

- `deps: move the picky pin to 7.0.0-rc.26` and
  `deps: use released dalek crates on Apple targets` — upstream's `847304f` did
  the same work, and went further: it dropped the dalek pins outright rather than
  moving them to released versions. Only `pkcs1 = "=0.8.0-rc.4"` remains, which
  the NyaTerm graph already resolves. The same commit removed `crates/dpapi`'s
  `curve25519-dalek "=5.0.0-rc.1"` pin, so `cargo check -p sspi` works at the
  workspace root again.
- `chore: publish 0.21.4's contents under version 0.21.0` — it existed only
  because `ironrdp-connector` 0.10.0 declares `sspi = "=0.21.0"`, and a Cargo
  `[patch]` replacement has to satisfy the original requirement. NyaTerm's IronRDP
  fork now relaxes that to `sspi = "0.21"`, matching upstream, so the version can
  stay where upstream put it.

## Why the base is the release commit and not upstream's tip

`master` is four commits further along, and one of them cannot be consumed:
`a9dfaec refactor!: enable `as_conversions` lint (#721)` changes
`credssp::TsRequest::buffer_len` from `-> u16` to `-> Result<u16>` across 63
files. It landed *after* `4172abe` set the version to 0.21.4 and did not bump the
version again, so upstream's tip is an unreleased breaking change sitting under a
released version number. `ironrdp-connector`'s `write_credssp_request` still calls
`usize::from(ts_request.buffer_len())`, on NyaTerm's fork *and* on IronRDP's own
`master`, so taking the tip breaks the RDP client. Revisit when upstream releases
that work under a version of its own and IronRDP adapts to it.

## Validation

On Windows 11, with the toolchain `rust-toolchain.toml` pins (1.97.1):

```sh
cargo check -p sspi   # clean
```

and, the way NyaTerm actually consumes it — an external package that
path-depends on this checkout alongside a stable `aes-gcm`:

```toml
[dependencies]
sspi = { path = "path/to/sspi-rs", default-features = false }
aes-gcm = "0.11"
```

`cargo check` on that package succeeds and its lock holds one version each of
`aes-gcm` (0.11.x), `picky` (7.0.0-rc.26) and `sspi` (0.21.4), which is the point
of pointing at this branch at all. `.github/workflows/nyaterm.yml` runs exactly
that check.

Windows and macOS helper builds plus a manual NLA test remain part of the
release matrix.
