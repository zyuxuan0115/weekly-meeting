---
layout: post
title:  "2022-12-12 apt-get pagerank results"
date:   2022-12-8 10:53:46 -0500
categories: data-cache 
---

|    pagerank   | deliquent load PCs<br> for each function| prefetch dist | execution time <br> original | execution time <br> optimized | 
| :----:        |    :----:							|   :----:			| :----: |  :----: |  :----: |  :----: |
| <strong>web-Google</strong><br>Vertices:916428<br> Degree:456     |    <strong>do_work</strong><br>PC:401500 72%| 32   | 29.899289(32)<br>28.906600(32)<br>28.819703(32)<br>29.006986(32)<br>29.161289(32) | 20.804068(32)<br>20.862082(32)<br>20.766731(32)<br>21.001575(32)<br>21.302885(32)|
| <strong>web-BerkStan</strong><br>Vertices:685231<br> Degree:249   |    <strong>do_work</strong><br>PC:401500 99%| <span style="color:steelblue">234x3<br>237<br>231</span>   | 13.734723(237)<br>13.583152(234)<br>13.818474(234)<br>13.873286(234)<br>13.946716(231) | 8.970430(237)<br>8.749127(234)<br>8.879961(234)<br>8.823190(234)<br>8.828055(231) |
| <strong>web-Stanford</strong><br>Vertices:281904<br> Degree:255    |   <strong>do_work</strong><br>PC:401500 91%| <span style="color:steelblue">212<br>205<br>185</span><br>32x4   | 5.176777(212)<br>5.032269(205)<br>5.041926(185)<br>5.147700(32)<br>5.135883(32)<br>5.201475(32)<br>5.140363(32) | 3.113612(212)<br>3.036380(205)<br>3.048690(185)<br>2.952322(32)<br>2.952870(32)<br>2.955477(32)<br>2.956764(32) |
| <strong>web-NotreDame</strong><br>Vertices:325729<br> Degree:3445   | <strong>do_work</strong><br>PC:401500 89%| 32x4<br><span style="color:steelblue">190x2<br>150</span>   | 10.278070(32)<br>10.366384(32)<br>10.318248(32)<br>10.291323(32)<br>10.060439(190)<br>10.401188(190)<br>10.092987(150)| 8.291560(32)<br>8.160605(32)<br>8.290371(32)<br>8.194220(32)<br>8.014428(190)<br>8.122975(190)<br>8.171565(150)| 
| <strong>roadNet-PA</strong><br>Vertices:1090920<br> Degree:9   | <strong>do_work</strong><br>PC:401500 93%| 32   | 21.585450(32)<br>21.556787(32)<br>20.167020(32)<br>20.176205(32)<br>20.513408(32) | 13.722496(32)<br>13.392380(32)<br>12.970168(32)<br>12.765780(32)<br>13.223683(32) |
| <strong>roadNet-CA</strong><br>Vertices:1971281<br> Degree:12  | <strong>do_work</strong><br>PC:401500 82%| <span style="color:steelblue">296<br>347</span><br>32x4  | 39.372540(296)<br>39.675003(347)<br>42.202213(32)<br>40.789358(32)<br>39.423965(32)<br>39.596660(32) | 27.626210(296)<br>27.676888(347)<br>27.141911(32)<br>26.565268(32)<br>27.210036(32)<br>26.928825(32) | 



|    pagerank<br>([20:][5,15])   | deliquent load PCs<br> for each function| prefetch dist | execution time <br> original | execution time <br> optimized | 
| :----:        |    :----:							|   :----:			| :----: |  :----: |  :----: |  :----: |
| <strong>web-Google</strong><br>Vertices:916428<br> Degree:456     |    <strong>do_work</strong><br>PC:401500 72%|  107<br>82<br>141<br>75<br>95  | 29.571503(107)<br>28.814451(82)<br>29.039713(141)<br>29.683625(75)<br>29.248524(95) | 21.171014(107)<br>21.151259(82)<br>20.738358(141)<br>21.211768(75)<br>21.156784(95) |
| <strong>web-BerkStan</strong><br>Vertices:685231<br> Degree:249   |    <strong>do_work</strong><br>PC:401500 99% |  5x2<br>6<br>4<br>7  | 13.910013(5)<br>13.976824(6)<br>13.991151(5)<br>13.976214(4)<br>13.874109(7) | 11.359223(5)<br>10.893891(6)<br>11.378976(5)<br>11.794827(4)<br>10.467014(7) |
| <strong>web-Stanford</strong><br>Vertices:281904<br> Degree:255    |   <strong>do_work</strong><br>PC:401500 91% |  43<br>64<br>46<br>45x2  | 5.071551(43)<br>5.054031(64)<br>5.030575(46)<br>4.998892(45)<br>4.951986(45) | 2.897825(43)<br>2.848110(64)<br>2.898914(46)<br>2.856269(45)<br>2.883441(45) |
| <strong>web-NotreDame</strong><br>Vertices:325729<br> Degree:3445   | <strong>do_work</strong><br>PC:401500 89%| 85<br>71<br>67<br>101<br>107<br>73 |10.308179(85)<br>10.437574(71)<br>10.335972(67)<br>10.378235(101)<br>9.945183(107)<br>10.390319(73)| 8.232225(85)<br>8.276260(71)<br>8.258278(67)<br>8.368134(101)<br>8.012450(107)<br>8.167111(73)| 
| <strong>roadNet-PA</strong><br>Vertices:1090920<br> Degree:9   | <strong>do_work</strong><br>PC:401500 93%| 62<br>63<br>83<br>102<br>79   | 20.585552(62)<br>20.694978(63)<br>20.380082(83)<br>20.762681(102)<br>20.281646(79) | 13.260643(62)<br>13.357712(63)<br>13.270489(83)<br>13.774247(102)<br>13.214693(79) |
| <strong>roadNet-CA</strong><br>Vertices:1971281<br> Degree:12  | <strong>do_work</strong><br>PC:401500 82%| 177<br>63<br>100<br>81<br>73   | 40.354689(177)<br>40.901820(63)<br>40.773215(100)<br>41.126592(81)<br>40.298437(73) | 27.933310(177)<br>27.099066(63)<br>27.324560(100)<br>27.239035(81)<br>26.302358(73) | 



synthetic workloads for pagerank.

|    pagerank<br>   | deliquent load PCs<br> for each function| prefetch dist | execution time <br> original | execution time <br> optimized | 
| :----:        |    :----:							|   :----:			| :----: |  :----: |  :----: |  :----: |
| Vertices:200000<br> Degree:200 | <strong>do_work</strong><br>PC:401500 99% | 32x4<br>235 | 13.538791(32)<br>13.517384(32)<br>13.179158(32)<br>13.215739(32)<br>13.273866(235) | 13.428654(32)<br>13.452878(32)<br>12.988537(32)<br>13.176782(32)<br>13.013297(235) |


|    pagerank<br>([20:][5,15])   | deliquent load PCs<br> for each function| prefetch dist | execution time <br> original | execution time <br> optimized | 
| :----:        |    :----:							|   :----:			| :----: |  :----: |  :----: |  :----: |
| Vertices:200000<br> Degree:200 | <strong>do_work</strong><br>PC:401500 99% | 182<br>141<br>120<br>167<br>101 | 13.051599(182)<br>13.353129(141)<br>13.442306(120)<br>13.469791(167)<br>13.384551(101) | 13.028119(182)<br>13.406983(141)<br>13.277799(120)<br>13.349196(167)<br>13.258522(101) |




