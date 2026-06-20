# Lupus app-agent

`app-agent` is the optional **remote agent** for [Lupus](https://alpistesec.com) — a single gRPC
server that runs Lupus's engine (Metasploit RPC, scanning tools, the workspace database, file
operations) on your own Linux host, so the Lupus Android app can drive it remotely instead of running
everything on the phone.

The phone never talks to Metasploit directly; it talks only to the agent over a single mutually
authenticated (mTLS + bearer token) gRPC port.

## Download

Grab the latest static binary from the [**Releases**](../../releases/latest) page:

- `app-agent-linux-amd64` — Linux x86-64
- `app-agent-linux-arm64` — Linux ARM64

They are statically linked (no dependencies). Make it executable and run it:

```bash
chmod +x app-agent-linux-amd64
./app-agent-linux-amd64 --version
```

## Prerequisites

The host needs the **Metasploit Framework** (provides `msfrpcd`), **PostgreSQL** (the Metasploit
database), and the scanning tools you intend to use (`nmap`, `nuclei`). Only the agent's gRPC port
(`50051`) needs to be reachable from the phone — `msfrpcd` and PostgreSQL stay on `127.0.0.1`.

## Quick start (enrollment)

Start PostgreSQL and `msfrpcd`, then run the agent in enrollment mode (it auto-generates its TLS
certificates into `--certs-dir`):

```bash
./app-agent-linux-amd64 \
  --listen 0.0.0.0:50051 \
  --certs-dir ~/.app-agent/certs \
  --cert-sans "<your-public-ip>" \
  --msf-host 127.0.0.1 --msf-port 55599 --msf-user app --msf-pass app --msf-ssl=true \
  --enrollment-mode \
  --allowed-ips "<your-phone-ip>/32"
```

The agent prints an 8-character **enrollment code**. In the Lupus app's connection screen, enter the
host, port `50051` and the code, then tap **Enroll & Connect**. On later restarts, run the agent
without `--enrollment-mode` — the app reuses the saved certificates and token.

Full setup, options and security model: **<https://alpistesec.com/docs#remote>**.

## Responsible use

Lupus and this agent are for lawful, authorized security testing and education only. Only target
systems and networks you own or have explicit written permission to test.
