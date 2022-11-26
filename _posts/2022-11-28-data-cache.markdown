---
layout: post
title:  "2022-11-28 data cache"
date:   2022-11-25 10:53:46 -0500
categories: data-cache 
---
|    pagerank   | func that causes<br>most LLC-miss | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:							|   :----:			| :----: |
| <strong>web-Google</strong><br>Vertices:916428<br> Degree:456  |   main: 32.85% <br> do_work: 15.41%   |Function Name:    main<br>PC:401e9b  43%<br>PC:401e7a 29%<br>Function Name: do_work<br>PC:401460 72%| 32   | 
| <strong>web-BerkStan</strong><br>Vertices:685231<br> Degree:249  |   do_work: 34.36% <br> main: 13.32% |Function Name:    main<br>PC:401e9b  40%<br>PC:401e7a 37%<br>Function Name: do_work<br>PC:401460 99%| <span style="color:blue">1/3/4/32</span>   | 
| <strong>web-Stanford</strong><br>Vertices:281904<br> Degree:255  |   main: 25.07% <br> do_work: 23.38%   |Function Name:    main<br>PC:401e9b  47%<br>PC:401e7a 38%<br>Function Name: do_work<br>PC:401460 91%| 32   | 
| <strong>web-NotreDame</strong><br>Vertices:325729<br> Degree:3445  |   do_work: 5.19% <br> main: 3.15%   |Function Name: do_work<br>PC:401460 89%| 32   | 
| <strong>roadNet-PA</strong><br>Vertices:1090920<br> Degree:9  |   do_work: 65.16% <br> do_work: 26.16%   |Function Name: do_work<br>PC:401460 93%| 32   | 
| <strong>roadNet-CA</strong><br>Vertices:1971281<br> Degree:12  |   do_work: 62.36% <br> do_work: 25.53%   |Function Name: do_work<br>PC:401460 82%| 32   | 



|    bfs   | func that causes<br>most LLC-miss | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:							|   :----:			| :----: |
| <strong>loc-brightkite_edges</strong><br>Vertices:36681<br> Degree:456  |   do_work: 20.39% <br> main: 0.05%   |Function Name:    do_work<br>PC:401471  99%<br>Function Name: main<br>PC:401826 44% <br>PC:401b5b 36%| 32   | 
| <strong></strong>Vertices:200000<br> Degree:2  |   do_work: 99.34% <br> main: 0.06%   |Function Name:    do_work<br>PC:401471  99% | 102/107/100/96 | 
| <strong></strong>Vertices:100000<br> Degree:4  |   do_work: 98.31% <br> main: 0.14%   |Function Name:    do_work<br>PC:401471  98% | 205/200/202/32x3 | 
| <strong></strong>Vertices:300000<br> Degree:6  |   do_work: 99.82% <br> main: 0.01%   |Function Name:    do_work<br>PC:401471  99% | 286/32/282  | 
| <strong></strong>Vertices:100000<br> Degree:8  |   do_work: 99.28% <br> main: 0.04%   |Function Name:    do_work<br>PC:401471  99% | 32   | 
| <strong></strong>Vertices:100000<br> Degree:16  |   do_work: 99.23% <br> main: 0.02%   |Function Name:    do_work<br>PC:401471  99%| 32   | 
| <strong></strong>Vertices:200000<br> Degree:32  |   do_work: 99.7% <br> main: 0.12%   |Function Name:    do_work<br>PC:401471  99% | 32   | 
| <strong></strong>Vertices:300000<br> Degree:64  |   do_work: 99.39% <br> main: 0.1%   |Function Name:    do_work<br>PC:401471  99% | 32   | 
| <strong></strong>Vertices:200000<br> Degree:128  |   do_work: 99.13% <br> main: 0.4%   |Function Name:    do_work<br>PC:401471  99% | 32   | 
| <strong></strong>Vertices:200000<br> Degree:150  |   do_work: 98.55% <br> main: 0.29%   |Function Name:    do_work<br>PC:401471  99% | 32   | 
| <strong></strong>Vertices:300000<br> Degree:200  |   do_work: 97% <br> main: 0.47%   |Function Name:    do_work<br>PC:401471  98% | 32   | 
| <strong></strong>Vertices:300000<br> Degree:256  |   do_work: 96.83% <br> main: 0.62%   |Function Name:    do_work<br>PC:401471  98% | 32   | 
| <strong></strong>Vertices:200000<br> Degree:400  |   do_work: 94.79% <br> init_weights: 1.46%   |Function Name:    do_work<br>PC:401471  98% | 32   | 
| <strong></strong>Vertices:200000<br> Degree:512  |   do_work: 92.01% <br> main: 1.8%   |Function Name:    do_work<br>PC:401471  98% | 32   | 







|    dfs   | func that causes<br>most LLC-miss | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:							|   :----:			| :----: |
| <strong></strong>Vertices:800000<br> Degree:800  |   do_work: 29.33% <br>init_weights: 25.04%<br> main: 0.2%   |Function Name:    do_work<br>PC:4013e0  53%<br>PC:4013b4 24%<br> Function Name: init_weights<br>PC:402079 54%<br>PC:4020f2 44%<br>Function Name: main<br>PC:401695 100%| 96   | 

