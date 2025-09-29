# 纯函数：函数式编程的基石

在函数式编程的世界里，纯函数是最基本也是最重要的概念。理解纯函数是掌握函数式编程的第一步，它不仅是一种编程技术，更是一种思维方式。

## 什么是纯函数？

纯函数是指在相同的输入下，永远产生相同输出的函数，并且没有任何可观察的副作用。这个定义包含两个核心要点：

1. **引用透明性**：相同的输入永远产生相同的输出
2. **无副作用**：函数执行不会改变外部状态或产生可观察的外部影响

### 纯函数的示例

```javascript
// 纯函数
const add = (a, b) => a + b;
const square = (x) => x * x;

// 非纯函数
let counter = 0;
const increment = () => ++counter;
const random = () => Math.random();
```

## 纯函数的特性

### 1. 引用透明性

引用透明性意味着函数调用可以被其返回值替换，而不会改变程序的行为：

```javascript
const result = add(2, 3);
// 等价于
const result = 5;
```

这种特性使得代码更容易理解、测试和优化。

### 2. 无副作用

副作用包括：
- 修改外部变量
- 修改参数
- 进行I/O操作
- 抛出异常
- 打印日志

```javascript
// 有副作用的函数
let globalVar = 0;
const addToGlobal = (x) => {
  globalVar += x;
  return globalVar;
};

// 纯函数
const addToValue = (value, x) => value + x;
```

## 纯函数的优势

### 1. 可测试性

纯函数非常容易测试，因为不需要mock外部环境：

```javascript
// 测试纯函数
test('add function', () => {
  expect(add(2, 3)).toBe(5);
  expect(add(-1, 1)).toBe(0);
  expect(add(0, 0)).toBe(0);
});
```

### 2. 可组合性

纯函数可以像乐高积木一样组合使用：

```javascript
const compose = (f, g) => (x) => f(g(x));
const addThenSquare = compose(square, add);
console.log(addThenSquare(2, 3)); // square(add(2, 3)) = square(5) = 25
```

### 3. 并发安全

由于纯函数不共享状态，可以安全地在多线程环境中执行：

```javascript
// 多个线程可以同时执行这个函数，不会有竞态条件
const process = (data) => data.map(x => x * 2);
```

### 4. 缓存优化

纯函数的结果可以被缓存（记忆化）：

```javascript
const memoize = (fn) => {
  const cache = new Map();
  return (x) => {
    if (cache.has(x)) {
      return cache.get(x);
    }
    const result = fn(x);
    cache.set(x, result);
    return result;
  };
};

const memoizedFib = memoize(fibonacci);
```

## 实践中的纯函数

### 1. 避免全局状态

```javascript
// 不好的做法
let config = { debug: false };
const log = (message) => {
  if (config.debug) {
    console.log(message);
  }
};

// 好的做法
const log = (message, config) => {
  if (config.debug) {
    console.log(message);
  }
};
```

### 2. 不可变数据

```javascript
// 不好的做法
const addToCart = (cart, item) => {
  cart.push(item);
  return cart;
};

// 好的做法
const addToCart = (cart, item) => [...cart, item];
```

### 3. 依赖注入

```javascript
// 不好的做法
const fetchData = () => {
  return fetch('/api/data');
};

// 好的做法
const fetchData = (fetchFn) => {
  return fetchFn('/api/data');
};
```

## 纯函数的限制

纯函数虽然有很多优势，但也有其局限性：

1. **I/O操作**：与外部世界交互本质上是有副作用的
2. **性能考虑**：某些操作可能需要可变状态来保证性能
3. **状态管理**：某些应用场景需要维护状态

## 解决方案：函数式编程模式

### 1. 副作用隔离

```javascript
// 将副作用隔离到特定层
const pureBusinessLogic = (data) => {
  // 纯业务逻辑
  return processedData;
};

const impureLayer = () => {
  const data = fetchData(); // 副作用
  const result = pureBusinessLogic(data);
  saveData(result); // 副作用
};
```

### 2. 函数式数据结构

```javascript
// 使用不可变数据结构
const Immutable = require('immutable');
const list = Immutable.List([1, 2, 3]);
const newList = list.push(4); // 返回新列表，原列表不变
```

## 总结

纯函数是函数式编程的基石，它们提供了：
- 可预测性
- 可测试性
- 可组合性
- 并发安全

虽然在实际应用中完全避免副作用是不现实的，但通过将副作用隔离到特定层，我们可以在大部分代码中享受纯函数带来的好处。掌握纯函数的概念，将帮助你写出更优雅、更易维护的代码。

记住，函数式编程不是要完全消除副作用，而是要控制副作用，让它们在可控的范围内发生。通过这种方式，我们可以结合函数式编程的优点，同时保持与现实世界的交互能力。