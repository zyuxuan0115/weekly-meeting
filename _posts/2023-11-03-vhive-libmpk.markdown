---
layout: post
title:  "2023-11-03 vHive & libmpk"
date:   2023-11-03 1:53:46 -0500
categories: serverless functions
---
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

### libmpk
- [github repo](https://github.com/sslab-gatech/libmpk), & [paper](https://www.usenix.org/system/files/atc19-park-soyeon.pdf)
- normal `pkey_alloc()`, `pkey_free()` 
	+ explainations are [here](https://lwn.net/Articles/689395/)
	+ example is [here](https://www.phoronix.com/news/Linux-4.9-Mem-Protection-Keys)
	+ on Intel Skylake server CPUs, and CPUs after Skylake

