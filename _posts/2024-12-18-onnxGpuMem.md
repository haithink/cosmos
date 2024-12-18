---
title: "ONNX c++运行时占用大量显存的问题"
date: 2024-12-18
tags: [onnx, 显存]
---

本来也没太关注模型推理时的内存占用问题，但昨天说看看能不能降低ONNX执行模型推理时的显存占用大小。问了几个老司机，都没关注过这个问题。

模型本身只有5MB，但推理时占用显存1.5GB。一开始以为时有个输出节点输出内容较大（300MB）导致的，但去掉这个节点，发现显存占用毫无变化。使用Pytorch加载对应格式的模型，发现占用的显存更多，看来ONNX还是比较节约显存的了。Pytorch加载随便设计的一个很小的卷积模型，占用显存都是1GB以上，看来不是模型本身的锅啊。

把目前搜集到的资料和代码备份下：

https://github.com/microsoft/onnxruntime/issues/13380<br>
https://github.com/microsoft/onnxruntime/issues/11903<br>
https://github.com/microsoft/onnxruntime/issues/11801#issuecomment-1151505137<br>
https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html#convolution-heavy-models<br>

实测有用的代码有：
```c++

	Ort::RunOptions run_options;
	// 降低峰值，但推理耗时略微上升
    run_options.AddConfigEntry("memory.enable_memory_arena_shrinkage", "cpu:0;gpu:0");
	
	OrtCUDAProviderOptions cuda_options{};

    cuda_options.device_id = 0;  // 指定要使用的 GPU 设备 ID
	// 如下选项确实有用
    cuda_options.arena_extend_strategy = 1;  // 内存分配策略
    // cuda_options.gpu_mem_limit = 1000 * 1024 * 1024;  // 设置 GPU 显存限制（例如：2GB）
    // cuda_options.cudnn_conv_algo_search = OrtCudnnConvAlgoSearchHeuristic;  // cuDNN 卷积算法搜索策略
    // cuda_options.do_copy_in_default_stream = 1;  // 是否在默认流中进行内存拷贝	

	sessionOptions.AppendExecutionProvider_CUDA(cuda_options);	
```
