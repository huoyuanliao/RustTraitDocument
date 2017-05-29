#Rust Result Handle
- 参考:[简谈 Rust 中的错误处理]:http://lotabout.me/2017/rust-error-handling/
 - 参考:[Rust Book 1st](https://doc.rust-lang.org/book/error-handling.html#composing-custom-error-types)
##错误处理是每种编程语言都会遇到的问题，不同的编程语言都基于自己对错误的理解实现了自己的处理机制：
1.   Python:   try exception
2.   C:        错误值包含在返回值中，返回值可能代表正确，也可能代表错误，需要由API编写者和调用者实现达成申明。    
3.   Go-lang:  多返回值，多返回值错误处理代码，在GO语言项目中大量存在，几乎成为Go语言编程的一大特点。多返回值给调用者造成了一些代码负担，但是也因为将错误值和正确值进行区别返回，使返回值定义与正确逻辑分离，摒弃了C的弊端。但是在代码中还是会造成处理错误代码的逻辑夹杂在正常逻辑中。
---
##Rust的错误处理
Rust通过类型系统的支持，从而避免了Null的存在。从类型系统的角度来说，Rust是一门没有Null的语言。
###Option<T>
```
    pub enum Option<T> {
        None,
        Some(T),
    }
```
1 Some究竟是什么：Option作为一个enum，统一了None和Some(T)类型。从而使返回值的类型得到统一。标准库的所有错误逻辑都源自于这样的类型系统的统一。
    Some在标准库中仅仅就是一个泛型类型pub struct<T> Some(T);Some(T)使用struct的new type 类型，将所有的类型统一为Some<T>类型。从而使Option类型成为一个标准的泛型结构。
2 Option<T> 怎么用：Option作为Rust的错误处理的标准类型之一，主要为程序员提供“要么是None要么是某一个类型值”的抽象。这种抽象似的程序员能够得到一个统一的类型值，但是其中可能包含正确的返回值，也可能是一个错误。这个时候需要调用者手动的去解开这个结构，并具体的处理里面遇到的None或者Some<T>情况。这也是Rust强制调用者对错误情况进行处理的设计逻辑。较为基础的处理方法是match：
    ···
    match Some(100) {
        Some(x) => println!("this is a valid value:{}",x),
        None => println!("this is None value"),
    }
    ···
3 我们可以通过查看Option的libstd文档找到所有的Option方法:[Option](https://doc.rust-lang.org/std/option/enum.Option.html)
    这里罗列部分常用的方法，当我们不想判断Option值，而直接假设Option内部是Some<T>，并取出T的内容时，则直接使用Some(100).unwrap()
    就可以得到Some内部的具体值。其他的方法类似，map方法处理Some的情况，并在遇到None时返回。

    * expect()
    * is_some()
    * is_none()
    * map()
    * map_or()
    * map_or_else()
    * ok_or()
    * ok_or_else()
    * and()
    * and_then()
    * or()
    * or_eles()
    * unwrap()
    * unwrap_or()
    * unwrap_or_else()
    * unwrap_or_default()

###Result<T,E>[](https://doc.rust-lang.org/std/result/enum.Result.html)
```
    pub enum Result<T, E> {
        Ok(T),
        Err(E),
    }
```
1   T,E是啥意思：
    T和E在这里都代表泛型类型，其中使用T作为Type的简写，意思是代表一个通用的泛型，E用作Error的简写，代表Error类型的泛型。
    所以Result<T,E>是一个包含两个泛型参数的enum类型。他将Ok(T)和Err(E)统一为Result类型。
2   Ok(T),Err(E) 又是啥意思：
    Ok(),Err()在这里和Some的作用是一致的，使用struct的new type类型，将不同的类型包裹为Ok,Err类型。
    如果有C++的templete基础，这里应该是相当好理解的。
3   Result为啥多了一个泛型E，
    这个E主要用于错误传递。因为Option只能传递None和Some<T>的情况，在实际使用时，这是不够的，基础库开发人员或者API调用者都可能遇到多种错误的情况，开发人员需要将这些错误出现的情况传递出去，让上层调用者做出更好的处理和判断。特别是对于某些鲁棒性要求较高的项目，需要拦截，并处理遇到的各种错误。
4   这个泛型的E如何传递和包裹错误呢：
    Result<T,E>在标准库中的使用非常多，标准库API也通常返回某种聚类T,E类型的Result给调用者，方便调用者判断具体的错误信息，并做出合适的处理。
    ```
    std::fs::File;
    pub struct File {
        inner: fs_imp::File,
    }

    impl File {
        ...
        pub fn open<P: AsRef<Path>>(path: P) -> io::Result<File> {
            OpenOptions::new().read(true).open(path.as_ref())
        }
        ...
    }
    ```
    标准库中结构体File的open方法接受一个path路径，并返回io::Result<File> 类型。这里io::Result<File> 是对标准Result<T,E>的type简写：
    `pub type Result<T> = result::Result<T, Error>;` 这里的Error类型就是io::Error类型。所以io::Result<File>类型的意思是：Result要么返回一个File结构，要么返回一个io::Error类型。
    ```
    std::io::Error
    pub struct Error {
        repr: Repr,
    }
    ```
5. Result<T,E>的常用方法：
    * unwrap()
    * is_ok()
    * is_err()
    * ok()
    * err()
    * as_ref()
    * as_mut()
    * map()
    * map_err()
    * iter()
    * iter_mut()
    * and()
    * and_then()
    * or()
    * or_else()
    * unwrap_or()
    * unwrap_or_else()
    * expect()
    * unwrap_err()
    * expect_err()
    * unwrap_or_default()
6. 可以看到Result和Option存在发部分的相同method，因为Result除了增加了Err类型的表达外，本质上是一样的。

##思考:
    Rust实际上鼓励开发人员定义API或者在调用API时，将结果都包裹到Reust、Option等类型中，并强制要求API调用者处理可能出现的错误情况，调用者可以通过标准库提供的method粗暴的假设错误情况不可能发生(unwrap)，也向调用者提供更加灵活的错误类型转换(map_err)，或者当剥开类型包裹后只处理正确的情况，而原值返回错误类型(map)。目的只有一个：调用者要明确的编写代码，应对错误出现时业务代码应该采取的应对方法，否则类型系统就会出现错误。
##自定义错误类型:
    在Rust Book[](https://doc.rust-lang.org/book/error-handling.html#composing-custom-error-types)中有详细的自定义错误类型的编写方法，其中利用了From trait,Error trait来生成符合通用标准的错误类型，并通过From trait在编译器类型推断时，生成正确的类型转换代码，不需要再显式的进行错误类型转换。