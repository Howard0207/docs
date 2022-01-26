# SCHEDULER

## API介绍

*  ImmediatePriority
*  UserBlockingPriority 
*  NormalPriority
*  IdlePriority
*  LowPriority
*  unstable_runWithPriority
*  unstable_next
*  unstable_scheduleCallback
*  unstable_cancelCallback
*  unstable_wrapCallback
*  unstable_getCurrentPriorityLevel
*  shouldYieldToHost
*  unstable_getFirstCallbackNode
*  getCurrentTime
*  forceFrameRate

以上是Scheduler导出的所有api了，下面逐步介绍



### 任务优先级

```js
  // 下面导出的5个api代表5种任务的优先级，每个优先级任务对应一个过期时间(ms)
  ImmediatePriority  => IMMEDIATE_PRIORITY_TIMEOUT
  UserBlockingPriority  => USER_BLOCKING_PRIORITY_TIMEOUT
  NormalPriority => NORMAL_PRIORITY_TIMEOUT
  IdlePriority => IDLE_PRIORITY_TIMEOUT
  LowPriority => LOW_PRIORITY_TIMEOUT
  
   // 最大过期时间
   var maxSigned31BitInt = 1073741823;
   // Times out immediately 立即过期
   var IMMEDIATE_PRIORITY_TIMEOUT = -1;
   // Eventually times out 多久后过期
   var USER_BLOCKING_PRIORITY_TIMEOUT = 250;
   var NORMAL_PRIORITY_TIMEOUT = 5000;
   var LOW_PRIORITY_TIMEOUT = 10000;
   // Never times out 永不过期
   var IDLE_PRIORITY_TIMEOUT = maxSigned31BitInt;

```



### unstable_runWithPriority

将当前的priority 赋值为 传入的priorityLevel，以priorityLevel的优先级执行eventHandler。eventHandler任务结束后恢复priority 为前一次的状态。

* react-dom 中存在自己的任务优先级，scheduler也有自己的任务优先级。二者各自独立，但在react-dom中二者是有映射关系的。
* 当执行eventHandler这个任务过程中可能会存在获取当前任务优先级的需求。所以，为什么这里会有对当前任务有优先级赋值，恢复的过程。
* 如果eventHandler任务中没有获取当前任务优先级的需求。那他的用法和直接调用eventHandler没有差别。

```js
function unstable_runWithPriority(priorityLevel, eventHandler) {
    switch (priorityLevel) {
        case ImmediatePriority:
        case UserBlockingPriority:
        case NormalPriority:
        case LowPriority:
        case IdlePriority:
          break;
        default:
          priorityLevel = NormalPriority;
      }
      var previousPriorityLevel = currentPriorityLevel;
      currentPriorityLevel = priorityLevel;
      try {
        return eventHandler();
      } finally {
        currentPriorityLevel = previousPriorityLevel;
      }
}
```



### unstable_next



```js
function unstable_next(eventHandler) {
  var priorityLevel;
  switch (currentPriorityLevel) {
    case ImmediatePriority:
    case UserBlockingPriority:
    case NormalPriority:
      // Shift down to normal priority
      priorityLevel = NormalPriority;
      break;
    default:
      // Anything lower than normal priority should remain at the current level.
      priorityLevel = currentPriorityLevel;
      break;
  }

  var previousPriorityLevel = currentPriorityLevel;
  currentPriorityLevel = priorityLevel;

  try {
    return eventHandler();
  } finally {
    currentPriorityLevel = previousPriorityLevel;
  }
}
```



### unstable_scheduleCallback

这个方法创建任务，并将任务添加到任务队列。

```js
function unstable_scheduleCallback(priorityLevel, callback, options) {}
// 任务队列中的任务如下
var newTask = {
    id: taskIdCounter++,
    callback,
    priorityLevel,
    startTime,
    expirationTime,
    sortIndex: -1,
  };
```



### unstable_cancelCallback

这个方法就是取消已经排队中的任务。

打个比方：俩人来到车站买票，车马上到站了，网上售票停止，此时2人排队分别买票，谁先排到谁买，另一个人停止买票，即另一个人任务取消。

在react中呢，react的任务有同步任务，有异步任务。react和scheduler都保存这个任务，具体谁工作，谁先排到谁先执行，另一个取消任务。

```js
function unstable_cancelCallback(task) {
  task.callback = null;
}
```



### unstable_wrapCallback



### unstable_getCurrentPriorityLevel

实现非常简单，就是将当前任务的优先级返回

```js
function unstable_getCurrentPriorityLevel() {
  return currentPriorityLevel;
}
```



### shouldYieldToHost

判断当前是否要将当前工作线程交还给主线程。

```js
function shouldYieldToHost() {
    return getCurrentTime() >= deadline;
}
```



### unstable_getFirstCallbackNode

获取队头任务

```js
function unstable_getFirstCallbackNode() {
  return peek(taskQueue);
}
```



### getCurrentTime

这里面的功能是获取当前的一个相对时间。

相对页面加载或者相对js执行的一个时间。

实现如下

```js
let getCurrentTime;
const hasPerformanceNow =
  typeof performance === 'object' && typeof performance.now === 'function';

if (hasPerformanceNow) {
  const localPerformance = performance;
  getCurrentTime = () => localPerformance.now();
} else {
  const localDate = Date;
  const initialTime = localDate.now();
  getCurrentTime = () => localDate.now() - initialTime;
}
```



### forceFrameRate

这个api主要是根据帧率来调整scheduler执行每个任务的时间。默认每个任务执行5 ms

```js
function forceFrameRate(fps) {
    if (fps > 0) {
        yieldInterval = Math.floor(1000 / fps);
    } else {
        // reset the framerate
        yieldInterval = 5;
    }
}
```











