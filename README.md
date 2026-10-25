# ROSData

**Status: proof-of-concept prototype.** This project is for experimentation and
learning. Features are incomplete, interfaces may change, and it is not
production-ready.

Shared Protocol Buffer schemas and generated C++ library for communication
between [ROSAdapter](../ROSAdapter/README.md) and [ROSVis](../ROSVis/README.md).
This project defines the communication contract; it does not open sockets or
provide a UI data model.

## Message definitions

All messages use the `robotics` package.

| File | Contents |
| --- | --- |
| `message.proto` | Envelope with `request_id` and a `oneof` payload. |
| `topicsRequest.proto` | Topic query, topic descriptions, and query response. |
| `subscribeRequest.proto` | Subscription request and response. |
| `unsubscribeRequest.proto` | Unsubscription request and response. |
| `dataResponse.proto` | Topic data with string or unsigned integer payloads. |
| `exitRequest.proto` | Adapter exit request. |

Schema definitions do not imply that every operation is implemented by the
applications. Topic queries and exit handling are currently implemented in
ROSAdapter; subscription and data-streaming paths remain incomplete.

## Dependencies and build

- CMake 3.16 or later and a C++ compiler compatible with your Protobuf version.
- Protocol Buffers compiler (`protoc`) and C++ development library.

Use a compatible compiler/runtime installation. Neither Qt nor ROS 2 is needed
to build this library by itself.

```sh
cmake -S . -B build/local -DCMAKE_BUILD_TYPE=Debug
cmake --build build/local --parallel
```

CMake generates `.pb.h` and `.pb.cc` files in the build directory and builds the
static target `rosdata`. Edit the `.proto` files rather than generated sources.

## Use from another project

For an application checked out alongside `ROSData`:

```cmake
add_subdirectory(
    "${CMAKE_CURRENT_SOURCE_DIR}/../ROSData"
    "${CMAKE_CURRENT_BINARY_DIR}/rosdata"
)

# Replace your_target with an existing executable or library target.
target_link_libraries(your_target PRIVATE rosdata)
```

The target exports the generated-header directory and its Protobuf runtime
dependency. Consumers can include the generated headers directly:

```cpp
#include "message.pb.h"

robotics::Message message;
message.mutable_topics_request();
```

## Wire format

ROSAdapter and ROSVis frame each serialized envelope as:

```text
[4-byte payload length, big-endian][serialized robotics::Message]
```

The applications limit payloads to 16 MiB. Framing is implemented in their
Asio transport code, not in this library. Binary messages are not newline-delimited.

## Schema evolution

The protocol is experimental and has no stable compatibility guarantee yet.
Rebuild both applications after schema changes. Preserve field numbers and
meanings when compatibility matters; reserve removed field numbers and names.
New payload types also require application dispatch handling. No installed
CMake package or automated test suite is currently provided.
