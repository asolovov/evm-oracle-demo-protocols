# Changelog

All notable changes to `evm-oracle-demo-protocols` are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] — TBD

Initial release. Cut by the human after task 05 (`uw-oracle-price-service`) confirms downstream subtree integration end-to-end.

### Added

- `common/v1/blockchain.proto` — EVM-native types: `EthAddress`, `Hash`, `Wei` (decimal-string to avoid uint256 overflow), `BlockNumber`, `LogCursor`, `EventMeta`.
- `price/v1/price.proto` — `PriceService` with `GetPrice` and `Subscribe`; `AggregatedPrice` carries median + per-source `SourceContribution` breakdown.
- `oracle/v1/oracle.proto` — `OracleService` with `TriggerUpdate`, `SetHeartbeat`, `GetSubmissionStatus`; `SubmissionStatus.Status` covers the pending/confirmed/failed/dropped tx lifecycle.
- `indexer/v1/indexer.proto` — `IndexerService` with `ListEvents`, `GetRequest`, and `StreamEvents`; opaque `Event.payload` lets consumers decode with their own ABI bindings.
- `buf breaking` baseline against `origin/<base_ref>`; first PR after this tag locks the wire format.
- All `.github/workflows/` actions SHA-pinned (NFR-07).

### Removed

- Template example `user/` service.
- Template `common/v1/status.proto` and `common/v1/types.proto` (UUID, Money, Address, CommonStatus) — none of which the oracle pipeline uses.

[Unreleased]: https://github.com/asolovov/evm-oracle-demo-protocols/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/asolovov/evm-oracle-demo-protocols/releases/tag/v0.1.0
