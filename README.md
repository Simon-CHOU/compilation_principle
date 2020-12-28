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

## MOOC

- [USTC(本提纲来源)](https://www.bilibili.com/video/av32233569/)
- [SE系列](https://study.163.com/series/1001245004.htm)
- [HIT](https://www.bilibili.com/video/av17649289)
- [Stanford](https://www.bilibili.com/video/av27845355)

## 绘图工具

- [graphviz](https://graphviz.gitlab.io/documentation/)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Graphviz Preview](https://marketplace.visualstudio.com/items?itemName=EFanZh.graphviz-preview) 