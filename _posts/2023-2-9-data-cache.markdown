---
layout: post
title:  "2023-2-9 data cache"
date:   2023-2-9 7:53:46 -0500
categories: data-cache 
---
### pagerank # iteration = 100
- directed graph

| input name  | # nodes | # edges | execution time (sec)| 
| :----: | :----:| :----:| :----: | 
| Cit-Patents | 3,774,768 | 16,518,948 | 210.613649 |
| soc-pokec-relationships | 1,632,803 | 30,622,564 | 83.689856 | 
| wiki-topcats | 1,791,489 |28,511,807 | 67.773231 |
| roadNet-TX | 1,379,917 | 1,921,660 | 28.719360 |
| sx-superuser| 194,085 | 1,443,339 |11.797546|
| sx-superuser-a2q | 167,981 | 430,033 |10.325830|
| sx-askubuntu | 159,316 | 964,437 |8.763955|
| amazon0601 | 403,394 | 3,387,388 |6.951533 | 
| amazon0312 | 	400,727 | 3,200,440 | 6.905545| 
| sx-superuser-c2a | 101,052 | 534,239 |6.318929|
| amazon0505 | 410,236 | 3,356,824 |6.299512 | 
| email-EuAll | 265,214 | 420,045 | 6.260060 |
| sx-superuser-c2q | 94,548 | 479,067 | 6.228259|
| amazon0302 | 	262,111 | 1,234,877 | 3.120691 | 
| soc-sign-Slashdot090221 | 82,140 | 549,202 | 1.484171 |
| soc-sign-Slashdot090216 | 81,867 | 545,671 | 1.458236 |
| soc-sign-Slashdot081106 | 77,357 | 516,575 | 1.330841 |
| as-caida20071105 | 26,475 | 106,762 |0.501582  |
| wiki-talk-temporal | 1,140,149 | 7,833,140 | Could not allocate memory |
| soc-LiveJournal1 | 4,847,571 | 68,993,773 | killed by OS during initialization|
| Wiki-Talk | 2,394,385 | 5,021,410 | killed by OS during initialization|
| soc-sign-epinions | 131,828 | 841,372 | killed by OS during initialization| 
| sx-stackoverflow | 2,601,977 | 63,497,050 | killed by OS during initialization |
| sx-stackoverflow-a2q | 2,464,606 | 17,823,525| killed by OS during initialization |
| sx-stackoverflow-c2q | 1,655,353 | 20,268,151| killed by OS during initialization |
| sx-stackoverflow-c2a | 1,646,338 | 25,405,374| killed by OS during initialization |
| gplus_combined | 107,614 | 13,673,453 | crash | 


- undirected graph

| input name  | # nodes | # edges | execution time (sec)| 
| :----: | :----:| :----:| :----: | 
| com-youtube | 1,134,890 | 2,987,624 | 63.444812 |
| roadNet-TX | 1,379,917 | 1,921,660 | 28.719360 |
| com-amazon | 334,863 | 925,872 | 11.132960 |
| com-dblp | 317,080 | 1,049,866 |9.623910 |
| loc-gowalla_edges | 196,591 | 950,327 | 4.819170 | 
| com-LiveJournal | 3,997,962 | 34,681,189	 | killed by OS during initialization |
| com-friendster | 65,608,366 | 1,806,067,135 | killed by OS during initialization |
| as-skitter | 1,696,415 | 11,095,298 | killed by OS during initialization | 
| com-orkut | 3,072,441 | 117,185,083 | killed by OS during initialization | 

### vtune top-down results
- vtune is installed at: `/opt/intel/oneapi`
    + before using vtune, must run `source /opt/intel/oneapi/vtune/2023.0.0/amplxe-vars.sh` first
    + to run vtune `vtune -collect uarch-exploration -data-limit=4096 -target-pid `
- vtune result

    | input name  | Back-End Bound | Memory Bound |  DRAM Bound | Memory Bandwidth | Memory Latency |
    | :----: | :----:| :----:| :----: | :----:| :----: | 
    | soc-pokec-relationships | 83.8% | 75.1% | 74.6% | 76.8% | 17.2% | 
    | wiki-topcats | 89.1% | 80.3% | 76.9% | 64.0% | 28.5% |
    | roadNet-TX | 86.7% | 73.7% | 68.8% | 73.6% | 21.2% | 
    | com-youtube | 72.0% | 62.3% | 57.7% |  54.9% | 24.1% |
    | sx-superuser | 50.2% | 38.3% | 44.3% | 45.5% | 27.5% | 
    | com-amazon | 48.2% | 35.0% | 34.3% | 37.1% | 45.7% | 
    | cit-Patents | 79.8% | 71.1% | 68.3% | 50.6% | 40.0% | 