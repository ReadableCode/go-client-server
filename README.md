# Go Client Server App

A minimal client and server for sending a text command to a machine over
HTTP, in one Go file with no dependencies outside the standard library.

The same binary runs in either role. In server mode it listens for commands
and echoes each one back. In client mode it takes a list of servers, tries
them in order, and sends the command to the first one that answers. It is a
small starting point for remote command handling with simple failover, and a
quick way to check that one machine can reach another.

## How it works

- **Server** (`--mode=server`): serves `POST /command` on the given port. It
  reads the request body as the command, logs `Received command: <body>` and
  replies `Received: <body>`. The command is only echoed, never executed.
- **Client** (`--mode=client`): splits `--servers` on commas and POSTs the
  command as `text/plain` to `http://<addr>/command` on each address in turn,
  with a 2 second timeout per server. It logs the reply of the first server
  that responds and stops there. If none respond it logs `No servers
  responded`.
- Any HTTP client can stand in for the Go client, as the curl examples below
  show.

## Flags

| Flag        | Default  | Meaning                                                  |
| ----------- | -------- | -------------------------------------------------------- |
| `--mode`    | `server` | `server` or `client`; anything else exits with an error  |
| `--port`    | `8080`   | Port the server listens on                               |
| `--servers` | (none)   | Client only: comma-separated `IP:port` list, required    |
| `--command` | `ping`   | Client only: command text to send                        |

## Requirements

Go (the module declares `go 1.26.5`).

## Run the server

```bash
go run main.go --mode=server --port=8080
```

## Run the client

```bash
go run main.go --mode=client --servers=127.0.0.1:8080,203.0.113.10:8080 --command="ping"
```

Example output when the first server is down and the second answers (log
timestamps removed):

```text
Trying http://127.0.0.1:8081/command...
Connection failed: Post "http://127.0.0.1:8081/command": dial tcp 127.0.0.1:8081: connect: connection refused
Trying http://127.0.0.1:8080/command...
Server 127.0.0.1:8080 responded: Received: ping
```

## Use curl as the client

- Linux

```bash
curl -X POST http://127.0.0.1:8080/command -d "ping"
```

- Windows (PowerShell)

```powershell
Invoke-WebRequest -Uri http://127.0.0.1:8080/command -Method POST -Body "ping"
```
