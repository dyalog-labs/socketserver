# Changelog

All notable changes to this project are documented here. The format is based
on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.0.0]

First public release.

### Added

- TCP socket server evaluating APL expressions over a network connection.
- Text protocol (`T`) — newline-delimited expressions, one result line per input line.
- JSON protocol (`J`) — newline-delimited `{"ex": ...}` objects echoed back with an `r` result field, or an `e` field carrying `⎕DMX` on error.
- `Start` / `Stop` with state guards against double-start and double-stop.
- Auto-start via the `SOCKET_INIT` environment variable.
- `Version` field reporting the running server version.
- Configurable `BufferSize` and `WaitTimeout`.
- `install.apls` installer that fetches the namespace from the GitHub release and writes it into the Dyalog startup session, with permission-aware error messages.
- Test suite (`Test.apls`) exercising both protocols end to end.
- GitHub Actions workflows for CI testing and tagged releases.

[Unreleased]: https://github.com/dyalog-labs/socketserver/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/dyalog-labs/socketserver/releases/tag/v1.0.0
