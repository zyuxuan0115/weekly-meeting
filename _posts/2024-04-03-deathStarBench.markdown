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

#### The processor in apache thrift


- Once a client does connect, we need to process RPC calls made over the connection. The inner while loop uses the processor `process()` method to process client RPC requests 6. The `process()` method takes care of everything required to process one RPC request. The `process()` method will read the client’s RPC message from the network, determine which handler method to call, unpack the parameters using the I/O stack, and call the handler. When the handler returns with a result, the processor serializes the result back to the client using the I/O stack. The inner loop breaks when the client disconnects, which causes the `process()` method to return 0 6.

- Note that the TProcessor `process()` method takes three parameters, `proc.process (proto, proto, and nullptr)`. In this example, we pass the protocol twice. The processor uses the first parameter for reading and the second for writing. We’ll look at in/out protocol stacks in detail in section 10.4, “Using factories.” The third parameter to the `process()` method is the Processor context, which we’re not supporting in our simple server. This parameter works in conjunction with an optional TProcessorEventHandler. Processor event handlers allow us to hook processor events (pre/post I/O stack read/write operations) without hacking the Apache Thrift source code. You can find a processor event handler example in chapter 8, “Implementing services.”


#### How does one function call another function
- For example [UniqueIDService](https://github.com/delimitrou/DeathStarBench/blob/master/socialNetwork/gen-cpp/UniqueIdService.cpp#L291)

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

- the RPC in [DeathStarBench](https://github.com/delimitrou/DeathStarBench/blob/master/socialNetwork/gen-cpp/UniqueIdService.h#L224)
- If we want to convert C++ code to C, we need some good JSON parser in C
	+ the [json parser library](https://www.json.org/json-en.html) in C
	+ <strong>another useful json parser</strong> [jsmn](https://github.com/zserge/jsmn/tree/master)
	  * [example code of jsondump](https://github.com/zserge/jsmn/blob/master/example/jsondump.c)
- Stateful serverless functions;
	+ disable all the writes
