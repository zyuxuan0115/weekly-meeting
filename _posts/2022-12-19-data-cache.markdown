---
layout: post
title:  "2022-12-19 data cache"
date:   2022-12-13 10:53:46 -0500
categories: data-cache 
---

|    pagerank<br> 1000 iterations   | prefetch dist | execution time <br> original | execution time <br> optimized | average<br> speedup |
| :----:        |    :----:							|   :----:			| :----: |  :----: |  :----: |  :----: |
| <strong>web-Google</strong><br>Vertices:916428<br> Degree:456   |   32   | 28.596261(32)<br>28.825631(32)<br>29.023874(32)<br>28.744529(32)<br>29.537645(32) | 21.337055(32)<br>20.768182(32)<br>21.362526(32)<br>20.829610(32)<br>21.191370(32) | 27.112% |
| <strong>web-BerkStan</strong><br>Vertices:685231<br> Degree:249   |   242<br>236x2<br>231<br>238   | 14.705026(242)<br>14.272114(236)<br>14.425358(231)<br>14.531888(236)<br>14.677635(238) | 8.882512(242)<br>9.076098(236)<br>8.808531(231)<br>8.811421(236)<br>9.073147(238) | 38.505% |
| <strong>web-BerkStan</strong><br>optimized binary <br>from web-Google  |   32     |  | 8.834033(32)<br>8.888177(32)<br>8.880781(32)<br>8.925810(32)<br>8.972979(32) | 38.712% |
| <strong>web-BerkStan</strong><br>([20:],[5,15])  | 166<br>134<br>123<br>6<br>7 | 14.507541(166)<br>14.983151(134)<br>14.999300(123)<br>14.400081(6)<br>14.345355(7)     | 9.057653(166)<br>9.077510(134)<br>8.956206(123)<br>10.673848(6)<br>10.581121(7) | 39.107%(120+)<br>26.058%(10-) | 


The execution time for producing an optimized binary (`perf record` time (20s) is included):

|      | real | user | system | 
| :----: |:----:|   :----:| :----: | 
| web-Google  |  3m15.850s| 3m26.829s | 2m40.915s | 
| web-BerkStan | 3m12.146s | 3m23.843s | 2m38.148s |
| web-Stanford | 2m43.879s | 3m0.540s | 2m43.761s |
| web-NotreDame| 3m8.183s | 3m11.062s | 2m51.289s |
| roadNet-PA | 3m31.861s | 3m47.807s | 2m41.504s |
| roadNet-CA | 3m43.796s | 3m58.652s | 2m42.920s |