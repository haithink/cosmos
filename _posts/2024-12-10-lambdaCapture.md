objA---
title: "获取YouTube栏目视频标题"
date: 2024-12-10
tags: [c++, lambda, 多线程]
---

今天开发一个功能，需要对接到同事的代码，看到其中一个代码片段
```c++
    std::vector<std::thread> threads;
    for (int i = 0; i < objA.size(); i++)
    {
        threads.push_back(std::thread([this, &objA, i, &res]()
                                        { 
                                            xxx[i]->func(objA[i], res[i].first, res[i].second); 
                                        }));
    }

    for (auto &th : threads)
    {
        if (th.joinable())
        {
            th.join();
        }
    }
```
以下讨论基于 objA的大小为2。


自己需要把这个代码用到另外一个封装的接口上，即，xxx改为yyy，第一反应觉得捕获列表太长了。于是，把[this, &objA, i, &res]直接改成了[&], 然后运行的时候就出现奇葩的内存报错问题。然后测试发现 i的上限如果是1，那么不会崩溃，如果i从1开始那么会崩溃。单独测试每个xxx[i]->func调用也不会报错。但如果像上面这样，就会出现某个xxx[i]对象的成员变量像是释放了一样，体现出随机值。

问AI，AI说了些无关痛痒的问题。然后又看了一遍代码，才发现**i**这个变量单独拎出来使用了值捕获，顿时意识到问题。如果i使用引用捕获和传递，那么for循环的时候i变成2，thread构造函数里面此时访问的i也变成2，那么自然就越界了！！！除非此时thread已经用合理的i构造完毕，但目前看这种概率非常低。。。

这个问题困扰了自己至少一个消失！

