# Runtime 宏介绍


## Rust 宏
* https://doc.rust-lang.org/book/ch19-06-macros.html 
* Rust 提供了两种宏
  * 声明宏
  * 过程宏


## Runtime 常用的宏
* `decl_storage` 宏定义 runtime 模块的存储单元。
```Rust
// https://github.com/substrate-developer-hub/substrate-node-template/blob/master/pallets/template/src/lib.rs
pub trait Trait: frame_system::Trait {
	type Event: From<Event<Self>> + Into<<Self as frame_system::Trait>::Event>;
}
decl_storage! {
	trait Store for Module<T: Trait> as TemplateModule {
		Something get(fn something): Option<u32>;
	}
}
```

* `decl_event` 宏定义事件。runtime 通过触发事件通知用户重要状态的转换。
```Rust
// https://github.com/substrate-developer-hub/substrate-node-template/blob/master/pallets/template/src/lib.rs
decl_event!(
	pub enum Event<T> where AccountId = <T as frame_system::Trait>::AccountId {
		SomethingStored(u32, AccountId),
	}
);
```
```Rust
Self::deposit_event(RawEvent::SomethingStored(something, who));
```

* `decl_error` 宏定义错误信息。
  * 可调用函数里的错误类型
    * 不能给它们添加数据
    * 通过 metadata 暴露给客户端
    * 错误发生时触发 system.ExtrinsicFailed 事件，包含了对应错误的信息
```Rust
// https://github.com/substrate-developer-hub/substrate-node-template/blob/master/pallets/template/src/lib.rs
decl_error! {
	pub enum Error for Module<T: Trait> {
		NoneValue,
		StorageOverflow,
	}
}

```
* `decl_module` 宏定义可调用函数，每一个外部交易都会触发一个可调用函数，并根据交易体信息也就是函 数参数，更新链上状态。
```Rust
// https://github.com/substrate-developer-hub/substrate-node-template/blob/master/pallets/template/src/lib.rs
decl_module! {
	pub struct Module<T: Trait> for enum Call where origin: T::Origin {
		type Error = Error<T>;
		fn deposit_event() = default;
		#[weight = 10_000 + T::DbWeight::get().writes(1)]
		pub fn do_something(origin, something: u32) -> dispatch::DispatchResult {
			let who = ensure_signed(origin)?;
			Something::put(something);
			Self::deposit_event(RawEvent::SomethingStored(something, who));
			Ok(())
		}
		#[weight = 10_000 + T::DbWeight::get().reads_writes(1,1)]
		pub fn cause_error(origin) -> dispatch::DispatchResult {
			let _who = ensure_signed(origin)?;
			match Something::get() {
				None => Err(Error::<T>::NoneValue)?,
				Some(old) => {
					let new = old.checked_add(1).ok_or(Error::<T>::StorageOverflow)?;
					Something::put(new);
					Ok(())
				},
			}
		}
	}
}
```
  * Runtime 模块里的保留函数
    * https://github.com/paritytech/substrate/blob/master/frame/support/src/dispatch.rs
    * `deposit_event` 函数，如果 pallets 用到 Events 的话，必须用这个函数初始化
    * `on_initialize` 函数，在每个区块的开头执行
    * `on_runtime_upgrade` 函数，当有 runtime 升级时才会执行，用来迁移数据
    * `integrity_test` 函数，集成测试
    * `on_finalize` 函数，在每个区块结束时执行
    * `offchain_worker` 函数，开头且是链外执行，不占用链上的资源

* `construct_runtime` 宏加载模块
```Rust
// https://github.com/substrate-developer-hub/substrate-node-template/blob/master/runtime/src/lib.rs
construct_runtime!(
	pub enum Runtime where
		Block = Block,
		NodeBlock = opaque::Block,
		UncheckedExtrinsic = UncheckedExtrinsic
	{
		System: frame_system::{Module, Call, Config, Storage, Event<T>},
		RandomnessCollectiveFlip: pallet_randomness_collective_flip::{Module, Call, Storage},
		Timestamp: pallet_timestamp::{Module, Call, Storage, Inherent},
		Aura: pallet_aura::{Module, Config<T>, Inherent},
		Grandpa: pallet_grandpa::{Module, Call, Storage, Config, Event},
		Balances: pallet_balances::{Module, Call, Storage, Config<T>, Event<T>},
		TransactionPayment: pallet_transaction_payment::{Module, Storage},
		Sudo: pallet_sudo::{Module, Call, Config<T>, Storage, Event<T>},
		TemplateModule: template::{Module, Call, Storage, Event<T>},
	}
);
```


## `cargo expand` 把宏代码展开，得到 Rust 的标准语法代码
* https://github.com/dtolnay/cargo-expand
* https://github.com/zhubaiyuan/substrate-node-template/blob/expand_pallet_template/pallets/template/src/expanded.rs


## 其它宏
* `sp_api::decl_runtime_apis` 宏定义 runtime api
  *  substrate/primitives/sr-api/proc-macro/src/lib.rs 
* `runtime_interface` 宏定义在 runtime 里可以调用的 Host 提供的函数
  *  substrate/primitives/runtime-interface/proc-macro/src/lib.rs 

