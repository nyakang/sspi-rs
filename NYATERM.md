# NyaTerm fork notes

This branch is upstream `sspi` plus CI. It carries **no patch**.

- Fork: <https://github.com/nyakang/sspi-rs>
- Upstream: <https://github.com/Devolutions/sspi-rs>
- Base revision: `f73e31c8f988894a7b11eb03ae887e0fbc8f874b`
  (`chore(release): recover sspi crate publication (#746)`, the `0.22.0` release)
- Branch: `nyaterm`

The branch now follows the published 0.22 line. NyaTerm's IronRDP fork consumes
`sspi = "0.22"` and adapts its CredSSP writer to the fallible
`TsRequest::buffer_len` API introduced on this line.

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
- `chore: publish 0.21.4's contents under version 0.21.0` — it existed only to
  satisfy an exact dependency in the older IronRDP baseline. Both upstream SSPI
  and NyaTerm's IronRDP fork now use the released 0.22 API.

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
`aes-gcm` (0.11.x), `picky` (7.0.0-rc.26) and `sspi` (0.22.0), which is the point
of pointing at this branch at all. `.github/workflows/nyaterm.yml` runs exactly
that check.

Windows and macOS helper builds plus a manual NLA test remain part of the
release matrix.
