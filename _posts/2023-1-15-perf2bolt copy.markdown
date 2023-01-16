---
layout: post
title:  "2022-1-15 BAT related"
date:   2023-1-15 10:53:46 -0500
categories: cont-opt 
---

- In [RewriteInstance::preprocessProfileData()](https://github.com/upenn-acg/BOLT/blob/main/bolt/lib/Rewrite/RewriteInstance.cpp#L2616) 
    + the function calls [ProfileReader->setBAT(&*BAT);](https://github.com/upenn-acg/BOLT/blob/main/bolt/lib/Rewrite/RewriteInstance.cpp#L2629)