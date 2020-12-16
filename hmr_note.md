参考链接

rust 学习网址 https://www.runoob.com/rust/rust-basic-syntax.html

redox 参考文档 https://www.redox-os.org/zh/docs/

## Rust相关语法

1. Rust的组织管理

- 箱Crate："箱"是二进制程序文件或者库文件，存在于"包"中。箱是树状结构的，它的树根是编译器开始运行时编译源文件所编译的程序。

- 包Package：一个包最多包含一个库"箱"，可以包含任意数量的二进制"箱"，但是至少包含一个"箱"（不管是库还是二进制"箱"）。创建工程时，工程目录下会建立一个 Cargo.toml 文件。工程的实质就是一个包，包必须由一个 Cargo.toml 文件来管理，该文件描述了包的基本信息以及依赖项。

- 模块Module：具有特定的功能代码

2. rust中的异常处理（6大断言和4个异常处理机制）

- 断言：assert!是用于断言布尔表达式是否为true，assert_eq!用于断言两个表达式是否相等，assert_ne!用于断言两个表达式是否不相等。当不符合条件时，断言会引发线程恐慌（panic!）

- 异常处理：Option、Result<T, E>、线程恐慌（Panic）、程序终止（Abort）

3. 属性声明(属性是在Rust语言元素上的元数据)

- 外部属性：一个属性声明在一个元素之前，对跟在后面的这个元素生效。外部属性用 #[] 声明。

- 内部属性：属性声明在一个元素，对此元素的整体生效，内部属性用#![]声明。

- 一些属性的说明

   feature 表示开启一些不稳定特性，只可在nightly版的编译器中使用

   cfg条件编译，表示只有当…的时候才会编译

   no_std表示不链接自带的std库

4. Rust的路径描述和作用域

- Rust中路径分隔符为::，绝对路径从Crate关键字开始描述，相对路径从 self 或 super 关键字或一个标识符开始描述。

- use 关键字能够将模块标识符引入当前作用域；使用#[macro_use] 可以使被注解的module模块中的宏应用到当前作用域中

5. 数据类型

   整数型：按照位长度和有无符号进行分类。

   浮点数型：支持 32 位浮点数（f32）和 64 位浮点数（f64）。
 
   布尔型：值只能为True或False。

   字符型：Rust的 char 类型大小为 4 个字节。

   复合类型：元组，用（）表示一组数据。


6. 宏的定义

   使用macro_rules!来创建宏，格式如下：

      macro_rules! 宏名称{

        (接受的参数) =>{
 
	      宏代码中的内容
  
        }
  
      }


## 组合算子

相关知识

一般来说程序失败，rust会使用panic处理，而对于可能不存在的情况，则使用标准库std里面的option<T>的枚举类型。option<T>用于不存在的可能性的情况，表现为以下两种形式

- Smoe(T):找到一个属于T类型的元素
- None:找不到相应元素

这两个选项可以通过match(类似于switch)显示进行处理,也可以使用unwrap隐式地进行处理。隐式处理要么返回 Some 内部的元素，要么就 panic。

match是处理Option的一个有效的方法，但是这个方法对于很多用例都比较繁琐（操作只有一个有效输入时)，由此引入组合算子(combinator）以模块化的方式来管理控制流

- map

Option中有一个内置方法map(),这个组合算子可以简单映射some->some，none->none的情况。多个不同的map调用可以更加灵活地链式连接在一起。
```
#![allow(dead_code)]

#[derive(Debug)] enum Food { Apple, Carrot, Potato }

#[derive(Debug)] struct Peeled(Food);
#[derive(Debug)] struct Chopped(Food);
#[derive(Debug)] struct Cooked(Food);

// 削水果皮。如果没有水果，就返回 `None`。
// 否则返回削好皮的水果。
fn peel(food: Option<Food>) -> Option<Peeled> {
    match food {
        Some(food) => Some(Peeled(food)),
        None       => None,
    }
}

// 和上面一样，我们要在切水果之前确认水果是否已经削皮。
fn chop(peeled: Option<Peeled>) -> Option<Chopped> {
    match peeled {
        Some(Peeled(food)) => Some(Chopped(food)),
        None               => None,
    }
}

// 和前面的检查类似，但是使用 `map()` 来替代 `match`。
fn cook(chopped: Option<Chopped>) -> Option<Cooked> {
    chopped.map(|Chopped(food)| Cooked(food))
}

// 另外一种实现，我们可以链式调用 `map()` 来简化上述的流程。
fn process(food: Option<Food>) -> Option<Cooked> {
    food.map(|f| Peeled(f))
        .map(|Peeled(f)| Chopped(f))
        .map(|Chopped(f)| Cooked(f))
}

// 在尝试吃水果之前确认水果是否存在是非常重要的！
fn eat(food: Option<Cooked>) {
    match food {
        Some(food) => println!("Mmm. I love {:?}", food),
        None       => println!("Oh no! It wasn't edible."),
    }
}

fn main() {
    let apple = Some(Food::Apple);
    let carrot = Some(Food::Carrot);
    let potato = None;//马铃薯不存在

    let cooked_apple = cook(chop(peel(apple)));
    //烹饪苹果之前需要先判断是否是水果->是否削皮->是否切块->是否烹饪
    let cooked_carrot = cook(chop(peel(carrot)));
    // 现在让我们试试简便的方式 `process()`。
    let cooked_potato = process(potato);

    eat(cooked_apple);
    eat(cooked_carrot);
    eat(cooked_potato);
```

- and_then

多层map链式调用比较复杂，引入and_then语句

and_then() 使用包裹的值（wrapped value）调用其函数输入并返回结果。 如果 Option 是 None，那么它返回 None。

```
#![allow(dead_code)]

#[derive(Debug)] enum Food { Noodles, Steak, apple }
#[derive(Debug)] enum Day { Monday, Tuesday, Wednesday }

// 我们没有苹果（ingredient）来制作苹果派，(有其他原材料)
fn have_ingredients(food: Food) -> Option<Food> {
    match food {
        Food::apple => None,
        _           => Some(food),
    }
}

// 我们拥有全部食物的食谱，除了面条
fn have_recipe(food: Food) -> Option<Food> {
    match food {
        Food::Noodles => None,
        _                => Some(food),
    }
}

// 做一份好菜，我们需要原材料和食谱这两者。
// 借助一系列 `match` 来表达相应的逻辑：
fn cookable_v1(food: Food) -> Option<Food> {
    match have_ingredients(food) {
        None       => None,
        Some(food) => match have_recipe(food) {
            None       => None,
            Some(food) => Some(food),
        },
    }
}

// 可以使用 `and_then()` 方便重写出更紧凑的代码：
fn cookable_v2(food: Food) -> Option<Food> {
    have_ingredients(food).and_then(have_recipe)
}

fn eat(food: Food, day: Day) {
    match cookable_v2(food) {
        Some(food) => println!("Yay! On {:?} we get to eat {:?}.", day, food),
        None       => println!("Oh no. We don't get to eat on {:?}?", day),
    }
}

//cookable_v2() 会产生一个 Option<Food>。使用 map() 替代 and_then() 将会得到 Option<Option<Food>>，对 eat() 来说是一个无效类型。

fn main() {
    //定义一个三元组
    let (Noodles, steak, apple) = (Food::Noodles, Food::Steak, Food::apple);

    eat(Noodles, Day::Monday);
    eat(steak, Day::Tuesday);
    eat(apple, Day::Wednesday);
}
```






