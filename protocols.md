# Protocol Documentation
<a name="top"></a>

## Table of Contents

- [oracle/v1/oracle.proto](#oracle_v1_oracle-proto)
    - [GetSubmissionStatusRequest](#oracle-v1-GetSubmissionStatusRequest)
    - [ListSubmissionsRequest](#oracle-v1-ListSubmissionsRequest)
    - [ListSubmissionsResponse](#oracle-v1-ListSubmissionsResponse)
    - [SetHeartbeatRequest](#oracle-v1-SetHeartbeatRequest)
    - [SetHeartbeatResponse](#oracle-v1-SetHeartbeatResponse)
    - [SubmissionStatus](#oracle-v1-SubmissionStatus)
  
    - [SubmissionStatus.Status](#oracle-v1-SubmissionStatus-Status)
  
    - [OracleService](#oracle-v1-OracleService)
  
- [price/v1/price.proto](#price_v1_price-proto)
    - [AggregatedPrice](#price-v1-AggregatedPrice)
    - [GetPriceRequest](#price-v1-GetPriceRequest)
    - [SourceContribution](#price-v1-SourceContribution)
    - [SubscribeRequest](#price-v1-SubscribeRequest)
  
    - [PriceService](#price-v1-PriceService)
  
- [common/v1/errors.proto](#common_v1_errors-proto)
    - [ErrorDetail](#common-v1-ErrorDetail)
    - [ErrorResponse](#common-v1-ErrorResponse)
  
- [common/v1/pagination.proto](#common_v1_pagination-proto)
    - [CursorPageRequest](#common-v1-CursorPageRequest)
    - [CursorPageResponse](#common-v1-CursorPageResponse)
    - [PageRequest](#common-v1-PageRequest)
    - [PageResponse](#common-v1-PageResponse)
  
- [indexer/v1/indexer.proto](#indexer_v1_indexer-proto)
    - [AssetRegisteredEvent](#indexer-v1-AssetRegisteredEvent)
    - [Event](#indexer-v1-Event)
    - [EventMeta](#indexer-v1-EventMeta)
    - [GetRequestRequest](#indexer-v1-GetRequestRequest)
    - [ListEventsRequest](#indexer-v1-ListEventsRequest)
    - [ListEventsResponse](#indexer-v1-ListEventsResponse)
    - [PriceFulfilledEvent](#indexer-v1-PriceFulfilledEvent)
    - [PriceRequestedEvent](#indexer-v1-PriceRequestedEvent)
    - [RequestStatus](#indexer-v1-RequestStatus)
    - [StreamEventsRequest](#indexer-v1-StreamEventsRequest)
  
    - [EventKind](#indexer-v1-EventKind)
    - [RequestStatus.Status](#indexer-v1-RequestStatus-Status)
  
    - [IndexerService](#indexer-v1-IndexerService)
  
- [Scalar Value Types](#scalar-value-types)



<a name="oracle_v1_oracle-proto"></a>
<p align="right"><a href="#top">Top</a></p>

## oracle/v1/oracle.proto



<a name="oracle-v1-GetSubmissionStatusRequest"></a>

### GetSubmissionStatusRequest
GetSubmissionStatusRequest selects a submission by exactly one of
req_id or tx_hash. Sending both, or neither, is rejected with
INVALID_ARGUMENT.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| req_id | [string](#string) |  | uint256 as base-10 decimal string. Empty when querying by tx_hash. &#34;0&#34; is reserved for heartbeat submissions and cannot be looked up here — use tx_hash for those. |
| tx_hash | [string](#string) |  | 32-byte tx hash, 0x-prefixed lowercase hex. Empty when querying by req_id. |






<a name="oracle-v1-ListSubmissionsRequest"></a>

### ListSubmissionsRequest
ListSubmissionsRequest pages over submissions in descending
submitted_at order. `asset_id` is an optional exact-match filter.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| asset_id | [string](#string) |  |  |
| page | [common.v1.PageRequest](#common-v1-PageRequest) |  |  |






<a name="oracle-v1-ListSubmissionsResponse"></a>

### ListSubmissionsResponse



| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| submissions | [SubmissionStatus](#oracle-v1-SubmissionStatus) | repeated |  |
| page | [common.v1.PageResponse](#common-v1-PageResponse) |  |  |






<a name="oracle-v1-SetHeartbeatRequest"></a>

### SetHeartbeatRequest
SetHeartbeatRequest reconfigures the heartbeat for one asset.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| asset_id | [string](#string) |  |  |
| interval_sec | [uint32](#uint32) |  | Heartbeat interval in seconds. 0 disables the heartbeat for this asset. |
| deviation_bps | [uint32](#uint32) |  | Deviation threshold in basis points (1 bp = 0.01%). If the newly aggregated price has moved by at least this much since the last on-chain price, a heartbeat fires regardless of `interval_sec`. 0 disables the deviation-based path; the heartbeat then runs strictly on the time schedule. |






<a name="oracle-v1-SetHeartbeatResponse"></a>

### SetHeartbeatResponse
SetHeartbeatResponse is intentionally empty — the configuration
is keyed by asset_id and idempotent on replay.






<a name="oracle-v1-SubmissionStatus"></a>

### SubmissionStatus
SubmissionStatus is the lifecycle of one `fulfillPrice` tx.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| req_id | [string](#string) |  | uint256 as base-10 decimal string. &#34;0&#34; for heartbeat submissions (no consumer request behind them). |
| asset_id | [string](#string) |  | Canonical asset id (matches PriceService.asset_id form). |
| tx_hash | [string](#string) |  | 32-byte tx hash, 0x-prefixed lowercase hex. Empty while STATUS_PENDING and the tx has not yet been broadcast. |
| submitted_price | [string](#string) |  | int256 as base-10 decimal string in Chainlink&#39;s 8-decimal scale (e.g. &#34;345020000000&#34; represents a USD price of 3450.20). Empty until the price has been signed. |
| submitted_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | When the tx was first broadcast (or rebroadcast after replace). Server-set; never trust client clocks here. |
| status | [SubmissionStatus.Status](#oracle-v1-SubmissionStatus-Status) |  |  |
| retry_count | [uint32](#uint32) |  | Number of replace-by-fee attempts so far. 0 means the tx was broadcast once and not bumped. |
| last_error | [string](#string) |  | Free-form error text from the last failed broadcast or revert. Empty unless STATUS_FAILED or a retry happened. |





 


<a name="oracle-v1-SubmissionStatus-Status"></a>

### SubmissionStatus.Status


| Name | Number | Description |
| ---- | ------ | ----------- |
| STATUS_UNSPECIFIED | 0 |  |
| STATUS_PENDING | 1 | Tx built and signed but not yet broadcast, or broadcast and still in the mempool. |
| STATUS_CONFIRMED | 2 | Mined and included with status=1 (success). |
| STATUS_FAILED | 3 | Mined and reverted, or rejected pre-broadcast (e.g. bad nonce, signature verification failure). |
| STATUS_DROPPED | 4 | Dropped from the mempool by the node without ever being mined. After 3 replace-by-fee retries we give up and mark dropped. |


 

 


<a name="oracle-v1-OracleService"></a>

### OracleService
OracleService is the signing &#43; on-chain submission surface of the
oracle pipeline. Its gRPC surface is admin &#43; read only — there is
no `TriggerUpdate` RPC.

Trigger model: oracle-service is a long-lived client of
`indexer.StreamEvents` with `kinds=[EVENT_KIND_PRICE_REQUESTED]`.
Each event delivered on that stream is already past-confirmations,
reorg-handled, and idempotency-keyed by `req_id`. The stream IS
the trigger; no separate RPC is needed.

Heartbeat updates are produced by an oracle-internal scheduler;
they do not enter via gRPC either. Operators reconfigure the
heartbeat schedule with SetHeartbeat below.

| Method Name | Request Type | Response Type | Description |
| ----------- | ------------ | ------------- | ------------|
| SetHeartbeat | [SetHeartbeatRequest](#oracle-v1-SetHeartbeatRequest) | [SetHeartbeatResponse](#oracle-v1-SetHeartbeatResponse) | SetHeartbeat — admin: configure the per-asset heartbeat schedule. Persisted; survives service restart. |
| GetSubmissionStatus | [GetSubmissionStatusRequest](#oracle-v1-GetSubmissionStatusRequest) | [SubmissionStatus](#oracle-v1-SubmissionStatus) | GetSubmissionStatus — read: status of a specific submission, looked up either by req_id (consumer-issued requests) or by tx_hash (heartbeat submissions, where req_id = &#34;0&#34;). |
| ListSubmissions | [ListSubmissionsRequest](#oracle-v1-ListSubmissionsRequest) | [ListSubmissionsResponse](#oracle-v1-ListSubmissionsResponse) | ListSubmissions — read: paginated submission history for the dashboard and for debugging. |

 



<a name="price_v1_price-proto"></a>
<p align="right"><a href="#top">Top</a></p>

## price/v1/price.proto



<a name="price-v1-AggregatedPrice"></a>

### AggregatedPrice
AggregatedPrice is the median across the live source set for an
asset at a point in time, with the per-source breakdown the
dashboard renders.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| asset_id | [string](#string) |  | Canonical asset id (see GetPriceRequest.asset_id). |
| median_price | [double](#double) |  | Median across included sources, in USD, as an IEEE-754 double. Always non-negative. |
| aggregated_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | Wall-clock time when this aggregation was produced. |
| sources | [SourceContribution](#price-v1-SourceContribution) | repeated | Per-source contributions. Includes both sources used in the median (`included = true`) and sources dropped by the freshness or deviation guard (`included = false`). Always populated even if empty — the dashboard renders the full set. |






<a name="price-v1-GetPriceRequest"></a>

### GetPriceRequest
GetPriceRequest selects an asset by its canonical id.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| asset_id | [string](#string) |  | Canonical asset id: lowercase symbol used as the bytes32 key on chain. Examples: &#34;weth&#34;, &#34;wbtc&#34;, &#34;xau&#34;, &#34;spx&#34;. |






<a name="price-v1-SourceContribution"></a>

### SourceContribution
SourceContribution is the single-source view of an aggregation
round — what the price was, how stale it was, and whether the
aggregator used it.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| source | [string](#string) |  | Source discriminator. Stable identifiers chosen at adapter implementation time: &#34;coingecko&#34;, &#34;binance&#34;, &#34;uniswap_v3&#34;, &#34;alpha_vantage&#34;, &#34;twelve_data&#34;, &#34;stooq&#34;. |
| price | [double](#double) |  | Price reported by this source, in USD, as an IEEE-754 double. |
| fetched_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | When price-service retrieved this datum. Bounded by the poller interval — typically within 30s of `aggregated_at` for crypto, much wider for RWA. |
| source_observed_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | When the upstream reports the price was observed (e.g. Binance ticker timestamp, Alpha Vantage close-of-bar). May lag fetched_at by hours for end-of-day RWA series. |
| age_sec | [int64](#int64) |  | Convenience: `now - source_observed_at` in seconds at the time this message was produced. Surfaced on the dashboard&#39;s freshness badge — clients SHOULD prefer this over recomputing from timestamps because aggregator and client clocks may drift. |
| included | [bool](#bool) |  | Whether this source contributed to the median. Sources are excluded on fetch error, schema mismatch, or deviation-guard rejection. The reason for exclusion is logged server-side; the wire format only carries the boolean. |






<a name="price-v1-SubscribeRequest"></a>

### SubscribeRequest
SubscribeRequest selects one or more assets to stream.
An empty asset_ids list is rejected with INVALID_ARGUMENT;
callers must opt in explicitly per-asset.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| asset_ids | [string](#string) | repeated | Asset ids to subscribe to. Same id form as GetPriceRequest. |





 

 

 


<a name="price-v1-PriceService"></a>

### PriceService
PriceService is the off-chain aggregation surface of the oracle.
It owns source polling and exposes the latest USD price per asset
to internal callers (oracle-service uses it when building
`fulfillPrice` payloads; rest-api uses it for the dashboard).

Pricing model: prices flow as IEEE-754 doubles end-to-end inside
the off-chain pipeline — sources return floats, aggregation runs on
floats, the wire format carries doubles. Conversion to the on-chain
int256 representation (Chainlink&#39;s 8-decimal scale) happens once,
inside oracle-service, when it builds the fulfillPrice call. Keep
the conversion in one place; do not scatter it across services.

| Method Name | Request Type | Response Type | Description |
| ----------- | ------------ | ------------- | ------------|
| GetPrice | [GetPriceRequest](#price-v1-GetPriceRequest) | [AggregatedPrice](#price-v1-AggregatedPrice) | GetPrice returns the most recently aggregated price for one asset. Returns NOT_FOUND if the asset is not tracked. |
| Subscribe | [SubscribeRequest](#price-v1-SubscribeRequest) | [AggregatedPrice](#price-v1-AggregatedPrice) stream | Subscribe streams aggregated prices for the requested assets. On stream open, the server pushes the current value for each subscribed asset; thereafter it pushes updates as new aggregations are produced. The stream stays open until the client cancels. |

 



<a name="common_v1_errors-proto"></a>
<p align="right"><a href="#top">Top</a></p>

## common/v1/errors.proto



<a name="common-v1-ErrorDetail"></a>

### ErrorDetail
ErrorDetail provides structured information about a single error.
Can be used to represent validation errors, business logic errors, etc.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| code | [string](#string) |  | Error code (e.g., INVALID_ARGUMENT, NOT_FOUND, PERMISSION_DENIED). Should follow gRPC status codes or custom application codes. |
| message | [string](#string) |  | Human-readable error message. |
| field | [string](#string) |  | Field name that caused the error (optional). Useful for validation errors to identify which field failed. |
| metadata | [google.protobuf.Any](#google-protobuf-Any) |  | Additional context-specific metadata (optional). Can contain any serialized message type for rich error details. |






<a name="common-v1-ErrorResponse"></a>

### ErrorResponse
ErrorResponse provides a consistent structure for error responses.
Use this for returning multiple errors in a single response.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| errors | [ErrorDetail](#common-v1-ErrorDetail) | repeated | List of errors that occurred. Multiple errors allow returning all validation failures at once. |
| trace_id | [string](#string) |  | Trace ID for correlating errors across services. Should match the trace ID in distributed tracing systems. |





 

 

 

 



<a name="common_v1_pagination-proto"></a>
<p align="right"><a href="#top">Top</a></p>

## common/v1/pagination.proto



<a name="common-v1-CursorPageRequest"></a>

### CursorPageRequest
CursorPageRequest represents cursor-based pagination parameters.
Use this for efficient pagination of large datasets or real-time data.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| cursor | [string](#string) |  | Opaque cursor string from previous response. Empty string for first page. |
| limit | [int32](#int32) |  | Maximum number of items to return. |






<a name="common-v1-CursorPageResponse"></a>

### CursorPageResponse
CursorPageResponse contains cursor pagination metadata.
Include this in list responses when using CursorPageRequest.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| next_cursor | [string](#string) |  | Cursor for fetching the next page. Empty if no more pages available. |
| has_more | [bool](#bool) |  | Indicates if more results are available. |






<a name="common-v1-PageRequest"></a>

### PageRequest
PageRequest represents offset/limit pagination parameters.
Use this for traditional page-based pagination.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| page | [int32](#int32) |  | Page number (1-indexed). First page is 1. |
| page_size | [int32](#int32) |  | Number of items per page. Maximum should be enforced by service. |






<a name="common-v1-PageResponse"></a>

### PageResponse
PageResponse contains metadata about paginated results.
Include this in list responses when using PageRequest.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| total_count | [int32](#int32) |  | Total number of items across all pages. |
| page | [int32](#int32) |  | Current page number (1-indexed). |
| page_size | [int32](#int32) |  | Number of items per page. |
| total_pages | [int32](#int32) |  | Total number of pages available. |





 

 

 

 



<a name="indexer_v1_indexer-proto"></a>
<p align="right"><a href="#top">Top</a></p>

## indexer/v1/indexer.proto



<a name="indexer-v1-AssetRegisteredEvent"></a>

### AssetRegisteredEvent
AssetRegisteredEvent — emitted by OracleRegistry.registerAsset.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| asset_id | [string](#string) |  | bytes32 asset id, 0x-prefixed lowercase hex. |
| aggregator | [string](#string) |  | PriceAggregator contract registered for this asset. 20-byte address, 0x-prefixed lowercase hex. |






<a name="indexer-v1-Event"></a>

### Event
Event is a single observed log with its typed decoded payload.
`kind` mirrors which variant of `payload` is set — clients can
dispatch on `kind` without unwrapping the oneof.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| meta | [EventMeta](#indexer-v1-EventMeta) |  |  |
| kind | [EventKind](#indexer-v1-EventKind) |  |  |
| price_requested | [PriceRequestedEvent](#indexer-v1-PriceRequestedEvent) |  |  |
| price_fulfilled | [PriceFulfilledEvent](#indexer-v1-PriceFulfilledEvent) |  |  |
| asset_registered | [AssetRegisteredEvent](#indexer-v1-AssetRegisteredEvent) |  |  |






<a name="indexer-v1-EventMeta"></a>

### EventMeta
EventMeta carries the chain-derived context for every observed
event: where it came from, when we saw it, and how confirmed it
is at observation time.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| contract_address | [string](#string) |  | Emitting contract address. 0x-prefixed lowercase hex of the 20-byte address. |
| tx_hash | [string](#string) |  | 32-byte tx hash, 0x-prefixed lowercase hex. |
| block_hash | [string](#string) |  | 32-byte block hash, 0x-prefixed lowercase hex. |
| block_number | [uint64](#uint64) |  | Block height where the log was emitted. |
| log_index | [uint32](#uint32) |  | Log index within the block (0-based, as reported by the node). |
| observed_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | Wall-clock time at which the indexer observed (decoded &#43; persisted) this event. NOT the block timestamp. |
| confirmations | [uint32](#uint32) |  | Number of blocks built on top of `block_number` at the time this message was produced. For StreamEvents this is always ≥ the confirmation threshold; for ListEvents it is whatever depth the reconciler last recorded. |






<a name="indexer-v1-GetRequestRequest"></a>

### GetRequestRequest
GetRequestRequest selects a request by its on-chain id.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| req_id | [string](#string) |  | uint256 as base-10 decimal string. |






<a name="indexer-v1-ListEventsRequest"></a>

### ListEventsRequest
ListEventsRequest filters and paginates the persisted event log.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| kinds | [EventKind](#indexer-v1-EventKind) | repeated | Empty = all kinds. Otherwise the union of the listed kinds. |
| asset_id | [string](#string) |  | Optional asset id filter (bytes32 hex). Applied only to events that carry an asset_id (PriceRequested, PriceFulfilled, AssetRegistered). |
| from_block | [uint64](#uint64) |  | Inclusive lower bound on block number. 0 = no lower bound. |
| to_block | [uint64](#uint64) |  | Inclusive upper bound on block number. 0 = no upper bound. |
| page | [common.v1.PageRequest](#common-v1-PageRequest) |  |  |






<a name="indexer-v1-ListEventsResponse"></a>

### ListEventsResponse



| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| events | [Event](#indexer-v1-Event) | repeated |  |
| page | [common.v1.PageResponse](#common-v1-PageResponse) |  |  |






<a name="indexer-v1-PriceFulfilledEvent"></a>

### PriceFulfilledEvent
PriceFulfilledEvent — emitted by PriceAggregator.fulfillPrice.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| req_id | [string](#string) |  | uint256 as base-10 decimal string. |
| asset_id | [string](#string) |  | bytes32 asset id, 0x-prefixed lowercase hex. |
| price | [string](#string) |  | int256 price as base-10 decimal string, in Chainlink&#39;s 8-decimal scale. |
| timestamp | [string](#string) |  | uint256 contract-emitted seconds-since-epoch as base-10 decimal string. Kept as a string per the uint256 convention; clients that need a Timestamp can parse and convert. |
| round_id | [string](#string) |  | uint80 round id (Chainlink-style) as base-10 decimal string. |






<a name="indexer-v1-PriceRequestedEvent"></a>

### PriceRequestedEvent
PriceRequestedEvent — emitted by PriceAggregator.requestPrice.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| req_id | [string](#string) |  | uint256 as base-10 decimal string. |
| asset_id | [string](#string) |  | bytes32 asset id, 0x-prefixed lowercase hex. |
| requester | [string](#string) |  | 20-byte requester address, 0x-prefixed lowercase hex. |






<a name="indexer-v1-RequestStatus"></a>

### RequestStatus
RequestStatus is the joined lifecycle of one consumer-issued
price request, as observed on chain.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| req_id | [string](#string) |  | uint256 as base-10 decimal string. |
| asset_id | [string](#string) |  | bytes32 asset id, 0x-prefixed lowercase hex. |
| status | [RequestStatus.Status](#indexer-v1-RequestStatus-Status) |  |  |
| requester | [string](#string) |  | 20-byte requester address, 0x-prefixed lowercase hex. |
| requested_tx_hash | [string](#string) |  | 32-byte tx hash of the PriceRequested tx, 0x-prefixed lowercase hex. |
| fulfilled_tx_hash | [string](#string) |  | 32-byte tx hash of the PriceFulfilled tx. Empty unless STATUS_FULFILLED. |
| fulfilled_price | [string](#string) |  | int256 price as base-10 decimal string in Chainlink&#39;s 8-decimal scale. Empty unless STATUS_FULFILLED. |
| requested_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | Block timestamp of the originating PriceRequested event. |
| fulfilled_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | Block timestamp of the fulfilling tx. Zero unless STATUS_FULFILLED. |






<a name="indexer-v1-StreamEventsRequest"></a>

### StreamEventsRequest
StreamEventsRequest selects which events to receive live.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| kinds | [EventKind](#indexer-v1-EventKind) | repeated | Empty = all kinds. Otherwise the union of the listed kinds. |
| asset_id | [string](#string) |  | Optional asset id filter (bytes32 hex). Same semantics as ListEventsRequest.asset_id. |
| from_block | [uint64](#uint64) |  | 0 = live-only. &gt; 0 = replay history from this block (inclusive) in chronological order, then continue live. |





 


<a name="indexer-v1-EventKind"></a>

### EventKind
EventKind enumerates the contract-emitted events the indexer
recognises. Used both as a filter on Stream/ListEvents and as the
discriminator that mirrors `Event.payload`.

| Name | Number | Description |
| ---- | ------ | ----------- |
| EVENT_KIND_UNSPECIFIED | 0 |  |
| EVENT_KIND_PRICE_REQUESTED | 1 |  |
| EVENT_KIND_PRICE_FULFILLED | 2 |  |
| EVENT_KIND_ASSET_REGISTERED | 3 |  |



<a name="indexer-v1-RequestStatus-Status"></a>

### RequestStatus.Status


| Name | Number | Description |
| ---- | ------ | ----------- |
| STATUS_UNSPECIFIED | 0 |  |
| STATUS_PENDING | 1 | PriceRequested observed; PriceFulfilled has not (yet) been observed past the confirmation threshold. |
| STATUS_FULFILLED | 2 | PriceFulfilled observed past the confirmation threshold. |
| STATUS_FAILED | 3 | The fulfilling tx reverted or the aggregator emitted a terminal failure event for this request. |


 

 


<a name="indexer-v1-IndexerService"></a>

### IndexerService
IndexerService is the chain-watching surface of the oracle.
It subscribes to PriceAggregator &#43; OracleRegistry events on the
single deployed chain, persists them past the configured
confirmation depth, exposes a paginated read-side, and serves a
long-lived event stream that oracle-service consumes as its
trigger source.

Single-chain only: every event is from our own deployed
contracts, so we type each one explicitly rather than carrying an
opaque payload.

| Method Name | Request Type | Response Type | Description |
| ----------- | ------------ | ------------- | ------------|
| ListEvents | [ListEventsRequest](#indexer-v1-ListEventsRequest) | [ListEventsResponse](#indexer-v1-ListEventsResponse) | ListEvents is a paginated historical query for the dashboard and the read-side. Filterable by event kind, asset, and block range. Sorted descending by (block_number, log_index). |
| StreamEvents | [StreamEventsRequest](#indexer-v1-StreamEventsRequest) | [Event](#indexer-v1-Event) stream | StreamEvents is a long-lived server stream. It emits events ONLY after they cross the indexer&#39;s confirmation threshold — consumers do not need to gate on `meta.confirmations`. When `from_block` is set the server replays history first (in chronological order), then continues live; when unset, the stream is live-only. |
| GetRequest | [GetRequestRequest](#indexer-v1-GetRequestRequest) | [RequestStatus](#indexer-v1-RequestStatus) | GetRequest returns the joined view of PriceRequested &#43; PriceFulfilled by req_id. Returns NOT_FOUND if the indexer has not observed a PriceRequested with that id yet. |

 



## Scalar Value Types

| .proto Type | Notes | C++ | Java | Python | Go | C# | PHP | Ruby |
| ----------- | ----- | --- | ---- | ------ | -- | -- | --- | ---- |
| <a name="double" /> double |  | double | double | float | float64 | double | float | Float |
| <a name="float" /> float |  | float | float | float | float32 | float | float | Float |
| <a name="int32" /> int32 | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. | int32 | int | int | int32 | int | integer | Bignum or Fixnum (as required) |
| <a name="int64" /> int64 | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. | int64 | long | int/long | int64 | long | integer/string | Bignum |
| <a name="uint32" /> uint32 | Uses variable-length encoding. | uint32 | int | int/long | uint32 | uint | integer | Bignum or Fixnum (as required) |
| <a name="uint64" /> uint64 | Uses variable-length encoding. | uint64 | long | int/long | uint64 | ulong | integer/string | Bignum or Fixnum (as required) |
| <a name="sint32" /> sint32 | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. | int32 | int | int | int32 | int | integer | Bignum or Fixnum (as required) |
| <a name="sint64" /> sint64 | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. | int64 | long | int/long | int64 | long | integer/string | Bignum |
| <a name="fixed32" /> fixed32 | Always four bytes. More efficient than uint32 if values are often greater than 2^28. | uint32 | int | int | uint32 | uint | integer | Bignum or Fixnum (as required) |
| <a name="fixed64" /> fixed64 | Always eight bytes. More efficient than uint64 if values are often greater than 2^56. | uint64 | long | int/long | uint64 | ulong | integer/string | Bignum |
| <a name="sfixed32" /> sfixed32 | Always four bytes. | int32 | int | int | int32 | int | integer | Bignum or Fixnum (as required) |
| <a name="sfixed64" /> sfixed64 | Always eight bytes. | int64 | long | int/long | int64 | long | integer/string | Bignum |
| <a name="bool" /> bool |  | bool | boolean | boolean | bool | bool | boolean | TrueClass/FalseClass |
| <a name="string" /> string | A string must always contain UTF-8 encoded or 7-bit ASCII text. | string | String | str/unicode | string | string | string | String (UTF-8) |
| <a name="bytes" /> bytes | May contain any arbitrary sequence of bytes. | string | ByteString | str | []byte | ByteString | string | String (ASCII-8BIT) |

