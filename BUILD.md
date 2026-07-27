# Vendored binary provenance

This add-on bundles pre-compiled Python extension modules under `bin/lib/`
so it runs without requiring `pip install` on the Splunk instance. These
binaries are platform- and CPython-ABI-specific and must be re-fetched (not
hand-edited) whenever a dependency version changes.

Third-party packages live under `bin/lib/` rather than directly in `bin/`.
Splunk adds every app's `bin/` to `sys.path` for any script it runs, across
apps, in shared/persistent Python worker processes — so generically-named
top-level packages (`OpenSSL`, `cryptography`, `cffi`, `six`, `certifi`, ...)
sitting straight in `bin/` can shadow, or be shadowed by, identically-named
packages vendored by other Splunk apps in the same worker. `oci_logging.py`
explicitly prepends `bin/lib` onto `sys.path` before importing any of these
(see the `sys.path.insert` near the top of the file, which must stay before
the `oci` / `multiprocess` / `certifi` imports), so only this add-on's own
script ever resolves these names — `bin/lib` is never added implicitly.

## Why multiple binaries per package

- `cffi` ships a version-specific extension (`_cffi_backend.cpXY-*`), so a
  separate build is required per targeted CPython minor version.
- `cryptography`'s Rust extension is built against the stable ABI
  (`abi3`), so **one** binary per OS covers all supported Python 3
  versions — no per-minor-version build needed.

## `cffi` is a real runtime dependency — do not remove it

Despite `cryptography` 48.x being Rust/abi3-based and never importing
`cffi` directly in its own source, this add-on's vendored `cryptography`
binary (`cryptography.hazmat.bindings._rust`, at least the `openssl`/`x509`
submodules) fails to import with `ModuleNotFoundError: No module named
'_cffi_backend'` if `_cffi_backend` isn't importable — confirmed via a
GitHub Actions `windows-latest` run with a clean `actions/setup-python`
interpreter and no ambient `cffi` installed anywhere.

This was previously (incorrectly) believed to be dead weight and removed:
static analysis showed no `import cffi` in `cryptography`'s source, and
manual verification (locally, and in a Splunk Docker container) showed no
failure after removing `bin/lib/cffi` and the `_cffi_backend` binaries.
Both of those environments turned out to already have a system-wide `cffi`
installed in their own Python's site-packages (the local dev machine's
Python, and Splunk's own bundled Python both had it pre-installed for
unrelated reasons), silently masking the real dependency — `_cffi_backend`
lives further down `sys.path` than `bin/lib`, so it was still found even
after our vendored copy was deleted. A genuinely clean interpreter with no
system-wide `cffi` anywhere is what actually surfaces this dependency, and
CI is exactly that: don't trust a "looks unused" conclusion from an
environment you haven't confirmed is clean of the package everywhere on
`sys.path`, not just in `bin/lib`.

`pycparser` is vendored alongside `cffi` because `cffi` imports it (used
for its C-parsing / API-mode support); it is not otherwise used directly by
this add-on.

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
and the module's pure-Python sources into `bin/lib/`, replacing the matching
`*.dist-info` directory. Repeat per Python tag needed (cp37 / cp39 / cp313
today) and per platform (win_amd64 / manylinux x86_64).

For `cryptography`, since the extension is `abi3`, only one Windows and one
Linux wheel is needed regardless of how many Python 3 versions are
supported — download with `--abi abi3` instead of a `cpXY` tag.

## When bumping a dependency version

1. Update this table.
2. Re-run the `pip download` commands above for every (package, Python tag,
   platform) combination this add-on claims to support.
3. Confirm no CPython tag in `bin/lib/` is left over from a Splunk version
   this add-on no longer supports (dead weight in the package).
4. Verify each new binary imports cleanly under its target interpreter
   before committing — ideally via CI on a clean runner, not just a local
   machine or container, both of which can have an ambient copy of a
   "removed" package sitting in their own Python installation and mask a
   real breakage (see the `cffi` note above).
