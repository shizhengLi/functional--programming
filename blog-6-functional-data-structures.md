# 函数式数据结构：不可变世界的构建块

函数式数据结构是函数式编程的基石。与传统的可变数据结构不同，函数式数据结构在创建后就不能被修改，每次"修改"都会创建一个新的数据结构。这种特性使得它们天然适合并发编程和状态管理。

## 函数式数据结构的特性

### 1. 持久性

函数式数据结构是持久的，这意味着对数据结构的修改不会影响现有的引用：

```javascript
// 传统的可变数组
const arr1 = [1, 2, 3];
const arr2 = arr1;
arr2.push(4);
console.log(arr1); // [1, 2, 3, 4] - arr1也被修改了

// 函数式不可变数组
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4];
console.log(arr1); // [1, 2, 3] - arr1保持不变
console.log(arr2); // [1, 2, 3, 4] - arr2是新数组
```

### 2. 结构共享

为了提高性能，函数式数据结构使用结构共享技术：

```javascript
// 结构共享示例
const original = { a: 1, b: { c: 2, d: 3 } };
const updated = { ...original, a: 2 };

// updated.b 和 original.b 是同一个对象引用
console.log(updated.b === original.b); // true
```

### 3. 引用透明性

函数式数据结构支持引用透明性，相同的操作总是产生相同的结果：

```javascript
const list = [1, 2, 3];
const result1 = list.map(x => x * 2);
const result2 = list.map(x => x * 2);
console.log(result1); // [2, 4, 6]
console.log(result2); // [2, 4, 6]
```

## 常见的函数式数据结构

### 1. 不可变列表

```javascript
class ImmutableList {
  constructor(head, tail = null) {
    this.head = head;
    this.tail = tail;
    this.length = tail ? tail.length + 1 : 1;
  }

  static of(...items) {
    return items.reduceRight((list, item) => new ImmutableList(item, list), null);
  }

  push(value) {
    return new ImmutableList(value, this);
  }

  map(fn) {
    return this.tail ? new ImmutableList(fn(this.head), this.tail.map(fn)) : new ImmutableList(fn(this.head));
  }

  filter(predicate) {
    if (!this.tail) {
      return predicate(this.head) ? new ImmutableList(this.head) : null;
    }

    const filteredTail = this.tail.filter(predicate);
    if (predicate(this.head)) {
      return new ImmutableList(this.head, filteredTail);
    }
    return filteredTail;
  }

  reduce(fn, initialValue) {
    let result = initialValue;
    let current = this;

    while (current) {
      result = fn(result, current.head);
      current = current.tail;
    }

    return result;
  }

  toArray() {
    const result = [];
    let current = this;

    while (current) {
      result.push(current.head);
      current = current.tail;
    }

    return result;
  }
}

// 使用示例
const list = ImmutableList.of(1, 2, 3, 4, 5);
const doubled = list.map(x => x * 2);
const evens = list.filter(x => x % 2 === 0);
const sum = list.reduce((acc, x) => acc + x, 0);

console.log(doubled.toArray()); // [2, 4, 6, 8, 10]
console.log(evens.toArray()); // [2, 4]
console.log(sum); // 15
```

### 2. 不可变二叉搜索树

```javascript
class BinarySearchTree {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }

  insert(value) {
    if (value < this.value) {
      return new BinarySearchTree(
        this.value,
        this.left ? this.left.insert(value) : new BinarySearchTree(value),
        this.right
      );
    } else if (value > this.value) {
      return new BinarySearchTree(
        this.value,
        this.left,
        this.right ? this.right.insert(value) : new BinarySearchTree(value)
      );
    }
    return this;
  }

  contains(value) {
    if (value === this.value) {
      return true;
    } else if (value < this.value && this.left) {
      return this.left.contains(value);
    } else if (value > this.value && this.right) {
      return this.right.contains(value);
    }
    return false;
  }

  inOrderTraversal() {
    const result = [];
    if (this.left) {
      result.push(...this.left.inOrderTraversal());
    }
    result.push(this.value);
    if (this.right) {
      result.push(...this.right.inOrderTraversal());
    }
    return result;
  }

  map(fn) {
    return new BinarySearchTree(
      fn(this.value),
      this.left ? this.left.map(fn) : null,
      this.right ? this.right.map(fn) : null
    );
  }
}

// 使用示例
const tree = new BinarySearchTree(5)
  .insert(3)
  .insert(7)
  .insert(1)
  .insert(9);

console.log(tree.contains(7)); // true
console.log(tree.contains(4)); // false
console.log(tree.inOrderTraversal()); // [1, 3, 5, 7, 9]

const doubledTree = tree.map(x => x * 2);
console.log(doubledTree.inOrderTraversal()); // [2, 6, 10, 14, 18]
```

### 3. 不可变队列

```javascript
class ImmutableQueue {
  constructor(front = [], rear = []) {
    this.front = front;
    this.rear = rear;
  }

  enqueue(value) {
    return new ImmutableQueue(this.front, [...this.rear, value]);
  }

  dequeue() {
    if (this.front.length === 0) {
      if (this.rear.length === 0) {
        return null;
      }
      // 反转rear作为新的front
      const newFront = [...this.rear].reverse();
      return new ImmutableQueue(newFront.slice(1), []);
    }
    return {
      value: this.front[0],
      queue: new ImmutableQueue(this.front.slice(1), this.rear)
    };
  }

  peek() {
    if (this.front.length === 0) {
      return this.rear.length > 0 ? this.rear[this.rear.length - 1] : null;
    }
    return this.front[0];
  }

  isEmpty() {
    return this.front.length === 0 && this.rear.length === 0;
  }

  size() {
    return this.front.length + this.rear.length;
  }
}

// 使用示例
const queue = new ImmutableQueue()
  .enqueue(1)
  .enqueue(2)
  .enqueue(3);

const result1 = queue.dequeue();
console.log(result1.value); // 1
console.log(result1.queue.peek()); // 2

const result2 = result1.queue.dequeue();
console.log(result2.value); // 2
console.log(result2.queue.peek()); // 3
```

## 高级函数式数据结构

### 1. 哈希映射（Trie实现）

```javascript
class HashMap {
  constructor(root = {}) {
    this.root = root;
  }

  set(key, value) {
    const newRoot = { ...this.root };
    let current = newRoot;

    for (let i = 0; i < key.length - 1; i++) {
      const char = key[i];
      if (!current[char]) {
        current[char] = {};
      }
      current = current[char];
    }

    current[key[key.length - 1]] = value;
    return new HashMap(newRoot);
  }

  get(key) {
    let current = this.root;

    for (const char of key) {
      if (!current[char]) {
        return undefined;
      }
      current = current[char];
    }

    return current;
  }

  has(key) {
    return this.get(key) !== undefined;
  }

  delete(key) {
    if (!this.has(key)) {
      return this;
    }

    const newRoot = JSON.parse(JSON.stringify(this.root));
    let current = newRoot;
    const path = [];

    for (let i = 0; i < key.length; i++) {
      const char = key[i];
      path.push({ node: current, char });
      if (i < key.length - 1) {
        current = current[char];
      }
    }

    // 删除最后的字符
    delete current[key[key.length - 1]];

    // 清理空节点
    for (let i = path.length - 2; i >= 0; i--) {
      const { node, char } = path[i];
      if (Object.keys(node[char]).length === 0) {
        delete node[char];
      }
    }

    return new HashMap(newRoot);
  }

  keys() {
    const result = [];
    const traverse = (node, prefix = '') => {
      for (const [char, value] of Object.entries(node)) {
        if (typeof value === 'object' && value !== null) {
          traverse(value, prefix + char);
        } else {
          result.push(prefix + char);
        }
      }
    };

    traverse(this.root);
    return result;
  }
}

// 使用示例
const map = new HashMap()
  .set('name', 'Alice')
  .set('age', '25')
  .set('email', 'alice@example.com');

console.log(map.get('name')); // Alice
console.log(map.has('age')); // true
console.log(map.keys()); // ['name', 'age', 'email']

const updatedMap = map.delete('age');
console.log(updatedMap.has('age')); // false
```

### 2. 不可变集合

```javascript
class ImmutableSet {
  constructor(items = new Set()) {
    this.items = items;
  }

  add(value) {
    const newItems = new Set(this.items);
    newItems.add(value);
    return new ImmutableSet(newItems);
  }

  delete(value) {
    const newItems = new Set(this.items);
    newItems.delete(value);
    return new ImmutableSet(newItems);
  }

  has(value) {
    return this.items.has(value);
  }

  size() {
    return this.items.size;
  }

  union(otherSet) {
    const newItems = new Set(this.items);
    for (const item of otherSet.items) {
      newItems.add(item);
    }
    return new ImmutableSet(newItems);
  }

  intersection(otherSet) {
    const newItems = new Set();
    for (const item of this.items) {
      if (otherSet.has(item)) {
        newItems.add(item);
      }
    }
    return new ImmutableSet(newItems);
  }

  difference(otherSet) {
    const newItems = new Set();
    for (const item of this.items) {
      if (!otherSet.has(item)) {
        newItems.add(item);
      }
    }
    return new ImmutableSet(newItems);
  }

  toArray() {
    return Array.from(this.items);
  }
}

// 使用示例
const set1 = new ImmutableSet([1, 2, 3, 4]);
const set2 = new ImmutableSet([3, 4, 5, 6]);

const union = set1.union(set2);
const intersection = set1.intersection(set2);
const difference = set1.difference(set2);

console.log(union.toArray()); // [1, 2, 3, 4, 5, 6]
console.log(intersection.toArray()); // [3, 4]
console.log(difference.toArray()); // [1, 2]
```

### 3. 函数式栈

```javascript
class ImmutableStack {
  constructor(items = []) {
    this.items = items;
  }

  push(value) {
    return new ImmutableStack([...this.items, value]);
  }

  pop() {
    if (this.items.length === 0) {
      return null;
    }
    return {
      value: this.items[this.items.length - 1],
      stack: new ImmutableStack(this.items.slice(0, -1))
    };
  }

  peek() {
    return this.items.length > 0 ? this.items[this.items.length - 1] : null;
  }

  isEmpty() {
    return this.items.length === 0;
  }

  size() {
    return this.items.length;
  }

  toArray() {
    return [...this.items];
  }
}

// 使用示例
const stack = new ImmutableStack()
  .push(1)
  .push(2)
  .push(3);

const result1 = stack.pop();
console.log(result1.value); // 3
console.log(result1.stack.peek()); // 2

const result2 = result1.stack.pop();
console.log(result2.value); // 2
console.log(result2.stack.peek()); // 1
```

## 实际应用场景

### 1. React状态管理

```javascript
// 使用函数式数据结构管理React状态
const useImmutableState = (initialState) => {
  const [state, setState] = useState(initialState);

  const updateState = (updater) => {
    setState(prevState => updater(prevState));
  };

  return [state, updateState];
};

// 在组件中使用
const UserProfile = () => {
  const [user, updateUser] = useImmutableState({
    name: 'Alice',
    age: 25,
    hobbies: ['reading', 'coding']
  });

  const addHobby = (hobby) => {
    updateUser(user => ({
      ...user,
      hobbies: [...user.hobbies, hobby]
    }));
  };

  const updateAge = (newAge) => {
    updateUser(user => ({
      ...user,
      age: newAge
    }));
  };

  return (
    <div>
      <h2>{user.name}, {user.age}</h2>
      <ul>
        {user.hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
      <button onClick={() => addHobby('gaming')}>Add Hobby</button>
      <button onClick={() => updateAge(user.age + 1)}>Increment Age</button>
    </div>
  );
};
```

### 2. 游戏状态管理

```javascript
// 游戏状态使用函数式数据结构
class GameState {
  constructor(players = [], currentPlayer = 0, gameOver = false) {
    this.players = players;
    this.currentPlayer = currentPlayer;
    this.gameOver = gameOver;
  }

  static create(players) {
    return new GameState(players, 0, false);
  }

  nextPlayer() {
    if (this.gameOver) {
      return this;
    }

    const nextIndex = (this.currentPlayer + 1) % this.players.length;
    return new GameState(this.players, nextIndex, this.gameOver);
  }

  updatePlayer(playerIndex, updates) {
    const newPlayers = this.players.map((player, index) =>
      index === playerIndex ? { ...player, ...updates } : player
    );

    return new GameState(newPlayers, this.currentPlayer, this.gameOver);
  }

  endGame() {
    return new GameState(this.players, this.currentPlayer, true);
  }

  getCurrentPlayer() {
    return this.players[this.currentPlayer];
  }
}

// 使用示例
const gameState = GameState.create([
  { name: 'Player 1', score: 0 },
  { name: 'Player 2', score: 0 }
]);

const nextState = gameState
  .updatePlayer(0, { score: 10 })
  .nextPlayer()
  .updatePlayer(1, { score: 5 })
  .nextPlayer();
```

### 3. 时间旅行调试

```javascript
// 时间旅行调试器
class TimeTravelDebugger {
  constructor(initialState) {
    this.states = [initialState];
    this.currentIndex = 0;
  }

  currentState() {
    return this.states[this.currentIndex];
  }

  dispatch(action) {
    const currentState = this.currentState();
    const newState = this.reducer(currentState, action);

    // 移除当前索引之后的所有状态
    const newStates = this.states.slice(0, this.currentIndex + 1);
    newStates.push(newState);

    this.states = newStates;
    this.currentIndex = this.states.length - 1;

    return newState;
  }

  undo() {
    if (this.currentIndex > 0) {
      this.currentIndex--;
    }
    return this.currentState();
  }

  redo() {
    if (this.currentIndex < this.states.length - 1) {
      this.currentIndex++;
    }
    return this.currentState();
  }

  reducer(state, action) {
    switch (action.type) {
      case 'INCREMENT':
        return { ...state, count: state.count + 1 };
      case 'DECREMENT':
        return { ...state, count: state.count - 1 };
      default:
        return state;
    }
  }
}

// 使用示例
const debugger = new TimeTravelDebugger({ count: 0 });
debugger.dispatch({ type: 'INCREMENT' }); // count: 1
debugger.dispatch({ type: 'INCREMENT' }); // count: 2
debugger.dispatch({ type: 'DECREMENT' }); // count: 1

const undoState = debugger.undo(); // count: 2
const redoState = debugger.redo(); // count: 1
```

## 性能优化策略

### 1. 批量操作

```javascript
// 批量更新策略
const batchUpdate = (dataStructure, operations) => {
  return operations.reduce((current, operation) => {
    return operation(current);
  }, dataStructure);
};

// 使用示例
const list = ImmutableList.of(1, 2, 3);
const updatedList = batchUpdate(list, [
  list => list.map(x => x * 2),
  list => list.filter(x => x > 3),
  list => list.push(10)
]);
```

### 2. 惰性求值

```javascript
// 惰性求值策略
class LazyList {
  constructor(head, tailThunk) {
    this.head = head;
    this.tailThunk = tailThunk;
    this.tail = null;
  }

  getTail() {
    if (!this.tail) {
      this.tail = this.tailThunk();
    }
    return this.tail;
  }

  map(fn) {
    return new LazyList(fn(this.head), () => this.getTail().map(fn));
  }

  filter(predicate) {
    if (predicate(this.head)) {
      return new LazyList(this.head, () => this.getTail().filter(predicate));
    }
    return this.getTail().filter(predicate);
  }

  take(n) {
    if (n <= 0) {
      return null;
    }
    return new LazyList(this.head, () => this.getTail().take(n - 1));
  }

  toArray() {
    const result = [];
    let current = this;

    while (current && result.length < 1000) { // 防止无限循环
      result.push(current.head);
      current = current.getTail();
    }

    return result;
  }
}

// 使用示例
const naturals = (n = 0) => new LazyList(n, () => naturals(n + 1));
const evens = naturals().filter(x => x % 2 === 0);
const first10Evens = evens.take(10);
console.log(first10Evens.toArray()); // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

### 3. 记忆化

```javascript
// 记忆化策略
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

const memoizedSetAdd = memoize((set, value) => set.add(value));
const memoizedListMap = memoize((list, fn) => list.map(fn));
```

## 总结

函数式数据结构是函数式编程的核心组成部分，它们提供了：

1. **不可变性**：数据创建后不能被修改，确保程序的可预测性
2. **持久性**：旧版本的仍然可用，支持时间旅行和撤销操作
3. **结构共享**：通过共享未修改的部分来优化性能
4. **线程安全**：天然适合并发编程环境
5. **引用透明性**：相同的操作总是产生相同的结果

虽然函数式数据结构在性能上可能不如可变数据结构，但通过结构共享、批处理、惰性求值等优化策略，我们可以在大多数情况下获得可接受的性能。

函数式数据结构特别适合：
- 需要高度并发和线程安全的应用
- 需要时间旅行调试和撤销功能的应用
- 需要高度可预测性和可测试性的应用
- 复杂状态管理的应用

掌握函数式数据结构需要改变思维方式，但一旦掌握，你会发现它们能够帮助你构建更加健壮、可维护和优雅的应用程序。