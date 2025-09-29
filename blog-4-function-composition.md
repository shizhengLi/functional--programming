# 函数组合：构建优雅的函数式管道

函数组合是函数式编程中最强大的工具之一。它允许我们将简单的函数组合成复杂的操作，就像用乐高积木搭建复杂的结构一样。理解函数组合是掌握函数式编程思维的关键。

## 什么是函数组合？

函数组合是将多个函数组合成一个新函数的过程。数学上，如果我们有两个函数 f 和 g，那么它们的组合 f ∘ g 定义为 (f ∘ g)(x) = f(g(x))。

### 基本概念

```javascript
// 简单的函数组合
const compose = (f, g) => (x) => f(g(x));

// 示例函数
const add1 = (x) => x + 1;
const multiply2 = (x) => x * 2;

// 组合函数
const add1ThenMultiply2 = compose(multiply2, add1);
console.log(add1ThenMultiply2(3)); // multiply2(add1(3)) = multiply2(4) = 8
```

## 组合的类型

### 1. 从右到左组合 (compose)

```javascript
// 标准组合：从右到左执行
const compose = (...fns) => (x) => {
  return fns.reduceRight((value, fn) => fn(value), x);
};

// 使用示例
const add = (a) => (b) => a + b;
const multiply = (a) => (b) => a * b;
const divide = (a) => (b) => b / a;

const calculate = compose(divide(2), multiply(3), add(1));
console.log(calculate(5)); // divide(2)(multiply(3)(add(1)(5))) = divide(2)(multiply(3)(6)) = divide(2)(18) = 9
```

### 2. 从左到右组合 (pipe)

```javascript
// 管道组合：从左到右执行
const pipe = (...fns) => (x) => {
  return fns.reduce((value, fn) => fn(value), x);
};

// 使用示例
const processUser = pipe(
  user => ({ ...user, fullName: `${user.firstName} ${user.lastName}` }),
  user => ({ ...user, email: user.fullName.toLowerCase().replace(' ', '.') + '@example.com' }),
  user => ({ ...user, createdAt: new Date().toISOString() })
);
```

### 3. 异步组合

```javascript
// 异步函数组合
const composeAsync = (...fns) => async (x) => {
  return fns.reduceRight(async (promise, fn) => {
    const value = await promise;
    return fn(value);
  }, Promise.resolve(x));
};

const pipeAsync = (...fns) => async (x) => {
  return fns.reduce(async (promise, fn) => {
    const value = await promise;
    return fn(value);
  }, Promise.resolve(x));
};

// 使用示例
const fetchUser = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

const processUserData = pipeAsync(
  fetchUser,
  user => ({ ...user, processed: true }),
  user => saveToDatabase(user)
);
```

## 组合的实际应用

### 1. 数据转换管道

```javascript
// 数据处理管道
const processData = pipe(
  data => data.filter(item => item.active),
  data => data.map(item => ({
    ...item,
    calculatedValue: item.value * item.multiplier
  })),
  data => data.sort((a, b) => b.calculatedValue - a.calculatedValue),
  data => data.slice(0, 10),
  data => data.map(item => ({
    id: item.id,
    name: item.name,
    value: item.calculatedValue
  }))
);

// 使用
const processedData = processData(rawData);
```

### 2. 中间件模式

```javascript
// Express风格中间件组合
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

// 中间件函数
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
};

const auth = (req, res, next) => {
  if (!req.headers.authorization) {
    res.status(401).send('Unauthorized');
    return;
  }
  next();
};

// 组合中间件
const app = composeMiddleware(logger, auth);
```

### 3. 验证链

```javascript
// 验证函数组合
const createValidator = (...validators) => {
  return (value) => {
    const errors = [];
    for (const validator of validators) {
      const result = validator(value);
      if (result !== true) {
        errors.push(result);
      }
    }
    return errors.length === 0 ? true : errors;
  };
};

// 验证函数
const required = (value) => {
  return value !== undefined && value !== null && value !== '' ? true : 'Required field';
};

const minLength = (min) => (value) => {
  return value.length >= min ? true : `Must be at least ${min} characters`;
};

const isEmail = (value) => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? true : 'Invalid email format';
};

// 组合验证器
const validateEmail = createValidator(required, minLength(5), isEmail);
```

## 高级组合技巧

### 1. 条件组合

```javascript
// 条件组合
const when = (condition, fn) => {
  return (x) => {
    if (condition(x)) {
      return fn(x);
    }
    return x;
  };
};

const unless = (condition, fn) => {
  return when((x) => !condition(x), fn);
};

// 使用示例
const processActiveUsers = pipe(
  when(user => user.age >= 18, user => ({ ...user, isAdult: true })),
  unless(user => user.active, user => ({ ...user, status: 'inactive' }))
);
```

### 2. 分支组合

```javascript
// 分支组合
const branch = (condition, trueFn, falseFn) => {
  return (x) => {
    if (condition(x)) {
      return trueFn(x);
    }
    return falseFn(x);
  };
};

// 使用示例
const processOrder = branch(
  order => order.total > 100,
  order => ({ ...order, discount: 0.1, shipping: 'free' }),
  order => ({ ...order, discount: 0, shipping: 'standard' })
);
```

### 3. 并行组合

```javascript
// 并行组合
const parallel = (...fns) => (x) => {
  return fns.map(fn => fn(x));
};

const parallelAsync = (...fns) => async (x) => {
  return Promise.all(fns.map(fn => fn(x)));
};

// 使用示例
const analyzeUser = parallel(
  user => user.age,
  user => user.name.length,
  user => user.email.split('@')[1]
);

const userStats = analyzeUser({ name: 'Alice', age: 25, email: 'alice@example.com' });
// [25, 5, 'example.com']
```

### 4. 收敛组合

```javascript
// 收敛组合：将多个函数的结果合并
const converge = (finalFn, ...fns) => {
  return (x) => {
    const results = fns.map(fn => fn(x));
    return finalFn(...results);
  };
};

// 使用示例
const calculateStats = converge(
  (count, sum, max) => ({ count, average: sum / count, max }),
  data => data.length,
  data => data.reduce((acc, x) => acc + x, 0),
  data => Math.max(...data)
);

const stats = calculateStats([1, 2, 3, 4, 5]);
// { count: 5, average: 3, max: 5 }
```

## 组合的实用工具

### 1. 函数提升

```javascript
// 将函数提升到特定位置
const lift = (fn, position) => {
  return (...args) => {
    const liftedArgs = [...args];
    liftedArgs[position] = fn(liftedArgs[position]);
    return liftedArgs;
  };
};

// 使用示例
const processThird = lift(x => x * 2, 2);
const result = processThird(1, 2, 3, 4); // [1, 2, 6, 4]
```

### 2. 函数适配器

```javascript
// 函数适配器：调整函数签名
const adapt = (fn, adapter) => {
  return (...args) => {
    const adaptedArgs = adapter(...args);
    return fn(...adaptedArgs);
  };
};

// 使用示例
const oldFunction = (a, b, c) => a + b + c;
const newFunction = adapt(oldFunction, (x, y, z) => [y, z, x]);
```

### 3. 记忆化组合

```javascript
// 为组合函数添加记忆化
const memoizedCompose = (...fns) => {
  const composed = compose(...fns);
  const cache = new Map();

  return (x) => {
    if (cache.has(x)) {
      return cache.get(x);
    }
    const result = composed(x);
    cache.set(x, result);
    return result;
  };
};
```

## 组合的设计原则

### 1. 函数签名一致性

```javascript
// 好的：函数签名一致
const add1 = (x) => x + 1;
const multiply2 = (x) => x * 2;
const square = (x) => x * x;

const pipeline = pipe(add1, multiply2, square);

// 不好的：函数签名不一致
const add = (a, b) => a + b;
const multiply = (a, b) => a * b;
// 难以组合
```

### 2. 单一职责

```javascript
// 好的：每个函数只做一件事
const formatDate = (date) => date.toISOString().split('T')[0];
const addPrefix = (prefix) => (str) => prefix + str;
const toUpperCase = (str) => str.toUpperCase();

const formatDisplayDate = pipe(
  formatDate,
  addPrefix('Date: '),
  toUpperCase
);
```

### 3. 点无风格

```javascript
// 点无风格：避免嵌套函数调用
const result = pipe(
  data => data.filter(x => x.active),
  data => data.map(x => x.value),
  data => data.reduce((acc, x) => acc + x, 0)
)(rawData);

// 而不是
const result = rawData
  .filter(x => x.active)
  .map(x => x.value)
  .reduce((acc, x) => acc + x, 0);
```

## 组合在真实项目中的应用

### 1. React组件

```javascript
// React高阶组件组合
const withAuth = (Component) => {
  return (props) => {
    const { isAuthenticated } = useAuth();
    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }
    return <Component {...props} />;
  };
};

const withData = (Component) => {
  return (props) => {
    const [data, setData] = useState(null);
    useEffect(() => {
      fetchData().then(setData);
    }, []);
    return <Component {...props} data={data} />;
  };
};

const EnhancedComponent = pipe(withAuth, withData)(MyComponent);
```

### 2. 状态管理

```javascript
// Redux中间件组合
const createStore = (reducer, initialState, enhancers) => {
  let state = initialState;
  const listeners = [];

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      const index = listeners.indexOf(listener);
      listeners.splice(index, 1);
    };
  };

  const getState = () => state;

  const store = { dispatch, subscribe, getState };

  return enhancers.reduce((enhancedStore, enhancer) =>
    enhancer(enhancedStore), store);
};
```

### 3. 数据处理

```javascript
// ETL管道
const etlPipeline = pipe(
  // Extract
  data => extractDataFromSource(data),
  // Transform
  data => data.map(item => transformItem(item)),
  data => data.filter(item => isValidItem(item)),
  data => data.map(item => enrichItem(item)),
  // Load
  data => loadDataToDestination(data)
);
```

## 总结

函数组合是函数式编程的核心概念，它提供了：

1. **模块化**：将复杂问题分解为简单函数
2. **可重用性**：小函数可以在多个地方使用
3. **可测试性**：每个函数都可以独立测试
4. **可维护性**：修改功能只需要修改对应的函数
5. **声明式**：代码更接近问题描述而非解决方案

掌握函数组合需要时间和实践，但一旦掌握，你会发现代码变得更加清晰、简洁和优雅。记住，函数组合不仅是一种技术，更是一种思维方式，它让你能够用更抽象的方式思考问题。

通过函数组合，我们可以构建出像数学公式一样精确和优美的代码，这就是函数式编程的魅力所在。