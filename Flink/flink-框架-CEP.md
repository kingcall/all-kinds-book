[toc]

## CEP 背景
- 在我们的实际生产中，随着数据的实时性要求越来越高，实时数据的量也在不断膨胀，在某些业务场景中需要根据连续的实时数据，发现其中有价值的那些事件。
> 比如，我们需要在大量的订单交易中发现那些虚假交易，在网站的访问日志中寻找那些使用脚本或者工具“爆破”登录的用户，或者在快递运输中发现那些滞留很久没有签收的包裹等。

## CEP 定义

- Flink CEP 是 Flink 的复杂事件处理库。它允许用户快速检测无尽数据流中的复杂模式也就是在无休止的事件流中检测由一个或者多个简单事件构成的匹配规则(从简单的事件流中发现复杂的事件联系)
- 不过 Flink CEP 仅可用于通过 DataStream API处理。
- Flink 的每个模式包含多个状态，模式匹配的过程就是状态转换的过程，每个状态(state)可以理解成由Pattern构成，为了从当前的状态转换成下一个状态，用户可以在Pattern上指定条件，用于状态的过滤和转换。
- 实际上Flink CEP 首先需要用户创建定义一个个pattern，然后通过链表将由前后逻辑关系的pattern串在一起，构成模式匹配的逻辑表达。然后需要用户利用NFACompiler，将模式进行分拆，创建出NFA(非确定有限自动机)对象，NFA包含了该次模式匹配的各个状态和状态间转换的表达式。


### Flink-cep 三种状态迁移边
- Take: 表示事件匹配成功，将当前状态更新到新状态，并前进到“下一个”状态；
- Procceed: 当事件来到的时候，当前状态不发生变化，在状态转换图中事件直接“前进”到下一个目标状态；
- IGNORE: 当事件来到的时候，如果匹配不成功，忽略当前事件，当前状态不发生任何变化。
CEP 的构成
模式的属性

### 模式之间的联系
- next/notnext（**严格连续**）
- followedBy/notflowedBy（**宽松连续**）
> 允许中间的不匹配 "a followedBy b ",对于[a,c,b1,b2] 可以匹配出 a,b1 然后状态就会被清除
- followedByAny（**非确定性宽松连续**）
> 之前匹配过的还可以再次使用 "a followedByAny b ",对于[a,c,b1,b2] 可以匹配出 a,b1和 a,b2

#### 个体模式
##### 单例模式
##### 循环模式

#### 组合模式
#### 模式组

### 核心概念

#### 模式的有效期
- within

#### 条件
##### 简单条件
- where
- or 
- until

##### 组合条件
- where().or()
- where().where()

##### 终止条件
- 如果使用了oneOrMore 建议使用 .until() 进行条件终止，避免状态无限制增大

##### 迭代条件

#### 量词

### NFA(Non-determined Finite Automaton 不确定的有限状态机)
### CEP的数据结构
#### ShareBuffer
- CEP的规则解析之后，本质上是一个不确定状态的转换机，所以在匹配过程中，每个状态会对应着一个或多个元素
### NFAState
```
private Queue<ComputationState> partialMatches; // 正在进行的匹配
private Queue<ComputationState> completedMatches; // 完成的匹配
```
- 每次接收到新的事件，都会遍历partialMatches来尝试匹配，看是否能够让partialMatch转化为completedMatch。

## 不足
### Pattern这方面
- 不能同时匹配多个Pattern
- 不能动态修改Pattern
- 不支持NotFollowBy结尾语法
- not 不支持 option

### 其他
-  性能问题


## 优势
- **擅长跨事件匹配**
- 支持Event和processing Time 语义
- 对延迟数据处理的友好API
