---
layout: post
title:  "2024-06-08 merge multiple functions"
date:   2024-06-08 1:53:49 -0500
categories: serverless
---
## Performance of Merging

![diagram](/assets/2024-06-08/call-graph-new.png)

### text-service (merged)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      38.271     0.000000            1         1.00
      43.231     0.100000           42         1.11
      46.335     0.500000          196         2.00
      49.183     0.800000          315         5.00
      51.551     0.900000          353        10.00
     197.887     0.990625          389       106.67
     197.887     0.992188          389       128.00
     199.423     0.992969          390       142.22
     201.855     0.996094          391       256.00
     201.855     0.996875          391       320.00
     297.727     0.997656          392       426.67
     297.727     1.000000          392          inf
#[Mean    =       50.556, StdDeviation   =       22.302]
#[Max     =      297.472, Total count    =          392]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  586 requests in 30.01s, 1.00MB read
Requests/sec:     19.53
Transfer/sec:     34.10KB
```

### text-service (original)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      45.823     0.000000            1         1.00
      49.375     0.100000           34         1.11
      53.727     0.500000          169         2.00
      56.735     0.800000          270         5.00
      58.655     0.900000          305        10.00
     200.191     0.990625          334       106.67
     227.967     0.995313          336       213.33
     227.967     0.996094          336       256.00
     302.335     0.997266          337       365.71
     302.335     1.000000          337          inf
#[Mean    =       58.950, StdDeviation   =       25.925]
#[Max     =      302.080, Total count    =          337]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  504 requests in 30.01s, 0.86MB read
Requests/sec:     16.80
Transfer/sec:     29.25KB
```



### write-home-timeline (merged)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      29.679     0.000000            1         1.00
      31.359     0.100000           56         1.11
      33.279     0.500000          274         2.00
      35.263     0.800000          441         5.00
      39.967     0.900000          494        10.00
     101.439     0.990625          544       106.67
     103.103     0.996875          547       320.00
     103.103     0.997266          547       365.71
     113.791     0.998242          548       568.89
     113.791     1.000000          548          inf
#[Mean    =       36.417, StdDeviation   =       12.476]
#[Max     =      113.728, Total count    =          548]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  815 requests in 30.01s, 291.14KB read
Requests/sec:     27.16
Transfer/sec:      9.70KB
```

### write-home-timeline (original)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      28.063     0.000000            1         1.00
      29.215     0.100000           58         1.11
      30.639     0.500000          291         2.00
      33.311     0.800000          464         5.00
      34.783     0.900000          522        10.00
      50.143     0.950000          551        20.00
     120.703     0.990625          575       106.67
     200.191     0.996875          580       320.00
     200.191     1.000000          580          inf
#[Mean    =       34.459, StdDeviation   =       17.794]
#[Max     =      200.064, Total count    =          580]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  866 requests in 30.01s, 309.28KB read
Requests/sec:     28.86
Transfer/sec:     10.31KB
```


### compose-post (merged)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      42.015     0.000000            1         1.00
      46.847     0.100000           33         1.11
      52.223     0.500000          158         2.00
      56.639     0.800000          247         5.00
      61.823     0.900000          278        10.00
      97.343     0.950000          293        20.00
     400.127     0.990625          306       106.67
     496.639     0.996875          308       320.00
     496.639     1.000000          308          inf
#[Mean    =       64.421, StdDeviation   =       56.500]
#[Max     =      496.384, Total count    =          308]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  469 requests in 30.00s, 515.00KB read
Requests/sec:     15.63                                                                                   Transfer/sec:     17.17KB
```

### compose-post (original)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)
 
     129.151     0.000000            1         1.00
     134.911     0.100000           12         1.11
     143.999     0.500000           62         2.00
     162.943     0.800000           96         5.00
     193.919     0.900000          108        10.00
     462.079     0.990625          118       106.67
     494.079     1.000000          119          inf

#[Mean    =      167.563, StdDeviation   =       73.999]
#[Max     =      493.824, Total count    =          119]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  179 requests in 30.01s, 196.02KB read
Requests/sec:      5.97
Transfer/sec:      6.53KB
```


### Summary

|        | 2 functions | 3 functions | 11 functions |
| :----: | :----: | :----: | :----: |
| original time (ms) | 30.639 | 53.727 | 143.999 |
| merged time (ms) | 33.279 | 46.335 | 52.223 |


## linker flags

From `man ld` or [this webpage](https://www.man7.org/linux/man-pages/man1/ld.1.html)

### --as-needed

- linker flag 1: `-Wl,--as-needed`
  + This option affects `ELF DT_NEEDED` tags for dynamic libraries mentioned on the command line after the `--as-needed` option. Normally the linker will add a `DT_NEEDED` tag for each dynamic library mentioned on the command line, regardless of whether the library is actually needed or not.  `--as-needed` causes a `DT_NEEDED` tag to only be emitted for a library that at that point in the link satisfies a non-weak undefined symbol reference from a regular object file or, if the library is not found in the `DT_NEEDED` lists of other needed libraries, a non-weak undefined symbol reference from another needed dynamic library.  Object files or libraries appearing on the command line after the library in question do not affect whether the library is seen as needed.

```bash
LINKER_FLAGS="-lstd$RUST_LIBSTD_LINKER_FLAG -lcurl -lcrypto -lm 
-lssl -lz -lpthread -lrustc_driver$RUST_LIBRUSTC_LINKER_FLAG 
-ltest$RUST_LIBTEST_LINKER_FLAG "
```

- before `--as-needed`

![s3](/assets/2024-06-08/s2.png)

- after `--as-needed`

![s3](/assets/2024-06-08/s3.png)

### --gc-setions

- linker flag 2: `-Wl,--gc-sections`
  + Enable garbage collection of unused input sections.
  + this flag must be using with `-ffunction-sections -fdata-sections` at compile time
    * [reference](https://gcc.gnu.org/onlinedocs/gnat_ugn/Compilation-options.html)
    * [reference 2](https://stackoverflow.com/questions/18115598/how-to-remove-all-unused-symbols-from-a-shared-library)
    * Note: for clang the flag is `$LLVM_DIR/llc -filetype=obj --function-sections --data-sections function.ll -o function.o` 
  + the size of the new binary: 9.8 MB (original 45 MB)
    * ![s1](/assets/2024-06-08/s1.png)

### --strip-debug

- linker flag 3: `-Wl,--strip-debug`
	+ Omit debugger symbol information (but not all symbols) from the output file

### Merged 11 functions: after adding the linker flags

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      43.743     0.000000            1         1.00
      47.807     0.100000           31         1.11
      52.031     0.400000          121         1.67
      53.087     0.500000          152         2.00
      57.791     0.800000          242         5.00
      64.543     0.900000          271        10.00
     102.079     0.950000          286        20.00
     400.383     0.990625          299       106.67
     499.199     0.996484          300       284.44
     502.783     1.000000          301          inf
#[Mean    =       66.038, StdDeviation   =       59.188]
#[Max     =      502.528, Total count    =          301]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  455 requests in 30.01s, 499.36KB read
Requests/sec:     15.16
Transfer/sec:     16.64KB
```


### Merged 11 functions: without adding linker flags

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      61.215     0.000000            1         1.00
      64.575     0.100000           25         1.11
      67.327     0.400000          101         1.67
      67.967     0.500000          121         2.00
      70.399     0.800000          194         5.00
      73.535     0.900000          218        10.00
     115.775     0.950000          230        20.00
     403.455     0.990625          240       106.67
     485.887     0.992188          241       128.00
     500.479     1.000000          242          inf
#[Mean    =       82.219, StdDeviation   =       64.579]
#[Max     =      500.224, Total count    =          242]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  361 requests in 30.01s, 394.57KB read
Requests/sec:     12.03
Transfer/sec:     13.15KB
```


### Original 11 functions (without merging)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

     140.287     0.000000            1         1.00
     145.151     0.100000           12         1.11
     154.239     0.500000           56         2.00
     169.599     0.800000           90         5.00
     191.615     0.900000          101        10.00
     398.079     0.950000          107        20.00
     494.335     0.990625          111       106.67
     524.799     1.000000          112          inf
#[Mean    =      178.449, StdDeviation   =       75.363]
#[Max     =      524.288, Total count    =          112]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  167 requests in 30.01s, 182.73KB read
Requests/sec:      5.57
Transfer/sec:      6.09KB
```

### Merged 3 functions: after adding the linker flags

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      30.079     0.000000            1         1.00
      33.023     0.100000           45         1.11
      36.127     0.400000          180         1.67
      36.703     0.500000          222         2.00
      40.095     0.800000          355         5.00
      43.071     0.900000          399        10.00
      96.447     0.950000          421        20.00
     202.367     0.990625          439       106.67
     300.799     0.996484          442       284.44
     301.567     0.998047          443       512.00
     301.567     1.000000          443          inf
#[Mean    =       45.075, StdDeviation   =       35.426]
#[Max     =      301.312, Total count    =          443]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  678 requests in 30.01s, 792.24KB read
Requests/sec:     22.59
Transfer/sec:     26.40KB
```

### Merged 3 functions: without adding linker flags

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      30.239     0.000000            1         1.00
      33.375     0.100000           45         1.11
      36.895     0.500000          223         2.00
      39.775     0.800000          358         5.00
      43.871     0.900000          402        10.00
     102.399     0.950000          424        20.00
     201.343     0.990625          442       106.67
     202.367     0.996094          445       256.00
     204.031     0.998047          446       512.00
     204.031     1.000000          446          inf
#[Mean    =       44.427, StdDeviation   =       30.612]
#[Max     =      203.904, Total count    =          446]
#[Buckets =           27, SubBuckets     =         2048]
```

### Original 3 functions (without merging)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      42.079     0.000000            1         1.00
      45.247     0.100000           35         1.11
      48.319     0.400000          139         1.67
      48.831     0.500000          172         2.00
      52.095     0.800000          276         5.00
      74.943     0.900000          310        10.00
     117.311     0.950000          327        20.00
     200.831     0.990625          341       106.67
     201.343     0.993750          342       160.00
     201.471     0.994531          343       182.86
     202.623     0.997266          344       365.71
     202.623     1.000000          344          inf
#[Mean    =       57.735, StdDeviation   =       31.385]
#[Max     =      202.496, Total count    =          344]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  524 requests in 30.01s, 607.97KB read
Requests/sec:     17.46
Transfer/sec:     20.26KB
```

### Merged 2 functions: after adding the linker flags

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      25.983     0.000000            1         1.00
      27.903     0.100000           57         1.11
      29.871     0.500000          284         2.00
      32.559     0.800000          452         5.00
      36.799     0.900000          508        10.00
      95.807     0.950000          536        20.00
     103.935     0.990625          559       106.67
     200.703     0.996875          563       320.00
     200.703     0.998047          563       512.00
     221.439     0.998242          564       568.89
     221.439     1.000000          564          inf
#[Mean    =       35.358, StdDeviation   =       21.201]
#[Max     =      221.312, Total count    =          564]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  851 requests in 30.01s, 198.62KB read
Requests/sec:     28.36
Transfer/sec:      6.62KB
```

### Merged 2 functions: without adding linker flags

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      27.279     0.000000            1         1.00
      29.135     0.100000           31         1.11
      31.343     0.500000          155         2.00
      33.919     0.800000          244         5.00
      37.311     0.900000          275        10.00
      73.407     0.950000          290        20.00
     198.399     0.990625          303       106.67
     199.295     0.996484          304       284.44
   18956.287     0.996875          305       320.00
   18956.287     1.000000          305          inf
#[Mean    =       98.198, StdDeviation   =     1081.320]
#[Max     =    18939.904, Total count    =          305]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  306 requests in 30.00s, 71.42KB read
  Socket errors: connect 0, read 0, write 0, timeout 9
Requests/sec:     10.20
Transfer/sec:      2.38KB
```


### Original 2 functions (without merging)

```
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

      23.935     0.000000            1         1.00
      26.511     0.100000           61         1.11
      28.095     0.500000          300         2.00
      29.903     0.800000          477         5.00
      33.503     0.900000          537        10.00
      94.015     0.950000          567        20.00
     105.151     0.990625          591       106.67
     193.151     0.998047          595       512.00
     194.047     0.998437          596       640.00
     194.047     1.000000          596          inf
#[Mean    =       33.430, StdDeviation   =       19.632]
#[Max     =      193.920, Total count    =          596]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  888 requests in 30.00s, 207.26KB read
Requests/sec:     29.60
Transfer/sec:      6.91KB
```

### Results

#### Execution time

| function name | original | merge  | merge with linker opt |
| :----: | :----:   | :----: | :----: |
| <strong>compose post</strong> <br> (11 functions)  | 154.239 ms | 67.967 ms | 53.087 ms |
| <strong>text service</strong> <br> (3 functions) | 48.831 ms | 36.895 ms | 36.703 ms |
| <strong>write home timeline</strong> <br> (2 functions) | 28.095 ms | 31.343 ms | 29.871 ms |

#### Binary size

| function name | original (1 caller function) | merge  | merge with linker opt |
| :----: | :----:   | :----: | :----: |
| <strong>compose post</strong> <br> (11 functions)  | 11 MB | 45 MB | 9.8 MB |
| <strong>text service</strong> <br> (3 functions) | 27 MB | 24 MB | 7 MB |
| <strong>write home timeline</strong> <br> (2 functions) | 29 MB | 35 MB | 4.3 MB |

#### Number of dynamically linked libraries

| function name | original (1 caller function) | merge  | merge with linker opt |
| :----: | :----:   | :----: | :----: |
| <strong>compose post</strong> <br> (11 functions)  | 10 | 51  | 51 |
| <strong>text service</strong> <br> (3 functions) | 10 | 51 | 51 |
| <strong>write home timeline</strong> <br> (2 functions) | 10 | 51 | 51 |

### Maybe an alternative way?

using Rust linker? 
- the [manual of rust](https://doc.rust-lang.org/rustc/linker-plugin-lto.html)
