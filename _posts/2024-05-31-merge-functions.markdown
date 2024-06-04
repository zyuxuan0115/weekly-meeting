---
layout: post
title:  "2024-05-31 merge functions"
date:   2024-05-31 1:53:49 -0500
categories: serverless
---
I guess the reason why there is a delay is because in our code, we need to dynamically load dynamic linked library to the program and then execute the function in the functions in those libraries. 
[make-static](https://www.ucc.asn.au/~dagobah/things/make-static.html)




#### Immediate exit after function is called
- exit code `-1`
- no `-flto` (link time optimization)
- have `-Oz` compile time optimization, similiar to `-O2`, but also reduce code size
- binary size: `37MB`

```
      17.823     0.000000            1         1.00
      19.583     0.500000          494         2.00
      20.639     0.800000          781         5.00
      22.175     0.900000          879        10.00
      36.223     0.990625          967       106.67
      54.239     0.999023          976      1024.00
      54.239     1.000000          976          inf
#[Mean    =       20.424, StdDeviation   =        3.502]
#[Max     =       54.208, Total count    =          976]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  1463 requests in 30.01s, 505.35KB read
Requests/sec:     48.76
Transfer/sec:     16.84KB
```

#### Is static linking faster than dynamic linking?
- [here](https://stackoverflow.com/questions/4667882/is-a-statically-linked-executable-faster-than-a-dynamically-linked-executable) is a stackoverflow answer:
  + Static linking produces a larger executable file than dynamic linking because it has to compile all of the library code directly into the executable. 
  + The benefit is a reduction in overhead from no longer having to call functions from a library, and anywhere from <strong>somewhat to noticeably faster load times</strong>.

- <strong>staticx</strong>
  + convert all dynamically linked binary to be statically linked.
    * [github repo](https://github.com/JonathonReinhart/staticx)
    * [online manual](https://staticx.readthedocs.io/en/latest/index.html)


