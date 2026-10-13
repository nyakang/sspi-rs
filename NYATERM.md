# NyaTerm fork notes

This branch follows upstream SSPI 0.23.0 with a consumer dependency
compatibility patch and fork CI.

- Fork: <https://github.com/nyakang/sspi-rs>
- Upstream: <https://github.com/Devolutions/sspi-rs>
- Base revision: `d87a57a7042cbefd1f794a9c3657b33d3cee8037` (upstream `master`, 2026-10-13)
- Branch: `nyaterm`

NyaTerm's IronRDP fork consumes `sspi = "0.23"` and retains its fallible
`TsRequest::buffer_len` handling. Details of previous releases below are
historical; the 2026-10-13 section records current validation.

## Patches

Require non-yanked picky-krb 0.13 and the ASN.1 minimum API versions needed
when resolving an existing consumer lockfile. Preserve upstream GSS validation.

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

## 2026-10-01 upstream merge

Merged upstream 0.22.1 without conflicts; this fork still carries no library
patch. The update includes Kerberos clock-skew/DER fixes and NTLM sealing-state
preservation. Windows validation: `cargo check -p sspi` and
`cargo test -p sspi --lib` (282 passed). The external consumer with a stable
`aes-gcm = "0.11"` passed `cargo check`; its lockfile contains exactly one
`aes-gcm` (0.11.1), `picky` (7.0.0-rc.26), and `sspi` (0.22.1). Upstream emits
two existing unreachable-public-item warnings in `src/secret.rs`.

## 2026-10-13 upstream rebase

Upstream `master` is fixed at `d87a57a7042cbefd1f794a9c3657b33d3cee8037`.
Previous published NyaTerm head: `37afc5f400193a219fdd1cf4c9d83f1fedac650c`.
The previous branch is retained locally as `backup/nyaterm-20261013`.

Replay commits before this documentation update:

- `c99270bc58c3255674bc68e0e7cb248915f56cca` ci: resolve sspi as an external dependency on the nyaterm branch
- `5c3fe5418838a228e36d2336de48adbd1cf2881f` docs: record the NyaTerm fork baseline and patch set
- `753d698bd41a0c5846d2a2f2b5d83dd34ce7913a` docs: update the NyaTerm fork baseline
- `e3856089c937a10858bb3feaa180893ab42e5a41` docs(fork): record SSPI baseline and validation
- `e20f58f883af70a5ea10768df1cb375e84c3d1dc` build(deps): require non-yanked GSS-compatible crypto APIs

SSPI is now 0.23.0. The previous crypto-release/SSPI compatibility decisions
remain in force. One new dependency patch requires the non-yanked picky-krb
0.13 release and minimum picky-asn1-der 0.5.7 / picky-asn1-x509 0.15.5 APIs.
Upstream references InvalidMechanismOid, which 0.12.4 lacks; 0.12.5 contains
it but is yanked. The 0.13 minimum dependencies otherwise allow versions missing
Length and iakerb5 when an existing application's lock is retained. This patch
keeps the upstream fail-closed GSS validation and updates requirements instead.

Windows: SSPI library tests (283), workspace fmt, the external consumer alongside
stable aes-gcm 0.11, and IronRDP client/connector/acceptor/VMConnect checks passed.
An old-lock NyaTerm workspace check also passed after targeted ASN.1 updates.
The consumer resolves one each of aes-gcm 0.11.1, picky rc.26 and SSPI 0.23.0.


Validation is on Windows; native macOS/Linux checks are delegated to the
branch workflow. Interactive GUI/file-drop and real-server authentication
checks were not performed by this update. Their outcomes must not be inferred
from scripted peers or compile checks.
