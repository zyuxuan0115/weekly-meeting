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
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

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

- exit code `0`
- no `-flto`
- have `-Oz`
- binary size: `37MB`

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      17.871     0.000000            1         1.00
      18.639     0.100000           99         1.11
      19.503     0.500000          495         2.00
      20.431     0.800000          784         5.00
      21.615     0.900000          882        10.00
      40.927     0.990625          971       106.67
      41.631     0.992188          973       128.00
      42.175     0.992969          974       142.22
      42.175     0.993750          974       160.00
      50.687     0.994531          975       182.86
      61.215     0.999023          980      1024.00
      61.215     1.000000          980          inf
#[Mean    =       20.325, StdDeviation   =        3.701]
#[Max     =       61.184, Total count    =          980]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  1462 requests in 30.01s, 504.99KB read
Requests/sec:     48.72
Transfer/sec:     16.83KB
```

- exit code `0`
- have `-flto`
- have `-Oz`
- binary size: `37MB`

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      18.367     0.000000            1         1.00
      26.687     0.500000          374         2.00
      27.519     0.800000          601         5.00
      28.079     0.900000          675        10.00
      35.423     0.990625          741       106.67
      60.479     0.998437          747       640.00
      60.479     0.998633          747       731.43
      60.831     0.998828          748       853.33
      60.831     1.000000          748          inf
#[Mean    =       26.647, StdDeviation   =        3.333]
#[Max     =       60.800, Total count    =          748]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  1123 requests in 30.01s, 388.05KB read
Requests/sec:     37.43
Transfer/sec:     12.93KB
```

#### Is static linking faster than dynamic linking?
- [here](https://stackoverflow.com/questions/4667882/is-a-statically-linked-executable-faster-than-a-dynamically-linked-executable) is a stackoverflow answer:
  + Static linking produces a larger executable file than dynamic linking because it has to compile all of the library code directly into the executable. 
  + The benefit is a reduction in overhead from no longer having to call functions from a library, and anywhere from <strong>somewhat to noticeably faster load times</strong>.

- <strong>staticx</strong>
  + convert all dynamically linked binary to be statically linked.
    * [github repo](https://github.com/JonathonReinhart/staticx)
    * [online manual](https://staticx.readthedocs.io/en/latest/index.html)

- I used staticx to replace all the dynamic linked library to be statically linked
  + the binary size: 71MB

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

    8396.799     0.000000            1         1.00
    8396.799     0.100000            1         1.11
    8396.799     0.200000            1         1.25
    8396.799     0.300000            1         1.43
    8454.143     0.400000            2         1.67
    8454.143     0.500000            2         2.00
    8454.143     0.550000            2         2.22
    8454.143     0.600000            2         2.50
    8454.143     0.650000            2         2.86
   12025.855     0.700000            3         3.33
   12025.855     1.000000            3          inf
#[Mean    =     9621.504, StdDeviation   =     1697.399]
#[Max     =    12017.664, Total count    =            3]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  3 requests in 30.01s, 1.10KB read
  Socket errors: connect 0, read 0, write 0, timeout 12
Requests/sec:      0.10
Transfer/sec:      37.38B
```
