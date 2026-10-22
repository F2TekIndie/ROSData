cmake_minimum_required(VERSION 3.16)
project(ROSData LANGUAGES CXX)

find_package(Protobuf REQUIRED MODULE)

add_library(rosdata STATIC
	message.proto
	topicsRequest.proto
	subscribeRequest.proto
	unsubscribeRequest.proto
	dataResponse.proto
	exitRequest.proto
)

protobuf_generate(
    TARGET rosdata
    LANGUAGE cpp
)

# Generated headers are written into this library's build directory.
target_include_directories(rosdata PUBLIC
    "${CMAKE_CURRENT_BINARY_DIR}"
)

target_link_libraries(rosdata PUBLIC
    protobuf::libprotobuf
)

