# CacheSystem

一个纯 C++17 标准库实现的**线程安全缓存系统**，实现了 LRU、LFU、ARC、LRU-K、LFU-Aging 五种缓存淘汰策略，核心代码约 1100 行，无任何外部依赖。

> 本项目基于 [youngyangyang04/KamaCache](https://github.com/youngyangyang04/KamaCache) 修改而来，感谢原作者 [代码随想录](https://programmercarl.com/) 的开源分享。

## 特性

- **五种淘汰策略**：LRU、LFU、ARC、LRU-K、LFU-Aging，覆盖不同业务场景
- **线程安全**：互斥锁 + 原子操作，保证多线程下数据一致性
- **缓存分片（Sharding）**：按 key 哈希分片，降低锁竞争，提升并发性能
- **LFU 老化机制**：防止历史高频数据长期占用缓存
- **零外部依赖**：纯 C++17 标准库，无需 Boost 等第三方库
- **跨平台**：Windows / Linux / macOS 均可编译运行

## 缓存策略

| 策略 | 淘汰思路 | 适用场景 |
|------|---------|---------|
| LRU | 淘汰最久未访问的数据 | 一般热点访问 |
| LFU | 淘汰访问频率最低的数据 | 长期稳定热点 |
| ARC | LRU 与 LFU 自适应平衡 | 访问模式多变 |
| LRU-K | 访问满 K 次才进入缓存 | 区分热点与冷数据 |
| LFU-Aging | LFU + 频率衰减 | 防止历史高频项霸占缓存 |

## 项目结构

```
CacheSystem/
├── Pudd1ngCachePolicy.h   # 抽象接口：put / get
├── LruCache.h             # LRU、LRU-K、LRU 分片
├── LfuCache.h             # LFU、LFU-Aging、LFU 分片
├── ArcCache/              # ARC 自适应缓存
│   ├── ArcCache.h         # ARC 顶层调度
│   ├── ArcCacheNode.h     # 节点定义
│   ├── ArcLruPart.h       # LRU 半边（主链表 + 幽灵链表）
│   └── ArcLfuPart.h       # LFU 半边（频次分桶 + 幽灵链表）
├── testAllCachePolicy.cpp # 测试程序
└── CMakeLists.txt
```

## 构建与运行

项目无外部依赖，一条命令即可编译。

### Linux / macOS

```bash
# 方式一：g++ 直接编译
g++ -std=c++17 -O2 -I. testAllCachePolicy.cpp -o main
./main

# 方式二：CMake
mkdir build && cd build
cmake .. && make
./main
```

### Windows（MinGW）

```bash
# 方式一：g++ 直接编译
g++ -std=c++17 -O2 -I. testAllCachePolicy.cpp -o main.exe
main.exe

# 方式二：CMake + Ninja
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
build\main.exe
```

## 测试场景

测试程序模拟三种典型访问模式，对比五种策略的命中率：

1. **热点数据访问**：70% 热点 + 30% 冷数据
2. **循环扫描**：顺序扫描 + 随机跳跃
3. **工作负载剧烈变化**：五个阶段，访问模式不断切换

## 什么是缓存

缓存是将高频访问的数据暂存到内存中，加速数据访问、降低延迟、提高吞吐率的关键手段。

服务器内存有限，无法把所有数据都放在内存中，因此需要一种机制，决定内存不够用时该淘汰哪些数据——这就是缓存淘汰策略。不同的策略适合不同的访问模式：

- 访问模式稳定 → LFU 更优
- 访问模式多变 → LRU / ARC 更优
- 扫描型负载 → ARC 显著优于 LRU

## 实际应用

缓存淘汰策略在各类系统中广泛应用：

- **Linux Kernel**：页缓存（Page Cache）用 LRU 淘汰物理内存不足时最近未使用的页面
- **Redis**：提供 volatile-lru、LFU 等多种淘汰策略（maxmemory-policy 配置）
- **PostgreSQL / ZFS**：使用 ARC 自适应缓存
- **Android / 浏览器**：LruCache 等内存缓存

## 许可证

[GPL v3](LICENSE)。基于 [youngyangyang04/KamaCache](https://github.com/youngyangyang04/KamaCache) 修改，保留原作者版权声明。
