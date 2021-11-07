# Visitor（访问者）

## 解决了什么问题

方便调用方对**复杂树状数据结构**的遍历，一般用于编译器 AST 的遍历接口

## 如何解决

树状结构库同时提供一个 Visitor 的 trait 以及相应的从任意树节点一直 walk 到叶子节点的方法，让调用方不必知道太多树结构的细节

## 真实案例

### syn::visit::Visitor

在开发过程宏或 codegen 库解析 Rust 源码输入时，可以用 syn::visit API 方便的遍历输入的 Rust AST 树

#### servo 浏览器 to_css 相关过程宏

[servo 源码](https://github.com/servo/servo/blob/a3af32155fe74ab886862a56a75af06dee9ea9d5/components/style_derive/to_css.rs#L111)
曾用过 syn::visit::visitor，不过在 [这个 commit](https://github.com/servo/servo/commit/f2758950284610265b9245beed179c8729715a6b)
中重构了 Visitor 的写法

#### 静态分析统计 await 调用次数

虽然用 grep 等文本搜索工具也能统计 await 个数，但是会将**注释**中的 await 代码错误的统计进去

假设想统计以下代码内容的 await 个数:

```rust
// const CODE: &str = r#"
async fn foo() {}

async fn a() {
    foo().await;
    foo().await;
}

struct B;
impl B {
    async fn new() -> Self {
        a().await;
        todo!();
    }
}
// "#;
```

如果自己手动遍历 syn::File 的 item 会是这么写:

```rust
let file = syn::parse_str::<syn::File>(CODE).unwrap();
for item in file.items {
    match item {
        syn::Item::Fn(fn_) => {
            for stmt in fn_.block.stmts {
                match stmt {
                    syn::Stmt::Local(_) => todo!(),
                    syn::Stmt::Item(_) => {
                        // TODO match item recursive?
                        todo!()
                    },
                    syn::Stmt::Expr(_) => todo!(),
                    syn::Stmt::Semi(_, _) => todo!(),
                }
            }
        },
        syn::Item::Struct(struct_) => {
            match struct_.fields {
                syn::Fields::Named(_) => todo!(),
                syn::Fields::Unnamed(_) => todo!(),
                syn::Fields::Unit => todo!(),
            }
        },
        // ...
        _ => todo!()
    }
}
```

当用 match 层层解析 File 的 Item 到 Stmt::Item 时会发现怎么又要解析 Item,

于是被迫重写遍历文件 AST 的函数改成递归，然后又要读 syn 很多结构体的定义才能层层解析下来

如果使用 syn::visit 提供的 API 则无需关心太多 AST 树的细节

```rust
fn use_visitor_count_await() {
    struct AwaitVisitor {
        await_count: u32
    }
    impl<'ast> syn::visit::Visit<'ast> for AwaitVisitor {
        fn visit_expr_await(&mut self, _i: &'ast syn::ExprAwait) {
            self.await_count += 1;    
        }
    }
    let mut await_visitor = AwaitVisitor { await_count: 0 };
    let file = syn::parse_str::<syn::File>(CODE).unwrap();
    syn::visit::visit_file(&mut await_visitor, &file);
    dbg!(await_visitor.await_count);
}
```

我只需要在 Visitor::visit_expr_await() 回调中将 await 计数器自增，不需要关心怎么递归层层遍历 Item 到叶子节点

### rustc_ast::visit::Visitor

rustc 源码中除了 AST 提供 Visitor 便于遍历，HIR 和 MIR 也提供了 Visitor trait

其它 rustc package 中大量使用 rustc_ast package 的 visit 相关方法遍历 AST 树
