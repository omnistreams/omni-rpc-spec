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
   omnistreams Multiplexer.sendControlMessage.
4. Streaming RPC requests (either stream parameter, stream response, or both)
   are handled using Multiplexer.sendControlMessage and
   Multiplexer.createConduit. There are 3 cases:

   1. Single param, stream response: Multiplexer.sendControlMessage for request,
      Multiplexer.createConduit for response.
   2. Stream param, single response: Multiplexer.createConduit for request,
      Multiplexer.sendControlMessage for reponse.
   3. Stream param, stream response: Multiplexer.createConduit for request and
      response.

# Differences from other RPC systems

There are several excellent RPC systems in the wild, in particular
[gRPC](https://grpc.io/), 
[Cap'n Proto](https://capnproto.org/rpc.html), and 
[Apache Thrift](https://thrift.apache.org/).


## No IDL
Most RPC systems use an IDL such os
[Protocol Buffers](https://developers.google.com/protocol-buffers/) to
specify a strict interface between endpoints. This helps ensure proper use
of endpoints, and also has the huge benefit of enabling code generation so
you get function type signatures that match the IDL spec. The
downside is that it requires a build step, and adds a fairly complex
dependency. omni-rpc does not require a specific IDL. All parameters are simply
encoded in the message format, which by default is JSON. Although use of and
IDL for encoding messages is encouraged with omni-rpc, it is not required.
