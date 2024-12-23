---
title: "LibTorch Tensor结构转为OpenCV数据结构"
date: 2024-12-23
tags: [Torch, OpenCV]
---

最近工作上需要将LibTorch Tensor结构，具体来说是特征点坐标信息和描述符信息，转为OpenCV的KeyPoint和Mat结构，调用后续API。直接用AI写了个，发现效率非常低，只能继续改改，改了以后，性能从几百ms降低到0.1ms左右。

```c++
void convertToOpenCV(const torch::Tensor& pts, const torch::Tensor& desc,
                     std::vector<cv::KeyPoint>& keypoints, cv::Mat& descriptors) {
    Timer xx("convertToOpenCV");

    // 确保输入Tensor是CPU上的浮点数类型，并且是连续的
    auto pts_cpu = pts.to(torch::kCPU).to(torch::kFloat32).contiguous();
    auto desc_cpu = desc.to(torch::kCPU).to(torch::kFloat32).contiguous();

    // 获取关键点数量
    int num_keypoints = pts_cpu.size(0);

    // 清空输出变量
    keypoints.clear();
    keypoints.reserve(num_keypoints);  // 预分配空间以避免多次分配

    // 创建描述符矩阵
    descriptors.create(num_keypoints, 128, CV_32F);

    // 使用指针直接访问Tensor数据
    const float* pts_data = pts_cpu.data_ptr<float>();
    const float* desc_data = desc_cpu.data_ptr<float>();

    for (int i = 0; i < num_keypoints; ++i) {
        // 从 pts Tensor 中提取 x 和 y 坐标
        float x = pts_data[i * 2];
        float y = pts_data[i * 2 + 1];

        // 创建 cv::KeyPoint 对象并添加到 vector 中
        cv::KeyPoint kp(x, y, 1.0f); // 这里的 size 可以根据需要调整
        keypoints.push_back(kp);

        // 将描述符数据复制到 OpenCV Mat
        std::memcpy(descriptors.ptr<float>(i), desc_data + i * 128, 128 * sizeof(float));
    }
}
```