---
layout: post
title:  "2023-11-30 nightcore-workloads"
date:   2023-11-30 1:53:46 -0500
categories: serverless functions
---

### How clang merge serverless functions foo and bar
- in the example of [foo](https://github.com/zyuxuan0115/nightcore/blob/asplos-release/examples/c/foo.c#L35) calling [bar](https://github.com/zyuxuan0115/nightcore/blob/asplos-release/examples/c/bar.c#L36), we have the function invocation from `foo`

```cpp
int ret = context->invoke_func_fn(context->caller_context, 
                                  "Bar", input, input_length,
                                  &bar_output, &bar_output_length);
``` 

- and the return function from the callee function `bar`

```cpp
context->append_output_fn(context->caller_context,
                          output_buffer, input_length + sizeof(kOutputSuffix));
```

- this is how the function pointer of [faas_invoke_func_fn_t]() is defined

```cpp
typedef int (*faas_invoke_func_fn_t)(
    void* caller_context, const char* func_name,
    const char* input_data, size_t input_length,
    const char** output_data, size_t* output_length);
```

- replace the RPC with 

```cpp
int ret = callee (context->caller_context, input, input_length, &bar_output, &bar_output_length);
```

- also need to create a normal callee function

```cpp
int callee(void*, const char* input, size_t input_length, const char** output_data, size_t* output_length){
	// callee function
  *output_data = (char*)malloc(...);
  *output_size = input_length + sizeof(kOutputSuffix);
  return 0;
}
```

### From nightcore's C++ workloads
- I made both `movieReview` and `socialNetwork` work on cloudlab machines 
- I checked if the `movieReview` workload contains the invocation `invoke_func_fn` 
	+ only nightcore [FaasWorker.h](https://github.com/zyuxuan0115/nightcore-test/blob/main/moviereview_singlenode/DeathStarBench/mediaMicroservices/src/FaasWorker.h) contains [invoke_func_fn_](https://github.com/zyuxuan0115/nightcore-test/blob/main/moviereview_singlenode/DeathStarBench/mediaMicroservices/src/FaasWorker.h)

![s1](/assets/2023-11-29/S1.png)

- I also check `socialNetwork` workload and it shows the same result.

```cpp
class ClientTransport : public apache::thrift::transport::TVirtualTransport<ClientTransport> {
public:
   ClientTransport(FaasWorker* parent, const std::string& func_name)
      : parent_(parent), func_name_(func_name) {}

   void write(const uint8_t* buf, uint32_t len) {
      out_buf_.write(buf, len);
   }

   uint32_t read(uint8_t* buf, uint32_t len) {
      return in_buf_.read(buf, len);
   }

   uint32_t readAll(uint8_t* buf, uint32_t len) {
      return in_buf_.readAll(buf, len);
   }

   void flush() override {
      uint8_t* data;
      uint32_t data_length;
      out_buf_.getBuffer(&data, &data_length);
      const char* output;
      size_t output_length;
      if (parent_->invoke_func_fn_(parent_->caller_context_, func_name_.c_str(),
                                   reinterpret_cast<const char*>(data),
                                   static_cast<size_t>(data_length),
                                   &output, &output_length) != 0) {
         throw apache::thrift::transport::TTransportException(
         apache::thrift::transport::TTransportException::UNKNOWN, "invoke_func call failed");
      }
      out_buf_.resetBuffer();
      in_buf_.resetBuffer(reinterpret_cast<uint8_t*>(const_cast<char*>(output)),
                          static_cast<uint32_t>(output_length));
      }

private:
   FaasWorker* parent_;
   std::string func_name_;
   apache::thrift::transport::TMemoryBuffer in_buf_;
   apache::thrift::transport::TMemoryBuffer out_buf_;
};
```

- will there be any other ways for nightcore to call another serverless function?

- also, in real workloads, the way how one serverless function is declared is different from the sample code
	+ For example in [MovieReviewService.cpp](https://github.com/zyuxuan0115/nightcore-test/blob/main/moviereview_singlenode/DeathStarBench/mediaMicroservices/src/MovieReviewService/MovieReviewService.cpp)

### Should we depend on nightcore?
- how Nightcore works is different from the other FaaS runtime.
	+ it has no scheduler and it assume all serverless functions from 1 micro service running on the same machine 
	+ what does other serverless look like?


### the discussion with Konstantinos and Haoran

![s2](/assets/2023-11-29/s2.png)

