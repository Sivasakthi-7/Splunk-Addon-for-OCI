# Binary File Declaration
bin/lib/_cffi_backend.cpython-39-x86_64-linux-gnu.so: This binary file is provided along with the cffi module (required by the vendored cryptography build) and source code for the same can be found at https://pypi.org/project/cffi/
bin/lib/_cffi_backend.cpython-313-x86_64-linux-gnu.so: This binary file is provided along with the cffi module (required by the vendored cryptography build) and source code for the same can be found at https://pypi.org/project/cffi/
bin/lib/_cffi_backend.cp39-win_amd64.pyd: This binary file is provided along with the cffi module (required by the vendored cryptography build) and source code for the same can be found at https://pypi.org/project/cffi/
bin/lib/_cffi_backend.cp313-win_amd64.pyd: This binary file is provided along with the cffi module (required by the vendored cryptography build) and source code for the same can be found at https://pypi.org/project/cffi/
bin/lib/cryptography/hazmat/bindings/_rust.abi3.so: This binary file is provided along with the cryptography module (cp39-abi3, covers Python 3.9-3.13) and source code for the same can be found at https://pypi.org/project/cryptography/
bin/lib/cryptography/hazmat/bindings/_rust.pyd: This binary file is provided along with the cryptography module (cp39-abi3, covers Python 3.9-3.13) and source code for the same can be found at https://pypi.org/project/cryptography/

# Platform-Specific Binaries
# This add-on bundles compiled extensions for two platforms (win_amd64, manylinux
# x86_64) and two CPython ABI tags: cp39 and cp313 for cffi's version-specific
# _cffi_backend extension, and one cp39-abi3 build per OS for cryptography's Rust
# bindings (abi3 spans CPython 3.9 through 3.13, so one binary per OS covers that
# whole range). This matches Splunk's own bundled-Python versions across the
# supported Splunk Enterprise releases (CPython 3.9 on 9.3-10.2, CPython 3.13 on
# 10.3+). Binaries are downloaded pre-built from the official PyPI wheels for
# cffi and cryptography (see the Binary File Declaration above for source links)
# and are never compiled from local/unreviewed sources.
