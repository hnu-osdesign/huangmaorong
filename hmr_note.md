参考链接

rust 学习网址 https://www.runoob.com/rust/rust-basic-syntax.html

redox 参考文档 https://www.redox-os.org/zh/docs/

## 目录

Rust相关语法

redox整体框架

src/lib.rs代码解释

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

## redox整体框架

1. 内核各个模块部分

   allocator  堆分配器

   arch            架构 

   common       共享数据结构

   consts     常量

   context   上下文管理

   devices   独立于架构的devices

   elf       用来解析elf可执行文件的elf文件

   event      事件处理

   externs   外部功能块

   log       

   memory    内存管理

   panic      异常处理

   scheme    文件系统处理程序

   sync      同步基元

   syscall   系统调用

   time 

2. 宏

3. 功能

   cpu_count  获取当前处于活动状态的CPU数

   cpu_id       获取当前CPU的调度id

   kmain        主CPU的内核入口点，Crate负责调用这个

   kmain_ap   辅助CPU的主要内核入口点

   ksignal           允许异常处理程序独立的内核发送信号

   userspace_init   运行initfs: bin/init进程初始化用户空间

4. 各个模块详细说明

- 堆分配器模块（allocator）
   mod.rs文件的功能：映射堆页；初始化全局堆。
   
   linked_list.rs和slab.rs主要是为mod.rs文件中提供一些函数实现，例如mod.rs中的初始化全局堆用到了linked_list.rs的init函数，里面还有Allocator结构体的定义。
   
- 架构模块（arch)
   mod.rs文件：声明x86_64的使用

   X86_64文件中的各个模块
     debug    调试支持
     device    设备
     gdt       全局段号记录表
     idt       中断号记录表
     interrupt 中断说明中断指令
     ipi       处理器间中断
     macros     宏
     paging     分页，虚拟内存
     pti
     start      初始化和启动功能
     stop       停止功能

- 共享数据结构（common)

   mod.rs文件的功能:声明了int_like模块的作用域，定义了一个具有简单功能的宏dbg(什么功能）
   
   int_like.rs文件的功能:定义了一些宏的功能,例如交换，比较。
   
 - 上下文管理 （context）
   内部分为很多功能模块：context.rs  file.rs  list.rs  memory.rs  signal.rs switch.rs  timeout.rs
   
   context.rs:定义上下文管理的结构体
   
   file.rs:定义文件管理方案和文件编号
   
   list.rs:上下文列表
   
   memory.rs:上下文的存储（页）
   
   signal.rs:信号处理
   
   timeout.rs:超时处理
   

## src/lib.rs代码解释

```
{
    //标识CPU的唯一编号，用于调度
    #[thread_local]
    static CPU_ID: AtomicUsize = ATOMIC_USIZE_INIT;
    //获取当前CPU的编号
    #[inline(always)]
    pub fn cpu_id() -> usize {
        CPU_ID.load(Ordering::Relaxed)
    }
}
```
```
{
    //记录所有可以安排调度的CPU数
    static CPU_COUNT : AtomicUsize = ATOMIC_USIZE_INIT;
    //获取当前活跃的CPU数
    #[inline(always)]   //内联属性
    pub fn cpu_count() -> usize {
        CPU_COUNT.load(Ordering::Relaxed)
    }
}
```
```
{
    static mut INIT_ENV: &[u8] = &[];//u8表示无符号型的8位整数
    pub extern fn userspace_init() {
        let path = b"initfs:/bin/init";//初始化路径
        let env = unsafe { INIT_ENV };//允许读取可变静态变量
    assert_eq!(syscall::chdir(b"initfs:"), Ok(0));//将chdir的结果和OK(0)的结果比较，判断路径是否打开成功，下面几句用法类似
    assert_eq!(syscall::open(b"debug:", syscall::flag::O_RDONLY).map(FileHandle::into), Ok(0));
    assert_eq!(syscall::open(b"debug:", syscall::flag::O_WRONLY).map(FileHandle::into), Ok(1));
    assert_eq!(syscall::open(b"debug:", syscall::flag::O_WRONLY).map(FileHandle::into), Ok(2));
    let fd = syscall::open(path, syscall::flag::O_RDONLY).expect("failed to open init");
    let mut args = Vec::new();
        args.push(path.to_vec().into_boxed_slice());
    let mut vars = Vec::new();
        for var in env.split(|b| *b == b'\n') {
            if ! var.is_empty() {
                vars.push(var.to_vec().into_boxed_slice());
            }
        }
    syscall::fexec_kernel(fd, args.into_boxed_slice(), vars.into_boxed_slice()).expect("failed to execute init");
    panic!("init returned");
    }
}
```

```
{
    //这是主CPU的内核入口点。
    pub fn kmain(cpus: usize, env: &'static [u8]) -> ! {
        CPU_ID.store(0, Ordering::SeqCst);
        CPU_COUNT.store(cpus, Ordering::SeqCst);
        unsafe { INIT_ENV = env };
    //初始化第一个上下文, stored in kernel/src/context/mod.rs
        context::init();
    let pid = syscall::getpid();//获取进程pid
        println!("BSP: {:?} {}", pid, cpus);
        println!("Env: {:?}", ::core::str::from_utf8(env));
    match context::contexts_mut().spawn(userspace_init) {
            Ok(context_lock) => {
                let mut context = context_lock.write();
                context.rns = SchemeNamespace::from(1);
                context.ens = SchemeNamespace::from(1);
                context.status = context::Status::Runnable;
            },
            Err(err) => {
                panic!("failed to spawn userspace_init: {:?}", err);
            }
        }
    loop {
            unsafe {
                //屏蔽中断
                interrupt::disable();
                if context::switch() {//上下文切换
                    interrupt::enable_and_nop();
                } else {
                    //启用中断，然后停止CPU（以节省电源），直到下一个中断真正被触发。
                    interrupt::enable_and_halt();
                }
            }
        }
    }
}
```
    辅助cpu的主要内核入口点和主cpu的类似






