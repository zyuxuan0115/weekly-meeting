---
layout: post
title:  "2024-05-31 merge functions"
date:   2024-05-31 1:53:49 -0500
categories: serverless
---

### the reversed results

#### original write-home-timeline

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      28.143     0.000000            1         1.00
      30.511     0.100000           55         1.11
      31.567     0.500000          274         2.00
      32.431     0.800000          437         5.00
      33.311     0.900000          490        10.00
     195.455     0.990625          538       106.67
     195.583     0.992188          540       128.00
     195.583     0.992969          540       142.22
     195.583     0.993750          540       160.00
     197.247     0.995313          541       213.33
     199.039     0.996484          543       284.44                                                                    199.039     1.000000          543          inf
#[Mean    =       36.751, StdDeviation   =       23.434]
#[Max     =      198.912, Total count    =          543]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  813 requests in 30.01s, 290.46KB read
Requests/sec:     27.09
Transfer/sec:      9.68KB
```

#### merged write-home-timeline (callee: social-graph-get-followers)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      31.839     0.000000            1         1.00
      33.503     0.100000           49         1.11
      35.423     0.500000          240         2.00
      37.983     0.800000          382         5.00
      45.119     0.900000          430        10.00
     195.711     0.990625          473       106.67
     196.095     0.992188          474       128.00
     198.143     0.996094          476       256.00
     198.783     0.998047          477       512.00
     198.783     1.000000          477          inf
#[Mean    =       41.792, StdDeviation   =       23.274]
#[Max     =      198.656, Total count    =          477]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  712 requests in 30.01s, 254.38KB read
Requests/sec:     23.73
Transfer/sec:      8.48KB
```

#### the result of merging 2 functions
- caller: write-home-timeline
- callee: social-graph-get-followers

| | reported by wrk | reported by function |
|:---: | :---: | :---: |
| original | 31.567 ms | 23.5 ms |
| merged | 35.423 ms | 15 ms |

### Immediate exit after function is called

![s4](/assets/2024-05-31/s4.png)

#### Unmerged function (like calling an empty function)

![s2](/assets/2024-05-31/s2.png)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       5.439     0.000000            1         1.00
       8.119     0.100000          232         1.11
       8.559     0.500000         1192         2.00
       8.695     0.800000         1851         5.00
       8.855     0.900000         2075        10.00
      10.127     0.990625         2280       106.67
      44.159     0.999023         2299      1024.00
      49.279     0.999609         2301      2560.00
      49.279     1.000000         2301          inf
#[Mean    =        8.620, StdDeviation   =        1.767]
#[Max     =       49.248, Total count    =         2301]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  3452 requests in 30.01s, 1.16MB read
Requests/sec:    115.05
Transfer/sec:     39.72KB
```

#### Merged fucntion v1 & v2

![s3](/assets/2024-05-31/s3.png)

- exit code `0` (or `1` -- terminate the program with error)
- no `-flto` (link time optimization)
- have `-O2` (compile time optimization)
- binary size: `20MB`

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

#### Merged function v3

- exit code `0`
- have `-flto`
- have `-O2`
- binary size: `20MB`

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

### more about dynamic linking 
I guess the reason why there is a delay is because in our code, we need to dynamically load dynamic linked library to the program and then execute the function in the functions in those libraries. 
[make-static](https://www.ucc.asn.au/~dagobah/things/make-static.html)

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

### Merging 3 functions
