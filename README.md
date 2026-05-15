# evm-oracle-demo-protocols

Protobuf definitions and gRPC service contracts for the **uw-oracle** demo — a pull-based, multi-source price oracle covering 5 crypto and 5 RWA assets. Single source of truth for the wire formats spoken by every Go service in the stack.

> Forked from [`andskur/protocols-template`](https://github.com/andskur/protocols-template) and trimmed to the surfaces this project needs.

## What lives here

| Package          | Purpose                                                                  |
|------------------|--------------------------------------------------------------------------|
| `common/v1`      | Shared types: `EthAddress`, `Hash`, `Wei`, `BlockNumber`, `LogCursor`, `EventMeta`, cursor/offset pagination, structured error envelope. |
| `price/v1`       | `PriceService` — `GetPrice`, `Subscribe`. Consumed by `oracle-service` and `rest-api`. |
| `oracle/v1`      | `OracleService` — `TriggerUpdate`, `SetHeartbeat`, `GetSubmissionStatus`. Consumed by `indexer-service` and admin tooling. |
| `indexer/v1`     | `IndexerService` — `ListEvents`, `GetRequest`, `StreamEvents`. Consumed by `rest-api`. |

## Repo layout

```
.
├── common/v1/          # Shared messages (blockchain types, pagination, errors)
├── price/v1/           # Price service contract
├── oracle/v1/          # Oracle service contract
├── indexer/v1/         # Indexer service contract
├── buf.yaml            # buf workspace + STANDARD + PACKAGE_DIRECTORY_MATCH lint
├── buf.gen.yaml        # buf generate plugin config (protoc-gen-go + grpc-go)
├── Makefile            # buf-lint, buf-generate, protoc-generate-all, validate
├── docs/               # Generic buf/protoc usage guides
├── scripts/            # add-service, validate helpers
└── .github/workflows/  # CI (lint, breaking, generate), docs, release
```

Generated `*.pb.go` and `*_grpc.pb.go` files are **not committed** — every downstream service runs codegen at build time. See `.gitignore`.

## Local development

Install tooling:

```bash
make install   # buf + protoc-gen-go + protoc-gen-go-grpc
```

Validate before committing:

```bash
make buf-lint               # STANDARD + PACKAGE_DIRECTORY_MATCH + PACKAGE_VERSION_SUFFIX
make buf-breaking           # diff against origin/main
make protoc-validate        # secondary check via raw protoc
make validate-ci            # all three together (what CI runs)
```

Generate Go bindings locally for inspection:

```bash
make buf-generate-all       # all packages
make buf-generate PACKAGE=price
make protoc-generate-all    # alt codegen path
```

## Versioning

Repository follows semver. Proto packages carry their own `vN` suffix:

| Change                                              | Repo bump | Proto bump |
|-----------------------------------------------------|-----------|------------|
| New message, new optional field, new RPC            | `vX.Y.Z+1`| same `vN`  |
| Field rename, type change, RPC removal, semantic break | `vX.Y+1.0` *(pre-1.0)* / `v(X+1).0.0` *(post-1.0)* | new `vN+1` package |

Breaking changes are gated by `buf breaking` in CI against `origin/<base_ref>`. The first PR after the v0.1.0 tag is what locks the baseline; from then on, breaking diffs need a new package version.

## Consuming this repo (downstream Go services)

Every Go service in the project pulls this repo in as a **git subtree** under `protocols/`. The subtree wire-up is done by the human at service-repo bootstrap time; consumers see the protos as if they were vendored files.

### One-time wire-up (per consuming service)

```bash
# In the consuming service repo, on a feature branch:
git remote add -f protocols https://github.com/asolovov/evm-oracle-demo-protocols.git
git subtree add --prefix=protocols protocols main --squash
```

### Pulling updates

```bash
git subtree pull --prefix=protocols protocols main --squash
```

### Generating Go bindings inside the consumer

The recommended pattern is to drive codegen from the consumer's own Makefile, pointing at `protocols/`:

```makefile
.PHONY: proto-generate
proto-generate:
	@buf generate protocols
```

…or, with raw `protoc`:

```makefile
proto-generate:
	@find protocols -name "*.proto" -exec protoc \
	  --proto_path=protocols \
	  --go_out=internal/gen --go_opt=paths=source_relative \
	  --go-grpc_out=internal/gen --go-grpc_opt=paths=source_relative \
	  {} \;
```

### `go.mod` replace (when iterating locally)

If you need to point a consumer at an in-flight branch of this repo without committing a subtree push:

```go
// go.mod (consumer)
require github.com/asolovov/evm-oracle-demo-protocols v0.1.0

replace github.com/asolovov/evm-oracle-demo-protocols => ../evm-oracle-demo-protocols
```

…then revert the `replace` once the changes land on `main` here and have been pulled back via `git subtree pull`.

## CI

Every PR runs:

1. `buf lint` — STANDARD + PACKAGE_DIRECTORY_MATCH + PACKAGE_VERSION_SUFFIX
2. `buf breaking` against `origin/<base_ref>` (PR-only)
3. `protoc` validation (defence-in-depth, catches buf-vs-protoc disagreements)
4. `buf generate` + `make protoc-generate-all` smoke tests

All third-party actions in `.github/workflows/` are pinned to commit SHAs (org policy NFR-07).

## Release

Releases are cut manually via the `Release` workflow's `workflow_dispatch` trigger. Choose `patch`, `minor`, or `major`; the workflow validates, tags, and publishes a GitHub Release populated from the commit log.

The `v0.1.0` tag (initial public release) is cut by the human once a downstream consumer has confirmed end-to-end integration. See `CHANGELOG.md`.

## License

MIT — see [LICENSE](LICENSE).
