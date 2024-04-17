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

![t3](/assets/2024-04-07/t3.png)

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

#### the RPC in DeathStarBench
- in one service, run client of another service.  
	+ For example, in [ComposePostService](https://github.com/delimitrou/DeathStarBench/blob/master/socialNetwork/src/ComposePostService/ComposePostHandler.h#L374)

```c++
  auto text_future =
      std::async(std::launch::async, &ComposePostHandler::_ComposeTextHelper,
                 this, req_id, text, writer_text_map);
  auto creator_future =
      std::async(std::launch::async, &ComposePostHandler::_ComposeCreaterHelper,
                 this, req_id, user_id, username, writer_text_map);
  auto media_future =
      std::async(std::launch::async, &ComposePostHandler::_ComposeMediaHelper,
                 this, req_id, media_types, media_ids, writer_text_map);
  auto unique_id_future = std::async(
      std::launch::async, &ComposePostHandler::_ComposeUniqueIdHelper, this,
      req_id, post_type, writer_text_map);

  Post post;
  auto timestamp =
      duration_cast<milliseconds>(system_clock::now().time_since_epoch())
          .count();
  post.timestamp = timestamp;

  post.post_id = unique_id_future.get();
  post.creator = creator_future.get();
  post.media = media_future.get();
  auto text_return = text_future.get();
  post.text = text_return.text;
  post.urls = text_return.urls;
  post.user_mentions = text_return.user_mentions;
  post.req_id = req_id;
  post.post_type = post_type;
```

#### Make OpenFaaS connect to MongoDB
- the post about how to do this is [here](https://www.openfaas.com/blog/get-started-with-python-mongo/)
- Manual: [mongodb sync client api in rust](https://docs.rs/mongodb/latest/mongodb/index.html#using-the-sync-api)
- [rust mongodb example from github](https://github.com/mehmetsefabalik/rust-mongodb-example/tree/master)
- [mongodb C client api](https://mongoc.org/libmongoc/current/mongoc_client_get_collection.html)
- JSON for rust - [serde](https://serde.rs/data-model.html)
- [how to send query to mongodb server from rust program](https://www.mongodb.com/docs/drivers/rust/current/fundamentals/crud/read-operations/query/)

#### Make OpenFaaS connect to Memcached
- [Deploying Memcached on Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/tutorials/deploying-memcached-on-kubernetes-engine)

```bash
HELM_VERSION=3.14.3
wget https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz
```

#### Make OpenFaaS connect to Redis
- tutorial from [github](https://gist.github.com/alexellis/e05a7b573ae22b209f0214d5766ff07e)
  + doesn't work
- Install Redis to Kubernetes using [helm](https://bitnami.com/stack/redis/helm)
- [Using Rust to run Redis client](https://redis.io/lp/redis-enterprise-rust/) 

#### Convert DeathStarBench to Rust serverless functions 

![t2](/assets/2024-04-07/t2.png)

- <strong>UniqueIdService</strong>

```bash
curl 127.0.0.1:8080/function/unique-id-service -d ""
```

- <strong>UrlShortenService</strong>

```bash
curl 127.0.0.1:8080/function/url-shorten-service -d "[\"http://google.com\",\"http://kate0115.net\"]"
```

- <strong>UserMentionService</strong>

```bash
curl 127.0.0.1:8080/function/user-mention-service -d "[\"Alice\",\"Bob\"]"
```

- <strong>TextService</strong>

```bash
curl 127.0.0.1:8080/function/text-service -d "Hey, this is @Yuxuan! Nice to meet you and welcome to my personal web: https://kate0115.net @tomwenisch "
```

- <strong>MediaService</strong>

```bash
curl 127.0.0.1:8080/function/media-service -d "{\"media_id\":[111,222],\"media_type\":[\"png\",\"jpg\"]}"
```

- <strong>RegisterUserWithId</strong>

```bash
curl 127.0.0.1:8080/function/register-user-with-id -d "{\"first_name\":\"Tom\",\"last_name\":\"Wenisch\",\"username\":\"twenisch\",\"password\":\"umichandgoogle\",\"user_id\":11028}"
curl 127.0.0.1:8080/function/register-user-with-id -d "{\"first_name\":\"Joe\",\"last_name\":\"Devietti\",\"username\":\"devietti\",\"password\":\"uwandupenn\",\"user_id\":11029}"
```

- <strong>RegisterUser</strong>

```bash
curl 127.0.0.1:8080/function/register-user -d "{\"first_name\":\"Yuxuan\",\"last_name\":\"Zhang\",\"username\":\"zyuxuan\",\"password\":\"umichandupenn\"}"
```

- <strong>ComposeCreatorWithUsername</strong>

```bash
curl 127.0.0.1:8080/function/compose-creator-with-username -d "zyuxuan"
```

- <strong>ComposeCreatorWithUserId</strong>

```bash
curl 127.0.0.1:8080/function/compose-creator-with-userid -d "{\"user_id\":11028,\"username\":\"twenisch\"}"
```

- <strong>GetUserId</strong>

```bash
curl 127.0.0.1:8080/function/get-user-id -d "zyuxuan"
```

- <strong>UserLogin</strong>

```bash
curl 127.0.0.1:8080/function/user-login -d "{\"username\":\"zyuxuan\",\"password\":\"umichandupenn\",\"secret\":\"idon'tknowwhatshouldbesecret\"}"
```

- <strong>SocialGraphInsertUser<strong>

```bash
curl 127.0.0.1:8080/function/social-graph-insert-user -d "11028"
```

- <strong>SocialGraphFollow<strong>
  + I didn't check the replica of redis, but the DeathStarBench checked.

```bash
curl 127.0.0.1:8080/function/social-graph-follow -d "{\"user_id\":11028,\"followee_id\":11029}"
```

- <strong>SocialGraphUnfollow</strong>
  + I didn't check the replica of redis, but the DeathStarBench checked.

```bash
curl 127.0.0.1:8080/function/social-graph-unfollow -d "{\"user_id\":11028,\"followee_id\":11029}"
```

- <strong>SocialGraphFollowWithUsername</strong>

```bash
curl 127.0.0.1:8080/function/social-graph-follow-with-username -d "{\"user_name\":\"twenisch\",\"followee_name\":\"devietti\"}"
```

- <strong>SocialGraphUnfollowWithUsername</strong>

```bash
curl 127.0.0.1:8080/function/social-graph-unfollow-with-username -d "{\"user_name\":\"twenisch\",\"followee_name\":\"devietti\"}"
```

- <strong>SocialGraphGetFollowers</strong>

```bash
curl 127.0.0.1:8080/function/social-graph-get-followers -d "???"
```

- <strong>SocialGraphGetFollowees</strong>

```bash
curl 127.0.0.1:8080/function/social-graph-get-followees -d "???"
```



#### can rust run multithread from another function?
- [stackoverflow](https://stackoverflow.com/questions/33938547/cannot-call-a-function-in-a-spawned-thread-because-it-does-not-fulfill-the-requ)

### JSON parser
- If we want to convert C++ code to C, we need some good JSON parser in C
	+ the [json parser library](https://www.json.org/json-en.html) in C
	+ <strong>another useful json parser</strong> [jsmn](https://github.com/zserge/jsmn/tree/master)
	  * [example code of jsondump](https://github.com/zserge/jsmn/blob/master/example/jsondump.c)
- Stateful serverless functions;
	+ disable all the writes
