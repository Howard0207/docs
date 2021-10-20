# Redux

> 在Redux，只有一家商店，它是唯一的真理来源。存储中的状态是不可变的，这使得我们更容易知道在哪里可以找到数据/状态。在Redux中，虽然有一个巨大的JSON对象来表示存储，但是您可以始终将代码拆分为多个reducer。这样，就可以用多个reducer在逻辑上分离关注点。对于许多开发人员来说，这是一种更直观的方法，因为他们可以始终引用应用程序状态的单个存储区，并且不存在与当前数据状态相关的重复或混淆的可能性。

- Redux只有一个存储——单一来源的真相
- 存储区中的状态是不可变的
- 操作会调用对存储的更改
- Reducers（减速器）更新状态

```js
function createStore(reducer, initialState) {
  let state = initialState;
  const listeners = [];
  function subscribe(listener) {
    listeners.push(listener);
    return function unSubscribe() {
      const index = listeners.indexOf(listener);
      listeners.splice(index, 1)
    }
  }

  function dispatch(action) {
    state = reducer(state, action);
    for (const listener of listeners) {
      listener();
    }
    return action;
  }

  function getState() {
    return state;
  }

  dispatch({ type: Symbol() })

  return {
    subscribe,
    dispatch,
    getState,
  };
}

const combineReducers = (reducers) => {
  const reducersKeys = Object.keys(reducers);
  return function combination(state = {}, action) {
    const nextState = {};
    let key, reducer, previousStateForKey, nextStateForKey;
    for (let i = 0; i < reducersKeys.length; i++) {
      key = reducersKeys[i];
      reducer = reducers[key];
      previousStateForKey = state[key];
      nextStateForKey = reducer(previousStateForKey, action);
      nextState[key] = nextStateForKey;
    }
    return nextState;
  };
};

```

Redux使用普通JavaScript对象作为数据结构来存储状态。使用Redux时，必须手动跟踪更新。在需要维护大量状态的应用程序中，这可能更困难。