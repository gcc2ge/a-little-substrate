# 如何学习 Substrate 源码
* 熟练使用 Substrate 的 Rust 文档: [crates.parity.io](https://crates.parity.io)
* 从 frame 下的感兴趣的 pallet 开始，比如资产相关的 balance, assets; 治理相关的 democracy, membership, collective。理解
  * 本模块的作用，提供了哪些存储和可调用函数
  * 本模块和其它模块如何交互
  * 本模块是如何在 Kusama/ Polkadot/ Substrate Node/ Substrate Node Template 的 runtime 中使用的
  * 为什么在链上需要这个模块或者逻辑
* 对 frame 的功能有一定了解之后，可以去探索更加底层的知识和架构，比如
  * runtime 模块里对存储单元的操作如何反应在数据库中的
  * 为什么使用 wasm，使用的 wasm 运行时的考量，host 和 runtime 之间的关系和如何互相调用
  * Substrate 如何使用 libp2p 实现的点对点网络，使用的已有协议和新的协议有哪些
  * 共识相关的接口和抽象，可以支持哪些共识，为什么，Substrate 写新的共识有哪些限制
  * 交易池在 Substrate 架构里的位置以及处理逻辑
  * 通过 git issue 发现最早添加这个功能的 context 是什么，有哪些值得关注的讨论
  * more
* 在探索底层的同时，也要注意业务
  * Substrate 的哪个层次可以支持我的业务，如果要修改我应该去哪里修改
  * 修改是不是通用的，是不是可以贡献回去

