# Introduction

omni-rpc is a barebones specification for doing RPC on top of
[omnistreams](https://github.com/omnistreams/omnistreams-spec). The default
usage is essentially just [JSON-RPC](https://www.jsonrpc.org/specification),
with added semantics for handling streaming data. However, this specification
does not prescribe a particular encoding format, so it could easily be
adapted for use with protocol buffers, flatbuffers, capnproto, etc, or even
a custom encoding. As with omnistreams in general, the underlying transport
is also not specified, though WebSockets is a good default. Both the transport
and encoding format should be indicated in the API documentation of an
omni-rpc endpoint.

# Overview

The spec can basically be summarized as follows:

1. A peer opens an omnistreams mux listener.
2. Another peer establishes a mux connection to the listener.
3. Normal RPC requests (single parameter, single response) are handled using
   omnistreams Multiplexer.sendMessage.
4. Streaming RPC requests (either stream parameter, stream response, or both)
   are handled using Multiplexer.sendMessage and
   Multiplexer.openConduit. There are 3 cases:

   1. Single param, stream response: Multiplexer.sendMessage for request,
      Multiplexer.openConduit for response.
   2. Stream param, single response: Multiplexer.openConduit for request,
      Multiplexer.sendMessage for reponse.
   3. Stream param, stream response: Multiplexer.openConduit for request and
      response.
      
The peers are responsible for keeping track of the requests (for example
using JSON-RPC request.id), and making sure the responses are matched
up.

# Differences from other RPC systems

There are several excellent RPC systems in the wild, in particular
[gRPC](https://grpc.io/), 
[Cap'n Proto](https://capnproto.org/rpc.html), and 
[Apache Thrift](https://thrift.apache.org/).


## No IDL
Most RPC systems I've seen use an IDL such as
[Protocol Buffers](https://developers.google.com/protocol-buffers/) to
specify a strict interface between endpoints. This helps ensure proper use
of endpoints, and also has the huge benefit of enabling code generation so
you get function type signatures that match the IDL spec. The
downside is that it requires a build step, and adds a fairly complex
dependency. omni-rpc does not require a specific IDL. All parameters are simply
encoded in the message format, which by default is JSON. Although use of an
IDL for encoding messages is encouraged with omni-rpc, it is not required.

## Web-first
Most web APIs use HTTP, and most RPC systems are designed primarily for
backend systems, microservices, etc. Althouth gRPC is working on a
[browser-compatible version](https://github.com/improbable-eng/grpc-web), 
gRPC was not originally designed to work this way (it's built for HTTP/2). grpc-web
required a lot of complex machinery (relays, etc) to work last time I checked.

omni-rpc is intended to be used in the browser, and implementators are
encouraged to provide WebSockets compatibility with all implementations.
