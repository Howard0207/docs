# Promise A+

[promise]()表示一个异步操作的最终结果。和[promise]()进行交互的主要方式是通过它的then方法，该方法注册回调要么接收一个[promise]()的最终值，要么接收[promise]()不能[fullfilled]()的原因。

规范详细描述了[then]()方法的欣慰，提供了一个可交互操作的基础，所有符合<span style="color:red">promise/A+</span>规范的[promise]()都可以依靠该基础来实现。因此，这个规范应该被认为是时分稳定的。



## 1. 术语

* 1.1. promise: 一个拥有负荷这个规范的行为的[then]()方法的对象或函数
* 1.2. thenable:定义了一个[then]()方法的对象或函数
* 1.3. 值(value): 任意合法的JavaScript值（包括 undefined， thenable， promise）。
* 1.4. 异常(exception)：使用throw语句抛出的一个值
* 1.5. 原因(reason)： 表示promise为什么被拒绝的一个值

## 2. 必要条件

### 2.1. Promise 状态

promise 必须是这三个状态中的一种： 等待态[pending](), 解决态[fullfilled]()或拒绝态[rejected]()

* 2.1.1. 当一个promise处于等待状态的时候：
  * 2.1.1.1. 可能变为<b>解决</b>或者<b>拒绝</b>状态
* 2.1.2. 当一个promise处于<b>解决</b>状态的时候：
  * 2.1.2.1. 一定不能转换为其它任何状态
  * 2.1.2.2. 必须有个不能改变的<b>值</b>
* 2.1.3. 当一个promise处于<b>拒绝</b>状态的时候：
  * 2.1.2.1. 一定不能转换为其它任何状态
  * 2.1.2.2. 必须有个不能改变的原因

## 2.2. then 方法

**Promise**必须提供一个**then**方法来访问当前或最终的值或原因。

**Promise**的**then**方法接收两个参数：

```js
promise.then(onFullfilled, onRejected)
```

* 2.2.1 onFullfilled   和 onRejected 都是可选参数

