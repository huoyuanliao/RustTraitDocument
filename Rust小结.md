#Rust小结
##Rust的通用编程方法
---
1. libs:
	+ API导出
	  - 编写一个库的主要目的就是导出基础方法和数据结构给开发人员使用。通常情况下是使用方法是：
		pub struct XXX {...} 
		pub enum XXX {...}
		pub fn func() {...}

	  - 从代码实践来看，Rust对API的导出并不集中，因为他并不像C/C++头文件一样可以将公开的数据结构和方法直接详细的罗列出来。这对于探索Rust库文件的功能和数据结构存在不直观的问题我的解决方法很简单：在源代码文件中，对源代码使用mod this {...} 进行包裹，这样源代码文件中的API均有一层mod的命名空间。利用该命名空间，在源代码文件的头部，将需要导出的代码接口使用 pub use self::this::XXXX；进行集中导出。这样就达到了将API接口、数据结构进行集中导出的目的。这与lib.rs的功能设定是重合的，lib.rs 能够统一的标注整个库代码的导出API，而源文件头部的pub use self::this::XXX更多的是为了对单一源文件进行详细的说明。

	+ enum和struct:
		- 在我的实践中发现，enum在通用库开发中使用更多。在更特别的情况下，enum几乎是为了支持Option/Result而存在的，所以在通用库开发中，enum也充当了类似的角色，使用enum定义自身的ErrorType成为常见的enum使用方法。当然enum+match的使用模式并不限于Option/Result的情况。enum作为Rust类型系统的主要类型，类似C/C++的Union,可以统一enum内部申明的所有类型，并在编译器进行类型推断的过程中，充当重要作用。例如：enum Option 统一了Some(T)和None类型，这两个类型都认为是Option类型。更准确的说，我们使用的是Option::Some(T)和Option::None类型。
		  enum的第二个用法: enum FileType { ReadOnly, WriteOnly, ReadWrite} 可以用于编写状态机代码，标识状态以及类型。
	+ Trait:
		- Trait是Rust类型系统中又一大特色，Trait类似interface或者protocol的概念，但是在Rust中Trait却承担了不止interface的角色，因为Trait支持default Trait以及depends。对于长期使用面向对象编程范式的coder可能并不会体会到面向对象编程的缺陷。但是随着对OOP编程范式的反思和functional program的流行，面向对象编程的一些缺陷逐渐暴露出来。OOP将数据资源设定为一个对象，并对该对象赋予method，这个思路的缺陷是，coder实际上从编码和API申明上，更加关注method而非该method所属的class。可以和python的鸭子类型做一下比较，Python代码中并不介意被调用则的class类型，只要该class 实现了__iter__魔术方法，则代表调用者就可以遍历该class类型。所以class除了封装了数据外，他的类型实际上并不重要。而在API中限定参数的class类型，反而掣肘了API的描述。
		- depends:
		  Rust基于Trait的类型系统，标准库将基础通用的特性进行分割，并单独暴露给调用者使用，这是不得不这样做的。这会导致一个问题，基础Trait多而繁杂，互相之间可以进行多种的组合，描述不同的特性。depends可以将多个Trait合并为新的Trait，并单独进行实现，这是一个不得不需要的处理Trait的方式。

2. bin:
	bin代码结构简单很多，所有的源文件都为main.rs服务。较为通行的bin代码结构通常包含src/bin/main.rs  src/lib.rs。在这种代码结构中，首先将自身需要实现的功能封装为库代码的形式在lib.rs中进行导出，并在bin代码中调用库自身导出的API，实现具体的功能。这有助于代码的组织和阅读。同时能够满足既编译成库又编译成bin的目的。方便程序员使用。

