# Vendored binary provenance

This add-on bundles a pre-compiled Python extension module under `bin/lib/`
so it runs without requiring `pip install` on the Splunk instance. This
binary is platform-specific and must be re-fetched (not hand-edited)
whenever the `cryptography` version changes.

Third-party packages live under `bin/lib/` rather than directly in `bin/`.
Splunk adds every app's `bin/` to `sys.path` for any script it runs, across
apps, in shared/persistent Python worker processes — so generically-named
top-level packages (`OpenSSL`, `cryptography`, `six`, `certifi`, ...)
sitting straight in `bin/` can shadow, or be shadowed by, identically-named
packages vendored by other Splunk apps in the same worker. `oci_logging.py`
explicitly prepends `bin/lib` onto `sys.path` before importing any of these
(see the `sys.path.insert` near the top of the file, which must stay before
the `oci` / `multiprocess` / `certifi` imports), so only this add-on's own
script ever resolves these names — `bin/lib` is never added implicitly.

## Why only one binary

`cryptography`'s Rust extension is built against the stable ABI (`abi3`),
so **one** binary per OS covers all supported Python 3 versions (3.7-3.13)
— no per-minor-version build needed. `pyOpenSSL` and `certifi` are pure
Python and need no compiled extension at all.

`cffi` and `pycparser` are deliberately **not** vendored, even though older
versions of `cryptography` used to need them: `cryptography` 48.x is fully
Rust/abi3-based and does not import `cffi` anywhere, and nothing else in
this add-on's dependency chain (`pyOpenSSL`, the `oci` SDK, `multiprocess`,
`dill`, `circuitbreaker`, `six`, `pytz`, `dateutil`) imports it either.
Vendoring them was confirmed dead weight (verified by removing them and
re-running the modular input's `--scheme` discovery and functional
crypto/SSL smoke tests under the real Splunk-bundled Python) and was
carrying a real cost: per-Python-tag binaries to re-fetch on every bump,
and a `pycparser` copy that had been silently broken (missing
`c_lexer.py`) for several releases without anything noticing, since
nothing on the runtime path ever imported it.

## Currently vendored targets

| Package        | Version | Python tags        | Platforms                    |
|-----------------|---------|---------------------|-------------------------------|
| cryptography   | 48.0.1  | abi3 (py3.7+)       | win_amd64, manylinux x86_64  |
| pyOpenSSL      | 26.0.0  | pure Python         | n/a                           |
| certifi        | 2024.7.4| pure Python         | n/a                           |

## Reproducing the binary

Use `pip download` with `--only-binary=:all:` against the target platform
— do not build locally, since the wheel must match the *target* OS/arch,
not the machine running the download:

```sh
# cryptography 48.0.1, abi3, Windows x86_64
pip download cryptography==48.0.1 \
  --only-binary=:all: \
  --implementation cp \
  --abi abi3 \
  --platform win_amd64 \
  -d ./download

# cryptography 48.0.1, abi3, Linux x86_64 (manylinux)
pip download cryptography==48.0.1 \
  --only-binary=:all: \
  --implementation cp \
  --abi abi3 \
  --platform manylinux2014_x86_64 \
  -d ./download
```

Unzip the resulting wheel and copy the compiled extension (`.pyd` / `.so`)
and the module's pure-Python sources into `bin/lib/`, replacing the
matching `*.dist-info` directory. Only one Windows and one Linux wheel are
needed regardless of how many Python 3 versions are supported, since the
extension is `abi3`.

## When bumping a dependency version

1. Update this table.
2. Re-run the `pip download` command above for each target platform.
3. Verify the new binary imports cleanly under the target interpreter(s)
   before committing.
4. If a future dependency bump reintroduces a `cffi` requirement, vendor it
   deliberately and re-verify it's actually reachable from the runtime
   import path (see the dead-weight note above) rather than copying it in
   by default.
