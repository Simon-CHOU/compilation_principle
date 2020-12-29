# 编译原理复习提纲挈领<sub>by周志寰</sub>

## 学习目标

1. 配合阅读 `Mastering Regular Expression 3rd Edition`，熟练使用正则表达式处理复杂文本
1. 学会定义文法并使用 [FLEX](https://www.geeksforgeeks.org/flex-fast-lexical-analyzer-generator/) 等命令行工具自动生成词法分析程序，应用审批链解析及类似需求
1. 为Java虚拟机、字节码分析、Java性能优化等学习做部分前置知识准备
1. 为学习形式化方法、提高工程代码质量，做部分前置知识准备

## 编译器概述

## 编译器结构

## 编译器实例

## 词法分析的任务

## 词法分析的手工构造

以上暂略，请直接查看文末MOOC

---

![从正则表达式到词法分析器](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/lexcial_analyzer_gen_flow.png)

## 正则表达式 RE(Regular Expression)

## 定义

对给定的字符集 `Σ={c1, c2, ..., cn}`

归纳定义： 

- 空串 `ε` (伊普西龙)是正则表达式
- 对于任意 `c ∈ Σ`，`c` 是正则表达式
- 如果 `M` 和 `N` 是正则表达式，则以下也是正则表达式
  - 选择 `M|N = {M, N}`
  - 连接 `MN = {mn| m∈M, n∈N}`
  - 闭包 `M* = {}`，文献当中经常称此种闭包运算为[Kleene闭包](https://baike.baidu.com/item/Kleene%E6%98%9F%E5%8F%B7/9319320?fr=aladdin)

### 形式表示

递归表示，联系数学归纳法理解

```javascript
e → ε // 空集
  | c  //单个字符
  | e | e  //选择
  | e e  //连接
  | e*  //闭包
```

## 有限状态自动机 Finite Automata

### 定义

`M = {Σ, S, q0, F, δ}`

- Σ（西格玛）：字母表
- S：状态集
- q0：初始状态
- F：终结状态
- δ（德尔塔）：转移函数

### 什么样的子串被认为可以接受

从初始状态开始，通过转移函数在状态集中改变状态，最后到达的状态属于 `F`，则认为串可被接受。

### 自动机例一

![fa_012ab](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/FA_012ab.png)

在上面这个自动机中：

- Σ={a, b}
- S={0, 1, 2}
- q0={0}
- F={2}
- δ={(q0,a)→q1, (q0,b)→q0, (q1,a)→q2, (q1,b)→q1, (q2,a)→q2, (q2,b)→q2}

### 自动机例二

![fa_01ab](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/FA_01ab.png)

在上面这个自动机中：

- Σ={a, b}
- S={0, 1}
- q0={0}
- F={1}
- δ={(q0,a)→q0, (q0,a)→q0, (q0,b)→q1, (q1,B)→q0, (q1,b)→q1}

对比以上两例不难发现：

- 在例一中，一个字符只有一个状态可以转移
- 在例二中，一个字符有多个状态可以转移
- 要判断一个串能否被自动机接受，例二比例一要复杂，因为每一步都需要遍历所有的转换函数，不是一次性就能判断。

## 自动机小结

确定有限状态自动机 `DFA (deterministic finite automata)`

- 对于任意字符，最多有一个状态可以转移
  - δ: S × Σ → S，×表示[笛卡尔积](https://baike.baidu.com/item/%E7%AC%9B%E5%8D%A1%E5%B0%94%E4%B9%98%E7%A7%AF?fromtitle=%E7%AC%9B%E5%8D%A1%E5%B0%94%E7%A7%AF&fromid=1434391)

非确定有限状态自动机 `NFA (non-deterministic finite automata)`

- 对于任意字符，有多于一个状态可以转移
  - δ: S × (Σ ∪ ε) → [℘](https://unicode-table.com/cn/2118/)(S)，℘表示[幂集（Power Set）](https://baike.baidu.com/item/%E5%B9%82%E9%9B%86/9555341?fr=aladdin)

### `DFA` 的实现

![fa_012ab](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/FA_012ab.png)

把自动机看做有向图，使用邻接矩阵实现，如下表：

|状态\字符|a|b|
|-|-|-|
|0|1|0|
|1|2|1|
|2|2|2|

## `RE` 转 `NFA`

### Thompson算法

- 基于对RE结构做归纳（数学归纳法）
  - 对基本的RE直接构造
  - 对符合的RE递归构造
- 递归算法，容易实现
  - 代码量很少

正则表达式的5种形式化定义与对应 `NFA` 的转化如下：

1. e → ε
![空串](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/thompson1.png)
1. e → c
![字符](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/thompson2.png)
1. e → e | e
![选择](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/thompson3.png)
1. e → e e
![连接](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/thompson4.png)
1. e → e*
![闭包](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/thompson5.png)

- Q：为什么转化出来的 `NFA` 都包含了很多 `ε`？
- A：保留ε的递归算法更加工整

练习1：构造C语言标识符的 `NFA`，以子母或下划线开头，后跟零个或多个子母、数字或下划线，`^[a-zA-z_]\w*$`， `\w` 匹配任何字类字符，包括下划线。与"[A-Za-z0-9_]"等效。
![c_identifier](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/thompson_nfa_c_identifier.png)

练习2：同上，十进制整数的 `DFA`。
![decimal_int](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/thompson_nfa_decimal_int.png)

不难从上面的练习看出正则的语法糖，如：`[c1-cn] == [c1|c2|...|cn]`
我们可以引入更多的语法糖，来简化构造

|语法糖|原始正则|真实意义|
|-|-|-|
|[c1-cn]|c1\|c2\|...\|cn|c1到cn中的任意一个|
|e+||一个或多个e|
|e?||零个或多个e|
|"a*"||a*自身，不是a的Kleene闭包|
|e{i,j}||i到j个e的连接|
|.||除'\n'外的任意字符|
<!--todo-->

语法糖的本质是对5种原始表达式形式的封装。同样的例子可以联想到，图灵机只定义了：赋值、跳转两种运算，汇编及各种高级语言都是对两种运算进行的不同抽象层次的封装，目的就在于简化构造。

练习3：画出`a(b|c)*`的 `NFA`
![abc_nfa](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/abc_nfa.png)

## [`NFA` 转 `DFA`](https://www.geeksforgeeks.org/program-implement-nfa-epsilon-move-dfa-conversion/)

### `ε-闭包`的计算：深度优先

```javascript
// ε-closure：基于深度优先遍历的算法
set closure = {};

void eps_closure (x)
  closure += {x}
  foreach (y: x--ε-→ y)
    if (!visited(y))
      eps_closure (y)
```
<!--todo-->
### [`ε-闭包`的计算：广度优先](http://personal.kent.edu/~rmuhamma/Compilers/MyCompiler/NFAtoDFA.htm)

```javascript
// ε-closure：基于广度优先遍历的算法
set closure = {};
Q = [];
void eps_closure (x) =
  Q = [x];
  while (Q not empty)
    q <- deQueue (Q)
    closure += q
    foreach (y: q--ε-→y)
      if(!visited(y))
        enQueue (Q, y)
```
<!--todo-->
### 子集构造算法：工作表算法

```javascript
// ε-closure：子集构造算法：工作表算法
q0 <- eps_closure (n0) //计算第一层"边界"，依然是BFS的思路
Q <- {q0};
workList <- q0
while (workList != [])
  remove q from workList
  foreach (character c) //如果字符集为ASCII码，则 character c 有256个，即遍历256遍
    t <- e-closure (delta (q, c)) //delta(q,c) 是q集合经过c可达的新状态集合,此集合求ε闭包，得到t，
    D[q, c] <- t //把t加进DFA中
    if(t not in Q)
      add t to Q and workList
```

以上一节练习3：`a(b|c)*` 的 `NFA` 为例，用子集构造法求ε闭包
![abc_nfa](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/abc_nfa.png)

|step|q0|Q|workList(FIFO)|q|c|delta(q,c)|t|D[q,c]，仅δ|
|-|-|-|-|-|-|-|-|-|
|1|{n0}|{{n0}}|{{n0}}|-|-|-|-|-|
|1|{n0}|{{n0}}|{}|{n0}|a|-|-|-|
|1|{n0}|{{n0}}|{}|{n0}|a|{n1}|-|-|
|1|{n0}|{{n0}}|{}|{n0}|a|{n1}|{n1,n2,n3,n4,n6,n9}|-|
|1|{n0}|{{n0}}|{}|{n0}|a|{n1}|{n1,n2,n3,n4,n6,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9}}|{{n1,n2,n3,n4,n6,n9}}|{n0}|a|{n1}|{n1,n2,n3,n4,n6,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9}}|{{n1,n2,n3,n4,n6,n9}}|{n0}|b|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9}}|{{n1,n2,n3,n4,n6,n9}}|{n0}|c|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9}}|{}|{n1,n2,n3,n4,n6,n9}|a|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9}}|{}|{n1,n2,n3,n4,n6,n9}|b|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9}}|{}|{n1,n2,n3,n4,n6,n9}|b|{n5}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9}}|{}|{n1,n2,n3,n4,n6,n9}|b|{n5}|{n3,n4,n5,n6,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9}}|{}|{n1,n2,n3,n4,n6,n9}|b|{n5}|{n3,n4,n5,n6,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9}}|{{n3,n4,n5,n6,n8,n9}}|{n1,n2,n3,n4,n6,n9}|b|{n5}|{n3,n4,n5,n6,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9}}|{{n3,n4,n5,n6,n8,n9}}|{n1,n2,n3,n4,n6,n9}|c|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9}}|{{n3,n4,n5,n6,n8,n9}}|{n1,n2,n3,n4,n6,n9}|c|{n7}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9}}|{{n3,n4,n5,n6,n8,n9}}|{n1,n2,n3,n4,n6,n9}|c|{n7}|{n3,n4,n6,n7,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9}}|{{n3,n4,n5,n6,n8,n9}}|{n1,n2,n3,n4,n6,n9}|c|{n7}|{n3,n4,n6,n7,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{n1,n2,n3,n4,n6,n9}|c|{n7}|{n3,n4,n6,n7,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n6,n7,n8,n9}}|{n3,n4,n5,n6,n8,n9}|a|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n6,n7,n8,n9}}|{n3,n4,n5,n6,n8,n9}|b|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n6,n7,n8,n9}}|{n3,n4,n5,n6,n8,n9}|b|{n5}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n6,n7,n8,n9}}|{n3,n4,n5,n6,n8,n9}|b|{n5}|{n3,n4,n5,n6,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n6,n7,n8,n9}}|{n3,n4,n5,n6,n8,n9}|b|{n3,n4,n5,n6,n8,n9}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n6,n7,n8,n9}}|{n3,n4,n5,n6,n8,n9}|c|{n7}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n6,n7,n8,n9}}|{n3,n4,n5,n6,n8,n9}|c|{n7}|{n3,n4,n6,n7,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{{n3,n4,n6,n7,n8,n9}}|{n3,n4,n5,n6,n8,n9}|c|{n7}|{n3,n4,n6,n7,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|a|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|b|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|b|{n5}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|b|{n5}|{n3,n4,n5,n6,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|b|{n5}|{n3,n4,n5,n6,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n6,n7,n8,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|c|{}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n6,n7,n8,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|c|{n7}|{}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n6,n7,n8,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|c|{n7}|{n3,n4,n6,n7,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n6,n7,n8,n9},b)→{n3,n4,n5,n6,n8,n9}}|
|1|{n0}|{{n0},{n1,n2,n3,n4,n6,n9},{n3,n4,n5,n6,n8,n9},{n3,n4,n6,n7,n8,n9}}|{}|{n3,n4,n6,n7,n8,n9}|c|{n7}|{n3,n4,n6,n7,n8,n9}|{({n0},a)→{n1,n2,n3,n4,n6,n9},({n1,n2,n3,n4,n6,n9},b)→{n3,n4,n5,n6,n8,n9},({n1,n2,n3,n4,n6,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n5,n6,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n5,n6,n8,n9},c)→{n3,n4,n6,n7,n8,n9},({n3,n4,n6,n7,n8,n9},b)→{n3,n4,n5,n6,n8,n9},({n3,n4,n6,n7,n8,n9},c)→{n3,n4,n6,n7,n8,n9}}|

通过子集构造法，我们得到了 `Q` 和 `D`，`Q` 为转化成的 `DFA` 的状态集 `S`，`D` 为 `DFA` 的转移函数 `δ`。
令：

- {n0} = q0
- {n1,n2,n3,n4,n6,n9} = q1
- {n3,n4,n5,n6,n8,n9} = q2
- {n3,n4,n6,n7,n8,n9} = q3

原来 `NFA` 中的终结状态是 `n9`，新构造出来的集合 `Q` 中，只要包含  `n9` 的状态就是终结状态，故 `q1`,`q2`,`q3` 都是接受状态。同理，包含起始状态 `n0` 的 `q0`，即为目标 `DFA` 的起始状态。

*思考：转化成的 `DFA` 可不可能有多个起始状态？*

由此可以得到 `DFA` 的定义：

- Σ={a, b}
- S={q0, q1, q2, q3}
- q0={q0}
- F={q1, q2, q3}
- δ={(q0,a)→q1,(q1,b)→q2,(q1,c)→q3,(q2,b)→q2,(q2,c)→q3,(q3,b)→q2,(q3,c)→q3}

绘制 `DFA` 如下图：
![abc_dfa](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/abc_dfa.png)
对比原 `NFA`：
![abc_nfa](https://raw.githubusercontent.com/Simon-CHOU/compilation_principle/master/img/abc_nfa.png)

至此，完整地实现了 `NFA` 到 `DFA`的转化。

## `DFA` 最小化

Hopcroft 算法

## 从 `DFA` 生成分析算法
## 语法分析的任务
## 上下文无关文法和推导
## 分析树和二义性文法
## 自顶向下分析
## 递归下降分析算法
##  LL(1)分析算法
### LL(1)分析的冲突处理
## LR(0)分析算法
## SLR分析算法
##  LR(1)分析算法
##  LR(1)分析工具
## 语法制导翻译
## 抽象语法树
## 抽象语法树的自动生成
## 语义分析的任务
## 语义规则及实现
## 符号表
## 语义分析中的其它问题

## [课程作业](https://mooc.study.163.com/learn/1000002001?tid=2403024009#/learn/testlist)

### 第一单元：编译器介绍

第一章作业

题目：（本题目是一个动手实践类题目，需要具备C语言和数据结构基础。）

在课程中，我们讨论一个小型的从表达式语言Sum到栈计算机Stack的编译器，

在附件中，你能找到对该编译器的一个C语言实现，但这个实现是不完整的，请你

把缺少的代码补充完整（不超过10行代码）。

加分题：实现常量折叠优化。

### 第二单元：词法分析（Part I）

第二单元作业：词法分析器

（10分）
在这部分中，你将使用图转移算法手工实现一个小型的词法分析器。

- 分析器的输入：存储在文本文件中的字符序列，字符取自ASCII字符集。文件中可能包括下面几种记号：关键字if、符合C语言标准的标识符、无符号整型数字、空格符、回车符\n。

- 分析器的输出：打印出所识别的记号的种类、及记号开始行号、开始列号信息。

注意：1. 忽略空格及回车符；2. 对于标识符和数字，要输出符号的具体词法单元（见下面的示例）。

【示例】对于下面的文本文件：

```txt
ifx if iif       if  234

iff     if
```

你的输出应该是（注意，因为文本显示的原因，列号信息可能不一定准确）：

```c
ID(ifx) (1, 1)

IF        (1, 4)

ID(iif)  (1, 8)

IF       (1, 13)

NUM(234) (1, 16)

ID(iff) (2, 1)

IF       (2, 8)
```

### 第三单元：词法分析（Part II）

第三单元作业
（9分）
给定如下的正则表达式 (a|b)((c|d)*)，请完成如下练习：

（1）使用Thompson算法，将该正则表达式转换成非确定状态有限自动机（NFA）；

（2）使用子集构造算法，将该上述的非确定有限状态自动机（NFA）转换成确定状态有限自动机（DFA）；

（3）使用Hopcroft算法，对该DFA最小化。

### 第四单元：语法分析（Part I）

算术表达式的语法分析器
（10分）
在这个题目中，要求你完成一个针对算术表达式的语法分析器。该算术表达式的上下文无关文法是：

```c
E -> E + T

   | E - T

   | T

T -> T * F

   | T / F

   | F

F -> num

   | (E)
```

请下载我们提供的C代码框架，并把其中缺少的部分补充完整。

代码下载地址：http://staff.ustc.edu.cn/~bjhua/mooc/exp.zip

### 第五单元：语法分析（Part II）

台式计算器的设计与实现
（10分）
在这个题目中，你将实现一个简单的台式计算器。这个台式计算器的功能像在最后一个讲义中演示的例子一样：即用户可以在控制台上交互输入算术表达式，你的程序判断该表达式是否合法，不合法的话报错并退出运行。

你的程序涉及表达式的部分要支持如下的表达式：

```c
E -> n

     | E + E

     | E - E

     | E * E

     | E / E

     | (E)
```

其中n是任意的非负整数（注意：在我们演示的例子中，n只是单个字符的整数，所以这个地方你需要做些扩展，这些扩展同时需要涉及修改词法分析yylex函数）。

如果在Linux系统上，那么bison应该是默认安装可用的；如果你需要在Windows上完成，你可以下载Windows平台上的bison：http://staff.ustc.edu.cn/~bjhua/mooc/bison.exe

注意：1。安装目录不能包含空格汉字等特殊字符；2。安装完成后把安装目录加到环境变量中。

### 第六单元：语法制导翻译与抽象语法树

抽象语法树
（10分）
【抽象语法树】

在这个题目中，你将完整的实现抽象语法树（包括数据结构的定义、语法树的生成等）。首先，请下载我们提供的代码包：http://staff.ustc.edu.cn/~bjhua/mooc/ast.zip 

代码的运行方式是：

首先生成语法分析器：

```shell
  $ bison exp.y
```

然后生成编译器：

```shell
  $ gcc main.c exp.tab.c ast.c
```

最后使用编译器编译某个源文件：

```shell
  $ a.exe <test.txt
```

在提供的代码里，我们已经提供了抽象语法树的定义、若干操作、及由bison生成语法树的代码框架。你的任务是：

进一步完善该代码框架，使其能够分析减法、除法和括号表达式；（你需要修改语法树的定义，修改bison源文件及其它代码）

重新研究第一次作业中的从Sum编译到Stack的小型编译器代码，把他移植到目前的代码框架中，这样你的编译器能够从文本文件中读入程序，然后输出编译的结果。（注意，你必须扩展你的编译器，让他能够支持减法和除法。）

### 第七单元：语义分析

C--语言的语义分析器
（10分）
在这个题目中，你将亲自动手实现C--语义的语义分析器。具体的题目要求见：http://staff.ustc.edu.cn/~bjhua/mooc/semant.html

### 第八单元：代码生成

C--语言的代码生成器
（10分）
在这个题目中，你将亲自动手实现C--语义的语义分析器。具体的题目要求见：http://staff.ustc.edu.cn/~bjhua/mooc/semant.html

### [**期末考试**](https://mooc.study.163.com/learn/1000002001?tid=2403024009#/learn/examlist)

#### 1.（20分）

给定如下的正则表达式

```c
（a|b)(c|d)*
```

（1）使用Thompson算法将其转换成NFA；

（2）使用子集构造算法将NFA转换成DFA；

（3）使用Hopcroft算法将上述DFA最小化。

#### 2.（30分）

对于下述的两个上下文无关文法：

（第一个文法）

```txt
E -> n + E

   | n
```

（第二个文法）

```txt
E -> E + n

   | n
```

问题：

简要说明为什么上述两个文法产生的语言是一样的；

哪个文法可以用LL(1)分析；为什么？

哪个文法可以用LR(1)分析；为什么？

用LR分析法，对这两个文法的分析效率哪个高？为什么？

#### 3.（10分）

在C语言中，函数在使用前必须声明其原型；而在Java中，调用其它类中的方法不需要提前进行原型的声明。这两种不同的规定，对语义检查器有什么影响？

#### 4.（10分）

在类型检查前就进行编译优化会有什么问题？请简单解释。

#### 5.（20分）

考虑如下两种不同结合性的表达式：

（第一种）

```c
((k1+k2)+...+kn-1)+kn
```

（第二种）

```c
k1+(k2+...(kn-1 + kn))
```

对栈式计算机生成代码的话，哪种表达式会消耗更多的操作数栈的空间。简要解释你的结论。

#### 6.（10分）

你学习本课程有何收获？对本课程有什么意见建议？对下轮授课有何期待？请提出你的建议。【任何答案，除非没有作答，都将得到满分。】

## MOOC

- [USTC Cimpiler 2014 Fall](http://staff.ustc.edu.cn/~bjhua/courses/compiler/2014/),[网易云课堂](https://study.163.com/instructor/3357972.htm),[B站搬运：BV1m7411d7iS](https://www.bilibili.com/video/BV1m7411d7iS)
- [SE系列](https://study.163.com/series/1001245004.htm)
- [HIT](https://www.bilibili.com/video/av17649289)
- [Stanford](https://www.bilibili.com/video/av27845355)

## 参考书

- 课程的主要参考书：《编译器工程》（第二版）[Keith D. Cooper, Linda Torczon — Engineering: A Compiler 2nd Edition](https://www.amazon.com/Engineering-Compiler-Keith-Cooper/dp/012088478X) [pdf下载](http://www.r-5.org/files/books/computers/compilers/writing/Keith_Cooper_Linda_Torczon-Engineering_a_Compiler-EN.pdf)
- 课程的其它参考书：
  - 《现代编译器实现---C语言描述》
  - 《编译原理：技术与工具》
  - 《高级编译器设计与实现》

## 绘图工具

- [graphviz](https://graphviz.gitlab.io/documentation/)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Graphviz Preview](https://marketplace.visualstudio.com/items?itemName=EFanZh.graphviz-preview) 