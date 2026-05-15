# Protocol Documentation
<a name="top"></a>

## Table of Contents

- [user/v1/user.proto](#user_v1_user-proto)
    - [CreateUserRequest](#user-v1-CreateUserRequest)
    - [GetUserRequest](#user-v1-GetUserRequest)
    - [User](#user-v1-User)
  
    - [UserService](#user-v1-UserService)
  
- [common/v1/errors.proto](#common_v1_errors-proto)
    - [ErrorDetail](#common-v1-ErrorDetail)
    - [ErrorResponse](#common-v1-ErrorResponse)
  
- [common/v1/status.proto](#common_v1_status-proto)
    - [CommonStatus](#common-v1-CommonStatus)
  
- [common/v1/types.proto](#common_v1_types-proto)
    - [Address](#common-v1-Address)
    - [Money](#common-v1-Money)
    - [UUID](#common-v1-UUID)
  
- [common/v1/pagination.proto](#common_v1_pagination-proto)
    - [CursorPageRequest](#common-v1-CursorPageRequest)
    - [CursorPageResponse](#common-v1-CursorPageResponse)
    - [PageRequest](#common-v1-PageRequest)
    - [PageResponse](#common-v1-PageResponse)
  
- [Scalar Value Types](#scalar-value-types)



<a name="user_v1_user-proto"></a>
<p align="right"><a href="#top">Top</a></p>

## user/v1/user.proto



<a name="user-v1-CreateUserRequest"></a>

### CreateUserRequest
CreateUserRequest is the request message for CreateUser.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| email | [string](#string) |  | User&#39;s email address (required). |
| name | [string](#string) |  | User&#39;s display name (required). |






<a name="user-v1-GetUserRequest"></a>

### GetUserRequest
GetUserRequest is the request message for GetUser.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| id | [common.v1.UUID](#common-v1-UUID) |  | ID of the user to retrieve (UUID, 16 bytes). |






<a name="user-v1-User"></a>

### User
User represents a user account in the system.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| id | [common.v1.UUID](#common-v1-UUID) |  | Unique user identifier (UUID, 16 bytes). |
| email | [string](#string) |  | User&#39;s email address (must be unique). |
| name | [string](#string) |  | User&#39;s display name. |
| status | [common.v1.CommonStatus](#common-v1-CommonStatus) |  | User&#39;s account status. |
| created_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | Timestamp when the user was created. |
| updated_at | [google.protobuf.Timestamp](#google-protobuf-Timestamp) |  | Timestamp when the user was last updated. |





 

 

 


<a name="user-v1-UserService"></a>

### UserService
UserService provides operations for managing user accounts.

| Method Name | Request Type | Response Type | Description |
| ----------- | ------------ | ------------- | ------------|
| GetUser | [GetUserRequest](#user-v1-GetUserRequest) | [User](#user-v1-User) | GetUser retrieves a user by ID. |
| CreateUser | [CreateUserRequest](#user-v1-CreateUserRequest) | [User](#user-v1-User) | CreateUser creates a new user. |

 



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





 

 

 

 



<a name="common_v1_status-proto"></a>
<p align="right"><a href="#top">Top</a></p>

## common/v1/status.proto


 


<a name="common-v1-CommonStatus"></a>

### CommonStatus
CommonStatus represents standard entity states across all services.
Use this enum for consistent status representation.

| Name | Number | Description |
| ---- | ------ | ----------- |
| COMMON_STATUS_UNSPECIFIED | 0 | Default unspecified value (required by Buf linting). |
| COMMON_STATUS_ACTIVE | 1 | Entity is active and available for use. |
| COMMON_STATUS_INACTIVE | 2 | Entity is temporarily inactive but not deleted. |
| COMMON_STATUS_DELETED | 3 | Entity is soft-deleted and should not be displayed. |


 

 

 



<a name="common_v1_types-proto"></a>
<p align="right"><a href="#top">Top</a></p>

## common/v1/types.proto



<a name="common-v1-Address"></a>

### Address
Address represents a physical mailing address.
Follows international address format standards.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| street | [string](#string) |  | Street address line 1 (e.g., &#34;123 Main St&#34;). |
| city | [string](#string) |  | City or locality name. |
| state | [string](#string) |  | State, province, or region. |
| postal_code | [string](#string) |  | Postal or ZIP code. |
| country | [string](#string) |  | ISO 3166-1 alpha-2 country code (e.g., US, GB, DE). |






<a name="common-v1-Money"></a>

### Money
Money represents a monetary amount with currency.
Uses integer representation to avoid floating-point precision issues.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| amount | [int64](#int64) |  | Amount in the smallest currency unit (e.g., cents for USD). For USD, $10.50 would be represented as 1050. |
| currency | [string](#string) |  | ISO 4217 currency code (e.g., USD, EUR, GBP). |






<a name="common-v1-UUID"></a>

### UUID
UUID represents a universally unique identifier.
Stored as 16 bytes in binary format for efficiency.


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| value | [bytes](#bytes) |  | UUID value as 16 bytes. Use this instead of string representation to save space. |





 

 

 

 



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

