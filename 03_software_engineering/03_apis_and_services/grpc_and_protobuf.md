---
layer: 03_software_engineering
type: engineering
tool: gRPC
status: growing
tags: [grpc, protobuf, rpc, streaming, microservices, code-generation]
created: 2026-03-05
---

# gRPC and Protocol Buffers

## Purpose

gRPC is an open-source RPC framework developed by Google, built on HTTP/2 and Protocol Buffers (protobuf). It provides a strongly-typed, high-performance, bidirectional-streaming-capable communication layer primarily suited for internal microservice communication, latency-sensitive workloads, and any scenario where network efficiency and strong schema contracts matter more than human readability.

Protocol Buffers (protobuf) is the interface definition language (IDL) and binary serialization format used by gRPC. A `.proto` file defines messages (data structures) and services (RPC methods); code generators produce type-safe client and server stubs in any supported language.

## Architecture

### Protocol Buffers: `.proto` Syntax

**Scalar types**:

| proto3 type | Default | Go | Python | Java |
|-------------|---------|-----|--------|------|
| `int32` | 0 | `int32` | `int` | `int` |
| `int64` | 0 | `int64` | `int` | `long` |
| `float` | 0.0 | `float32` | `float` | `float` |
| `double` | 0.0 | `float64` | `float` | `double` |
| `bool` | false | `bool` | `bool` | `boolean` |
| `string` | `""` | `string` | `str` | `String` |
| `bytes` | `b""` | `[]byte` | `bytes` | `ByteString` |

**Messages and enums**:
```protobuf
syntax = "proto3";
package user.v1;
option go_package = "github.com/example/user/v1;userv1";

import "google/protobuf/timestamp.proto";

enum UserRole {
  USER_ROLE_UNSPECIFIED = 0;  // Always define 0 as UNSPECIFIED for forward compat
  USER_ROLE_VIEWER = 1;
  USER_ROLE_EDITOR = 2;
  USER_ROLE_ADMIN = 3;
}

message User {
  int64 id = 1;
  string email = 2;
  string name = 3;
  UserRole role = 4;
  google.protobuf.Timestamp created_at = 5;
  repeated string tags = 6;          // repeated = list/array
  map<string, string> metadata = 7;  // map type
}

message Address {
  oneof location {                   // oneof: exactly one field set
    string street_address = 1;
    string po_box = 2;
  }
  string city = 3;
  string country_code = 4;
}
```

**Field numbers** are part of the wire format — never reuse or delete them (add reserved statements instead):
```protobuf
message OldMessage {
  reserved 3, 5;           // reserved field numbers
  reserved "old_field";    // reserved field names
}
```

**Well-known types** (import from `google/protobuf/`):
- `Timestamp`: Unix epoch seconds + nanoseconds
- `Duration`: seconds + nanoseconds
- `Empty`: no-op request/response
- `FieldMask`: partial update specification
- `Any`: typed serialized message (like `interface{}`)
- `Struct`: dynamic JSON-like structure

### Service Definitions

```protobuf
service UserService {
  // Unary RPC
  rpc GetUser (GetUserRequest) returns (GetUserResponse);

  // Server streaming RPC
  rpc ListUsers (ListUsersRequest) returns (stream User);

  // Client streaming RPC
  rpc BatchCreateUsers (stream UserCreate) returns (BatchCreateResponse);

  // Bidirectional streaming RPC
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}

message GetUserRequest  { int64 user_id = 1; }
message GetUserResponse { User user = 1;     }

message ListUsersRequest {
  int32 page_size = 1;
  string page_token = 2;
  string filter = 3;
}

message BatchCreateResponse {
  repeated User created_users = 1;
  repeated string failed_emails = 2;
}
```

### gRPC Communication Patterns

**Unary RPC** — one request, one response (standard request-response):
```
Client → Request → Server
Client ← Response ← Server
```
Use when: CRUD operations, command execution, queries where response fits in memory.

**Server streaming RPC** — one request, stream of responses:
```
Client → Request → Server
Client ← Response 1 ←
Client ← Response 2 ←
Client ← Response N ←  (stream ends)
```
Use when: large result sets (paginated without extra round trips), real-time updates subscription, file downloads, log tailing.

**Client streaming RPC** — stream of requests, one response:
```
Client → Request 1 →
Client → Request 2 →
Client → Request N →  (stream ends)
Client ← Response ← Server
```
Use when: bulk uploads, batch processing where all data is sent before processing, large file uploads.

**Bidirectional streaming RPC** — two independent streams:
```
Client ← → Server  (both send independently)
```
Use when: chat applications, collaborative editing, real-time telemetry with feedback, game state synchronization.

## Implementation Notes

### Code Generation

Install tools:
```bash
pip install grpcio grpcio-tools  # Python
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest  # Go
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

Generate code:
```bash
# Python
python -m grpc_tools.protoc \
  -I proto \
  --python_out=src/generated \
  --grpc_python_out=src/generated \
  proto/user/v1/user.proto

# Go
protoc \
  --go_out=. --go_opt=paths=source_relative \
  --go-grpc_out=. --go-grpc_opt=paths=source_relative \
  -I proto proto/user/v1/user.proto
```

**Buf** is the modern alternative to raw `protoc`:
```yaml
# buf.gen.yaml
version: v1
plugins:
  - plugin: go
    out: gen/go
    opt: paths=source_relative
  - plugin: go-grpc
    out: gen/go
    opt: paths=source_relative
```
`buf generate` handles imports, linting, breaking change detection.

### Python Server Example

```python
import grpc
from concurrent import futures
from generated import user_pb2, user_pb2_grpc

class UserServicer(user_pb2_grpc.UserServiceServicer):
    async def GetUser(self, request, context):
        user = await user_repository.get(request.user_id)
        if not user:
            await context.abort(grpc.StatusCode.NOT_FOUND, f"User {request.user_id} not found")
        return user_pb2.GetUserResponse(user=to_proto(user))

    async def ListUsers(self, request, context):
        async for user in user_repository.stream_all(page_size=request.page_size):
            yield to_proto(user)  # yield each item into the stream

async def serve():
    server = grpc.aio.server(
        options=[
            ("grpc.max_send_message_length", 50 * 1024 * 1024),   # 50 MB
            ("grpc.max_receive_message_length", 50 * 1024 * 1024),
        ]
    )
    user_pb2_grpc.add_UserServiceServicer_to_server(UserServicer(), server)
    server.add_insecure_port("[::]:50051")
    await server.start()
    await server.wait_for_termination()
```

### Python Client Example

```python
import grpc
from generated import user_pb2, user_pb2_grpc

async def main():
    async with grpc.aio.insecure_channel("localhost:50051") as channel:
        stub = user_pb2_grpc.UserServiceStub(channel)

        # Unary
        response = await stub.GetUser(user_pb2.GetUserRequest(user_id=42))
        print(response.user.email)

        # Server streaming
        async for user in stub.ListUsers(user_pb2.ListUsersRequest(page_size=100)):
            print(user.name)
```

### Interceptors

Interceptors are gRPC's middleware equivalent — they wrap each RPC call for cross-cutting concerns.

```python
class AuthInterceptor(grpc.aio.ServerInterceptor):
    async def intercept_service(self, continuation, handler_call_details):
        metadata = dict(handler_call_details.invocation_metadata)
        token = metadata.get("authorization", "").removeprefix("Bearer ")
        if not verify_token(token):
            async def abort(request, context):
                await context.abort(grpc.StatusCode.UNAUTHENTICATED, "Invalid token")
            return grpc.unary_unary_rpc_method_handler(abort)
        return await continuation(handler_call_details)

class LoggingInterceptor(grpc.aio.ServerInterceptor):
    async def intercept_service(self, continuation, handler_call_details):
        start = time.perf_counter()
        handler = await continuation(handler_call_details)
        # wrap handler to log after completion ...
        return handler
```

### Status Codes and Error Handling

gRPC defines its own status codes (not HTTP):

| Code | Name | HTTP equiv | When to use |
|------|------|------------|-------------|
| 0 | OK | 200 | Success |
| 1 | CANCELLED | - | Client cancelled |
| 2 | UNKNOWN | 500 | Unknown error |
| 3 | INVALID_ARGUMENT | 400 | Bad input |
| 4 | DEADLINE_EXCEEDED | 504 | Timeout |
| 5 | NOT_FOUND | 404 | Resource missing |
| 6 | ALREADY_EXISTS | 409 | Duplicate |
| 7 | PERMISSION_DENIED | 403 | Not authorized |
| 14 | UNAVAILABLE | 503 | Service down, retry |
| 16 | UNAUTHENTICATED | 401 | No/invalid credentials |

Rich error details: use `google.rpc.Status` with `google.rpc.BadRequest`, `google.rpc.ErrorInfo` for structured errors beyond a string message.

### TLS and Authentication

```python
# Server with TLS
server_credentials = grpc.ssl_server_credentials(
    [(open("server.key", "rb").read(), open("server.crt", "rb").read())]
)
server.add_secure_port("[::]:443", server_credentials)

# Client with TLS
channel_credentials = grpc.ssl_channel_credentials(
    root_certificates=open("ca.crt", "rb").read()
)
channel = grpc.secure_channel("api.example.com:443", channel_credentials)

# mTLS (mutual TLS): pass client cert to ssl_channel_credentials
```

Token-based auth: pass as metadata (equivalent to HTTP headers):
```python
metadata = [("authorization", f"Bearer {token}")]
response = await stub.GetUser(request, metadata=metadata)
```

### gRPC-Web and Transcoding

**gRPC-Web**: allows browser clients to call gRPC services via a proxy (Envoy or grpc-web proxy). Uses HTTP/1.1 with a special encoding layer. Does not support client or bidirectional streaming.

**gRPC transcoding**: Envoy or grpc-gateway can expose a REST/JSON API that proxies to a gRPC backend. Define HTTP rules in `.proto` annotations:
```protobuf
import "google/api/annotations.proto";

service UserService {
  rpc GetUser (GetUserRequest) returns (GetUserResponse) {
    option (google.api.http) = {
      get: "/v1/users/{user_id}"
    };
  }
}
```

## Trade-offs

| Dimension | gRPC | REST/JSON |
|-----------|------|-----------|
| Payload size | Compact binary (protobuf) | Verbose text (JSON) |
| Performance | ~5–10× faster serialization | Slower for high-frequency messages |
| Human readability | Binary, needs tooling to inspect | Human-readable, curl-friendly |
| Browser support | Limited (grpc-web proxy needed) | Native |
| Schema evolution | Strictly versioned, backward compat rules | Flexible but fragile |
| Streaming | First-class (4 patterns) | Server-sent events / WebSockets (separate) |
| Tooling complexity | Code gen pipeline, protoc/buf | Simple, ubiquitous |
| Ecosystem | Growing; strong in Go, Java, C++ | Ubiquitous across all languages |
| Error model | Rich typed status codes | HTTP status codes + ad-hoc JSON |
| Learning curve | Steeper (IDL, codegen, proto rules) | Gentle |

**Use gRPC when**:
- Low-latency internal microservice-to-microservice communication
- Streaming is a requirement (real-time data, large transfers)
- Strong schema contracts and breaking-change detection matter
- Polyglot services benefit from generated clients in multiple languages
- Network bandwidth is constrained (mobile, IoT, high-frequency trading)

**Prefer REST when**:
- Public-facing APIs consumed by browsers or third parties
- Team is unfamiliar with protobuf tooling and the overhead is not justified
- Debugging simplicity is critical (curl, browser DevTools)
- Rapid iteration where schema changes are frequent and backward compat is not enforced

## References

- Protocol Buffers Language Guide: https://protobuf.dev/programming-guides/proto3/
- gRPC documentation: https://grpc.io/docs/
- Buf toolchain: https://buf.build/docs/
- Google API design guide: https://cloud.google.com/apis/design
- gRPC status codes: https://grpc.github.io/grpc/core/md_doc_statuscodes.html

## Links
-
