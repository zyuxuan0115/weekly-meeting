---
layout: post
title:  "2023-2-9 data cache"
date:   2023-2-9 7:53:46 -0500
categories: data-cache 
---
### pagerank # iteration = 100

| input name  | # nodes | # edges | execution time (sec)| 
| :----: | :----:| :----:| :----: | 
| soc-pokec-relationships | 1,632,803 | 30,622,564 | 83.689856 | 
| wiki-topcats | 1,791,489 |28,511,807 | 67.773231 |
| roadNet-TX | 1,379,917 | 1,921,660 | 28.719360 |
| sx-superuser| 194,085 | 1,443,339 |11.797546|
| amazon0601 | 403,394 | 3,387,388 |6.951533 | 
| email-EuAll | 265,214 | 420,045 | 6.260060 |
| soc-sign-Slashdot081106 | 77,357 | 516,575 | 1.330841 |
| wiki-talk-temporal | 1,140,149 | 7,833,140 | killed by OS during initialization |
| soc-LiveJournal1 | 4,847,571 | 68,993,773 | killed by OS during initialization|
| Wiki-Talk | 2,394,385 | 5,021,410 | killed by OS during initialization|
| soc-sign-epinions | 131828 | 841372 | killed by OS during initialization| 
| sx-stackoverflow-a2q | 2,464,606 | 17,823,525| killed by OS during initialization |
| sx-stackoverflow-c2q | 1,655,353 | 20,268,151| killed by OS during initialization |
| gplus_combined | 107,614 | 13,673,453 | crash | 
| Cit-Patents | 3,774,768 | 16,518,948 | seg fault|

### vtune top-down results
- vtune is installed at: `/opt/intel/oneapi`
    + before using vtune, must run `source /opt/intel/oneapi/vtune/2023.0.0/amplxe-vars.sh` first
    + to run vtune `amplxe-cl -collect uarch-exploration -data-limit=4096 -target-pid `
- how to run vtune



