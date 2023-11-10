---
layout: post
title:  "2023-11-03 vHive & libmpk"
date:   2023-11-03 1:53:46 -0500
categories: serverless functions
---
### libmpk
- [github repo](https://github.com/sslab-gatech/libmpk), & [paper](https://www.usenix.org/system/files/atc19-park-soyeon.pdf)
- protection key virtualization
	+ even if we use libmpk, we only have 16 keys.
	+ ![s1](/assets/2023-11-03/s1.png) 
- lazy inter-thread key synchronization
	+ is used to accelerate `mpk_mprotect()`
	+ ![s2](/assets/2023-11-03/s2.png)
- metadata integrity
	+ ensure the integrity of the mapping info

### pkey (Intel MPK)
- normal `pkey_alloc()`, `pkey_free()` 
	+ explainations are [here](https://lwn.net/Articles/689395/)
	+ example is [here](https://www.phoronix.com/news/Linux-4.9-Mem-Protection-Keys)
	+ on [Intel Skylake](https://en.wikipedia.org/wiki/List_of_Intel_Xeon_processors_(Skylake-based)) server CPUs, and CPUs after Skylake
- `mprotect` with `pkey` [here](https://man7.org/linux/man-pages/man2/mprotect.2.html)

Before a pkey can be used, it must first be allocated with `pkey_alloc()`. An application calls the `WRPKRU` instruction directly in order to change access permissions to memory covered with a key. In this example `WRPKRU` is wrapped by a C function called `pkey_set()`.

```c++
pkey = pkey_alloc(0, PKEY_DENY_WRITE);
ptr = mmap(NULL, PAGE_SIZE, PROT_NONE, MAP_ANONYMOUS|MAP_PRIVATE, -1, 0);
ret = pkey_mprotect(ptr, PAGE_SIZE, PROT_READ | PROT_WRITE, pkey);
// ... application runs here
```

Now, if the application needs to update the data at 'ptr', it can gain access, do the update, then remove its write access:

```c++
pkey_set(pkey, 0); // clear PKEY_DENY_WRITE
*ptr = foo; // assign something
pkey_set(pkey, PKEY_DENY_WRITE); // set PKEY_DENY_WRITE again
```

Now when it frees the memory, it will also free the pkey since it is no longer in use:

### vHive
- written in GO
	+ how to install GO
		* first [apt install](https://stackoverflow.com/questions/17480044/how-to-install-the-current-version-of-go-in-ubuntu-precise)
		* then upgrade GO to [version 1.19](https://go.dev/dl/)
- how to quick start [here](https://github.com/vhive-serverless/vhive/blob/6a0c478d2c9f/docs/quickstart_guide.md)
	+ some tools vHive uses:
		* [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)
		* [containerd](https://containerd.io/)
		* [firecracker](https://firecracker-microvm.github.io/)
			- Firecracker is a virtual machine monitor (VMM) that uses the Linux Kernel-based Virtual Machine (KVM) to create and manage microVMs. 
		* `pushd` & `popd` & `screen -dmS <name>` & `tee -a`
			- `pushd dir1`: similiar to `cd dir1`
			- `popd`: pop the path of the old pwd
			- `screen -dmS <name>`: Start as daemon: Screen session in detached mode.
- about commit hash
	+ `b0d116cc39d32524b2a2f62002ce4ac7f9d624f7` 
		* can be built successfully on cloudlab machines.
	+ `c49a166ee4a3038418759b28d32a12b78b5ad9d7` 
		* has the `examples/deployer` code
- github issue [reported](https://github.com/vhive-serverless/vHive/issues/875)
	+ `kubectl get pods -A`


