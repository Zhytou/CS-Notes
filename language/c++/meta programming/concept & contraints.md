# 概念 & 约束

C++20中的concept和constraint主要用来取代enable_if这类traits机制,实现函数重载和SFINAE。

概念(concept)和约束(constraint)的主要定义和作用如下:

概念(concept)定义了模板参数类型需要满足的条件和属性,类似接口。比如Sortable概念规定类型必须支持<和=操作等。

约束(constraint)指定模板参数类型需要满足的概念。比如要求模板参数T满足Copyable和LessThanComparable概念。

使用concepts和constraints可以很清晰地定义函数模板可接受的类型,以及函数行为。比如只允许满足Sortable概念的类型调用排序函数。

比enable_if机制表达能力更强,不再需要依赖内置类型 trait。定义概念和约束更面向接口而不是实现。

满足constraints的函数将自动参与函数重载解析,实现SFINAE效果。不再需要enable_if配合特化实现重载嵌套。

所以概念与constraints取代了原有C++11/14标准库中的许多traits定义,如type_traits,也替代了enable_if这类机制。它们提供了一个更高级的编译期类型检查系统。

但原有的一些trait仍可用于运行期类型信息查询,两者不冲突,实现不同的目的。concepts主要用于编译期类型检查和 constrain 编程。
