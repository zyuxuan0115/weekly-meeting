---
layout: post
title:  "2024-04-03 DeathStarBench"
date:   2024-04-03 1:53:49 -0500
categories: serverless functions
---
### Beldi's serverless functions from DeathStarBench
- paper is [here](https://www.usenix.org/conference/osdi20/presentation/zhang-haoran)
- Beldi's modified DeathStarBench is [here](https://github.com/eniac/Beldi/tree/master/internal/media/core)
- Question: for example `MovieId` contains write (store the data to the database)
  + how do we handle these? 
	+ suggestion: allow data inconsistency.
- Question: which functions are going to be written in Rust?

### About DeathStarBench 
#### Use thrift compiler to generate code
- I'm just curious about how to use Apache thrift Compiler
	* apache thrift will automatically generate server template for your service
	* then you can decide the 
	* [here](https://github.com/apache/thrift/tree/master/tutorial/cpp) is an example
  * I also have an example [here](https://github.com/zyuxuan0115/faas-cpp-test/tree/main/apache_thrift_example)

#### The overall structure

- When you generate C++ code for an Apache Thrift service, the IDL compiler creates
  + a client
  + a processor
  + an interface (serviceIF) for the service. 
- It also creates a `TProcessorFactory` subclass and an interface factory for the service. 
	+ The processor factory is implemented, but the interface factory is abstract and must be implemented by the user.
- Multiplexed processors and multiplexed protocols allow a single server to host multiple services

#### The server

- The derive classes of `TServer`

![t0](/assets/2024-04-07/t0.png)

- Detail about different derive classes

![t1](/assets/2024-04-07/t1.png)

- The `main()` function in this example implements the server behavior. <strong>The IDL compiler-generated Processor class takes care of reading network requests from clients and dispatching calls to the correct handler method.</strong> In the example, the Processor class, MessageProcessor, is constructed with an instance of the MessageHandler service. As always, the C++ framework wants all objects wrapped in a `shared_ptr`. 

#### The processor in apache thrift - handle incoming request

From [this book](https://drive.google.com/file/d/11ZcqoKKuy_eCtQEUJvDP6FMH1t5e3J1a), I know

- Once a client does connect, we need to process RPC calls made over the connection. The inner while loop uses the processor `process()` method to process client RPC requests. The `process()` method takes care of everything required to process one RPC request. The `process()` method will read the client’s RPC message from the network, determine which handler method to call, unpack the parameters using the I/O stack, and call the handler. When the handler returns with a result, the processor serializes the result back to the client using the I/O stack. The inner loop breaks when the client disconnects, which causes the `process()` method to return 0.

- Note that the TProcessor `process()` method takes three parameters, `proc.process (proto, proto, and nullptr)`. In this example, we pass the protocol twice. The processor uses the first parameter for reading and the second for writing. We’ll look at in/out protocol stacks in detail in section 10.4, “Using factories.” The third parameter to the `process()` method is the Processor context, which we’re not supporting in our simple server. This parameter works in conjunction with an optional TProcessorEventHandler. Processor event handlers allow us to hook processor events (pre/post I/O stack read/write operations) without hacking the Apache Thrift source code. 


#### The client in apache thrift - class that allow other service to connect to the server
- For example, in [UniqueIDService](https://github.com/delimitrou/DeathStarBench/blob/master/socialNetwork/gen-cpp/UniqueIdService.cpp#L291), they have:

```cpp
void UniqueIdServiceClient::send_ComposeUniqueId(const int64_t req_id, const PostType::type post_type, const std::map<std::string, std::string> & carrier)
{
  int32_t cseqid = 0;
  oprot_->writeMessageBegin("ComposeUniqueId", ::apache::thrift::protocol::T_CALL, cseqid);

  UniqueIdService_ComposeUniqueId_pargs args;
  args.req_id = &req_id;
  args.post_type = &post_type;
  args.carrier = &carrier;
  args.write(oprot_);

  oprot_->writeMessageEnd();
  oprot_->getTransport()->writeEnd();
  oprot_->getTransport()->flush();
}
```

- I think client is the client code that teach you how to connect to the server.
	+ the other microservice can use your client code to send request to your server
	+ like `UniqueIdServiceClient` connected to `UniqueIdServiceServer`
	+ but this client won't be used in `UniqueIdService` but in `ComposePostService`


- the RPC in [DeathStarBench](https://github.com/delimitrou/DeathStarBench/blob/master/socialNetwork/gen-cpp/UniqueIdService.h#L224)
- If we want to convert C++ code to C, we need some good JSON parser in C
	+ the [json parser library](https://www.json.org/json-en.html) in C
	+ <strong>another useful json parser</strong> [jsmn](https://github.com/zserge/jsmn/tree/master)
	  * [example code of jsondump](https://github.com/zserge/jsmn/blob/master/example/jsondump.c)
- Stateful serverless functions;
	+ disable all the writes
