# 前言

有些小伙伴可能看到这道题目就已经蒙圈了！甚至可能看不懂这道题目在干什么。其实这是一道比较考察基础的面试题，当你明白原理之后，你可能会感觉这道题也就那么回事，但是如果你没有思绪，那你可能觉得这道题很难。

今天我们就来彻底看看这到底在做什么妖！

# 1.题目简介

这道题目很简单，面试官给出下面一段代码，问你输出什么？

**代码如下：**

```xml
let a = 3
console.log(a + a++ * ++a);
```
小伙伴们别看题目简单就不以为意，通常题目越简单，给我们留下的坑就越多！
# 2.到底考察什么？

代码如此简单的一道题到底能够考察些我们什么知识呢？

别看题目简单，考察的知识点还挺多的，大概总结如下。

**知识点：**

* 运算符优先级
* ++运算符
* 表达式
可以看出，如此简单的一行代码就考察了我们三个知识点。

如果我们仔细阅读题目的话，也不难看出它要考察哪些知识点，比如我们分析一下代码：

>代码：a + a++ * ++a
>1.代码是一个整体表达式，这无疑是考察我们最基本的表达式运算。
>2.代码中含有多个不同运算符，无疑是考察我们对运算符优先级的理解
>3.代码中还有++运算符，这是一个比较特殊的而运算符，需要单独考察我们。
# 3.运算符优先级复习

想要正确解答这道题目，运算符优先级的知识点我们是逃不过的，这里我们直接给出一张表，让大家复习一遍运算符的优先级规则。

**运算符优先级：**

|优先级|运算符|    |结合律|
|:----|:----|:----|:----|
|1|后缀运算符：[]    ()    ·    ->    ++    --(类型名称){列表}|    |从左到右|
|2|一元运算符：++    --    !    ~    +    -    *    &    sizeof_Alignof|    |从右到左|
|3|类型转换运算符：(类型名称)|    |从右到左|
|4|乘除法运算符：*    /    %|    |从左到右|
|5|加减法运算符：+    -|    |从左到右|
|6|移位运算符：<<    >>|    |从左到右|
|7|关系运算符：<<=    >>=|    |从左到右|
|8|相等运算符：==    !=|    |从左到右|
|9|位运算符 AND：&|    |从左到右|
|10|位运算符 XOR：^|    |从左到右|
|11|位运算符 OR：\||    |从左到右|
|12|逻辑运算符 AND：&&|    |从左到右|
|13|逻辑运算符 OR：\|\||    |从左到右|
|14|条件运算符：?:|    |从右到左|
|15|赋值运算符：<br>     =         +=        -=       *=       /=      %=       &=       ^=      \|=   <br>   <<=      >>=|    |从右到左|
|16|逗号运算符：，|    |从左到右|

上面是一张完成的运算符优先级表格，小伙伴们可以简单过一遍，重点查看题目中涉及到的运算符。

**注意**：上表中的运算符优先级是基于 C 语言排的。

# 4.++运算符复习

++运算符是一个比较特殊的运算符，所以我们这里单独拿出来复习一遍。之所以特殊，是因为把++放在变量前面和放在变量后面是两种完全不同的效果，比如下面的案例。

**++a 代码示例：**

```xml
let a = 1;
console.log(++a); // 2
```
**a++代码示例：**
```xml
let a = 1;
console.log(a++); // 1
console.log(a); // 2
```
它们的区别也是显而易见的，从上面代码输出结果可以看出，++a 是先将 a 加 1 之后输出结果，而 a++是输出结果之后再让 a 加 1。
# 5.题目解答

解答题目之前我们必须知道运算符优先级以及++运算符的知识点，题目中的表达式比较长，为了更好的解题，我们很有必要将表达式划分为几段，这也就是我们常说的**逐个攻破**！

如何划分表达式就需要用到我们的运算符优先级知识了，我们先将题目中涉及到的运算符排个序。

## 5.1 运算符排序

**代码**：a + a++ * ++a

**涉及的运算符**：+ 、 ++ 、 *

**运算符优先级排序**：++ > * > +

## 5.2 表达式分段

我们可以看到题目中的运算符总共有 3 级，因此我们可以考虑把这个表达式分为 3 段，而且需要根据运算符的优先级来分。

通常来说，小伙伴们可能会觉得哪个优先级更高，就先从哪里入手，但是我们发现这道题目这样做似乎不太利于我们理解，所以我们不防反过来，先从优先级低的入手。

**第一次分段：**

>通过+运算符分段，共分为两端：
>1. a
>2. a++ * ++a
**第二次分段：**

>再通过*运算符分段，此时将代码分为了三段
>1. a
>2. a++
>3.++a
我们发现经过两次分段题目中的表达式就变为了 3 段，也达到了我们的效果了，接下来就看如何计算了。

## 5.3 分段计算

表达式被分为了三段，那么我们分别计算 3 段表达式即可。

**题目回顾：**

```xml
let a = 3
console.log(a + a++ * ++a);
```

**先计算 a++：**

```xml
a++ = 3
```
之所这儿输出 3，是因为++表达式放在变量后面是先输出结果，再计算 a+1，所以 a++整体为 3.
再计算++a：

```xml
++a = 5
```
这里是最容易出错的地方，因为我们前面先执行了 a++，所以此时的 a 应该是 4，然后执行++a 的时候，这一个整体就变为了 5。
**分块计算后的表达式：**

我们可以将每一块的计算结果写成数字代入表达式看一下

```xml
3 + 3 * 5 = 18
```
上面那个表达式相信大家一眼就能看出来了吧，有些小伙伴可能对第一个 a=3 有一点点疑问：为什么++a 的时候 a 变为了 4，而第一个 a 还是为 3 呢？
因为我们再 JS 解释执行代码的时候其实都是从左往右开始的，当碰到第一个 a 时，就是 a 变量，直接赋为 3 即可，而到后面的++a 和 a++都是表达式，都是需要计算得到结果的。

## **5.4 最终结果**

通过上面的分析和计算，我们很容易得出答案：18。

**浏览器测试：**

![图片](data:image/png;base64,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)


# 总结

有些小伙伴觉得这道题非常的变态，实际项目中根本不可能这样写。事实却是如此，但是为什么各大厂还是喜欢考这种面试题呢？大概是为了最快的考察你的基础知识和应变能力吧，毕竟每天面试的人那么多。




