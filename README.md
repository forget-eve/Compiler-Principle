# 课前须知
> 评分与密码学大同小异，作业和点名不计入总分，但是作为调分的依据，作业分计入标准（是否准时，是否抄袭）。
> 
> 课程难度:前难后稍难，不建议自学和考前突击。
> 
> 期末闭卷，主要为概念理解和掌握方法。
> 
> 学期总评:期末80，实验20

# 第一章:引论
## 1.1 编译器概述
> 以下为编译的各个阶段
### 1.1.1 词法分析
<a href="#/?id=second">详见第二章</a>

> 又称线性分析或扫描
> 
> 字符流是指源程序的编程语句，类似 `position=initial+rate*60`
```mermaid
graph LR
A[字符流] -->|编程语言语法规则| B(记号流)
```
> 注:记号流格式:`<记号名，属性名>`，记号名是同类词法单元共用名称，属性值为区分其他同类词法单元实例的特征值，若某类词法单元只有一个实例，则属性值可省略。例如
```mermaid
graph LR
A[position] -->|有同类initial,rate实例,存在属性值| B(形成记号< id,1 >)
C[=] -->|无同类| D(形成记号< assign >)
E[60] -->|无同类| F(形成记号< number >)
```
最终字符流转化为记号流 `<id，1><=><id，2><+><id，3><*><60>`（为直观某些部分改写）

### 1.1.2 语法分析
### 1.1.3 语义分析
### 1.1.4 中间代码生成
### 1.1.5 代码优化
### 1.1.6 代码生成
### 1.1.7 符号表管理
### 1.1.8 阶段的分组
### 1.1.9 解释器
## 1.2 编译器技术的应用（只做了解，不做考察）

# 第二章 词法分析
<span id="second"></span>

```mermaid
graph LR
A[源程序] --> B[词法分析器]
B <--> C[符号表]
C <--> D[语法分析器]
B --> |记号token|D
D --> |取下一个记号|B
```

- 词法分析器：把构成源程序的字符流翻译成记号流，还完成和用户接口的一些任务
  > 如： `position = initial + rate * 60 → <id，1><=><id，2><+><id，3><*><60>`
  > 1. 剥去源程序的注解和（由空格、制表或换行符等引起的）空白；
  > 2. 把来自编译器各个阶段的错误信息和源程序联系起来；
  > 3. 如果源语言支持宏定义，那么对它们的预处理也可以在词法分析器完成。
- 记号：具有独立含义的最小词法单位
- 围绕词法分析器的自动生成展开
- 介绍正规式、状态转换图和有限自动机概念

## 2.1 词法记号与属性
### 2.1.1 词法记号、模式、词法单元
#### 定义
- 词法记号（简称记号）：是由记号名和属性值构成的二元组，*属性值不是必须项*。记号名为语法分析的输入符号。
  > *记号名是代表一类词法单元的抽象名字，比如标识符 <sup id="back1"><a href="#/?id=footnote1">1✍</a></sup> relation（关系，如<,>...，统称为relation）和某个特定的关键字如if（就是代表if判断）*
  > 
  > **粗俗理解就是关键词是某个编程语言中的语法词或者内置函数名，标识符为符合某种规则的变量或者函数名称**

- 模式：一个记号的模式描述属于该记号的词法单元形式。 ***在一个关键字作为一个记号的情况下，它的模式就是构成该关键字的字符序列。如if为模式为i,f***
  > 对于标识符和其他一些记号，他们的模式有更复杂的结构并且有很多字符串可以 **匹配** 它们，比如relation中的模式有<,>...。
  > ***为便于理解，不妨认为模式就是一个匹配规则，也就是当某个实例匹配到一个匹配规则时，也就是匹配到了一个记号名的模式***
  > > ***举个例子（C语言中）：printf("Total=%d\n",score);该语句中printf和score是匹配到id模式（可见下表即由字母开头的字母数字串）的词法单元，而"Total=%d\n"是匹配到litera模式(引号 " 和 " 之间任意不含引号本身的字符串)的词法单元***
- 词法单元（又称单词）：是源程序中匹配一个记号模式的字符序列，由词法分析器识别为该记号的一个实例。

<div align="center">
  <h6><strong>记号的例子</strong></h6>
  <table>
  <thead>
    <tr>
      <th>记号名</th>
      <th>词法单元例举</th>
      <th>模式的非形式描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>if</td>
      <td>if</td>
      <td>字符i, f</td>
    </tr>
    <tr>
      <td>for</td>
      <td>for</td>
      <td>字符f, o, r</td>
    </tr>
    <tr>
      <td>relation</td>
      <td>&lt;, &le;, =, …</td>
      <td>&lt; 或 &le; 或 = 或 …</td>
    </tr>
    <tr>
      <td>id</td>
      <td>sum, count, D5</td>
      <td>由字母开头的字母数字串</td>
    </tr>
    <tr>
      <td>number</td>
      <td>3.1, 10, 2.8E12</td>
      <td>任何数值常数</td>
    </tr>
    <tr>
      <td>literal</td>
      <td>"seg. error"</td>
      <td>引号 " 和 " 之间任意不含引号本身的字符串</td>
    </tr>
  </tbody>
</table>
</div>

#### 历史上词法定义中的一些问题
- 忽略空格（空格无意义，除非在字符串中，不作为语法单元的分隔符）带来的困难
	> DO 8 I = 3. 75 等同于 DO8I=3. 75 ，只有看到小数点，才知道DO不是关键字，而DO8I为一个关键字.
 
	> DO 8 I = 3, 75有7个记号，只有看到逗号才知道DO是关键字，8为语句标号，I为标识符。
- 关键字不保留
	> IF THEN THEN THEN=ELSE；ELSE …

- 关键字、保留字和标准标识符的区别
  > 保留字是语言预先确定了含义的词法单元。
  > > ***便于理解，相当于没有功能的关键词，还是不能当普通标识符使用。***
  
  > 标准标识符也是预先确定了含义的标识符，但程序可以重新声明它的含义。

### 2.1.2 词法记号的属性
> 为了区分同一个记号中的不同单词，比如记号relation中的单词<,>...所以需要给记号以属性，用属性来记住记号的附加信息，以便需要时使用。
> ***通俗来讲，记号名影响语法分析的决策，属性影响记号的翻译***

- 例如：position = initial + rate * 60的记号和属性值：
	- [x] <id，指向符号表中position条目的指针>
	- [x] < assign_op >(此处没有属性值，是因为记号名已经足以判断词法单元)
	- [x] <id，指向符号表中initial条目的指针>
	- [x] < add_op >
	- [x] <id，指向符号表中rate条目的指针>
	- [x] < mul_op >
	- [x] <number，整数值60>

#### 2.1.3 词法错误
- 词法分析器对源程序采取非常局部的观点
  	> 例：难以发现下面的错误

  	> fi (a == f (x) ) 
- 在实数是“数字串.数字串”格式下，可以发现下面的错误
	> 123.x
- 紧急方式的错误恢复
	> 删掉当前若干个字符，直至能读出正确的记号
- 错误修补
	> 进行增、删、替换和交换字符的尝试

<div id="footnote" style="font-size:10px">
	<h6 id="footnote1">
		[1]:标识符（Identifier）是指用来表示程序中变量、函数、类、对象或其他命名实体的名称或符号。标识符通常由字母、数字和下划线组成，并且必须遵循特定的命名规则和约定，以确保程序的正确性和可读性。不同的编程语言可能对标识符的命名规则有不同的要求，但通常包括以下一般规则：1.标识符必须以字母（包括大写和小写字母）或下划线(_)开头。2.标识符可以包含字母、数字和下划线。3.标识符通常区分大小写，意味着大写字母和小写字母被认为是不同的标识符。4.标识符不能是编程语言的关键字或保留字，这些关键字具有特殊含义，用于编程语言的语法和语义。<a href="#/?id=back1">➹返回</a>
	</h6>
</div>
