---
layout: post
title:  "2022-12-05 data cache"
date:   2022-11-28 10:53:46 -0500
categories: data-cache 
---

|    pagerank   | deliquent load PCs<br> for each function| prefetch dist | execution time <br> original | execution time <br> optimized | 
| :----:        |    :----:							|   :----:			| :----: |  :----: |  :----: |  :----: |
| <strong>web-Google</strong><br>Vertices:916428<br> Degree:456     |    <strong>main</strong><br>PC:401e9b  43%<br>PC:401e7a 29%<br> <strong>do_work</strong><br>PC:401460 72%| 32   | 0.030814<br>0.029949 | 0.022874<br>0.023516 |
| <strong>web-BerkStan</strong><br>Vertices:685231<br> Degree:249   |    <strong>main</strong><br>PC:401e9b  40%<br>PC:401e7a 37%<br> <strong>do_work</strong><br>PC:401460 99%| <span style="color:steelblue">1/3/4/32</span>   | 0.015497(4)<br>0.015798(4) | 0.010233(4)<br>0.010588(4) |
| <strong>web-Stanford</strong><br>Vertices:281904<br> Degree:255    |    <strong>main</strong><br>PC:401e9b  47%<br>PC:401e7a 38%<br> <strong>do_work</strong><br>PC:401460 91%| 32   | 0.005862<br>0.006024 | 0.003551<br>0.003584 |
| <strong>web-NotreDame</strong><br>Vertices:325729<br> Degree:3445   | <strong>do_work</strong><br>PC:401460 89%| 32   | 0.011595<br>0.010907 | 0.009481<br>0.009318| 
| <strong>roadNet-PA</strong><br>Vertices:1090920<br> Degree:9   | <strong>do_work</strong><br>PC:401460 93%| 32   | 0.022309<br>0.022705 | 0.014735<br>0.014760 |
| <strong>roadNet-CA</strong><br>Vertices:1971281<br> Degree:12  | <strong>do_work</strong><br>PC:401460 82%| <span style="color:steelblue">32/5x2</span>   | 0.050215(5)<br>0.051477(5) | 0.035544(5)<br>0.036379(5) | 



|    bfs   | deliquent load PCs<br> for each function| prefetch dist |execution time <br> original | execution time <br> optimized | 
| :----:    |   :----:	|   :----:	| :----: | :----: | :----: |
| <strong>p2p-Gnutella30</strong><br>Vertices:36681<br> Degree:  |  <strong>do_work</strong><br>PC:401471  70% |  32  |  0.003414<br>0.003299| 0.003350<br>0.003353 |
| <strong>loc-brightkite_edges</strong><br>Vertices:36681<br> Degree:456  |  <strong>do_work</strong><br>PC:401471  99%<br> <strong>main</strong><br>PC:401826 44% <br>PC:401b5b 36%| 32   |35.670361<br>35.651051 | 35.808314<br>35.279528 | 
| Vertices:200000<br> Degree:2  |   <strong>do_work</strong><br>PC:401471  99% | <span style="color:steelblue">102/107/<br>100/96</span> | 104.834(111)<br>107.769(101) | 104.783(111)<br>108.163(101)|
| Vertices:100000<br> Degree:4  |   <strong>do_work</strong><br>PC:401471  98% | <span style="color:steelblue">205/200/<br>202/32x3</span> | 11.646(196)<br>11.880(209) | 11.902(196)<br>11.892(209)| 
| Vertices:300000<br> Degree:6  |   <strong>do_work</strong><br>PC:401471  99% | <span style="color:steelblue">286/32x2/282</span>  | 94.722(315)<br>97.477(318) | 96.457(315)<br>95.788(318)|
| Vertices:100000<br> Degree:8  |   <strong>do_work</strong><br>PC:401471  99% | 32   | 6.738500<br>6.759814 | 6.715438<br>6.947777 | 
| Vertices:100000<br> Degree:16  |   <strong>do_work</strong><br>PC:401471  99% | 32   | 4.156175<br>4.245715 | 4.272679<br>4.287155 | 
| Vertices:200000<br> Degree:32  |   <strong>do_work</strong><br>PC:401471  99% | 32   | 18.635840<br>19.226318 | 17.406571<br>17.340986 | 
| Vertices:300000<br> Degree:64  |   <strong>do_work</strong><br>PC:401471  99% | 32   | 39.042188<br>39.979307 | 40.633055<br>41.062946 | 
| Vertices:200000<br> Degree:128  |   <strong>do_work</strong><br>PC:401471  99% | 32   | 12.508692<br>12.507585 | 12.311354<br>12.815299 | 
| Vertices:200000<br> Degree:150  |   <strong>do_work</strong><br>PC:401471  99% | 32   | 11.226(353)<br>11.491(364) | 11.42(353)<br>11.197(364)|
| Vertices:300000<br> Degree:200  |   <strong>do_work</strong><br>PC:401471  98% | 32   | 25.933(456)<br>26.321597 | 25.986(456)<br>24.500941 |
| Vertices:300000<br> Degree:256  |   <strong>do_work</strong><br>PC:401471  98% | 32   | 22.860<br>22.651(457)| 24.030<br>24.003(457) | 
| Vertices:200000<br> Degree:400  |   <strong>do_work</strong><br>PC:401471  98% | 32   | 9.018355<br>9.070743 | 9.009553<br>9.040988 | 
| Vertices:200000<br> Degree:512  |   <strong>do_work</strong><br>PC:401471  98% | 32   | 9.326931<br>9.186632 | 9.524787<br>9.283280 |







|    dfs   | deliquent load PCs<br> for each function| prefetch dist | deliquent load PCs<br> for each function| prefetch dist |
| :----:        |    :----:	 |   :----:			| :----: | :----: |
| Vertices:800000<br> Degree:100  |   <strong>do_work</strong><br>PC:4013e0  55%<br>PC:4013b4 30%<br>  <strong>init_weights</strong><br>PC:402079 76% | 32   | 0.670312<br>0.662219 | 0.651666<br>0.646888 |
| Vertices:800000<br> Degree:200  |   <strong>do_work</strong><br>PC:4013e0  49%<br>PC:4013b4 34%<br>  <strong>init_weights</strong><br>PC:402079 57%<br>PC:4020f2 40% | 32   | 1.191269<br>1.189988 | 1.151985<br>1.160511 |
| Vertices:800000<br> Degree:300  |   <strong>do_work</strong><br>PC:4013e0  51%<br>PC:4013b4 31%<br>  <strong>init_weights</strong><br>PC:402079 52%<br>PC:4020f2 45% | 32   | 1.750412<br>1.682633 | 1.662303<br>1.676280 | 
| Vertices:900000<br> Degree:400  |   <strong>do_work</strong><br>PC:4013e0  50%<br>PC:4013b4 29%<br>  <strong>init_weights</strong><br>PC:402079 52%<br>PC:4020f2 46%<br> <strong>main</strong><br>PC:401695 100%| <span style="color:steelblue">1/2/<br>8/32</span>   | 2.485635(7)<br>2.485459(2) | 2.464855(7)<br>2.496771(2)|
| Vertices:800000<br> Degree:600  |   <strong>do_work</strong><br>PC:4013e0  49%<br>PC:4013b4 29%<br>  <strong>init_weights</strong><br>PC:402079 54%<br>PC:4020f2 43% | 32 | 3.186280<br>3.177909(113) | 3.164332<br>3.199197(113) |
| Vertices:800000<br> Degree:800  |   <strong>do_work</strong><br>PC:4013e0  53%<br>PC:4013b4 24%<br>  <strong>init_weights</strong><br>PC:402079 54%<br>PC:4020f2 44%<br> <strong>main</strong><br>PC:401695 100%| <span style="color:steelblue">96/75/94/<br>54/32</span>   | 4.205640(32)<br>4.229977 | 4.178431(32)<br>4.130079| 



|    bc   | deliquent load PCs<br> for each function| prefetch dist | execution time <br> original | execution time <br> optimized | 
| :----:        |    :----:							|   :----:			| :----: | :----: |  :----: |
| Vertices:56384<br> Degree:8  |    <strong>do_work</strong><br>PC:401561  70% | 32   | 78.197884<br>77.895855 | 67.574543<br>67.412499 |
| Vertices:40000<br> Degree:10  |   <strong>do_work</strong><br>PC:401561 55%<br>PC:401565 25% | 32   | 47.212300<br>47.383155 | 42.218670<br>42.056944 |
| Vertices:20000<br> Degree:50  |   <strong>do_work</strong><br>PC:401561  39%<br>PC:40156b 33% | 32   | 24.023(445)<br>24.144(293) | 23.207(445)<br>23.187(293)| 
| Vertices:20000<br> Degree:100  |  <strong>do_work</strong><br>PC:401561  73% | <span style="color:steelblue">32x3/394/418</span>   |41.965(372)<br>41.559(399)| 40.331(372)<br>40.764(399)|
 | Vertices:20000<br> Degree:200  | <strong>do_work</strong><br>PC:401561  83% | <span style="color:steelblue"></span>32   |86.736(380)<br>85.793(379) | 151.152(380)<br>149.333(379)|
| Vertices:20000<br> Degree:300  |  <strong>do_work</strong><br>PC:401561  90% | <span style="color:steelblue">390/336/32/327</span>   | 137.194(395)<br>149.333(422) | 228.027(395)<br>227.285(422) |
| Vertices:10000<br> Degree:500  |  <strong>do_work</strong><br>PC:401561  62%<br>PC:401565 36% | <span style="color:steelblue">32/375/377/<br>392/346</span>   | 50.337(371)<br>49.825(32) | 89.602(371)<br>72.573(32) | 
| Vertices:20000<br> Degree:700  |  <strong>do_work</strong><br>PC:401561  77% | <span style="color:steelblue">400/392x2/405</span>   | 330.147(468)<br>327.785(382) | 536.279(468)<br>530.097(382)|
| Vertices:10000<br> Degree:1000  | <strong>do_work</strong><br>PC:401561  84% | <span style="color:steelblue">278/279/293</span>   | 102.086(313)<br>102.650(299) |  174.034(313)<br>171.788(299)   |




|    sssp   |  deliquent load PCs<br> for each function| prefetch dist | execution time <br> original | execution time <br> optimized | 
| :----:    |    :----:		|   :----:	| :----: | :----: |
 Vertices:100000<br> Degree:5  |     <strong>do_work</strong><br>PC: 401739 62%<br>PC:401746 36% | 32   | 72.024923<br>72.461733 | 70.580886<br>70.372966 |
 Vertices:100000<br> Degree:50 |    <strong>do_work</strong><br>PC: 401739 79% | <span style="color:steelblue">298/427/<br>32/429</span>  | 187.709(431)<br>188.253(439) | 192.303(431)<br>191.526(439) |
 Vertices:100000<br> Degree:100 |     <strong>do_work</strong><br>PC: 401739 84% | <span style="color:steelblue">429/412/299</span>  | 224.3(429)<br>223.607(319) | 231.641(429)<br>230.736(319)|
 Vertices:100000<br> Degree:250|    <strong>do_work</strong><br>PC: 401739 87% | <span style="color:steelblue">334/330/458</span>  | 295.647(439)<br>297.419(332) | 302.076(439)<br>305.930(332) |
 Vertices:100000<br> Degree:350 |   <strong>do_work</strong><br>PC: 401739 84% | <span style="color:steelblue">431x2/426</span>  | 282.891(301)<br>285.975(454) | 292.527(301)<br>297.341(454) |
 Vertices:100000<br> Degree:500 |   <strong>do_work</strong><br>PC: 401739 68%<br>PC: 401746 31% | <span style="color:steelblue">453/412/<br>452/399</span>  | 304.531(422)<br>309.046(331) | 329.614(422)<br>331.354(331)|
 Vertices:100000<br> Degree:750 |   <strong>do_work</strong><br>PC: 401739 84% | <span style="color:steelblue">301/309/<br>375/306</span>  | 306.793(318)<br>310.242(305) | 330.141(318)<br>329.943(305)|
 Vertices:100000<br> Degree:1000|   <strong>do_work</strong><br>PC: 401739 85% |  <span style="color:steelblue">377/368/<br>362/374</span> | 299.637<br>301.897(385) | 315.954<br>320.730(385)|
