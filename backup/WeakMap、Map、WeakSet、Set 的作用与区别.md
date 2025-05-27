# WeakMap、Map、WeakSet、Set 的作用与区别

## 1. Map（映射）

### 作用
- 存储键值对的集合，键可以是**任意类型**（对象、原始值等）
- 保持键的插入顺序

### 特点
```javascript
const map = new Map();
const keyObj = {id: 1};

map.set(keyObj, 'value');  // 键是对象
map.set('name', 'Alice');  // 键是字符串
map.set(42, 'answer');     // 键是数字

console.log(map.get(keyObj)); // 'value'
console.log(map.size);        // 3
```

### 与普通Object的区别
| 特性          | Map              | Object          |
|---------------|------------------|-----------------|
| 键的类型      | 任意值           | String/Symbol   |
| 顺序          | 插入顺序         | 无序            |
| 大小          | .size 属性       | 手动计算        |
| 性能          | 频繁增删更优     | -               |

## 2. WeakMap（弱映射）

### 作用
- 专用于以**对象作为键**的键值对存储
- 不阻止垃圾回收（弱引用）

### 特点
```javascript
const weakMap = new WeakMap();
let keyObj = {id: 1};

weakMap.set(keyObj, 'private data');

keyObj = null; // 当对象没有其他引用时，会被垃圾回收
// weakMap中的对应条目会自动清除
```

### 与Map的区别
| 特性          | WeakMap          | Map             |
|---------------|------------------|-----------------|
| 键类型        | 只接受对象       | 任意值          |
| 可迭代        | 不可迭代         | 可迭代          |
| 自动清理      | 键被GC时自动删除 | 需手动删除      |
| 使用场景      | 私有数据/缓存    | 通用键值存储    |

## 3. Set（集合）

### 作用
- 存储**唯一值**的集合（任何类型）
- 保持插入顺序

### 特点
```javascript
const set = new Set();
set.add(1);
set.add('text');
set.add({name: 'obj'});
set.add(1); // 重复值会被忽略

console.log(set.size); // 3
console.log(set.has('text')); // true
```

## 4. WeakSet（弱集合）

### 作用
- 存储**唯一对象**的集合
- 不阻止垃圾回收（弱引用）

### 特点
```javascript
const weakSet = new WeakSet();
let obj1 = {id: 1};
let obj2 = {id: 2};

weakSet.add(obj1);
weakSet.add(obj2);

console.log(weakSet.has(obj1)); // true

obj1 = null; // 对象被垃圾回收后，自动从WeakSet移除
```

### 与Set的区别
| 特性          | WeakSet          | Set             |
|---------------|------------------|-----------------|
| 值类型        | 只接受对象       | 任意值          |
| 可迭代        | 不可迭代         | 可迭代          |
| 自动清理      | 值被GC时自动移除 | 需手动删除      |
| 使用场景      | 对象标记/跟踪    | 通用值集合      |

## 使用场景对比

| 结构      | 典型使用场景                                                                 |
|-----------|----------------------------------------------------------------------------|
| Map       | 需要键值对且键可能为非字符串的场景，如DOM节点作为键                         |
| WeakMap   | 存储对象的私有数据，避免内存泄漏（如Vue3响应式系统）                       |
| Set       | 去重、集合运算（并集/交集/差集）                                          |
| WeakSet   | 标记/跟踪对象而不影响其生命周期（如检查对象是否已处理过）                   |

## 关键区别总结

1. **引用强度**：
   - WeakMap/WeakSet是弱引用，不影响垃圾回收
   - Map/Set是强引用，会阻止垃圾回收

2. **可迭代性**：
   - Map/Set可迭代（forEach/for...of）
   - WeakMap/WeakSet不可迭代

3. **键/值限制**：
   - WeakMap键必须是对象，WeakSet值必须是对象
   - Map键和Set值可以是任意类型