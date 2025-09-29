# 函数式设计模式：优雅解决复杂问题

函数式设计模式是函数式编程思想的结晶，它们为我们提供了一套经过验证的解决方案来处理常见的编程问题。这些模式不同于传统的面向对象设计模式，它们更注重函数的组合、数据的不可变性和声明的表达方式。

## 函数式设计模式的核心原则

### 1. 函数作为一等公民

在函数式编程中，函数可以像其他值一样被传递、存储和操作：

```javascript
// 函数作为参数
const apply = (fn, x) => fn(x);

// 函数作为返回值
const createMultiplier = (factor) => (x) => x * factor;

// 函数作为数据结构的一部分
const operations = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b
};
```

### 2. 纯函数和引用透明性

函数式设计模式依赖于纯函数的概念：

```javascript
// 纯函数：相同的输入总是产生相同的输出
const add = (a, b) => a + b;

// 引用透明性：函数调用可以被其返回值替换
const result = add(2, 3); // 等价于 const result = 5;
```

## 核心函数式设计模式

### 1. 函数组合模式

函数组合模式是将多个函数组合成一个新函数的模式。

```javascript
// 基本组合
const compose = (...fns) => (x) => fns.reduceRight((v, f) => f(v), x);
const pipe = (...fns) => (x) => fns.reduce((v, f) => f(v), x);

// 使用示例
const add1 = (x) => x + 1;
const multiply2 = (x) => x * 2;
const toString = (x) => x.toString();

const processNumber = pipe(add1, multiply2, toString);
console.log(processNumber(3)); // "8"

// 实际应用：数据处理管道
const processUserData = pipe(
  user => ({ ...user, fullName: `${user.firstName} ${user.lastName}` }),
  user => ({ ...user, email: user.fullName.toLowerCase().replace(' ', '.') + '@example.com' }),
  user => ({ ...user, processedAt: new Date().toISOString() })
);
```

### 2. 高阶函数模式

高阶函数模式使用函数来创建或返回其他函数。

```javascript
// 函数工厂
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

// 使用示例
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

// 记忆化模式
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

### 3. 柯里化模式

柯里化是将多参数函数转换为一系列单参数函数的模式。

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

// 使用示例
const curriedAdd = curry((a, b, c) => a + b + c);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6

// 实际应用：配置函数
const createLogger = curry((level, prefix, message) => {
  console.log(`[${level.toUpperCase()}] ${prefix}: ${message}`);
});

const infoLogger = createLogger('info', 'App');
const errorLogger = createLogger('error', 'App');

infoLogger('Application started');
errorLogger('Something went wrong');
```

### 4. 函子模式

函子模式提供了将函数应用到包装值的能力。

```javascript
// Maybe函子
class Maybe {
  constructor(value) {
    this.value = value;
  }

  static of(value) {
    return new Maybe(value);
  }

  map(fn) {
    return this.value === null || this.value === undefined
      ? Maybe.of(null)
      : Maybe.of(fn(this.value));
  }

  getOrElse(defaultValue) {
    return this.value === null || this.value === undefined
      ? defaultValue
      : this.value;
  }
}

// 使用示例
const getUser = (id) => {
  const users = { 1: { name: 'Alice' }, 2: { name: 'Bob' } };
  return Maybe.of(users[id]);
};

const getUserName = (user) => {
  return Maybe.of(user.name);
};

const getUserNameById = (id) => {
  return getUser(id)
    .map(user => user.name)
    .getOrElse('Unknown');
};

console.log(getUserNameById(1)); // "Alice"
console.log(getUserNameById(3)); // "Unknown"
```

### 5. 单子模式

单子模式扩展了函子，提供了链式操作的能力。

```javascript
// Maybe单子
class Maybe {
  constructor(value) {
    this.value = value;
  }

  static of(value) {
    return new Maybe(value);
  }

  map(fn) {
    return this.value === null || this.value === undefined
      ? Maybe.of(null)
      : Maybe.of(fn(this.value));
  }

  flatMap(fn) {
    return this.value === null || this.value === undefined
      ? Maybe.of(null)
      : fn(this.value);
  }
}

// Either单子
class Either {
  static left(value) {
    return new Left(value);
  }

  static right(value) {
    return new Right(value);
  }
}

class Left extends Either {
  map() {
    return this;
  }

  flatMap() {
    return this;
  }

  fold(leftFn, rightFn) {
    return leftFn(this.value);
  }
}

class Right extends Either {
  map(fn) {
    return Either.right(fn(this.value));
  }

  flatMap(fn) {
    return fn(this.value);
  }

  fold(leftFn, rightFn) {
    return rightFn(this.value);
  }
}

// 使用示例
const parseJson = (str) => {
  try {
    return Either.right(JSON.parse(str));
  } catch (error) {
    return Either.left(error.message);
  }
};

const validateUser = (user) => {
  if (!user.name) {
    return Either.left('Missing name');
  }
  return Either.right(user);
};

const processUserData = (jsonStr) => {
  return parseJson(jsonStr)
    .flatMap(validateUser)
    .fold(
      error => ({ success: false, error }),
      user => ({ success: true, user })
    );
};
```

### 6. 惰性求值模式

惰性求值模式延迟计算直到真正需要结果时。

```javascript
// 惰性序列
class LazySequence {
  constructor(generator) {
    this.generator = generator;
    this.cache = [];
    this.exhausted = false;
  }

  take(n) {
    const result = [];
    for (let i = 0; i < n; i++) {
      const next = this.next();
      if (next === undefined) break;
      result.push(next);
    }
    return result;
  }

  next() {
    if (this.cache.length > 0) {
      return this.cache.shift();
    }

    if (this.exhausted) {
      return undefined;
    }

    const result = this.generator.next();
    if (result.done) {
      this.exhausted = true;
      return undefined;
    }

    return result.value;
  }

  map(fn) {
    return new LazySequence(function* () {
      let item;
      while ((item = this.next()) !== undefined) {
        yield fn(item);
      }
    }.bind(this));
  }

  filter(predicate) {
    return new LazySequence(function* () {
      let item;
      while ((item = this.next()) !== undefined) {
        if (predicate(item)) {
          yield item;
        }
      }
    }.bind(this));
  }
}

// 使用示例
const naturalNumbers = new LazySequence(function* () {
  let i = 0;
  while (true) {
    yield i++;
  }
});

const evenNumbers = naturalNumbers.filter(x => x % 2 === 0);
const squares = evenNumbers.map(x => x * x);

console.log(squares.take(5)); // [0, 4, 16, 36, 64]
```

### 7. 状态模式

函数式状态模式通过不可变状态和状态转换来管理应用状态。

```javascript
// 状态机模式
class StateMachine {
  constructor(initialState, transitions) {
    this.state = initialState;
    this.transitions = transitions;
  }

  transition(action, payload) {
    const currentState = this.state;
    const possibleTransitions = this.transitions[currentState];

    if (!possibleTransitions || !possibleTransitions[action]) {
      throw new Error(`Invalid transition: ${action} from state ${currentState}`);
    }

    const nextState = possibleTransitions[action];
    const newState = typeof nextState === 'function'
      ? nextState(currentState, payload)
      : nextState;

    this.state = newState;
    return newState;
  }

  getCurrentState() {
    return this.state;
  }
}

// 使用示例
const orderStateMachine = new StateMachine('pending', {
  pending: {
    confirm: (state, payload) => ({
      status: 'confirmed',
      items: payload.items,
      total: payload.total
    }),
    cancel: 'cancelled'
  },
  confirmed: {
    ship: (state, payload) => ({
      ...state,
      status: 'shipped',
      tracking: payload.tracking
    }),
    cancel: 'cancelled'
  },
  shipped: {
    deliver: (state, payload) => ({
      ...state,
      status: 'delivered',
      deliveryDate: payload.date
    })
  },
  cancelled: {},
  delivered: {}
});

// 状态转换示例
orderStateMachine.transition('confirm', {
  items: ['item1', 'item2'],
  total: 100
});

orderStateMachine.transition('ship', {
  tracking: 'TRACK123'
});

orderStateMachine.transition('deliver', {
  date: new Date().toISOString()
});
```

### 8. 中间件模式

中间件模式通过函数组合来处理请求/响应周期。

```javascript
// 中间件模式
const createMiddleware = (...middlewares) => {
  return (initialContext) => {
    let index = 0;

    const dispatch = (context) => {
      if (index >= middlewares.length) {
        return context;
      }

      const middleware = middlewares[index++];
      return middleware(context, dispatch);
    };

    return dispatch(initialContext);
  };
};

// 使用示例
const loggerMiddleware = (context, next) => {
  console.log('Request:', context.request);
  const result = next(context);
  console.log('Response:', context.response);
  return result;
};

const authMiddleware = (context, next) => {
  if (!context.request.headers.authorization) {
    context.response = { status: 401, body: 'Unauthorized' };
    return context;
  }
  return next(context);
};

const dataProcessor = (context, next) => {
  context.response = {
    status: 200,
    body: `Processed: ${context.request.body}`
  };
  return context;
};

// 组合中间件
const app = createMiddleware(
  loggerMiddleware,
  authMiddleware,
  dataProcessor
);

// 使用
const result = app({
  request: {
    headers: { authorization: 'Bearer token' },
    body: 'Hello, World!'
  },
  response: {}
});
```

## 实际应用场景

### 1. 表单处理

```javascript
// 表单处理的函数式模式
const createFormHandler = (validators, transformers) => {
  return (formData) => {
    // 验证
    const errors = [];
    for (const [field, validator] of Object.entries(validators)) {
      if (!validator(formData[field])) {
        errors.push(`${field} is invalid`);
      }
    }

    if (errors.length > 0) {
      return { success: false, errors };
    }

    // 转换
    const transformed = Object.entries(transformers).reduce(
      (acc, [field, transformer]) => ({
        ...acc,
        [field]: transformer(formData[field])
      }),
      {}
    );

    return { success: true, data: transformed };
  };
};

// 使用示例
const userFormHandler = createFormHandler(
  {
    name: value => value && value.length > 2,
    email: value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    age: value => value >= 18
  },
  {
    name: value => value.trim(),
    email: value => value.toLowerCase(),
    age: value => parseInt(value)
  }
);

const result = userFormHandler({
  name: '  Alice  ',
  email: 'ALICE@EXAMPLE.COM',
  age: '25'
});
```

### 2. 数据处理管道

```javascript
// 数据处理管道
const createDataPipeline = (...processors) => {
  return (data) => {
    return processors.reduce((currentData, processor) => {
      return processor(currentData);
    }, data);
  };
};

// 处理器
const filterActive = (data) => data.filter(item => item.active);
const enrichData = (data) => data.map(item => ({
  ...item,
  processed: true,
  timestamp: Date.now()
}));
const sortByPriority = (data) => data.sort((a, b) => b.priority - a.priority);
const limitResults = (limit) => (data) => data.slice(0, limit);

// 使用示例
const processUsers = createDataPipeline(
  filterActive,
  enrichData,
  sortByPriority,
  limitResults(10)
);

const users = [
  { id: 1, name: 'Alice', active: true, priority: 3 },
  { id: 2, name: 'Bob', active: false, priority: 2 },
  { id: 3, name: 'Charlie', active: true, priority: 1 }
];

const processedUsers = processUsers(users);
```

### 3. API客户端

```javascript
// API客户端的函数式模式
const createApiClient = (baseUrl) => {
  const request = (endpoint, options = {}) => {
    return fetch(`${baseUrl}${endpoint}`, options)
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.json();
      });
  };

  return {
    get: (endpoint) => request(endpoint, { method: 'GET' }),
    post: (endpoint, data) => request(endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    }),
    put: (endpoint, data) => request(endpoint, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    }),
    delete: (endpoint) => request(endpoint, { method: 'DELETE' })
  };
};

// 使用示例
const api = createApiClient('https://api.example.com');

const fetchUser = (id) => api.get(`/users/${id}`);
const updateUser = (id, data) => api.put(`/users/${id}`, data);
const deleteUser = (id) => api.delete(`/users/${id}`);

// 组合API调用
const getUserProfile = (id) => {
  return fetchUser(id)
    .then(user => Promise.all([
      user,
      api.get(`/users/${id}/posts`),
      api.get(`/users/${id}/comments`)
    ]))
    .then(([user, posts, comments]) => ({
      ...user,
      posts,
      comments
    }));
};
```

## 高级模式

### 1. 依赖注入模式

```javascript
// 依赖注入的函数式模式
const createService = (dependencies) => {
  return (config) => {
    const service = {
      ...dependencies,
      config
    };

    return {
      execute: (operation) => operation(service)
    };
  };
};

// 使用示例
const databaseService = {
  query: (sql) => Promise.resolve([]),
  execute: (sql) => Promise.resolve({ affectedRows: 1 })
};

const loggerService = {
  info: (message) => console.log(`[INFO] ${message}`),
  error: (message) => console.error(`[ERROR] ${message}`)
};

const userService = createService({
  database: databaseService,
  logger: loggerService
});

const createUser = (service) => async (userData) => {
  service.logger.info(`Creating user: ${userData.name}`);

  try {
    const result = await service.database.execute(
      'INSERT INTO users (name, email) VALUES (?, ?)',
      [userData.name, userData.email]
    );

    service.logger.info(`User created successfully`);
    return result;
  } catch (error) {
    service.logger.error(`Failed to create user: ${error.message}`);
    throw error;
  }
};

// 使用
const service = userService({ environment: 'production' });
const createUserOperation = createUser(service);
createUserOperation({ name: 'Alice', email: 'alice@example.com' });
```

### 2. 策略模式

```javascript
// 策略模式的函数式实现
const createStrategyManager = (strategies) => {
  return (context) => {
    const strategyKey = context.type || context.action;
    const strategy = strategies[strategyKey];

    if (!strategy) {
      throw new Error(`No strategy found for: ${strategyKey}`);
    }

    return strategy(context);
  };
};

// 使用示例
const paymentStrategies = {
  credit_card: (context) => {
    console.log(`Processing credit card payment: ${context.amount}`);
    return { success: true, method: 'credit_card' };
  },
  paypal: (context) => {
    console.log(`Processing PayPal payment: ${context.amount}`);
    return { success: true, method: 'paypal' };
  },
  bank_transfer: (context) => {
    console.log(`Processing bank transfer: ${context.amount}`);
    return { success: true, method: 'bank_transfer' };
  }
};

const processPayment = createStrategyManager(paymentStrategies);

processPayment({ type: 'credit_card', amount: 100 });
processPayment({ type: 'paypal', amount: 50 });
```

### 3. 观察者模式

```javascript
// 观察者模式的函数式实现
class Observable {
  constructor() {
    this.observers = new Set();
  }

  subscribe(observer) {
    this.observers.add(observer);
    return {
      unsubscribe: () => this.observers.delete(observer)
    };
  }

  notify(data) {
    this.observers.forEach(observer => {
      try {
        observer(data);
      } catch (error) {
        console.error('Observer error:', error);
      }
    });
  }
}

// 使用示例
const eventBus = new Observable();

const userCreatedObserver = (user) => {
  console.log(`User created: ${user.name}`);
  sendWelcomeEmail(user.email);
};

const userUpdatedObserver = (user) => {
  console.log(`User updated: ${user.name}`);
  updateSearchIndex(user);
};

eventBus.subscribe(userCreatedObserver);
eventBus.subscribe(userUpdatedObserver);

// 发布事件
eventBus.notify({ name: 'Alice', email: 'alice@example.com' });
```

## 总结

函数式设计模式为我们提供了一套强大的工具来解决复杂问题：

1. **组合优于继承**：通过函数组合而不是类继承来构建复杂的逻辑
2. **声明式编程**：专注于"做什么"而不是"怎么做"
3. **不可变性**：使用不可变数据来避免副作用和状态管理问题
4. **高阶抽象**：使用高阶函数来创建抽象和重用代码
5. **类型安全**：通过函数签名和类型约束来确保代码的正确性

这些模式在以下场景中特别有用：
- 数据处理和转换
- 状态管理
- 异步操作
- 配置和依赖注入
- 事件处理和响应式编程

掌握这些模式需要时间和实践，但一旦掌握，你会发现它们能够帮助你写出更加优雅、可维护和可测试的代码。函数式设计模式不仅是技术，更是一种思维方式，它们让我们能够以更加抽象和数学化的方式来思考编程问题。

记住，模式是工具，不是规则。根据具体的需求和上下文来选择合适的模式，才能真正发挥函数式编程的力量。