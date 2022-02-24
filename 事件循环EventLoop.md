# 事件循环EventLoop

## 完整事件循环执行顺序

- 从上至下执行所有的同步代码
- 执行过程中将遇到的宏任务与微任务添加至响应的队列
- 同步代码执行完毕后，执行满足条件的微任务回调
- 微任务队列执行完毕后执行所有满足需求的宏任务回调
- 循环事件环操作
- 注意：每执行一个宏任务之后就会立刻检查微任务队列



## Nodejs 事件循环机制

- timers

- pending callbacks

- idle,prepare

- poll

- check

- close callbacks

### 队列说明

- timers: 执行 setTimeout 与 setInterval 回调

- pending callbacks: 执行系统操作的回调， 例如 TCP，UDP 等

- idle, prepare: 只在系统内部进行使用

- poll: 执行与 I/O 相关的回调

- check: 执行 setImmediate 中的回调

- close callbacks: 执行 close 事件的回调

## Nodejs 完整事件环

- 执行同步代码，讲不通的任务添加至相应的队列

- 所有同步代码执行后回去执行满足条件的微任务

- 所有微任务代码执行后会执行 timer 队列中满足的宏任务

- timer 中所有宏任务执行完成后就会依次切换队列

- 注意: 在完成队列切换之前会先清空微任务代码