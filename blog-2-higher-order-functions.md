# 高阶函数：函数式编程的瑞士军刀

高阶函数是函数式编程中最强大的概念之一。它们让函数从简单的值计算器变成了可以操纵其他函数的元工具，极大地扩展了编程的可能性。

## 什么是高阶函数？

高阶函数是指满足以下至少一个条件的函数：
1. **接受函数作为参数**
2. **返回一个函数**

这个看似简单的定义，却蕴含着函数式编程的精髓：函数作为一等公民。

### 基本示例

```javascript
// 接受函数作为参数
const apply = (fn, x) => fn(x);

// 返回函数
const createMultiplier = (factor) => (x) => x * factor;

// 同时满足两个条件
const compose = (f, g) => (x) => f(g(x));
```

## 常见的高阶函数模式

### 1. 函数组合

函数组合是将多个函数组合成一个新函数的技术：

```javascript
// 简单组合
const compose = (f, g) => (x) => f(g(x));
const pipe = (...fns) => (x) => fns.reduce((v, f) => f(v), x);

// 使用示例
const add1 = (x) => x + 1;
const multiply2 = (x) => x * 2;
const square = (x) => x * x;

const transform = pipe(add1, multiply2, square);
console.log(transform(3)); // square(multiply2(add1(3))) = square(multiply2(4)) = square(8) = 64
```

### 2. 函数柯里化

柯里化是将多参数函数转换为一系列单参数函数的技术：

```javascript
// 手动柯里化
const add = (a) => (b) => a + b;

// 自动柯里化
const curry = (fn) => {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return (...moreArgs) => curried(...args, ...moreArgs);
  };
};

const curriedAdd = curry((a, b, c) => a + b + c);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
```

### 3. 函数记忆化

记忆化是缓存函数结果以提高性能的技术：

```javascript
const memoize = (fn) => {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
};

const memoizedFib = memoize((n) => {
  if (n <= 1) return n;
  return memoizedFib(n - 1) + memoizedFib(n - 2);
});
```

## 实用的高阶函数

### 1. Array.prototype.* 方法

JavaScript数组方法是最常用的高阶函数：

```javascript
// map：转换数组的每个元素
const doubled = [1, 2, 3].map(x => x * 2); // [2, 4, 6]

// filter：过滤数组元素
const evens = [1, 2, 3, 4].filter(x => x % 2 === 0); // [2, 4]

// reduce：聚合数组元素
const sum = [1, 2, 3, 4].reduce((acc, x) => acc + x, 0); // 10

// forEach：执行副作用
[1, 2, 3].forEach(x => console.log(x));
```

### 2. 自定义高阶函数

```javascript
// 重复执行函数
const repeat = (times, fn) => {
  return (x) => {
    let result = x;
    for (let i = 0; i < times; i++) {
      result = fn(result);
    }
    return result;
  };
};

const doubleTwice = repeat(2, x => x * 2);
console.log(doubleTwice(5)); // 20

// 条件执行
const when = (condition, fn) => {
  return (x) => {
    if (condition(x)) {
      return fn(x);
    }
    return x;
  };
};

const whenEven = when(x => x % 2 === 0, x => x / 2);
console.log(whenEven(4)); // 2
console.log(whenEven(5)); // 5
```

## 高阶函数在实际应用中

### 1. 事件处理

```javascript
// 创建事件处理器工厂
const createEventHandler = (eventName, dataProcessor) => {
  return (event) => {
    const processedData = dataProcessor(event.data);
    console.log(`${eventName}:`, processedData);
  };
};

const userClickHandler = createEventHandler('click', (data) => ({
  ...data,
  timestamp: Date.now()
}));
```

### 2. 中间件模式

```javascript
// Express风格中间件
const middleware = (req, res, next) => {
  console.log('Request:', req.url);
  next();
};

// 组合中间件
const composeMiddleware = (...middlewares) => {
  return (req, res) => {
    let index = 0;
    const next = () => {
      if (index < middlewares.length) {
        middlewares[index++](req, res, next);
      }
    };
    next();
  };
};
```

### 3. 数据验证

```javascript
// 创建验证器
const createValidator = (rules) => {
  return (data) => {
    const errors = [];
    for (const [field, rule] of Object.entries(rules)) {
      if (!rule.validator(data[field])) {
        errors.push({
          field,
          message: rule.message
        });
      }
    }
    return errors;
  };
};

const userValidator = createValidator({
  name: {
    validator: name => name && name.length > 2,
    message: 'Name must be at least 3 characters'
  },
  age: {
    validator: age => age >= 18,
    message: 'Age must be at least 18'
  }
});
```

## 高阶函数的进阶技巧

### 1. 函数装饰器

```javascript
// 日志装饰器
const withLogging = (fn) => {
  return (...args) => {
    console.log(`Calling ${fn.name} with args:`, args);
    const result = fn(...args);
    console.log(`Result:`, result);
    return result;
  };
};

// 性能监控装饰器
const withPerformance = (fn) => {
  return (...args) => {
    const start = performance.now();
    const result = fn(...args);
    const end = performance.now();
    console.log(`${fn.name} took ${end - start}ms`);
    return result;
  };
};

// 组合装饰器
const enhance = (fn, ...decorators) => {
  return decorators.reduce((enhancedFn, decorator) =>
    decorator(enhancedFn), fn);
};

const enhancedFunction = enhance(
  myFunction,
  withLogging,
  withPerformance
);
```

### 2. 函数式链式调用

```javascript
// 链式操作
const pipe = (...fns) => (initialValue) => {
  return fns.reduce((value, fn) => fn(value), initialValue);
};

const processData = pipe(
  data => data.filter(item => item.active),
  data => data.map(item => ({ ...item, processed: true })),
  data => data.sort((a, b) => a.priority - b.priority),
  data => data.slice(0, 10)
);
```

### 3. 异步高阶函数

```javascript
// 异步函数组合
const composeAsync = (...fns) => (initialValue) => {
  return fns.reduce(async (promise, fn) => {
    const value = await promise;
    return fn(value);
  }, Promise.resolve(initialValue));
};

// 并行处理
const parallel = (...fns) => (input) => {
  return Promise.all(fns.map(fn => fn(input)));
};

// 重试机制
const withRetry = (fn, maxRetries = 3) => {
  return async (...args) => {
    for (let i = 0; i < maxRetries; i++) {
      try {
        return await fn(...args);
      } catch (error) {
        if (i === maxRetries - 1) throw error;
        await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
      }
    }
  };
};
```

## 高阶函数的设计原则

### 1. 单一职责

每个高阶函数应该只做一件事：

```javascript
// 不好的做法
const processAndLog = (data, logger) => {
  const processed = data.map(x => x * 2);
  logger(processed);
  return processed;
};

// 好的做法
const process = (fn) => (data) => data.map(fn);
const log = (logger) => (data) => {
  logger(data);
  return data;
};
```

### 2. 可组合性

设计时要考虑函数的组合：

```javascript
// 设计可组合的函数
const add = (x) => (y) => x + y;
const multiply = (x) => (y) => x * y;

// 可以轻松组合
const addThenMultiply = compose(multiply(2), add(1));
```

### 3. 类型安全

考虑函数的类型签名：

```javascript
// 类型安全的高阶函数
const map = (fn) => (array) => {
  if (!Array.isArray(array)) {
    throw new TypeError('Expected an array');
  }
  return array.map(fn);
};
```

## 总结

高阶函数是函数式编程的核心概念，它们提供了：

1. **抽象能力**：可以将通用模式抽象成可重用的函数
2. **组合能力**：可以像乐高积木一样组合不同的函数
3. **扩展能力**：可以为现有函数添加新功能而不修改原函数
4. **表达力**：可以写出更简洁、更声明式的代码

掌握高阶函数是成为一名优秀的函数式程序员的关键。通过高阶函数，我们可以：
- 减少代码重复
- 提高代码可读性
- 增强代码的可维护性
- 实现复杂的逻辑

记住，高阶函数不仅是技术，更是一种思维方式。当你开始用函数式的眼光看世界时，你会发现很多问题都可以用高阶函数优雅地解决。