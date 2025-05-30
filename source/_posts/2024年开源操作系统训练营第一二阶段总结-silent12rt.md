---
title: 2024年开源操作系统训练营第一和二阶段总结-silent12rt
date: 2024-11-08 14:57:58
categories:
    - 2024秋冬季开源操作系统训练营 
tags:
    - author:silent12rt
    - repo:https://github.com/LearningOS/2024a-rcore-Silent12rt.git
---

# rustlings  总结
Rustlings 是学习 Rust 编程语言的极佳练习工具，它包含了多个由浅入深的练习题目，帮助学习者快速掌握 Rust 的基础知识和重要概念。
### 1. 变量与可变性
- Rust 中的变量默认是不可变的（immutable），即变量在声明后无法更改。要让变量可变，必须显式添加 `mut` 关键字。
- 这种默认不可变性帮助开发者避免无意的状态变化，提高代码的安全性和可维护性。

### 2. 数据类型
- Rust 是静态类型语言，编译器会在编译阶段检查数据类型。
- Rust 支持多种数据类型，包括标量类型（整型、浮点型、布尔型、字符）和复合类型（元组、数组等）。

### 3. 所有权机制
- Rust 的所有权系统是其内存安全性和性能的重要保障。
- 每个值在同一时间只能有一个所有者，当所有者变量超出作用域时，内存会自动释放。所有权的转移、借用和引用（可变和不可变）是理解 Rust 内存管理的关键。

### 4. 借用与引用
- 借用（Borrowing）允许在不转移所有权的情况下使用数据。
- Rust 有严格的借用规则：在同一作用域中，只允许一个可变引用或多个不可变引用，确保内存安全。

### 5. 结构体与枚举
- 结构体（Struct）用于将不同的数据组合成一个复合类型，枚举（Enum）用于定义一组可能的状态或值。
- Rust 的枚举非常强大，支持绑定数据，并且可以与模式匹配一起使用，帮助更清晰地处理复杂的逻辑分支。

### 6. 模式匹配
- `match` 表达式和 `if let` 是 Rust 中处理分支的主要工具，尤其是当处理枚举和结果类型（`Result`）时。
- `match` 语法不仅简洁，还能避免遗漏某些分支，确保代码的健壮性。

### 7. 错误处理
- Rust 提供了 `Result` 和 `Option` 类型来进行错误处理和空值处理。
- 使用 `unwrap`、`expect`、`match` 等方式处理这些类型，开发者可以编写出更健壮的代码，避免程序在运行时崩溃。

### 8. 所有权的移动与复制
- 移动（Move）：当变量的所有权被转移时，源变量将不可用。
- 复制（Copy）：对于实现了 `Copy` 特征的类型（如基本数据类型），赋值不会转移所有权，而是直接复制。

### 9. 特征

- Rust 中的特征类似于其他语言的接口，用于定义一组方法签名，供结构体或枚举实现。
- 特征使得 Rust 支持多态，通过泛型和特征约束实现代码的复用和接口一致性。

### 10. 智能指针
- Rust 的标准库中提供了 `Box`、`Rc` 和 `RefCell` 等智能指针类型，帮助管理内存和共享数据。
- `Box` 实现堆分配，`Rc` 实现引用计数，`RefCell` 提供运行时的可变性检查，用于实现更复杂的数据结构。

### 11. 并发编程
- Rust 的所有权机制让多线程编程更加安全。
- 使用 `std::thread` 库可以方便地创建线程，并且借助 `Arc` 和 `Mutex` 等类型来实现线程间的数据共享和同步。

### 12. 生命周期（Lifetimes）
- Rust 使用生命周期注解来管理引用的生命周期，确保程序不会引用无效数据。
- 生命周期的概念是 Rust 内存安全的重要组成部分，编译器会自动推断大部分生命周期，但有时需要显式标注。


# OS 实现

第二阶段主要分为八个章节，每个章节层层递进，深入揭示操作系统的底层逻辑以及实现原理。

### lab1
为了实现 sys_task_info 系统调用，首先在 TaskManager 中为任务控制块 (TCB) 扩展结构体，加入如下字段：sys_call_times:[u32;MAX_SYSCALL_NUM]，然后在在mod.rs中增加increase_sys_call和get_sys_call_times函数，进而在syscall函数中调用increase_sys_call函数，

在系统调用处理逻辑中，维护当前任务的系统调用次数计数，每次进入系统调用时在数组中相应位置加一。在 sys_task_info 系统调用实现中，将当前任务的状态（应为 Running）、系统调用次数、以及通过 get_time() 获取的任务总运行时间写入 TaskInfo 结构体。

### lab2
- 完成sys_get_time和sys_task_info函数，需要定义一个translated_struct_ptr，它通过页表（PageTable）将一个指向结构体（`*mut T`）的指针翻译为对应的物理地址，并返回一个可变引用（&'static mut T），这样可以直接操作映射后的物理内存。在sys_task_info函数中，需要获取系统调用时间和任务运行时间，所以需要在task.rs中定义并实现它们。
- 完成sys_mmap和sys_munmap函数，用于申请和释放虚拟内存映射。sys_mmap 通过指定的起始地址 start、长度 len 和内存页属性 port 来映射一段物理内存到虚拟地址空间。该函数会检查起始地址对齐情况、port 的合法性，并将内存页映射为可读、可写或可执行。sys_munmap 则用于取消内存映射，释放从 start 开始的一段虚拟内存。（注意，需要判断地址是否对齐）

### lab3
- 实现了自定义的spawn的系统调用来创建新进程，以便简化进程创建的过程而无需使用fork+exec的组合操作。通过sys_spawn，直接从指定的路径启动目标程序，成功返回子程序的进程id，否则返回-1。
- 实现了stride调度算法，为每个进程设置优先级与其调度权重，使得系统资源能够更公平分配。`stride`调度通过设置初始优先级与动态步长（pass值），优先调度累计步长最小的进程，并对选中的进程步长进行累加调整，确保各进程的运行比例与优先级成正比。我还新增了 `sys_set_priority` 系统调用，以便动态调整进程优先级，增强调度灵活性。

### lab4
在本次作业中，我实现了三个系统调用：linkat、unlinkat 和 fstat，以支持硬链接和文件状态获取。
1. linkat：该调用用于创建一个文件的硬链接。它接收原有文件路径和新链接路径作为参数，并将新路径指向与原文件相同的磁盘块。在实现中，我确保在创建链接时不允许同名文件的存在，避免潜在的未定义行为。
2. unlinkat：此调用用于删除文件的链接。当调用 unlinkat 时，如果文件的引用计数降至零，它将回收与文件关联的 inode 及其数据块。我的实现确保正确处理文件的彻底删除，维护文件系统的一致性。
3. fstat：该调用获取指定文件描述符的状态信息。它将文件状态结构体填充以提供有关文件的详细信息，如大小、权限和时间戳等，便于用户或其他系统调用进行进一步处理。
通过这三个系统调用的实现，我的文件系统支持了硬链接的创建与删除，以及文件状态的查询，从而增强了其功能和灵活性。

### lab5
我实现的功能包括对进程和线程的资源管理，主要是通过互斥量（mutex）和信号量（semaphore）来控制并发访问。具体而言，为每个线程维护了四个关键数据结构：m_allocation 和 s_allocation 用于记录已分配的互斥量和信号量的数量，m_need 和 s_need 则用于跟踪线程尚需的资源数量。在系统调用中，加入了对死锁的检测逻辑，确保在尝试获取新资源之前检查当前资源的可用性。如果发现潜在的死锁情况，则会拒绝资源请求，并返回相应的错误代码。此外，还实现了调整资源需求的方法，以便在资源分配和释放时动态更新状态。



