---
layout: post
title:  "2024-03-17 RPG2 presentation"
date:   2024-03-17 1:53:49 -0500
categories: data cache
---

### When you are asked insert data prefetch during program's execution
#### What to prefetch?
- prefetch accuracy

#### When to prefetch?
- prefetch timeliness
	+ <strong>too early</strong>: prefetch a lot useless data and will finally be kicked out of the cache by the new data
	+ <strong>too late</strong>: when the data is needed by the processor, the data has not been brought to the cache
 
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

#### How to insert prefetch at runtime?
- if prefetch is inserted, all the PC dependent instructions will need to be updated 

#### useful figures are at 
- 2023-04-21
- 2024-05-05
- 2024-05-22 
