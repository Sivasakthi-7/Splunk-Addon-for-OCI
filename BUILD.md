# Vendored binary provenance

This add-on bundles pre-compiled Python extension modules under `bin/` so it
runs without requiring `pip install` on the Splunk instance. These binaries
are platform- and CPython-ABI-specific and must be re-fetched (not
hand-edited) whenever a dependency version changes.

## Why multiple binaries per package

- `cffi` ships a version-specific extension (`_cffi_backend.cpXY-*`), so a
  separate build is required per targeted CPython minor version.
- `cryptography`'s Rust extension is built against the stable ABI
  (`abi3`), so **one** binary per OS covers all supported Python 3
  versions — no per-minor-version build needed.

## Currently vendored targets

| Package        | Version | Python tags        | Platforms                    |
|-----------------|---------|---------------------|-------------------------------|
| cffi           | 1.17.1  | cp37, cp39, cp313   | win_amd64, manylinux x86_64  |
| cryptography   | 48.0.1  | abi3 (py3.8+)       | win_amd64, manylinux x86_64  |
| pyOpenSSL      | 26.0.0  | pure Python         | n/a                           |
| certifi        | 2024.7.4| pure Python         | n/a                           |

## Reproducing a binary

Use `pip download` with `--only-binary=:all:` against the target platform/ABI
— do not build locally, since the wheel must match the *target* OS/arch, not
the machine running the download:

```sh
# Example: cffi 1.17.1 for CPython 3.13 on Windows x86_64
pip download cffi==1.17.1 \
  --only-binary=:all: \
  --python-version 313 \
  --implementation cp \
  --abi cp313 \
  --platform win_amd64 \
  -d ./download

# Example: cffi 1.17.1 for CPython 3.13 on Linux x86_64 (manylinux)
pip download cffi==1.17.1 \
  --only-binary=:all: \
  --python-version 313 \
  --implementation cp \
  --abi cp313 \
  --platform manylinux2014_x86_64 \
  -d ./download
```

Unzip the resulting wheel and copy the compiled extension (`.pyd` / `.so`)
and the module's pure-Python sources into `bin/`, replacing the matching
`*.dist-info` directory. Repeat per Python tag needed (cp37 / cp39 / cp313
today) and per platform (win_amd64 / manylinux x86_64).

For `cryptography`, since the extension is `abi3`, only one Windows and one
Linux wheel is needed regardless of how many Python 3 versions are
supported — download with `--abi abi3` instead of a `cpXY` tag.

## When bumping a dependency version

1. Update this table.
2. Re-run the `pip download` commands above for every (package, Python tag,
   platform) combination this add-on claims to support.
3. Confirm no CPython tag in `bin/` is left over from a Splunk version this
   add-on no longer supports (dead weight in the package).
4. Verify each new binary imports cleanly under its target interpreter
   before committing.
