---
name: Thread pool isolation
description: Each data source (AMAP, Qixinbao, Network, Youzan) must have its own independent thread pool, never share
type: feedback
---

每个数据源必须使用独立线程池，不能复用，否则会互相阻塞导致卡死。

**Why:** 用户明确指出线程池复用会"卡死"，说明之前遇到过相关问题。并行数据源（高德、启信宝、联网搜索、有赞小程序查询）如果共享线程池，慢的数据源会占满线程，导致快的也被阻塞。嵌套 CompletableFuture 提交到同一线程池会导致线程饥饿死锁。

**How to apply:** 在 crm-master 的 Enrichment 模块中，为每个数据源配置独立的 ThreadPoolExecutor（独立的 core/max/queue）。新增数据源也需各自独立线程池。
