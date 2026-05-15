# AGENTS.md

This file provides guidance to coding agents (e.g. Claude Code, claude.ai/code) when working with code in this repository.

## Repository purpose

Go module `go.bytebuilders.dev/cert-manager-webhook-ace` — a [cert-manager](https://cert-manager.io/) ACME DNS-01 webhook solver that proxies challenge records through **AppsCode's DNS Proxy for Cloudflare**. Lets cert-manager solve DNS-01 challenges without giving each cluster a Cloudflare API token directly — the cluster talks to the AppsCode DNS proxy instead, which holds the credentials centrally.

The produced binary is `cert-manager-webhook-ace`. Runs as a Kubernetes deployment that cert-manager calls into via its webhook solver interface.

## Architecture

- `main.go` — entry point. Registers the webhook solver implementation with cert-manager's webhook framework.
- `cloudflare/` — the actual challenge plumbing: presents/cleans up TXT records through the AppsCode DNS Proxy for Cloudflare.
- `Dockerfile.in` (PROD, distroless), `Dockerfile.dbg` (debian) — two image variants (no UBI).
- `hack/`, `Makefile` — AppsCode build harness.
- `vendor/` — checked-in deps.

This is a **cert-manager webhook solver**, not a controller — its job is to satisfy cert-manager's `Present` / `CleanUp` interface during ACME challenges.

## Common commands

- `make ci` — full CI pipeline.
- `make build` / `make all-build` — host or all-platform build.
- `make fmt`, `make lint`, `make unit-tests` / `make test` — standard.
- `make verify` — codegen + module-tidy verification.
- `make container` / `make push` / `make release` — image build/publish flow.

## Conventions

- Module path is `go.bytebuilders.dev/cert-manager-webhook-ace` (vanity URL); imports must use that.
- License: `LICENSE`. Sign off commits (`git commit -s`).
- Vendor directory is checked in; keep `go mod tidy && go mod vendor` clean.
- Webhook contract is owned by cert-manager — pin the `cert-manager` dep deliberately when bumping. The webhook framework's `Present` / `CleanUp` signatures change rarely but breaking them silently corrupts ACME challenges.
- The Cloudflare proxy interaction lives in `cloudflare/` — if you add another DNS backend, add a sibling package and pick the backend at startup; don't branch inside the solver.
- Two Dockerfiles, one binary — keep `Dockerfile.in` and `Dockerfile.dbg` in sync.
