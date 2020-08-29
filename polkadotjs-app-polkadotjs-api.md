# Polkadot-JS App 和 Polkadot-JS API
* 问：访问 Polkadot-JS App 的网址是什么？
  * 答：https://polkadot.js.org/apps

* 问：Polkadot-JS App 和 Polkadot-JS API 的区别是什么？
  * 答：名字上一看就有不同，一个尾缀是 App,一个尾缀是 API；
  * Polkadot-JS App 是官方的前端与 Substrate 网络交互，而 Polkadot-JS API 则提供了官方的 JS 库连去 Substrate 网络。

* 问：你可以在 Polkadot-JS App 内做什么操作?
  * 答：查看 Substrate 网络 区块信息；
  * 对 Substrate 网络作出交易 (Extrinsics)；
  * 有一个 javascript 编辑器，可对 Substrate 网络写出基础 javascript 与之互动；
  * 可以对一个信息以某个账号作签名

* 问：哪些网络是 Polkadot-JS App 里默认有支持的？
  * 答：Kusama；
  * Kulupu；
  * Centrifuge

* 问：如果在 Substrate 端加了自定义类型，我们在 Polkadot-JS App 里需要作什么才能支持连到这个 Substrate 节点？
  * 答：在 Setting 里, Developer tab 里，加自定义的 JSON 对象。

* 问：在 Polkadot-JS API 里，数字默认是用哪个类型代表？
  * 答：[bn.js](https://github.com/indutny/bn.js/)

* 问：我要查询 Substrate 链上的存储变量,并订阅它的变更，应该用以下哪个方法？
  * 答：`const unsub = await api.query.<pallet>.<storage>(value => {...})`

* 问：我要对 Substrate 链上发出一次交易，但 **不需要** 订阅交易处理状态，应该用以下哪个方法？
  * 答：
`const val = await api.tx.<module>.<extrinsics>().signAndSend()`

* 问：Polkadot-JS API 内哪个组件负责取得 [Polkadot-JS extension](https://github.com/polkadot-js/extension) 里的钱包资料？
  * 答： [@polkadot/keyring](https://www.npmjs.com/package/@polkadot/keyring)

* 问：Github 上的 Substrate repo 地址
  * https://github.com/paritytech/substrate

