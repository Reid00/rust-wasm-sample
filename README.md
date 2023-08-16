
# Rust 如何使用WASM
## 安装
1. 安装Rust, 参考[官网](https://www.rust-lang.org/tools/install)
2. install wasm-pack `cargo install wasm-pack`
3. [可选] install cargo-generate `cargo install cargo-generate`
4. Python3 用于启动一个简单的 HTTP 服务器

## 案例
1. 创建一个rust lib 项目 `cargo new --lib hello-wasm`
2. 打开项目,结构如下
```sh
-rw-r--r-- 1 zhangbl 197121 3203 Aug 16 15:41 Cargo.lock
-rw-r--r-- 1 zhangbl 197121  224 Aug 16 15:51 Cargo.toml
-rw-r--r-- 1 zhangbl 197121  314 Aug 16 17:35 index.html # 后面新建的
drwxr-xr-x 1 zhangbl 197121    0 Aug 16 18:13 pkg/       # 后面生成的
drwxr-xr-x 1 zhangbl 197121    0 Aug 16 15:27 src/      
drwxr-xr-x 1 zhangbl 197121    0 Aug 16 16:01 target/ 
```

打开src/lib.rs, 删除原先内容，用以下内容取代:
```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
    pub fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("Hello, {}!", name));
}
```

同时在cargo.toml 新增下面内容:
```toml
[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen="0.2"
```

3. 代码解析
- `use wasm_bindgen::prelude::*;`：导入了 wasm_bindgen 库的预导入（prelude）模块，它包含了一些常用的宏和类型。这样可以方便地在代码中使用 wasm_bindgen 提供的功能。
- `#[wasm_bindgen]`：这是一个属性宏，用于标记下面的函数和外部接口需要与 JavaScript 进行绑定
- `extern 块`：在 extern 块内部使用 pub fn alert(s: &str) 定义了一个外部函数接口。这个函数声明了一个参数 s，类型为字符串引用（&str）。 **从 Rust 调用 JavaScript 中的外部函数**
- `pub fn greet(name: &str)`：这是一个公共函数 greet 的定义，它接受一个字符串引用参数 name。**生成 JavaScript 可以调用的 Rust 函数**
- `alert(&format!("Hello, {}!", name))`：在 greet 函数中调用了外部接口函数 alert。它使用 format! 宏将传入的 name 参数插入到格式化的字符串中，然后调用 alert 函数显示警告框，内容为拼接好的问候信息。

4. toml 解析
[lib] 告诉编译器，将要编译的类型是c 语言的动态类型 库

5. 编译
`wasm-pack build --target web` 请注意这步很慢，耗时会很久，请去喝茶。
编译完成之后，出现pkg 包，里面提供了我们项目名的mywasm_bg.wasm 
```sh
total 38
-rw-r--r-- 1 zhangbl 197121  1116 Aug 16 17:39 mywasm.d.ts        
-rw-r--r-- 1 zhangbl 197121  4910 Aug 16 17:39 mywasm.js
-rw-r--r-- 1 zhangbl 197121  2503 Aug 16 16:03 mywasm_bg.js       
-rw-r--r-- 1 zhangbl 197121 17307 Aug 16 18:13 mywasm_bg.wasm     
-rw-r--r-- 1 zhangbl 197121   287 Aug 16 17:39 mywasm_bg.wasm.d.ts
-rw-r--r-- 1 zhangbl 197121   213 Aug 16 18:13 package.json  
```
6. 在项目目录下，新建index.html
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>hello-wasm example</title>
</head>

<body>
    <script type="module">
        import init, { greet } from "./pkg/mywasm.js";
        init().then(() => {
            greet("WebAssembly")
        });
    </script>
</body>

</html>
```

7. 运行 `python -m http.server` 启动一个简单的web 服务, 让后访问 `http://localhost:8000`
![hello](https://cdn.staticaly.com/gh/Reid00/picx-images-hosting@master/20230816/image.35alc2e7oya0.png)
