引用[下面的Demo](https://dengjianping.github.io/2019/03/05/%E8%B0%88%E4%B8%80%E8%B0%88Fn,-FnMut,-FnOnce%E7%9A%84%E5%8C%BA%E5%88%AB.html)。
```Rust
#[derive(Debug)]
struct E {
    a: String,
}

impl Drop for E {
    fn drop(&mut self) {
        println!("destroyed struct E");
    }
}

fn fn_once<F>(func: F) where F: FnOnce() {
    println!("fn_once begins");
    func();
    // func(); // @1
    println!("fn_once ended");
}


fn test_fn_once() {
    let e = E { a: "fn_once".to_string() };
    // let f = move || println!("fn_once move calls: {:?}", e); // @2
    let f = || println!("fn_once calls: {:?}", e);
    fn_once(f);
    println!("main ended");
}


fn fn_<F>(func: F) where F: Fn() {
    println!("fn begins");
    func();
    func();
    println!("fn ended");
}

fn test_fn(){
    let e = E { a: "fn".to_string() };
    let f = move || println!("fn move calls: {:?}", e);
    // let f = || println!("fn calls: {:?}", e);
    fn_(f);
    println!("main ended");
}

fn fn_mut<F>(mut func: F) where F: FnMut() {
    println!("fn begins");
    func();
    func();
    println!("fn ended");
}

fn test_fn_mut(){
    let e = E { a: "fn_mut".to_string() };
    let f = move || println!("fn_mut move calls: {:?}", e);
    // let f = || println!("fn_mut calls: {:?}", e);
    fn_(f);
    println!("main ended");
}
```

1. FnOnce
    检查输出，可以发现在main函数退出后，e才销毁。
    ```
    fn_once begins
    fn_once calls: E { a: "fn_once" }
    fn_once ended
    main ended
    destroyed struct E
    ```
    但如果使用`@2`处的代码，则输出如下。发现e被move到闭包里面去了，并且在闭包退出时就被销毁了。
    ```
    fn_once begins
    fn_once move calls: E { a: "fn_once" }
    destroyed struct E
    fn_once ended
    main ended
    ```
    如果取消`@1`处的注释，那么编译报错。这也是符合闭包是FnOnce的认知的。
1. Fn
    不使用move版本
    ```
    fn begins
    fn calls: E { a: "fn" }
    fn ended
    main ended
    destroyed struct E
    ```
    使用move版本
    ```
    fn begins
    fn move calls: E { a: "fn" }
    fn ended
    destroyed struct E
    main ended
    ```
1. FnMut
    将上面的代码简单改成FnMut，发现无论是move还是非move版本都无法编译，给出`cannot borrow as mutable`错误。
    需要改成`mut func: F`才行。