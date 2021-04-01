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

### 2.2.1 **onFullfilled**   和 **onRejected** 都是可选参数

* 2.2.1.1. 如果onFullfilled不是一个函数，它必须被忽略
* 2.2.1.2. 如果onRejected不是一个函数，它必须被忽略

### 2.2.2. 如果**onFullfilled**是一个函数

* 2.2.2.1. 它必须在**promise**被**解决**后调用，promise的**值**作为它的第一个参数。
* 2.2.2.2. 它一定不能再promise被解决前调用。
* 2.2.2.3. 它一定不能被调用多次。

### 2.2.3. 如果**onRejected**是一个函数

* 2.2.3.1. 它必须在**promise**被**拒绝**之后调用，promise的**原因**作为它的第一个参数
* 2.2.3.2. 它一定不能在**promise**被<b>拒绝</b>之前调用
* 2.2.3.3. 它一定不能被调用多次。

### 2.2.4. 在执行上下文栈中只包含平台代码之前， onFullfilled 或onRejected 一定不能被调用【3.1】

### 2.2.5. ***onFullfilled***  和 ***onRejected***  一定被作为函数调用（没有***this***值）【3.2】

### 2.2.6. 同一个***promise*** 上的 ***then*** 可能被调用多次

- 2.2.6.1. 如果***promise*** 被解决，所有响应的onFullfilled回调必须按照他们原始调用then的顺序执行
- 2.2.6.2. 如果***promise*** 被拒绝，所有响应的onRejected回调必须按照他们原始调用then的顺序执行

###  2.2.7. ***then***  方法必须返回一个 ***promise*** 对象 【3.3】

- 2.2.7.1. 如果 ***onFullfilled***  或 ***onRejected***  返回一个值x，运行promise 解决程序[[Resovle]]
- 2.2.7.2. 如果 ***onFullfilled***  或 ***onRejected***  抛出一个异常e， promise2必须用e作为原因被拒绝
- 2.2.7.3. 如果 ***onFullfilled***  不是一个函数并且 promise1 被解决， promise2必须用与promise1相同的值被解决
- 2.2.7.4. 如果 ***onRejected***  不是一个函数并且promise1 被拒绝， promise2必须用与promise1相同的原因被拒绝。

##  2.3. Promise 解决程序

***promise*** 解决程序是一个抽象操作，它以一个***promise*** 和一个值作为输入，我们将其表示为[[Resolve]] (promise,x)。如果x是一个***thenable*** ，它尝试让promise采用 x 的状态， 并假设x的行为至少在某种程度上类似于***promise*** 。否则，它将会用x解决***promise***。

这种***thenable*** 特性使得 ***promise*** 的实现更具有通用性： 只要其暴露一个遵循Promise/A+ 协议的then 方法即可。这同时也使遵循***Promise/A+***  规范的实现可以与那些不太规范但可用的实现能良好的共存。

要运行[[Resolve]] (promise, x), 需要执行如下步骤：

* 2.3.1. 如果***promise*** 和 ***x*** 引用同一个对象，用一个 ***TypeError***  作为原因来拒绝Promise
* 2.3.2. 如果 ***x*** 是一个promise，采用它的状态： 【3.4】
  * 2.3.2.1. 如果 ***x*** 是<b>等待态</b>， ***promise***  必须保持<b>等待</b>状态， 直到 ***x***  被<b>解决</b>或<b>拒绝</b>。
  * 2.3.2.2. 如果 ***x*** 是<b>解决态</b>，用相同的值<b>解决</b> ***promise***
  * 2.3.2.3. 如果 ***x*** 是<b>拒绝态</b>，用相同的原因<b>拒绝</b> ***promise***
* 2.3.3. 否则，如果x是一个对象或函数
  * 2.3.3.1. 让then成为x.then。【3.5】
  * 2.3.3.2. 如果检索属性x.then导致抛出了一个异常e， 用 e 作为原因拒绝 promise
  * 2.3.3.3. 如果then是一个函数，用x作为this调用它。then方法的参数为两个回调函数，第一个参数叫做resolvePromise，第二个参数叫做rejectPromise：
    * 2.3.3.3.1. 如果resolvePromise用一个值y调用，运行[[Resolve]] (promise, y)。 译者注： 这里再次调用[[Resolve]] (promise, y), 因为y可能还是promise。
    * 2.3.3.3.2. 如果rejectPromise用一个原因r调用，用r拒绝promise。译者注： 这里如果 r 为promise的话， 依旧会直接reject, 拒绝的原因就是promise。 并不会等到promise被解决或拒绝
    * 2.3.3.3.3. 如果resolvePromise 和 rejectPromise 都被调用，或者对同一个参数进行多次调用， 那么第一次调用优先， 以后的调用都会被忽略。 <b>译注：</b>这里主要针对thenable， promise的状态一旦改变就不会再更改
    * 2.3.3.3.4. 如果调用then抛出了一个异常e，
      * 2.3.3.3.4.1. 如果resolvePromise 或rejectPromise 已经被调用， 请忽略它
      * 2.3.3.3.4.2. 否则， 用e作为原因拒绝promise
  * 2.3.3.4. 如果then 不是一个函数， 用x作为值解决promise
* 2.3.4. 如果x不是一个对象或函数，用x作为值解决promise



##  3. 注解

* 3.1.  这里“平台代码”意味着引擎、环境以及promise的实现代码。 在时间中， 这需要确保**onFullfilled** 和 **onRejected** 异步的执行， 并且应该在then方法被调用的那一轮事件循环之后 用新的执行栈执行。 这可以用如**setTimeout**或**setImmediate** 这样的 “宏任务”机制实现， 或者用如 **MutatioinObserver** 或 **process.nextTick** 这样的**“微任务”**机制实现。 由于promise的实现被考虑为“平台代码”， 因此在自身处理程序被调用时可能已经包含了一个任务调度队列。
* 3.2. 严格模式下， 它们中的this将会是undefined； 在费严格模式， this将会是全局对象。
* 3.3. 加入实现满足所有需求， 可以允许promise2 == promise1 。 每一个实现都应该记录是否能够产生promise2 == promise1 以及什么情况下会出现 promise2 === promise1。
* 3.4. 通常， 只有x来自于当前实现，才知道它是一个真正的promise。 这条规则允许那些特例实现采用复合一直要求的Promise的状态。
* 3.5. 这个程序首先存储x.then的引用， 之后测试那个引用，然后再调用那个引用，这样避免了多次访问x.then属性。 此类预防措施对于确保访问者属性的一致性非常重要， 因为访问者属性的值可能在两次检索之间发生变化。
* 3.6. 实现不应该在thenable 链的深度上做任意限制，并且假设超过那个任意限制将会无限递归。只有真正的循环才应该引发一个TypeError；如果遇到一个无限循环的thenable， 永远执行递归是正确的行为。



