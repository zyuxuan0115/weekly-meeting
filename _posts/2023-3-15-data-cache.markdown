---
layout: post
title:  "2023-3-15 DMon and perf record -p"
date:   2023-3-15 7:53:46 -0500
categories: data-cache 
---
### dmon
- successfully run dmon's selective profiling
- successfully build dmon's libPrefetch.so llvm pass
- libPrefetch.so doesn't work

### pg^2
- build the first version of pg^2 which can change the prefetch distance at runtime
![pr-syn](/assets/2023-03-04/pr.png)

### performance counter results
- from previous pr's results
![old-pr](/assets/2023-01-24/v200000-d200.png)

![s1](/assets/2023-03-15/s1.png)
 
- <strong>real original</strong>: execution time <strong>6.685039</strong> sec
```
# Samples: 77K of event 'MEM_LOAD_RETIRED.L3_MISS'
# Event count (approx.): 22215890
#
# Overhead  Command   Shared Object     Symbol
# ........  ........  ................  ...............................
#
    97.06%  pagerank  pagerank          [.] do_work
     1.24%  pagerank  pagerank          [.] main
     1.03%  pagerank  [unknown]         [k] 0xffffffffba263a7a
     0.30%  pagerank  [unknown]         [k] 0xffffffffba2741e0
     0.13%  pagerank  [unknown]         [k] 0xffffffffba263cd5
     0.04%  pagerank  [unknown]         [k] 0xffffffffba27c291
     0.04%  pagerank  [unknown]         [k] 0xffffffffba25454d
```

- <strong>synthetic original</strong>: execution time <strong>6.551384</strong> sec 
```
# Samples: 76K of event 'MEM_LOAD_RETIRED.L3_MISS'
# Event count (approx.): 22322765
#
# Overhead  Command          Shared Object      Symbol
# ........  ...............  .................  .......................
#
    97.21%  pagerank_syn_or  pagerank_syn_orig  [.] do_work
     1.07%  pagerank_syn_or  pagerank_syn_orig  [.] main
     1.06%  pagerank_syn_or  [unknown]          [k] 0xffffffffba263a7a
     0.30%  pagerank_syn_or  [unknown]          [k] 0xffffffffba2741e0
     0.16%  pagerank_syn_or  [unknown]          [k] 0xffffffffba263cd5
     0.04%  pagerank_syn_or  [unknown]          [k] 0xffffffffba27c291
     0.04%  pagerank_syn_or  [unknown]          [k] 0xffffffffba25454d
     0.03%  pagerank_syn_or  [unknown]          [k] 0xffffffffba273083
```

- <strong>perfetch dist = 28</strong>: execution time <strong>7.568303</strong> sec
```
# Samples: 79K of event 'MEM_LOAD_RETIRED.L3_MISS'
# Event count (approx.): 8357256
#
# Overhead  Command      Shared Object       Symbol
# ........  ...........  ..................  .........................
#
    92.31%  pagerank_28  pagerank_28         [.] do_work
     2.98%  pagerank_28  [unknown]           [k] 0xffffffffba263a7a
     2.85%  pagerank_28  pagerank_28         [.] main
     0.84%  pagerank_28  [unknown]           [k] 0xffffffffba2741e0
     0.45%  pagerank_28  [unknown]           [k] 0xffffffffba263cd5
     0.12%  pagerank_28  [unknown]           [k] 0xffffffffba27c291
```

- <strong>prefetch dist = 6</strong>: execution time <strong>6.312855</strong> sec
```
# Samples: 64K of event 'MEM_LOAD_RETIRED.L3_MISS'
# Event count (approx.): 4938538
#
# Overhead  Command     Shared Object     Symbol
# ........  ..........  ................  .......................
#
    89.33%  pagerank_6  pagerank_6        [.] do_work
     4.80%  pagerank_6  pagerank_6        [.] main
     3.13%  pagerank_6  [unknown]         [k] 0xffffffffba263a7a
     1.38%  pagerank_6  [unknown]         [k] 0xffffffffba2741e0
     0.47%  pagerank_6  [unknown]         [k] 0xffffffffba263cd5
     0.21%  pagerank_6  [unknown]         [k] 0xffffffffba27c291
     0.14%  pagerank_6  [unknown]         [k] 0xffffffffba273083
     0.14%  pagerank_6  [unknown]         [k] 0xffffffffba25454d
```



- <strong>manually inject prefetch from prefetch_dist=6 binary to synthetic_orig binary</strong>: execution time <strong>6.126617</strong> sec 
```
# Samples: 61K of event 'MEM_LOAD_RETIRED.L3_MISS'
# Event count (approx.): 4535020
#
# Overhead  Command          Shared Object     Symbol
# ........  ...............  ................  ............................
#
    86.98%  pagerank_syn_6_  pagerank_syn_6_   [.] do_work
     5.25%  pagerank_syn_6_  pagerank_syn_6_   [.] main
     4.93%  pagerank_syn_6_  [unknown]         [k] 0xffffffffba263a7a
     1.54%  pagerank_syn_6_  [unknown]         [k] 0xffffffffba2741e0
     0.42%  pagerank_syn_6_  [unknown]         [k] 0xffffffffba263cd5
     0.23%  pagerank_syn_6_  [unknown]         [k] 0xffffffffba27c291
     0.18%  pagerank_syn_6_  [unknown]         [k] 0xffffffffba25454d
     0.16%  pagerank_syn_6_  [unknown]         [k] 0xffffffffba273083
```

- problem about perf record LLC miss
	+ if perf is attached to a process by `perf record -e <LLC_MISS event> -p <PID>`, it cannot accurately record the LLC miss.
