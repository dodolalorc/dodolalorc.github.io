# Rust编程区块链项目 BlockChain in Rust


{{< admonition type=info title="GitHub仓库地址" open=false >}}
GitHub 仓库地址：[BlockChain](https://github.com/dodolalorc/BlockChain)
{{< /admonition >}}

## 一、产品方案

### 项目目标

实现一个轻量的简单高效的区块链系统项目，用于学习和研究。

- 学习区块链的相关知识，实现一个由 Rust 实现的简单区块链系统。
- 能够生成创世区块，支持区块的创建、验证和链式储存，实现工作量证明算法。
- 利用本机端口构建服务系统，实现交易的广播、挖矿功能、简单的交易校验功能。

### 用户需求分析

- 用户可以通过命令行启动本地服务，在`main.rs`中配置路由，参考下方的[使用手册](#使用手册)，进行创建交易、挖矿、查看信息等。
- 通过时间戳、`lock_time`、签名校验等完成简单的对交易信息的核对。
- 系统具有一定的可扩展性，用户可以通过调整 API 接口快速调试。

### 主要功能

本产品主要由`src/`下的`.rs`文件实现主要功能。

设计了区块链和区块的数据结构，并完成了简单的新建区块、新建区块链以及设置创世区块、添加交易到交易池、挖矿打包交易、交易广播、计算 Merkle 树根哈希等功能。

设计了一条交易信息的各种数据结构，包括其交易输入、交易输出、锁定时间，还实现了签名交易和广播行为。

## 二、技术方案

### 技术选型

- 编程语言：Rust（高性能、内存安全、适合区块链开发）。
- 数据序列化：使用  `serde`  和  `serde_json`  进行完成序列化。
- 时间戳：使用`chrono`获取当前时间。
- 哈希算法：通过`sha2`库的 SHA-256 计算哈希值。
- 加密算法：使用 Rust 的加密库`ring`实现哈希、签名等功能。
- 共识算法：实现 PoW（工作量证明）共识机制。
- 网络通信：使用 Rust 的异步网络库`Tokio`实现 P2P 通信。

### 模块划分

- 区块模块：区块的创建、验证和链式存储。
- 交易模块：交易的创建、签名和验证。
- 网络模块：节点发现、消息广播和数据同步。
- 共识模块：实现共识算法。
- 存储模块：数据持久化存储。

`main.rs`：定义了创建节点、消息广播等功能的网络模块。

`block_chain.rs`：定义了区块链和区块的数据结构，并完成了简单的新建区块、新建区块链以及设置创世区块、添加交易到交易池、挖矿打包交易、交易广播、计算 Merkle 树根哈希等功能。

`hash_function.rs`：主要定义了常用的哈希函数。

`serialization.rs`：定义了序列化和反序列化的方法。

`transaction.rs`：定义了一条交易信息的各种数据结构，包括其交易输入、交易输出、锁定时间，还实现了签名交易和广播行为。

### 系统结构

#### 区块链结构

区块链的核心是由一系列按顺序链接的区块组成的链式结构。每个区块包含以下关键信息：

- 时间戳（timestamp）：区块创建的时间。
- 哈希值（merkle_root）：当前区块的唯一标识，通过加密算法生成。
- 前一区块哈希（prev_block_hash）：指向前一个区块的哈希值，用于维护链的连续性。

在实现中，区块链可以通过一个动态数组（ `Vec<Block>`）来存储所有区块，确保区块的顺序和完整性。

#### 创世区块

创世区块是区块链中的第一个区块，它的生成标志着区块链的初始化。创世区块的特点包括：

- 前一区块哈希为空：由于没有前驱区块，其前一区块哈希值通常设置为空或特定标识（如 "0"）。
- 哈希值计算：根据区块的内容（索引、时间戳、前一区块哈希和数据）生成唯一的哈希值，并存储到区块中。

创世区块的生成是区块链启动的必要步骤，为后续区块的添加奠定基础。

#### 挖矿算法

挖矿是区块链中生成新区块的关键过程，其核心是通过计算找到一个满足特定条件的哈希值。具体实现如下：

1. 难度目标：设定一个哈希值的难度条件（例如，哈希值的前几位必须为 "0"）。
2. 随机数（Nonce）：通过不断尝试不同的随机数，结合区块的其他信息（索引、时间戳、前一区块哈希和数据），计算哈希值。
3. 哈希验证：检查生成的哈希值是否满足难度条件。如果满足，则挖矿成功；否则，继续尝试新的随机数。

挖矿算法的实现确保了区块链的安全性和去中心化特性，同时也为新区块的生成提供了动力。

以上设计使得区块链系统能够实现基础的区块的创建、链接和验证，同时通过挖矿算法保证网络的共识和安全。

## 三、使用手册

拉取本项目

```bash
git clone git@github.com:dodolalorc/BlockChain.git
```

启动项目，开启本地服务器

```bash
cargo run
```

本地服务器接口使用：

- 添加交易

```bash
curl -X POST http://127.0.0.1:3030/transaction -H "Content-Type: application/json" -d '{"value":100,"lock_time":0}'
```

- 查看交易池

```bash
curl http://127.0.0.1:3030/pool
```

- 查看区块链区块部分

```bash
curl http://127.0.0.1:3030/blocks
```

- 挖矿

```bash
 curl -X POST http://127.0.0.1:3030/mine
```

### 实验截图

建立交易及交易池状态

![](https://img.dodolalorc.cn/i/2025/03/06/67c9967b9075f.png)

挖矿

![](https://img.dodolalorc.cn/i/2025/03/06/67c99785d7f18.png)

## 四、项目要求

一：计划

1. 完成概念学习：区块链
2. 完成需求设计：产品方案、技术方案
3. 完成项目实现：Rust 实现，至少输出创世区块

二：要求

1. 结合下面给的和自己搜索到的参考资料、书籍、视频等学习区块链基础知识
2. 结合个人兴趣和所学内容，自己确定一个区块链领域的项目目标（如，实现基本的挖矿）
3. 有了目标，自己学习怎么写产品方案、技术方案、可以多人组队研究
4. 基于产品方案和技术方案实现你的需求，要结合之前讲的数据结构和算法内容

_注意：至少是实现一个简单的区块链，运行并输出创世区块_

三：参考资料

1. 区块链教程 https://liaoxuefeng.com/books/blockchain/introduction/index.html
2. 区块链学习路线 https://zjubca.github.io/roadmap/
3. go 实现的 demo https://github.com/Jeiwan/blockchain_go
4. B 站区块链项目实战 https://www.bilibili.com/video/BV145411t7qp/?vd_source=ca616b8d8161186b30bdd62e4e044e42

![](https://img.dodolalorc.cn/i/2025/03/03/67c5515c5fd95.png)

