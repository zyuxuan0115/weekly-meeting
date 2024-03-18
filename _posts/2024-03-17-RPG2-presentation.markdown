---
layout: post
title:  "2024-03-17 RPG2 presentation"
date:   2024-03-17 1:53:49 -0500
categories: serverless functions
---

### When you are asked insert data prefetch during program's execution
#### What to prefetch?
- prefetch accuracy

#### When to prefetch?
- prefetch timeliness
	+ too early: kick out of the cache by the new data
	+ too late:
 
```c
for (int i=0; i<1024; i++){
  a[i] = b[i] * c[i];
}
```

```c
for (int i=0; i<N; i++){
  for (int j=0; j<M; j++){
    a[i] = b[c[i]+j];
  }
}
```
