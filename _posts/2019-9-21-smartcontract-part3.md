---
layout: post
title: 智能合约part3
---
前言
现在大部分的语法教程都是0.4.24版本的，但是命令行自动下载的solidity 0.5.0以上的，所以亲测现在还必须得会 0.5.0的语法，总体变动是不大的。而且想比如其他
方法比如降版本，更快一些（之前暑假因为合约版本的问题还卡了几天/(ㄒoㄒ)/~~）。总体语法都是基于0.4.24，只是有些地方需要略微修改。可以配合remix学习。
比如，

    .call()不仅可以获知远程调用执行成功与否,还将获得远程调用执行的返回值

    ABI解码做了新的处理规范,有效防御了"短地址攻击"

    address地址类型细分成 address和 address payable

    uintY和 bytesX不能直接转换

    回退函数必须显式声明为 external可见性

    构造函数必须用 constructor关键字定义

    用于抛出异常的 throw关键字弃用, 函数状态可变性修饰符必须用 view,不能混用 constant和 view

    ...

下面我们将对这些改变一一予以介绍,最后给出一个示例代码,对比展示新旧版solidity代码写法的区别,供大家参考.

显式声明

函数可见性

    函数可见性必须显式声明. 之前, 函数如果不显式声明,将默认 public可见性.

    public: constructor构造函数必须声明为 public可见性,否则编译报错.

    external: 回退函数(fallback function), 接口(interface)的函数必须声明为 external可见性,否则编译报错.

存储位置

    结构体(struct),数组(array),映射(mapping)类型的变量必须显式声明存储位置( storage, memeory, calldata),包括函数参数和返回值变量都必须显式声明.

    external 的函数参数需显式声明为 calldata.

合约与地址

    contract合约类型不再包括 address类型的成员函数,必须显式转换成 address地址类型才能使用 send(), transfer(), balance等与之相关的成员函数/变量成员.

    address地址类型细分为 address和 address payable,只有 address payable可以使用 transfer(), send()函数.

    address payable类型可以直接转换为 address类型, 反之不能.

    但是 address x可以通过 address(uint160(x)),强制转换成 address payable类型.

    如果 contract A不具有 payable的回退函数, 那么 address(A)是 address类型.

    如果 contract A具有 payable的回退函数, 那么 address(A)是 address payable类型.

    msg.sender属于 address payable类型.

转换与填充(PADDING)

UINTY与 BYTESX

    因为填充(padding)的时候, bytesX是右填充(低比特位补0),而 uintY是左填充(高比特位补0),二者直接转换可能会导致不符合预期的结果,所以现在当 bytesX和 uintY长度大小不一致(即X*8 != Y)时,不能直接转换,必须先转换到相同长度,再转换到相同类型.

    10进制数值不能直接转换成 bytesX类型, 必须先转换到与 bytesX相同长度的 uintY,再转换到 bytesX类型

    16进制数值如果长度与 bytesX不相等,也不能直接转换成 byteX类型

ABI

    字面值必须显式转换成类型才能使用 abi.encodePacked()

    ABI编码器在构造外部函数入参和 abi.encode()会恰当地处理 bytes和 string类型的填充(padding),若不需要进行填充,请使用 abi.encodePacked()

    ABI解码器在解析函数入参和 abi.decode()时,如果发现 calldata太短或超长,将直接抛出异常,而不是像之前自动填充(补0)和截断处理,从而有效地遏制了短地址攻击.

    .call()族函数( .call(), .delegatecall(), .staticcall())和 哈希函数( keccak256(),sha256(), ripemd160())只接受一个参数 bytes,且不进行填充(padding)处理.

    .call()空参数必须写成 .call("")

    .call(sig,a,b,c)必须写成 .call(abi.encodeWithSignature(sig,a,b,c)),其他类推

    keccak256(a,b,c)必须写成 keccak256(abi.encodePacked(a,b,c)),其他类推

    另外, .call()族函数之前只返回函数执行成功是否的 bool, 现在还返回函数执行的返回值,即 (bool,bytes memory). 所以之前 boolresult=.call(sig,a,b,c)现在必须写成 (boolresult,bytes memory data)=.call(sig,a,b,c).

不允许的写法

在之前版本的solidity编译,以下不允许的写法只会导致 warnings报警,现在将直接 errors报错.

    不允许声明0长度的定长数组类型.

    不允许声明0结构体成员的结构体类型.

    不允许声明未初始化的 storage变量.

    不允许定义具有命名返回值的函数类型.

    不允许定义非编译期常量的 constant常量. 如 uintconstant time=now;是不允许的.

    不允许 0X(X大写)做16进制前缀,只能用 0x.

    不允许16进制数和单位名称组合使用. 如 value=0xffether必须写成 value=0xff*1ether.

    不允许小数点后不跟数字的数值写法. 如 value=255.0ether不能写成 value=255.ether.

    不允许使用一元运算符 +. 例如 value=1ether不能写成 value=+1ether.

    不允许布尔表达式使用算术运算.

    不允许具有一个或多个返回值的函数使用空返回语句.

    不允许未实现的函数使用修饰符(modifier).

    不允许 msg.value用在非 payable函数里以及此函数的修饰符(modifier)里.

废弃的关键字/函数

    years时间单位已弃用,因为闰年计算容易导致各种问题.

    var已弃用,请用 uintY精确声明变量长度.

    constant函数修饰符已弃用,不能用作修饰函数状态可变性, 请使用 view关键字.

    throw关键字已弃用,请使用 revert(), require(), assert()抛出异常.

    .callcode()已弃用,请使用 .delegatecall(). 但是注意,在内联汇编仍可使用.

    suicide()已弃用, 请使用 selfdestruct().

    sha3()已弃用,请使用 keccak256().

构造函数

    构造函数必须用 constructor关键字定义. 之前,并未强制要求,既可以用合约同名函数定义构造函数,也可以用 constructor关键字定义.

    不允许调用没有括号的基类构造函数.

    不允许在同一继承层次结构中多次指定基类构造函数参数.

    不允许调用带参数但具有错误参数计数的构造函数.如果只想在不提供参数的情况下指定继承关系，请不要提供括号.

其他

    do...while循环里的 continue不再跳转到循环体内,而是跳转到 while处判断循环条件,若条件为假,就退出循环.这一修改更符合一般编程语言的设计风格.

    实现了C99风格的作用域:

    变量必须先声明,后使用.之前,是可以先使用,后声明,现在会导致编译报错.

    只能在相同或嵌套作用域使用.比如 if(){...}, do{...}while();, for{...}块内声明的变量,不能用在块外.

    变量和结构体的循环依赖递归限制在256层.

    pure和 view函数在EVM内部采用 staticcall操作码实现(EVM版本>=拜占庭),而非之前的 call操作码,这使得状态不可更改(state changes disallowed)在虚拟机层面得到保证.
