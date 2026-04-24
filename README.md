# Socket — Dyalog APL Socket Server

A TCP socket server for Dyalog APL that evaluates APL expressions over a network connection. Supports plain-text and JSON protocols, backed by Dyalog's [Conga](https://docs.dyalog.com/latest/Conga%20User%20Guide.pdf) library.

## Installation

Run the installer to copy the namespace into Dyalog's startup session, making it available in every workspace automatically:

```sh
sudo dyalogscript install.apls
```

This writes `Socket.apln` to `$DYALOG/StartupSession/Dyalog/`.

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

Send newline-delimited APL expressions. Each line is evaluated independently and the formatted result is sent back, one result per line.

```
→ 2+2
← 4
→ ⍳5
← 1 2 3 4 5
```

### JSON (`J`)

Send newline-delimited JSON objects, each with an `ex` field containing the APL expression to evaluate. The response is the original JSON object echoed back with an added `r` field containing the result.

```json
→ {"ex": "2+2"}
← {"ex": "2+2", "r": 4}
```

Errors are returned as the original JSON object with an `e` field containing Dyalog's `⎕DMX` error information.

```json
→ {"ex": "1÷0"}
← {"ex": "1÷0", "e": {"Message": "DOMAIN ERROR", ...}}
```

## Configuration

These variables can be set on the `Socket` namespace before calling `Start`:

| Variable | Default | Description |
|---|---|---|
| `BufferSize` | `10000` | Conga receive buffer size in bytes |
| `WaitTimeout` | `1000` | Milliseconds between each Conga wait poll |

## Requirements

- Dyalog APL 21.0 or later
- Conga (ships with Dyalog; loaded automatically from `[DYALOG]/ws/conga`)
