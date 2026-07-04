# OCI Logging Add-On 
## Release 3.0.1
- Added Python 3.9 compatibility support for Splunk Enterprise 9.x+ (Linux x86_64).
- Bundled the Python 3.9 `_cffi_backend` C-extension library (`bin/_cffi_backend.cpython-39-x86_64-linux-gnu.so`).
- Scoped the `python.version = python3.9` configuration strictly under the `[oci_logging]` modular input stanza in `default/inputs.conf`.
- Removed global `default/server.conf` configuration to prevent settings pollution and dependency conflicts with other Splunk add-ons (Sophos, AWS, GCP, MSCS).

## Release 3.0.0
- Validating OCI Streaming endpoint URL for HTTPS 
- Added support for pasting in OCI API Key.  This can be an RSA key or an OCI Console Key for a LOCAL OCI IAM user.
- Depreciated OCI API key upload feature

## Release 2.3.2
- Added retry logic for 50x responses from OCI Streaming
- Moved README.txt for Splunkbase AppInspect 
- Update OCI SDK to 2.99.0
## Release 2.3.0
- Native integration for Cloud Guard problems written to OCI Streams via OCI Events.
    - This allows any OCI Event written to the OCI Stream to be written into Splunk
- Improved Error handling for 401 and 404 errors
- Updated Splunk Lib
- Updated OCI SDK to 2.90.3
- Updated app file permissions
- Added a README.txt with binary documentation

## Release 2.2.2
- Updated OCI SDK to 2.88.1

## Release 2.2.1
- Local File System Key is the default
- Support for upload of OCI API Key file is moved to `More settings`
- Updated OCI Python SDK to 2.54.0
- Updated [README.md](README.md) with prerequisites and troubleshooting

## Release 2.2.0
- Added help text to inputs screen
- Restored Support for uploading an RSA OCI API Key 
- Support for local file system key file is moved to `More settings`
    - Local File System Key is required for Splunk Version 8.0
- Added logger.info message for unsupported data in the stream
- Added logger.info for when data is not returned.  This is track the plugin is running without using DEBUG
- PEP8 clean up

## Release 2.1.1
- Added release notes
- Converted to an OCI API key stored on the Heavy Forwarder instead of uploaded via the Web UI
- Support for OCI API Keys generated from the console
- Updated OCI Python SDK to 2.45.1
- Removed __pycache__ and other clean up
- Renamed input variable *Worker Processes* to *Number of partitions*