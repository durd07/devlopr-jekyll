---
title: Grpc
---

# gRPC

## gRPC lets you define four kinds of service method
1. **Unary RPCs** where the client sends a single request to the server and gets a single response back, just like a normal function call.  
    ```
    rpc SayHello(HelloRequest) returns (HelloResponse);
    ```

2. **Server streaming RPCs** where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. gRPC guarantees message ordering within an individual RPC call.  
    ```
    rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
    ```

3. **Client streaming RPCs** where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them and return its response. Again gRPC guarantees message ordering within an individual RPC call.  
    ```
    rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
    ```

4. **Bidirectional streaming RPCs** where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved.
    ````
    rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
    ````

## gRPC tips
- Synchronous vs. asynchronous
- [RPC life cycle](https://grpc.io/docs/what-is-grpc/core-concepts/#rpc-life-cycle)  
    **RPC termination In gRPC**, both the client and server make independent and local determinations of the success of the call, and their conclusions may not match. This means that, for example, you could have an RPC that finishes successfully on the server side (“I have sent all my responses!") but fails on the client side (“The responses arrived after my deadline!"). It’s also possible for a server to decide to complete before a client has sent all its requests.

## commands
```
protoc -I ../../protos --grpc_out=. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` route_guide.proto
protoc -I ../../protos --cpp_out=. route_guide.proto
```

# [protocal-buffers](https://developers.google.com/protocol-buffers/docs/proto3)
`.proto`
```
/* comments */
syntax = 'proto3';  // default `proto2`, must be first non-empty, non-comment line

import "myproject/other_protos.proto"

message SearchRequest {
    string query = 1;  // field number
    int32 page_number = 2;
    int32 result_per_page = 3;

    enum Corpus {
        UNIVERSAL = 0;
        WEB = 1;
    }

    Corpus corpus = 4;

    reserved 2, 15, 9 to 11;
    reserved "foo", "bar";
}
```

## to generate your classes
```
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```
- `IMPORT_PATH` specifies a directory in which to look for `.proto` files when resolving import directives. If omitted, the current directory is used. Multiple import directories can be specified by passing the `--proto_path` option multiple times; they will be searched in order. `-I=IMPORT_PATH` can be used as a short form of `--proto_path`.
- You can provide one or more output directives:
    - --cpp_out generates C++ code in DST_DIR. See the C++ generated code reference for more.
    - --java_out generates Java code in DST_DIR. See the Java generated code reference for more.
    - --python_out generates Python code in DST_DIR. See the Python generated code reference for more.
    - --go_out generates Go code in DST_DIR. See the Go generated code reference for more.
    - --ruby_out generates Ruby code in DST_DIR. Ruby generated code reference is coming soon!
    - --objc_out generates Objective-C code in DST_DIR. See the Objective-C generated code reference for more.
    - --csharp_out generates C# code in DST_DIR. See the C# generated code reference for more.
    - --php_out generates PHP code in DST_DIR. See the PHP generated code reference for more.

As an extra convenience, if the DST_DIR ends in .zip or .jar, the compiler will write the output to a single ZIP-format archive file with the given name. .jar outputs will also be given a manifest file as required by the Java JAR specification. Note that if the output archive already exists, it will be overwritten; the compiler is not smart enough to add files to an existing archive.
You must provide one or more .proto files as input. Multiple .proto files can be specified at once. Although the files are named relative to the current directory, each file must reside in one of the IMPORT_PATHs so that the compiler can determine its canonical name.
