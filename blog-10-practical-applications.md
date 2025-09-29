# 函数式编程的实战应用：从理论到实践的桥梁

函数式编程不仅仅是学术概念，它在现代软件开发中有着广泛而深入的应用。本文将探讨函数式编程在真实项目中的应用场景，展示如何将函数式原则转化为实际的解决方案。

## 函数式编程的应用领域

### 1. 前端开发

#### React中的函数式编程

```javascript
// 函数式React组件
import React, { useState, useEffect, useCallback, useMemo } from 'react';

// 纯函数组件
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // 纯函数：数据转换
  const formatUserData = useCallback((userData) => {
    return {
      ...userData,
      fullName: `${userData.firstName} ${userData.lastName}`,
      displayName: userData.nickname || userData.fullName,
      joinDate: new Date(userData.createdAt).toLocaleDateString(),
      isActive: userData.status === 'active'
    };
  }, []);

  // 纯函数：计算派生状态
  const calculateUserStats = useCallback((userData) => {
    return {
      postCount: userData.posts?.length || 0,
      followerCount: userData.followers?.length || 0,
      engagementRate: userData.posts?.length > 0
        ? (userData.likes || 0) / userData.posts.length
        : 0
    };
  }, []);

  // 使用useMemo避免重复计算
  const formattedUser = useMemo(() => {
    return user ? formatUserData(user) : null;
  }, [user, formatUserData]);

  const userStats = useMemo(() => {
    return formattedUser ? calculateUserStats(formattedUser) : null;
  }, [formattedUser, calculateUserStats]);

  // 函数式数据获取
  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        console.error('Failed to fetch user:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (!formattedUser) return <div>User not found</div>;

  return (
    <div className="user-profile">
      <div className="user-header">
        <h1>{formattedUser.displayName}</h1>
        <p className="join-date">Joined: {formattedUser.joinDate}</p>
        <span className={`status ${formattedUser.isActive ? 'active' : 'inactive'}`}>
          {formattedUser.isActive ? 'Active' : 'Inactive'}
        </span>
      </div>

      <div className="user-stats">
        <div className="stat">
          <strong>{userStats?.postCount || 0}</strong>
          <span>Posts</span>
        </div>
        <div className="stat">
          <strong>{userStats?.followerCount || 0}</strong>
          <span>Followers</span>
        </div>
        <div className="stat">
          <strong>{userStats?.engagementRate.toFixed(1)}%</strong>
          <span>Engagement</span>
        </div>
      </div>
    </div>
  );
};

// 自定义Hook：函数式状态管理
const useFunctionalState = (initialState, reducers) => {
  const [state, setState] = useState(initialState);

  const dispatch = useCallback((action, payload) => {
    if (reducers[action]) {
      setState(prevState => reducers[action](prevState, payload));
    }
  }, [reducers]);

  return [state, dispatch];
};

// 使用自定义Hook
const useTodoState = () => {
  const reducers = {
    add: (state, todo) => ({
      ...state,
      todos: [...state.todos, { ...todo, id: Date.now() }]
    }),
    toggle: (state, id) => ({
      ...state,
      todos: state.todos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    }),
    remove: (state, id) => ({
      ...state,
      todos: state.todos.filter(todo => todo.id !== id)
    }),
    filter: (state, filter) => ({
      ...state,
      filter
    })
  };

  return useFunctionalState({
    todos: [],
    filter: 'all'
  }, reducers);
};

// 函数式高阶组件
const withAuth = (Component) => {
  return (props) => {
    const { isAuthenticated } = useAuth();

    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }

    return <Component {...props} />;
  };
};

const withData = (Component, dataFetcher) => {
  return (props) => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      const fetchData = async () => {
        try {
          setLoading(true);
          const result = await dataFetcher(props);
          setData(result);
        } catch (error) {
          console.error('Failed to fetch data:', error);
        } finally {
          setLoading(false);
        }
      };

      fetchData();
    }, [props]);

    if (loading) return <div>Loading...</div>;
    if (!data) return <div>Error loading data</div>;

    return <Component {...props} data={data} />;
  };
};

// 组合高阶组件
const ProtectedUserDashboard = compose(
  withAuth,
  withData((props) => fetch(`/api/users/${props.userId}/dashboard`))
)(UserDashboard);
```

#### 状态管理：Redux Toolkit

```javascript
// 函数式Redux Toolkit
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 异步操作：函数式数据获取
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (filters = {}, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(filters)
      });

      if (!response.ok) {
        throw new Error('Failed to fetch users');
      }

      return response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// 函数式Reducer
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    users: [],
    loading: false,
    error: null,
    filters: {},
    cache: new Map()
  },
  reducers: {
    // 纯函数：更新过滤器
    setFilters: (state, action) => {
      state.filters = { ...state.filters, ...action.payload };
    },
    // 纯函数：清除缓存
    clearCache: (state) => {
      state.cache.clear();
    },
    // 纯函数：更新单个用户
    updateUser: (state, action) => {
      const { id, updates } = action.payload;
      state.users = state.users.map(user =>
        user.id === id ? { ...user, ...updates } : user
      );
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.users = action.payload;

        // 缓存结果
        const cacheKey = JSON.stringify(state.filters);
        state.cache.set(cacheKey, action.payload);
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

// 函数式选择器
import { createSelector } from '@reduxjs/toolkit';

const selectUsersState = (state) => state.users;
const selectFilters = (state) => state.users.filters;

// 记忆化选择器
const selectFilteredUsers = createSelector(
  [selectUsersState, selectFilters],
  (usersState, filters) => {
    return usersState.users.filter(user => {
      const matchesSearch = !filters.search ||
        user.name.toLowerCase().includes(filters.search.toLowerCase());
      const matchesStatus = !filters.status || user.status === filters.status;
      const matchesRole = !filters.role || user.role === filters.role;

      return matchesSearch && matchesStatus && matchesRole;
    });
  }
);

const selectUserStats = createSelector(
  [selectFilteredUsers],
  (users) => {
    return {
      total: users.length,
      active: users.filter(u => u.status === 'active').length,
      inactive: users.filter(u => u.status === 'inactive').length,
      byRole: users.reduce((acc, user) => {
        acc[user.role] = (acc[user.role] || 0) + 1;
        return acc;
      }, {})
    };
  }
);

// 函数式Hook
export const useUsers = () => {
  const dispatch = useDispatch();
  const users = useSelector(selectFilteredUsers);
  const stats = useSelector(selectUserStats);
  const { loading, error, filters } = useSelector(selectUsersState);

  const fetchUsersData = useCallback((newFilters = {}) => {
    dispatch(fetchUsers({ ...filters, ...newFilters }));
  }, [dispatch, filters]);

  const updateUserFilters = useCallback((newFilters) => {
    dispatch(usersSlice.actions.setFilters(newFilters));
  }, [dispatch]);

  const updateUser = useCallback((id, updates) => {
    dispatch(usersSlice.actions.updateUser({ id, updates }));
  }, [dispatch]);

  return {
    users,
    stats,
    loading,
    error,
    filters,
    fetchUsersData,
    updateUserFilters,
    updateUser
  };
};
```

### 2. 后端开发

#### Node.js中的函数式编程

```javascript
// 函数式Express中间件
const express = require('express');
const { pipe, curry, compose } = require('lodash/fp');

// 函数式中间件工厂
const createMiddleware = (config) => {
  return (req, res, next) => {
    // 中间件逻辑
    req.context = { ...req.context, ...config };
    next();
  };
};

// 函数式验证器
const createValidator = (schema) => {
  return (req, res, next) => {
    const { error } = schema.validate(req.body);
    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details
      });
    }
    next();
  };
};

// 函数式认证中间件
const createAuthMiddleware = (options = {}) => {
  const { role, strict = true } = options;

  return async (req, res, next) => {
    try {
      const token = req.headers.authorization?.replace('Bearer ', '');

      if (!token) {
        return res.status(401).json({ error: 'No token provided' });
      }

      const decoded = jwt.verify(token, process.env.JWT_SECRET);

      if (role && decoded.role !== role) {
        return res.status(403).json({ error: 'Insufficient permissions' });
      }

      req.user = decoded;
      next();
    } catch (error) {
      if (strict) {
        return res.status(401).json({ error: 'Invalid token' });
      }
      req.user = { role: 'guest' };
      next();
    }
  };
};

// 函数式路由处理器
const createHandler = (handler) => {
  return async (req, res) => {
    try {
      const result = await handler(req);
      res.json(result);
    } catch (error) {
      console.error('Handler error:', error);
      res.status(500).json({ error: 'Internal server error' });
    }
  };
};

// 函数式错误处理
const errorHandler = (error, req, res, next) => {
  console.error('Error:', error);

  const errorResponse = {
    error: error.message || 'Internal server error',
    timestamp: new Date().toISOString(),
    path: req.path
  };

  res.status(error.status || 500).json(errorResponse);
};

// 函数式日志中间件
const createLogger = (options = {}) => {
  const { level = 'info', format = 'json' } = options;

  return (req, res, next) => {
    const start = Date.now();

    res.on('finish', () => {
      const duration = Date.now() - start;
      const logData = {
        method: req.method,
        url: req.url,
        status: res.statusCode,
        duration,
        userAgent: req.get('User-Agent'),
        ip: req.ip,
        timestamp: new Date().toISOString()
      };

      if (format === 'json') {
        console.log(JSON.stringify(logData));
      } else {
        console.log(`${req.method} ${req.url} - ${res.statusCode} - ${duration}ms`);
      }
    });

    next();
  };
};

// 函数式速率限制
const createRateLimiter = (options = {}) => {
  const { windowMs = 60000, max = 100, keyGenerator = (req) => req.ip } = options;
  const requests = new Map();

  return (req, res, next) => {
    const key = keyGenerator(req);
    const now = Date.now();
    const windowStart = now - windowMs;

    // 清理过期请求记录
    if (!requests.has(key)) {
      requests.set(key, []);
    }

    const userRequests = requests.get(key).filter(time => time > windowStart);
    requests.set(key, userRequests);

    if (userRequests.length >= max) {
      return res.status(429).json({ error: 'Too many requests' });
    }

    userRequests.push(now);
    next();
  };
};

// 组合中间件
const app = express();

app.use(createLogger());
app.use(createRateLimiter());
app.use(express.json());

// 函数式路由
const apiRouter = express.Router();

// 用户路由
const userRouter = express.Router();

userRouter.get('/',
  createAuthMiddleware({ role: 'admin' }),
  createHandler(async (req) => {
    const { page = 1, limit = 10, search } = req.query;
    const users = await userService.getUsers({ page, limit, search });
    return users;
  })
);

userRouter.post('/',
  createValidator(userSchema),
  createHandler(async (req) => {
    const user = await userService.createUser(req.body);
    return { success: true, user };
  })
);

userRouter.get('/:id',
  createAuthMiddleware(),
  createHandler(async (req) => {
    const user = await userService.getUserById(req.params.id);
    return user;
  })
);

apiRouter.use('/users', userRouter);

// 函数式服务层
class UserService {
  constructor(userRepository, cacheService) {
    this.userRepository = userRepository;
    this.cacheService = cacheService;
  }

  async getUsers(filters = {}) {
    const cacheKey = `users:${JSON.stringify(filters)}`;
    const cached = await this.cacheService.get(cacheKey);

    if (cached) {
      return cached;
    }

    const users = await this.userRepository.find(filters);
    await this.cacheService.set(cacheKey, users, 300); // 5分钟缓存

    return users;
  }

  async createUser(userData) {
    // 数据转换和验证
    const processedData = this.processUserData(userData);
    const user = await this.userRepository.create(processedData);

    // 清除相关缓存
    await this.cacheService.delete('users:*');

    return user;
  }

  async getUserById(id) {
    const cacheKey = `user:${id}`;
    const cached = await this.cacheService.get(cacheKey);

    if (cached) {
      return cached;
    }

    const user = await this.userRepository.findById(id);
    await this.cacheService.set(cacheKey, user, 300);

    return user;
  }

  processUserData(userData) {
    return {
      ...userData,
      email: userData.email.toLowerCase(),
      createdAt: new Date(),
      updatedAt: new Date()
    };
  }
}

// 启动应用
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### 3. 数据处理

#### ETL流程的函数式实现

```javascript
// 函数式ETL管道
const { pipe, map, filter, reduce } = require('lodash/fp');

// 数据提取
const extractData = async (sources) => {
  const results = await Promise.all(
    sources.map(async (source) => {
      try {
        const data = await fetchData(source);
        return { source, data, success: true };
      } catch (error) {
        return { source, error: error.message, success: false };
      }
    })
  );

  return {
    successful: results.filter(r => r.success),
    failed: results.filter(r => !r.success),
    timestamp: new Date().toISOString()
  };
};

// 数据转换管道
const createTransformPipeline = (transformers) => {
  return pipe(
    // 数据清洗
    map(cleanData),
    // 数据验证
    filter(validateData),
    // 应用自定义转换器
    ...transformers.map(transformer => map(transformer)),
    // 数据标准化
    map(normalizeData),
    // 数据增强
    map(enrichData)
  );
};

// 数据清洗函数
const cleanData = (item) => {
  return {
    ...item,
    // 清理字符串
    name: item.name ? item.name.trim() : '',
    email: item.email ? item.email.toLowerCase().trim() : '',
    // 清理数字
    age: typeof item.age === 'number' ? item.age : parseInt(item.age) || 0,
    // 清理日期
    createdAt: item.createdAt ? new Date(item.createdAt) : new Date()
  };
};

// 数据验证函数
const validateData = (item) => {
  return item.name && item.email && item.age >= 0;
};

// 数据标准化函数
const normalizeData = (item) => {
  return {
    ...item,
    // 标准化电话号码
    phone: item.phone ? item.phone.replace(/\D/g, '') : '',
    // 标准化地址
    address: item.address ? item.address.toLowerCase() : '',
    // 计算衍生字段
    ageGroup: item.age < 18 ? 'minor' : item.age < 65 ? 'adult' : 'senior'
  };
};

// 数据增强函数
const enrichData = async (item) => {
  const [locationData, creditScore] = await Promise.all([
    geolocationService.getCoordinates(item.address),
    creditService.getScore(item.email)
  ]);

  return {
    ...item,
    location: locationData,
    creditScore,
    processedAt: new Date().toISOString()
  };
};

// 数据加载
const loadData = async (processedData, destination) => {
  const batchSize = 1000;
  const batches = [];

  for (let i = 0; i < processedData.length; i += batchSize) {
    batches.push(processedData.slice(i, i + batchSize));
  }

  const results = await Promise.all(
    batches.map(async (batch, index) => {
      try {
        await destination.save(batch);
        return { batch: index + 1, success: true, count: batch.length };
      } catch (error) {
        return { batch: index + 1, success: false, error: error.message };
      }
    })
  );

  return {
    totalBatches: batches.length,
    successful: results.filter(r => r.success),
    failed: results.filter(r => !r.success),
    totalRecords: processedData.length
  };
};

// ETL编排器
class ETLOrchestrator {
  constructor(sources, transformers, destination) {
    this.sources = sources;
    this.transformers = transformers;
    this.destination = destination;
    this.transformPipeline = createTransformPipeline(transformers);
  }

  async run() {
    console.log('Starting ETL process...');

    // 提取阶段
    console.log('Extracting data...');
    const extractResult = await extractData(this.sources);
    console.log(`Extracted ${extractResult.successful.length} sources, ${extractResult.failed.length} failed`);

    // 转换阶段
    console.log('Transforming data...');
    const allData = extractResult.successful.flatMap(result => result.data);
    const processedData = this.transformPipeline(allData);
    console.log(`Transformed ${processedData.length} records`);

    // 加载阶段
    console.log('Loading data...');
    const loadResult = await loadData(processedData, this.destination);
    console.log(`Loaded ${loadResult.successful.length} batches, ${loadResult.failed.length} failed`);

    return {
      extraction: extractResult,
      transformation: {
        inputCount: allData.length,
        outputCount: processedData.length
      },
      loading: loadResult,
      timestamp: new Date().toISOString()
    };
  }
}

// 使用示例
const sources = [
  { type: 'database', config: { host: 'db1', query: 'SELECT * FROM users' } },
  { type: 'api', config: { url: 'https://api.example.com/users' } },
  { type: 'file', config: { path: './data/users.csv' } }
];

const transformers = [
  (item) => ({ ...item, category: categorizeUser(item) }),
  (item) => ({ ...item, priority: calculatePriority(item) }),
  (item) => ({ ...item, tags: generateTags(item) })
];

const destination = {
  save: async (batch) => {
    return database.batchInsert('processed_users', batch);
  }
};

const orchestrator = new ETLOrchestrator(sources, transformers, destination);
const result = await orchestrator.run();
```

### 4. 实时数据处理

```javascript
// 函数式流处理
const { Subject, fromEvent, merge, combineLatest } = require('rxjs');
const { map, filter, debounceTime, distinctUntilChanged, switchMap } = require('rxjs/operators');

// 实时数据处理管道
class StreamProcessor {
  constructor() {
    this.dataStream = new Subject();
    this.errorStream = new Subject();
    this.resultStream = new Subject();

    this.setupPipeline();
  }

  setupPipeline() {
    this.dataStream.pipe(
      // 数据清洗
      map(this.cleanData),
      // 数据验证
      filter(this.validateData),
      // 去重
      distinctUntilChanged((a, b) => JSON.stringify(a) === JSON.stringify(b)),
      // 防抖
      debounceTime(1000),
      // 数据转换
      map(this.transformData),
      // 错误处理
      catchError(error => {
        this.errorStream.next(error);
        return of(null);
      }),
      // 过滤掉null值
      filter(data => data !== null)
    ).subscribe(processedData => {
      this.resultStream.next(processedData);
    });
  }

  cleanData(data) {
    return {
      ...data,
      timestamp: data.timestamp || new Date().toISOString(),
      value: typeof data.value === 'number' ? data.value : parseFloat(data.value) || 0,
      metadata: data.metadata || {}
    };
  }

  validateData(data) {
    return data.value !== null && data.value !== undefined;
  }

  transformData(data) {
    return {
      ...data,
      normalizedValue: this.normalizeValue(data.value),
      category: this.categorizeData(data),
      score: this.calculateScore(data)
    };
  }

  normalizeValue(value) {
    // 归一化到0-1范围
    return Math.max(0, Math.min(1, value / 100));
  }

  categorizeData(data) {
    if (data.normalizedValue > 0.8) return 'high';
    if (data.normalizedValue > 0.5) return 'medium';
    return 'low';
  }

  calculateScore(data) {
    return data.normalizedValue * 100;
  }

  sendData(data) {
    this.dataStream.next(data);
  }

  getResultStream() {
    return this.resultStream.asObservable();
  }

  getErrorStream() {
    return this.errorStream.asObservable();
  }
}

// 实时监控仪表板
class RealTimeDashboard {
  constructor() {
    this.processor = new StreamProcessor();
    this.metrics = {
      total: 0,
      high: 0,
      medium: 0,
      low: 0,
      average: 0
    };

    this.setupSubscriptions();
  }

  setupSubscriptions() {
    this.processor.getResultStream().subscribe(data => {
      this.updateMetrics(data);
      this.updateUI(data);
    });

    this.processor.getErrorStream().subscribe(error => {
      console.error('Processing error:', error);
      this.showError(error);
    });
  }

  updateMetrics(data) {
    this.metrics.total++;
    this.metrics[data.category]++;

    // 更新平均值
    const totalValue = this.metrics.total * this.metrics.average;
    this.metrics.average = (totalValue + data.score) / (this.metrics.total + 1);
  }

  updateUI(data) {
    // 这里更新DOM或其他UI元素
    console.log('UI Update:', data);

    // 发送到WebSocket客户端
    if (this.wsServer) {
      this.wsServer.broadcast({
        type: 'data',
        data: data,
        metrics: this.metrics
      });
    }
  }

  showError(error) {
    console.error('Dashboard Error:', error);

    if (this.wsServer) {
      this.wsServer.broadcast({
        type: 'error',
        error: error.message
      });
    }
  }

  processData(data) {
    this.processor.sendData(data);
  }
}

// 事件驱动架构
class EventBus {
  constructor() {
    this.events = new Map();
    this.middlewares = [];
  }

  on(event, handler) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event).push(handler);
  }

  emit(event, data) {
    return this.processEvent(event, data);
  }

  use(middleware) {
    this.middlewares.push(middleware);
  }

  async processEvent(event, data) {
    const handlers = this.events.get(event) || [];

    // 应用中间件
    let processedData = data;
    for (const middleware of this.middlewares) {
      processedData = await middleware(event, processedData);
    }

    // 执行处理器
    const results = await Promise.all(
      handlers.map(handler => handler(processedData))
    );

    return results;
  }
}

// 使用示例
const eventBus = new EventBus();

// 中间件
eventBus.use(async (event, data) => {
  console.log(`[${event}] Processing:`, data);
  return data;
});

// 事件处理器
eventBus.on('user.created', async (userData) => {
  await sendWelcomeEmail(userData.email);
  await updateUserAnalytics(userData);
});

eventBus.on('user.updated', async (userData) => {
  await updateSearchIndex(userData);
  await notifyConnections(userData);
});

eventBus.on('order.completed', async (orderData) => {
  await processPayment(orderData);
  await updateInventory(orderData);
  await sendConfirmationEmail(orderData);
});

// 在应用中发出事件
eventBus.emit('user.created', {
  id: 1,
  name: 'Alice',
  email: 'alice@example.com'
});
```

## 函数式编程的最佳实践

### 1. 项目结构

```
src/
├── functions/           # 纯函数库
│   ├── utils/
│   │   ├── validation.js
│   │   ├── transformation.js
│   │   └── formatting.js
│   ├── business/
│   │   ├── user.js
│   │   ├── order.js
│   │   └── analytics.js
│   └── composables/
│       ├── hooks.js
│       └── middleware.js
├── components/          # 函数式组件
│   ├── ui/
│   ├── forms/
│   └── layout/
├── services/           # 服务层
│   ├── api.js
│   ├── database.js
│   └── cache.js
├── store/              # 状态管理
│   ├── slices/
│   ├── selectors/
│   └── middleware.js
└── app.js              # 应用入口
```

### 2. 测试策略

```javascript
// 函数式测试工具
const { test, expect } = require('@jest/globals');
const { compose, pipe } = require('lodash/fp');

// 纯函数测试
test('add function should add two numbers correctly', () => {
  const add = (a, b) => a + b;
  expect(add(2, 3)).toBe(5);
  expect(add(-1, 1)).toBe(0);
  expect(add(0, 0)).toBe(0);
});

// 函数组合测试
test('function composition should work correctly', () => {
  const add1 = x => x + 1;
  const multiply2 = x => x * 2;
  const square = x => x * x;

  const pipeline = pipe(add1, multiply2, square);

  expect(pipeline(3)).toBe(64); // square(multiply2(add1(3))) = square(multiply2(4)) = square(8) = 64
});

// 异步函数测试
test('async function should handle errors correctly', async () => {
  const asyncOperation = async (shouldFail) => {
    if (shouldFail) {
      throw new Error('Operation failed');
    }
    return 'Success';
  };

  await expect(asyncOperation(true)).rejects.toThrow('Operation failed');
  await expect(asyncOperation(false)).resolves.toBe('Success');
});

// Mock和Stub
test('service should call API correctly', async () => {
  const mockApi = {
    get: jest.fn().mockResolvedValue({ data: 'test' })
  };

  const service = new UserService(mockApi);
  const result = await service.getData();

  expect(mockApi.get).toHaveBeenCalledWith('/data');
  expect(result).toEqual({ data: 'test' });
});

// 属性测试
const { property } = require('fast-check');

test('addition is commutative', () => {
  property([fc.integer(), fc.integer()], (a, b) => {
    return add(a, b) === add(b, a);
  });
});
```

## 总结

函数式编程在现代软件开发中有着广泛的应用：

1. **前端开发**：React组件、状态管理、数据处理
2. **后端开发**：Express中间件、API设计、数据库操作
3. **数据处理**：ETL流程、实时流处理、数据分析
4. **系统架构**：事件驱动、微服务、分布式系统

关键要点：
- **渐进式采用**：不必完全函数式，可以逐步采用
- **场景选择**：在合适的场景使用函数式编程
- **团队协作**：考虑团队的技术水平和接受度
- **性能考虑**：在性能关键的场景进行适当优化
- **测试优先**：函数式代码天然适合测试驱动开发

函数式编程不仅是一种编程范式，更是一种思维方式和解决问题的方法论。通过在实际项目中应用函数式编程原则，我们可以构建出更加健壮、可维护和可扩展的软件系统。

记住，最好的代码是既能解决问题又易于理解的代码。函数式编程为我们提供了实现这一目标的强大工具集。