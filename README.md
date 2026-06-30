# Dyalog APL Socket Server

A TCP socket server for Dyalog APL that evaluates APL expressions over a network connection. Supports plain-text and JSON protocols, uses Dyalog [Conga](https://docs.dyalog.com/latest/Conga%20User%20Guide.pdf) library.

## Why

The state you care about in an APL session is usually already in memory: arrays you have built up, intermediate results, the workspace as it stands right now. `Socket` lets a program outside Dyalog reach that state. A client opens a TCP connection, sends an expression, and reads the result back, evaluated against the running interpreter with nothing written to disk in between.

The wire format is newline-delimited text, or JSON if you want structured output, so the client needs no Dyalog-specific library. Typical uses are scripting a session from Python, poking at it from the shell with `nc` or `socat`, connecting an editor, exposing an APL computation as a small network service, or giving an AI agent a tool it can call to run APL and read back the result.

## Quick start

Start the server inside Dyalog:

```apl
⎕SE.Dyalog.Socket.Start 'T:localhost:11612'
```

Then talk to it from any shell:

```sh
echo '⍳5' | nc localhost 11612
# → 1 2 3 4 5
```

Or with the JSON protocol (`J:localhost:11612`):

```sh
echo '{"ex":"2+2"}' | nc localhost 11612
# → {"ex":"2+2","r":4}
```

## Installation

The installer fetches `Socket.apln` from GitHub and writes it to `$DYALOG/StartupSession/Dyalog/`, making it available in every workspace automatically.

**One line (Linux/macOS).** Pipe the installer straight into `dyalogscript`:

```sh
curl -fsSL https://github.com/dyalog-labs/socketserver/releases/latest/download/install.apls | tail -n +2 | dyalogscript /dev/stdin
```

`tail -n +2` strips the shebang line, which `dyalogscript` would otherwise try to evaluate as APL.

Writing into a system-wide Dyalog install usually needs elevated permissions. Put `sudo` on the `dyalogscript` end of the pipe, not in front of `curl` — otherwise only the download runs as root and the file write still fails:

```sh
curl -fsSL https://github.com/dyalog-labs/socketserver/releases/latest/download/install.apls | tail -n +2 | sudo dyalogscript /dev/stdin
```

This pipeline relies on `tail` and `/dev/stdin`, so it is Linux/macOS only. On Windows, use the download method below.

**Download and run.** If you'd rather read the script before trusting it, or you're on Windows, download `install.apls` from the [latest release](https://github.com/dyalog-labs/socketserver/releases/latest) and run it:

```sh
dyalogscript install.apls
```

Add `sudo` (Linux) or use an Administrator shell (Windows) if the write is denied.

**Manual install.** If you'd rather place the file yourself, download `Socket.apln` from the release and copy it into your Dyalog startup session folder:

```sh
cp Socket.apln "$DYALOG/StartupSession/Dyalog/"
```

`$DYALOG` is your Dyalog installation directory. On Windows the equivalent path is `%DYALOG%\StartupSession\Dyalog\`. Anything in this folder is loaded into `⎕SE` automatically when Dyalog starts.

## Usage

### Starting the server

```apl
rc msg←⎕SE.Dyalog.Socket.Start 'T:localhost:11612'   ⍝ text protocol on port 11612
rc msg←⎕SE.Dyalog.Socket.Start 'J:localhost:11612'   ⍝ JSON protocol on port 11612
```

The argument is `protocol:host:port` where protocol is `T` (text) or `J` (JSON).

On success `rc` is `0` and `msg` describes the listening port. On failure `rc` is `¯1` and `msg` is the error.

### Stopping the server

```apl
rc msg←⎕SE.Dyalog.Socket.Stop
```

### Auto-start via environment variable

Set `SOCKET_INIT` to a start argument before launching Dyalog and the server will start automatically:

```sh
SOCKET_INIT="J:localhost:11612" dyalog
```

## Protocols

### Text (`T`)

Send newline-delimited APL expressions. Each line is evaluated independently and the formatted result is sent back, one result line per input line, so a client can pair requests and responses without relying on read timeouts. A multi-line result (e.g. a matrix) spans multiple response lines.

```
→ 2+2
← 4
→ ⍳5
← 1 2 3 4 5
```

Expressions with no result (e.g. a silent assignment) reply with an empty line. Errors come back as a single line beginning with `Error:`.

### JSON (`J`)

Send newline-delimited JSON objects, each with an `ex` field containing the APL expression to evaluate. The response is the original JSON object echoed back with an added `r` field containing the result.

```json
→ {"ex": "2+2"}
← {"ex": "2+2", "r": 4}
```

Errors are returned as the original JSON object with an `e` field containing Dyalog's `⎕DMX` error information, including `EN` (error number), `EM` (error message, e.g. `DOMAIN ERROR`), `Message`, and `DM` (the diagnostic message vector).

```json
→ {"ex": "1÷0"}
← {"ex": "1÷0", "e": {"EM": "DOMAIN ERROR", "EN": 11, "Message": "Divide by zero", ...}}
```

High-rank results round-trip as nested arrays (a matrix becomes a list of rows). An expression with no result, or a line with no `ex` field, is echoed back unchanged with no `r` field added.

## Execution context

Expressions are evaluated in the namespace from which `Start` was called. They can see, and modify, names visible in that context, and global state persists between calls, so a client can build up state across multiple requests.

## Configuration

These variables can be set on the `Socket` namespace before calling `Start`:

| Variable | Default | Description |
|---|---|---|
| `BufferSize` | `10000` | Conga receive buffer size in bytes |
| `WaitTimeout` | `1000` | Milliseconds between each Conga wait poll |

## Security

The server evaluates **arbitrary APL** with the full privileges of the Dyalog process — file access, command execution, everything. Bind it to `localhost` (the default in the examples) unless you have a specific reason, and a trust boundary, to expose it more widely. Do not put it on a public interface without an authenticating proxy in front of it.

## Testing

Run the test suite with:

```sh
dyalogscript Test.apls
```

It starts real servers on local ports and exercises both protocols end to end, including error handling and per-line independence.

## Requirements

- Dyalog APL 20.0 or later
- Conga (ships with Dyalog; loaded automatically from `[DYALOG]/ws/conga`)
