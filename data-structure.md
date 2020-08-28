# Runtime 数据结构


## 区块链存储的不同点和约束
* 区块链特点
  * 开源可审查，对等节点，引入延迟和随机来达到共识
  * 链式、增量地存储数据
* 区块链依赖高效的键值对数据库
  * LevelDB
  * RocksDB
* 区块链存储相关的限制
  * 大文件直接存储在链上的成本很高
  * 链式的区块存储结构不利于对历史数据的索引
  * 另外一个约束是，在进行数值运算时不能使用浮点数


## Substrate 存储单元的类型
* Substrate 存储单元的特点
  * Rust 原生数据类型的子集，定义在核心库和 alloc 库中
  * 原生类型构成的映射类型
  * 满足一定的编解码条件


### 单值类型存储某种单一类型的值，如布尔、数值、枚举、结构体等
* 数值: u8,i8,u32,i32,u64,i64,u128,i128 
```Rust
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyUnsignedNumber get(fn unsigned_number): u8 = 10;
        MySignedNumber get(fn signed_number): i8;
    }
}
```
  * 常用方法
    * https://github.com/paritytech/substrate/blob/master/frame/support/src/storage/mod.rs
    * 增:MyUnsignedNumber::put(number);
    * 查:MyUnsignedNumber::get();
    * 改:MyUnsignedNumber::mutate(|v| v + 1);
    * 删:MyUnsignedNumber::kill();
  * 安全操作
    * https://github.com/rust-num/num-traits/blob/master/src/ops/checked.rs
    * https://github.com/paritytech/substrate/blob/master/primitives/arithmetic/src/traits.rs
    * 返回Result类型:checked_add, checked_sub, checked_mul, checked_div
    * 溢出返回饱和值:saturating_add,saturating_sub,saturating_mul

* 大整数: U256,U512
  * https://github.com/paritytech/parity-common/blob/master/primitive-types/src/lib.rs
```Rust
use sp_core::U256;
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyBigInteger get(fn my_big_integer): U256;
    }
}
```

* 布尔: bool
```Rust
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyBool get(fn my_bool): bool;
    }
}
```

* Vec<T> 类型
  * https://github.com/paritytech/substrate/blob/master/primitives/std/src/lib.rs
  * https://github.com/rust-lang/rust/blob/master/library/alloc/src/vec.rs
```Rust
use sp_std::prelude::*;
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyString get(fn my_string): Vec<u8>;
    }
}
```

* Percent,Permill,Perbill 类型
```Rust
use sr_primitives::Permill;
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyPermill get(fn my_permill): Permill;
    }
}
```
  * https://github.com/paritytech/substrate/blob/master/primitives/arithmetic/src/per_things.rs
  * 构造
    * Permill::from_percent(value);
    * Permill::from_parts(value);
    * Permill::from_rational_approximation(p,q);
  * 计算
    * permill_one.saturating_mul(permill_two);
    * my_permill * 20000 as u32

* Moment 时间类型
```Rust
pub trait Trait: system::Trait + timestamp::Trait {}
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyTime get(fn my_time): T::Moment;
    }
}
```
  * 获取链上时间:<timestamp::Module<T>>::get();

* AccountId 账户类型
```Rust
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyAccountId get(fn my_account_id): T::AccountId;
    }
}
```
  * 获取AccountId: let sender = ensure_signed(origin)?;

* struct 类型
```Rust
#[derive(Clone, Encode, Decode, Eq, PartialEq, Debug, Default)]
pub struct People {
    name: Vec<u8>,
    age: u8,
}
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyStruct get(fn my_struct): People;
    }
}
```

* enum 类型需要实现 Default 接口
```Rust
#[derive(Copy, Clone, Encode, Decode, Eq, PartialEq, Debug)]
pub enum Weekday {
	Monday,
	Tuesday,
	Wednesday,
	Other,
}
impl Default for Weekday {
	fn default() -> Self {
		Weekday::Monday
	}
}
impl From<u8> for Weekday {
	fn from(value: u8) -> Self {
		match value {
			1 => Weekday::Monday,
			2 => Weekday::Tuesday,
			3 => Weekday::Wednesday,
			_ => Weekday::Other,
		}
	}
}
decl_storage! {
	trait Store for Module<T: Trait> as DataTypeModule {
		MyEnum get(fn my_enum): Weekday;
	}
}
```


###  简单映射类型
* map 类型用来保存键值对，单值类型都可以用作key或者value
```Rust
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyMap get(fn my_map): map hasher(twox_64_concat) u8 => Vec<u8>;
    } 
}
```
```Rust
hasher: blake2_128_concat, twox_64_concat, identity
```
  * 常用用法
    * https://github.com/paritytech/substrate/blob/master/frame/support/src/storage/mod.rs
    * 插入一个元素:MyMap::insert(key, value);
    * 通过key获取value:MyMap::get(key);
    * 删除某个key对应的元素:MyMap::remove(key);
    * 覆盖或者修改某个key对应的元素
      * MyMap::insert(key, new_value);
      * MyMap::mutate(key, |old_value| old_value+1);


###  双键映射类型
* double_map 类型，使用两个key来索引value，用于快􏰁删除key1对应的任意 记录，也可以遍历key1对应的所有记录
```Rust
decl_storage! {
    trait Store for Module<T: Trait> as DataTypeModule {
        MyDoubleMap get(fn my_double_map): double_map hasher(blake2_128_concat) T::AccountId, hasher(blake2_128_concat) u32 => Vec<u8>;
    }
}
```
  * 常用用法
    * https://github.com/paritytech/substrate/blob/master/frame/support/src/storage/mod.rs
    * 插入一个元素:MyDoubleMap::<T>::insert(key1, key2, value);
    * 获取某一元素:MyDoubleMap::<T>::get(key1, key2);
    * 删除某一元素:MyDoubleMap::<T>::remove(key1, key2);
    * 删除 key1 对应的所有元素:MyDoubleMap::<T>::remove_prefix(key1);


## 存储的初始化
* 创世区块的数据初始化的三种方式
  * config()
  * build(clousure)
  * add_extra_genesis { ... }
```Rust
decl_storage! {
	trait Store for Module<T: Trait> as GenesisConfigModule {
		Something get(fn something) config(): Option<u32>;
		SomethingTwo get(fn something_two) build(|config: &GenesisConfig<T>| {
			Some(config.something_two + 1)
		}): Option<u32>;
		SomethingMap get(fn something_map): map hasher(blake2_128_concat) T::AccountId => u32;
	}
	add_extra_genesis {
		config(something_two): u32;
		config(some_account_value): Vec<(T::AccountId, u32)>;
		build(|config: &GenesisConfig<T>| {
			for (who, value) in config.some_account_value.iter() {
				SomethingMap::<T>::insert(who, value);
			}
		})
	}	
}
```


## 最佳实践
* 最小化链上存储
  * 哈希值
  * 设置列表容量
* Verify First, Write Last


##  其它Tips
* 可以通过pub关键字设置存储单元的可见范围
* 可以手动设置默认值，如 `MyUnsignedNumber get(fn unsigned_number): u8 = 10;`
* 在frame目录下查找对应的最新用法
* decl_storage 宏的说明文档
  * https://crates.parity.io/frame_support/macro.decl_storage.html

