---
layout: post
title:  "2022-11-28 data cache"
date:   2022-11-25 10:53:46 -0500
categories: data-cache 
---
|    pagerank   | func that causes<br>most LLC-miss | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:							|   :----:			| :----: |
| <strong>web-Google</strong><br>Vertices:916428<br> Degree:456  |   <strong>main</strong>: 32.85% <br> <strong>do_work</strong>: 15.41%   |Function Name:    <strong>main</strong><br>PC:401e9b  43%<br>PC:401e7a 29%<br>Function Name: <strong>do_work</strong><br>PC:401460 72%| 32   | 
| <strong>web-BerkStan</strong><br>Vertices:685231<br> Degree:249  |   <strong>do_work</strong>: 34.36% <br> <strong>main</strong>: 13.32% |Function Name:    <strong>main</strong><br>PC:401e9b  40%<br>PC:401e7a 37%<br>Function Name: <strong>do_work</strong><br>PC:401460 99%| <span style="color:blue">1/3/4/32</span>   | 
| <strong>web-Stanford</strong><br>Vertices:281904<br> Degree:255  |   <strong>main</strong>: 25.07% <br> <strong>do_work</strong>: 23.38%   |Function Name:    <strong>main</strong><br>PC:401e9b  47%<br>PC:401e7a 38%<br>Function Name: <strong>do_work</strong><br>PC:401460 91%| 32   | 
| <strong>web-NotreDame</strong><br>Vertices:325729<br> Degree:3445  |   <strong>do_work</strong>: 5.19% <br> <strong>main</strong>: 3.15%   |Function Name: <strong>do_work</strong><br>PC:401460 89%| 32   | 
| <strong>roadNet-PA</strong><br>Vertices:1090920<br> Degree:9  |   <strong>do_work</strong>: 65.16% <br> <strong>do_work</strong>: 26.16%   |Function Name: <strong>do_work</strong><br>PC:401460 93%| 32   | 
| <strong>roadNet-CA</strong><br>Vertices:1971281<br> Degree:12  |   <strong>do_work</strong>: 62.36% <br> <strong>do_work</strong>: 25.53%   |Function Name: <strong>do_work</strong><br>PC:401460 82%| 32   | 



|    bfs   | func that causes<br>most LLC-miss | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:							|   :----:			| :----: |
| <strong>loc-brightkite_edges</strong><br>Vertices:36681<br> Degree:456  |   <strong>do_work</strong>: 20.39% <br> <strong>main</strong>: 0.05%   |Function Name:    <strong>do_work</strong><br>PC:401471  99%<br>Function Name: <strong>main</strong><br>PC:401826 44% <br>PC:401b5b 36%| 32   | 
| Vertices:200000<br> Degree:2  |   <strong>do_work</strong>: 99.34% <br> main: 0.06%   |Function Name:    <strong>do_work</strong><br>PC:401471  99% | <span style="color:blue">102/107/100/96</span> | 
| Vertices:100000<br> Degree:4  |   <strong>do_work</strong>: 98.31% <br> <strong>main</strong>: 0.14%   |Function Name:    <strong>do_work</strong><br>PC:401471  98% | <span style="color:blue">205/200/202/<br>32x3</span> | 
| Vertices:300000<br> Degree:6  |   <strong>do_work</strong>: 99.82% <br> <strong>main</strong>: 0.01%   |Function Name:    <strong>do_work</strong><br>PC:401471  99% | <span style="color:blue">286/32x2/282</span>  | 
| Vertices:100000<br> Degree:8  |   <strong>do_work</strong>: 99.28% <br> <strong>main</strong>: 0.04%   |Function Name:    <strong>do_work</strong><br>PC:401471  99% | 32   | 
| Vertices:100000<br> Degree:16  |   <strong>do_work</strong>: 99.23% <br> <strong>main</strong>: 0.02%   |Function Name:    <strong>do_work</strong><br>PC:401471  99% | 32   | 
| Vertices:200000<br> Degree:32  |   <strong>do_work</strong>: 99.7% <br> <strong>main</strong>: 0.12%   |Function Name:    <strong>do_work</strong><br>PC:401471  99% | 32   | 
| Vertices:300000<br> Degree:64  |   <strong>do_work</strong>: 99.39% <br> <strong>main</strong>: 0.1%   |Function Name:    <strong>do_work</strong><br>PC:401471  99% | 32   | 
| Vertices:200000<br> Degree:128  |   <strong>do_work</strong>: 99.13% <br> <strong>main</strong>: 0.4%   |Function Name:    <strong>do_work</strong><br>PC:401471  99% | 32   | 
| Vertices:200000<br> Degree:150  |   <strong>do_work</strong>: 98.55% <br> <strong>main</strong>: 0.29%   |Function Name:    <strong>do_work</strong><br>PC:401471  99% | 32   | 
| Vertices:300000<br> Degree:200  |   <strong>do_work</strong>: 97% <br> <strong>main</strong>: 0.47%   |Function Name:    <strong>do_work</strong><br>PC:401471  98% | 32   | 
| Vertices:300000<br> Degree:256  |   <strong>do_work</strong>: 96.83% <br> <strong>main</strong>: 0.62%   |Function Name:    <strong>do_work</strong><br>PC:401471  98% | 32   | 
| Vertices:200000<br> Degree:400  |   <strong>do_work</strong>: 94.79% <br> <strong>init_weights</strong>: 1.46%   |Function Name:    <strong>do_work</strong><br>PC:401471  98% | 32   | 
| Vertices:200000<br> Degree:512  |   <strong>do_work</strong>: 92.01% <br> <strong>main</strong>: 1.8%   |Function Name:    <strong>do_work</strong><br>PC:401471  98% | 32   | 







|    dfs   | func that causes<br>most LLC-miss | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:							|   :----:			| :----: |
| Vertices:800000<br> Degree:100  |   <strong>do_work</strong>: 44.97% <br><strong>init_weights</strong>: 28.2%   |Function Name:    <strong>do_work</strong><br>PC:4013e0  55%<br>PC:4013b4 30%<br> Function Name: <strong>init_weights</strong><br>PC:402079 76% | 32   | 
| Vertices:800000<br> Degree:200  |   <strong>do_work</strong>: 37.01% <br><strong>init_weights</strong>: 31.44%<br> <strong>main</strong>: 0.04%   |Function Name:    <strong>do_work</strong><br>PC:4013e0  49%<br>PC:4013b4 34%<br> Function Name: <strong>init_weights</strong><br>PC:402079 57%<br>PC:4020f2 40% | 32   | 
| Vertices:800000<br> Degree:300  |   <strong>do_work</strong>: 35.94% <br><strong>init_weights</strong>: 26.88%<br> <strong>main</strong>: 0.03%   |Function Name:    <strong>do_work</strong><br>PC:4013e0  51%<br>PC:4013b4 31%<br> Function Name: <strong>init_weights</strong><br>PC:402079 52%<br>PC:4020f2 45% | 32   | 
| Vertices:900000<br> Degree:400  |   <strong>do_work</strong>: 32% <br><strong>init_weights</strong>: 26.26%<br> <strong>main</strong>: 0.03%   |Function Name:    <strong>do_work</strong><br>PC:4013e0  50%<br>PC:4013b4 29%<br> Function Name: <strong>init_weights</strong><br>PC:402079 52%<br>PC:4020f2 46%<br>Function Name: <strong>main</strong><br>PC:401695 100%| <span style="color:blue">1/2/8/32</span>   | 
| Vertices:800000<br> Degree:600  |   <strong>do_work</strong>: 26.91% <br><strong>init_weights</strong>: 25.25%   |Function Name:    <strong>do_work</strong><br>PC:4013e0  49%<br>PC:4013b4 29%<br> Function Name: <strong>init_weights</strong><br>PC:402079 54%<br>PC:4020f2 43% | 32 | 
| Vertices:800000<br> Degree:800  |   <strong>do_work</strong>: 29.33% <br><strong>init_weights</strong>: 25.04%<br> <strong>main</strong>: 0.2%   |Function Name:    <strong>do_work</strong><br>PC:4013e0  53%<br>PC:4013b4 24%<br> Function Name: <strong>init_weights</strong><br>PC:402079 54%<br>PC:4020f2 44%<br>Function Name: <strong>main</strong><br>PC:401695 100%| <span style="color:blue">96/75/94/54/32</span>   | 



|    bc   | func that causes<br>most LLC-miss | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:							|   :----:			| :----: |
| Vertices:56384<br> Degree:8  |   <strong>do_work</strong>: 48.41%  |Function Name:    <strong>do_work</strong><br>PC:401561  70% | 32   | 
| Vertices:40000<br> Degree:10  |   <strong>do_work</strong>: 12.53%  |Function Name:    <strong>do_work</strong><br>PC:401561 55%<br>PC:401565 25% | 32   | 
| Vertices:20000<br> Degree:50  |   <strong>do_work</strong>: 5.45%  |Function Name:   <strong>do_work</strong><br>PC:401561  39%<br>PC:40156b 33% | 32   | 
| Vertices:20000<br> Degree:100  |   <strong>do_work</strong>: 40.84%  |Function Name:    <strong>do_work</strong><br>PC:401561  73% | <span style="color:blue">32x3/394/418</span>   |
 | Vertices:20000<br> Degree:200  |  <strong>do_work</strong>: 27.41%  |Function Name:   <strong>do_work</strong><br>PC:401561  83% | <span style="color:blue"></span>32   |
| Vertices:20000<br> Degree:300  |   <strong>do_work</strong>: 87.95%<br> <strong>init_weights</strong>: 0.01%  |Function Name:  <strong>do_work</strong><br>PC:401561  90% | <span style="color:blue">390/336/32/327</span>   |
| Vertices:10000<br> Degree:500  |   <strong>do_work</strong>: 88.29%<br> <strong>init_weights</strong>: 0.02% |Function Name:    <strong>do_work</strong><br>PC:401561  62%<br>PC:401565 36% | <span style="color:blue">32/375/377/392/346</span>   | 
| Vertices:20000<br> Degree:700  |   <strong>do_work</strong>: 89.91%<br> <strong>init_weights</strong>: 0.01%  |Function Name:  <strong>do_work</strong><br>PC:401561  77% | <span style="color:blue">400/392x2/405</span>   |
| Vertices:10000<br> Degree:1000  |   <strong>do_work</strong>: 90.83%<br> <strong>init_weights</strong>: 0.01%  |Function Name:    <strong>do_work</strong><br>PC:401561  84% | <span style="color:blue">278/279/293</span>   | 




|    sssp   | func that causes<br>most LLC-miss | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:							|   :----:			| :----: |
 Vertices:100000<br> Degree:5  |   <strong>do_work</strong>: 99.64%  |Function Name:    <strong>do_work</strong><br>PC: 401739 62%<br>PC:401746 36% | 32   | 
 Vertices:100000<br> Degree:50 |   <strong>do_work</strong>: 99.28%<br><strong>make_dot_graph</strong>: 0.01%<br><strong>init_weights</strong>: 0.01%  |Function Name:   <strong>do_work</strong><br>PC: 401739 79% | 298/427  | 
 Vertices:100000<br> Degree:250|   <strong>do_work</strong>: 98.28%<br><strong>make_dot_graph</strong>: 0.06%<br><strong>init_weights</strong>: 0.03%  |Function Name:    <strong>do_work</strong><br>PC: 401739 87% | 334  | 