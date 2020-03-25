---
id: index.md
title: Index Types
sidebar_label: Index Types
---

# Milvus 索引类型

## 索引概览

> 注意：索引的实际构建方式（使用 CPU 或者 GPU）不影响索引本身对 CPU 或 GPU 搜索的支持。

### 仅支持 CPU 的 Milvus 包含的索引类型

<div class="table-wrapper" markdown="block">

| 名称       | 支持 CPU 建立索引 | 支持 CPU 搜索 | 支持浮点型向量 | 支持二值型向量 |
| ---------- | ----------------- | ------------- | -------------- | -------------- |
| `FLAT`     | ✔️                | ✔️            | ✔️             | ✔️             |
| `IVFLAT`   | ✔️                | ✔️            | ✔️             | ✔️             |
| `IVF_SQ8`  | ✔️                | ✔️            | ✔️             | ❌             |
| `IVF_SQ8H` | ❌                | ❌            | ✔️             | ❌             |
| `IVF_PQ`   | ✔️                | ✔️            | ✔️             | ❌             |
| `RNSG`     | ✔️                | ✔️            | ✔️             | ❌             |
| `HNSW`     | ✔️                | ✔️            | ✔️             | ❌             |

</div>

### 支持 GPU 的 Milvus 包含的索引类型

<div class="table-wrapper" markdown="block">

| 名称       | 支持 CPU 建立索引 | 支持 CPU 搜索 | 支持 GPU 建立索引                                | 支持 GPU 搜索                                    | 支持浮点型向量 | 支持二值型向量 |
| ---------- | ----------------- | ------------- | ------------------------------------------------ | ------------------------------------------------ | -------------- | -------------- |
| `FLAT`     | ✔️                | ✔️            | ✔️ (对于二值型向量，精确搜索索引不支持 GPU 索引) | ✔️ (对于二值型向量，精确搜索索引不支持 GPU 搜索) | ✔️             | ✔️             |
| `IVFLAT`   | ✔️                | ✔️            | ✔️ (对于二值型向量，倒排索引不支持 GPU 索引)     | ✔️ (对于二值型向量，倒排索引不支持 GPU 搜索)     | ✔️             | ✔️             |
| `IVF_SQ8`  | ✔️                | ✔️            | ✔️                                               | ✔️                                               | ✔️             | ❌             |
| `IVF_SQ8H` | ✔️                | ✔️            | ✔️                                               | ✔️                                               | ✔️             | ❌             |
| `IVF_PQ`   | ✔️                | ✔️            | ✔️ (仅对欧氏距离支持 GPU 索引)                   | ✔️ (仅对欧氏距离支持 GPU 搜索)                   | ✔️             | ❌             |
| `RNSG`     | ✔️                | ✔️            | ❌                                               | ❌                                               | ✔️             | ❌             |
| `HNSW`     | ✔️                | ✔️            | ❌                                               | ❌                                               | ✔️             | ❌             |

</div>

> 注意：对于不同索引类型，创建索引的参数和搜索参数也有所不同。详细信息请参考 [Milvus 基本操作](milvus_operation.md)。

## 索引详解

### `FLAT`

如果使用 `FLAT` 索引，向量会以浮点/二进制的方式存储，不做任何压缩处理。搜索时，所有的向量会依次解码并于要搜索的目标向量对比计算距离。

`FLAT` 提供 100%的检索召回率。相比其它索引方式，在搜索量不大的情况下速度最快。

### `IVFLAT`

在聚类时，向量被直接添加到各个分桶中，不做任何压缩。这种基于聚类，多簇搜索的方式搜索速度和准确性都不错。

### `IVF_SQ8`

运用 scalar quantizer 的向量索引，节省存储空间（缩减为原体积的约 1/4 大小）。相比 `FLAT` 搜索速度更快，比 `IVFLAT` 占用存储空间更小。

向量被量化为 8 字节的浮点数，可能造成搜索精度的损失。

### `IVF_SQ8H`

基于 `IVF_SQ8` 做了深层优化，但需要 CPU 和 GPU 都在的情况下才能使用。不同于 `IVF_SQ8`，`IVF_SQ8H` 使用基于 GPU 的 coarse quantizer，能极大减少标准量化时间，提高查询速度。

### `IVF_PQ`

基于乘积量化的索引类型，意思是将原来的向量空间分解为若干个低维向量空间的笛卡尔积，然后对分解得到的低维向量空间分别做量化。

向量大小可以缩减至原来大小的 1/16 甚至 1/32。该索引方式适用于低内存环境下的大规模向量搜索，但搜索精度会有损失，需注意权衡。

目前每个 sub-quantizer 仅支持 1, 2, 3, 4, 6, 8, 10, 12, 16, 20, 24, 28, 32 维。sub-quantizer 总数量仅支持 1, 2, 3, 4, 8, 12, 16, 20, 24, 28, 32, 40, 48, 56, 64, 96。

### `RNSG`

`RNSG` 是 Milvus 自研的一种索引方式，基于 `NSG` 索引做了各种优化。`NSG` 是一种基于图的索引算法，它可以 a) 降低图的平均出度；b) 缩短搜索路径；c) 缩减索引大小；d) 降低索引复杂度。

不同于 `NSG` 单个搜索的方式，`RNSG` 支持多个目标向量的并发搜索。

### `HNSW`

`HNSW` 索引基于 HNSW 构建。HNSW (Hierarchical Small World Graph) 是一种基于图的索引算法，可以增量建立多层结构并且将边根据特征距离半径进行分层。由于计算复杂度是对数，HNSW 对于高维数据非常高效。

与 `RNSG` 相比， `HNSW` 的运行效率和内存使用效率更高。`HNSW` 支持增量建立索引，而 `RNSG` 则不支持。但是，因为图需要加载到内存中，`HNSW` 的内存需求要大于 `RNSG`。

## 如何选择索引

若要为您的使用场景选择合适的索引，请参阅 [如何选择索引类型](https://milvus.io/cn/blogs/2019-12-03-select-index.md)。

关于索引和向量距离计算方法的选择，请访问 [距离计算方式](metric.md)。