# Facade（外观）

## 一句话介绍

Facade，中文术语叫「外观模式」，也叫「门面模式」。在经典设计模式中，归为结构型（Structural）模式分类，因为这种模式用于帮助构建结构。它可以为程序库、框架或其他复杂情况提供一个简单的接口。

## 解决了什么问题

在软件开发中，有时候要处理很多同类型的业务，但具体处理方式却不同的场景。因此，建立一个「门面」来达到统一管理和分发的目的。

Facade 模式，帮忙建立了统一的接口，使得调用复杂的子系统变得更加简单。因为 Facade 模式只包括应用真正关心的核心功能。


## 如何解决

心智图：


```text
                 +------------------+
+-------+        |                  |         +---------------+
|       |        |                  |         |  additional   |
|client +------> |     facade       +-------> |  facade       |
+-------+        |                  |         |               |
                 |                  |         |               |
                 +--+----+------+--++         +---------------+
                    |    |      |  |
           +--------+    |      |  +--------+
           |          +--+      +-+         |
           |          |           |         |
           v          |           v         v
       +---+---+  +---v--+   +----+--+  +---+----+
       |       |  |      |   |       |  |        |
       | system|  |system|   |system |  | system |
       |       |  |      |   |       |  |        |
       +-------+  +------+   +-------+  +--------+

```

## 真实案例

实现方式：

- 利用 「类型」 和 「Trait」： 
    - [log](https://github.com/rust-lang/log)
    - [mio](https://github.com/tokio-rs/mio)
    - [cranelift]()
        - [MachBackend](https://github.com/bytecodealliance/wasmtime/search?q=MachBackend)
        - [LowerBackend](https://github.com/bytecodealliance/wasmtime/search?q=LowerBackend)
- 模块 re-export： 
    - [Rust libstd reexport libcore](https://github.com/rust-lang/rust/tree/master/library/std/src/sys)
    - [Futures-rs]()
- 条件编译：[tikv/tikv](https://github.com/tikv/tikv/tree/master/components/tikv_alloc)


## 结语

