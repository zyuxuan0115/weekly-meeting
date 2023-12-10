---
layout: post
title:  "2023-12-08 Merge UniqID+ComposePost"
date:   2023-12-08 1:53:46 -0500
categories: serverless functions
---
### if we are to merge UniqueId & ComposePost
- there are 6394 functions in `UniqueIdService` in total
- we first need to get a list of functions symbols that occur in both `libUniqueIdService.so` and `libComposePostService.so`
	+ `_ZN14social_network11init_loggerEv`: src/logger.h
	+ `_ZN14social_network16load_config_fileERKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEEPN8nlohmann10basic_jsonISt3mapSt6vectorS5_blmdSaNS8_14adl_serializerEEE`: src/utils.h 
	+ `_ZN14social_network11load_configEPN8nlohmann10basic_jsonISt3mapSt6vectorNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEEblmdSaNS0_14adl_serializerEEE`: src/logger.h
	+ `_ZN14social_network11SetUpTracerERKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEES7_`: src/tracing.h
- all the functions are from some header files. So they can be removed


- need to add debug info to source code
	+ [here](https://github.com/zyuxuan0115/misc-2023-11-20/blob/main/dmon/llvm-passes/selective-prefetch/Prefetch.cpp#L254) is how DMon read debug info from LLVM's optimization pass
