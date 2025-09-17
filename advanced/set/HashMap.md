
### **Java Map 接口与 HashMap 超详细笔记**

#### **一、Map 接口概述**

1.  **什么是 Map？**
    *   Map 是一个**双列集合**，用于存储具有**映射关系**的数据（键值对，Key-Value Pair）。
    *   它的每个元素都包含两个部分：**Key** (键) 和 **Value** (值)。
    *   **Key 和 Value 可以是任何引用类型的数据**。

2.  **核心特性**：
    *   **Key 不允许重复**：每个 Key 只能映射到一个 Value。如果 put 了相同的 Key，新的 Value 会**覆盖**旧的 Value。
    *   **Value 允许重复**：不同的 Key 可以映射到相同的 Value。
    *   **一个 Key 最多只能对应一个 Value**。
    *   **常用实现类**：`HashMap`, `LinkedHashMap`, `TreeMap`, `Hashtable`。

3.  **Map  vs. Collection**:
    *   `Collection` 是**单列集合**，存储的是一个个独立的元素。
    *   `Map` 是**双列集合**，存储的是键值对。

---

#### **二、Map 接口的常用方法**

| 方法签名 | 返回值 | 功能描述 |
| :--- | :--- | :--- |
| `V put(K key, V value)` | `V` | 添加键值对。如果 key 已存在，则用新 value **替换**旧 value，并**返回旧的 value**；如果 key 不存在，则返回 `null`。 |
| `V get(Object key)` | `V` | 根据指定的 key 获取对应的 value。如果 key 不存在，返回 `null`。 |
| `V remove(Object key)` | `V` | 根据 key 删除对应的键值对。返回被删除的 value；如果 key 不存在，返回 `null`。 |
| `boolean containsKey(Object key)` | `boolean` | 判断集合中是否包含指定的 key。 |
| `boolean containsValue(Object value)` | `boolean` | 判断集合中是否包含指定的 value。 |
| `int size()` | `int` | 返回集合中键值对的数量。 |
| `boolean isEmpty()` | `boolean` | 判断集合是否为空。 |
| `void clear()` | `void` | 清空集合中的所有键值对。 |
| `Set<K> keySet()` | `Set<K>` | **非常重要**：获取所有 key 组成的 **Set 集合**。 |
| `Collection<V> values()` | `Collection<V>` | 获取所有 value 组成的 **Collection 集合**。 |
| `Set<Map.Entry<K, V>> entrySet()` | `Set<Map.Entry<K,V>>` | **非常重要**：获取所有键值对（Entry）组成的 **Set 集合**。 |
| `getOrDefault.()` | ` | **工作原理：**首先尝试通过键从Map中获取对应的值,如果键存在，返回对应的值,如果键不存在，返回指定的默认值(value) |

**代码示例：基本方法使用**
```java
import java.util.HashMap;
import java.util.Map;

public class MapBasicDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();

        // 1. put(K key, V value)
        map.put("Alice", 95);
        map.put("Bob", 88);
        map.put("Charlie", 92);
        Integer oldValue = map.put("Bob", 90); // 更新Bob的成绩
        System.out.println("Bob原来的分数是: " + oldValue); // 输出: 88
        System.out.println("当前Map: " + map); // {Alice=95, Bob=90, Charlie=92}

        // 2. get(Object key)
        Integer score = map.get("Alice");
        System.out.println("Alice的分数: " + score); // 95
        System.out.println("不存在的Key: " + map.get("David")); // null

        // 3. remove(Object key)
        Integer removedScore = map.remove("Charlie");
        System.out.println("被删除的Charlie的分数: " + removedScore); // 92
        System.out.println("删除后: " + map); // {Alice=95, Bob=90}

        // 4. containsKey / containsValue
        System.out.println("是否有Bob? " + map.containsKey("Bob")); // true
        System.out.println("是否有100分? " + map.containsValue(100)); // false

        // 5. size() / isEmpty()
        System.out.println("Map大小: " + map.size()); // 2
        map.clear();
        System.out.println("清空后Map是空的吗? " + map.isEmpty()); // true
    }
}
```

---

#### **三、HashMap 的基本使用与特点**

1.  **特点**：
    *   **基于哈希表实现**（数组 + 链表 + 红黑树，JDK8+）。
    *   **无序**：不保证元素的顺序（插入顺序和遍历顺序不一致）。
    *   **Key 和 Value 都允许为 `null`**。
    *   **非线程安全**。

2.  **工作原理简述（面试核心）**：
    *   **put 过程**：
        1.  计算 Key 的 **hashCode** 值。
        2.  通过**哈希函数**（`(n - 1) & hash`）计算出该键值对应存放在**数组**中的索引位置。
        3.  如果该位置为空，直接放入。
        4.  如果该位置不为空（哈希冲突），则比较新 Key 和该位置上所有已有 Key 的 **hashCode** 和 **equals**。
        5.  如果找到相同的 Key，则替换 Value。
        6.  如果没有相同的 Key，则以**链表**形式挂在后面（JDK7头插，JDK8尾插）。当链表长度超过 8 且数组长度超过 64 时，链表会**转换为红黑树**以提高性能。
    *   **get 过程**：类似 put，先计算哈希值找到数组索引，再遍历该索引下的链表或树，通过 `equals` 方法找到对应的 Key，返回其 Value。

3.  **决定 HashMap 性能的两个参数**：
    *   **初始容量 (Initial Capacity)**：创建 HashMap 时数组的默认大小。默认为 **16**。
    *   **负载因子 (Load Factor)**：决定数组何时扩容的比率。默认为 **0.75**。
    *   **扩容机制 (Rehashing)**：当元素数量超过 `容量 * 负载因子` 时，数组会进行扩容（大约翻倍），并**重新计算所有元素的位置**。这是一个相对耗时的操作。
    *   **优化建议**：如果能预估要存储的键值对数量，最好通过构造方法 `new HashMap<>(initialCapacity)` 指定一个合适的初始容量，避免频繁扩容。

---

#### **四、Map 的三种遍历方式（非常重要）**

假设我们有一个 Map：`Map<String, Integer> map = ...;`（已填充数据）

**方式一：键找值遍历（通过 `keySet()`）**
1.  获取所有 key 的 Set 视图：`Set<String> keys = map.keySet()`
2.  遍历 keySet，用 get(key) 方法获取每个 key 对应的 value。
```java
// 1. 增强 for 循环
for (String key : map.keySet()) {
    Integer value = map.get(key);
    System.out.println(key + " = " + value);
}

// 2. 迭代器
Iterator<String> it = map.keySet().iterator();
while (it.hasNext()) {
    String key = it.next();
    Integer value = map.get(key);
    System.out.println(key + " = " + value);
}
```
*   **优点**：逻辑简单直观。
*   **缺点**：**效率相对较低**。因为对于每个 key，都需要调用一次 `get(key)` 方法，而 `get(key)` 方法本身也是基于哈希计算和可能发生的链表/树遍历。如果 Map 很大，相当于遍历了两遍。

**方式二：键值对遍历（通过 `entrySet()`） - 强烈推荐**
1.  获取所有键值对（Entry）的 Set 视图：`Set<Map.Entry<String, Integer>> entries = map.entrySet()`
2.  遍历 entrySet，从每个 Entry 对象中分别获取 key 和 value。
```java
// 1. 增强 for 循环 (最常用)
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    Integer value = entry.getValue();
    System.out.println(key + " = " + value);
}

// 2. 迭代器
Iterator<Map.Entry<String, Integer>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<String, Integer> entry = it.next();
    String key = entry.getKey();
    Integer value = entry.getValue();
    System.out.println(key + " = " + value);
}
```
*   **优点**：**效率最高**。一次遍历就同时拿到了 key 和 value，无需再次查询。
*   **缺点**：代码稍微复杂一点。

**方式三：仅遍历值（通过 `values()`）**
*   当你只关心 value，不关心 key 时使用。
```java
Collection<Integer> values = map.values();
for (Integer value : values) {
    System.out.println(value);
}
```

**JDK 8+ Lambda 表达式遍历**
*   语法极其简洁，底层实现也是使用的 `entrySet`。
```java
map.forEach((key, value) -> {
    System.out.println(key + " = " + value);
});
```

---

#### **五、总结与对比**

| 方面 | 说明 |
| :--- | :--- |
| **Map 核心** | 存储**键值对**，**Key 唯一**，**Value 可重复**。 |
| **HashMap 特性** | **无序**、**允许 null**、**非线程安全**、**查询高效 (O(1))**。 |
| **去重原理** | 依赖 Key 的 **`hashCode()`** 和 **`equals()`** 方法。**必须正确重写**。 |
| **遍历方式选择** | **优先使用 `entrySet()` 遍历**，性能最好。`keySet()` + `get()` 效率较低。 |
| **性能优化** | 根据业务场景预估值对数量，**指定合适的初始容量**，避免频繁扩容。 |
| **线程安全** | `HashMap` 非线程安全。多线程环境下可使用 `ConcurrentHashMap` 或 `Collections.synchronizedMap(Map)` 包装。 |

**最佳实践建议：**
1.  绝大多数情况下，遍历 Map 请使用 **`for (Map.Entry<K, V> entry : map.entrySet())`** 方式。
2.  如果知道 Map 大致要存放多少数据，创建时使用 `new HashMap<>(initialCapacity)` 来指定初始容量，提升性能。
3.  如果用自定义类的对象作为 Key，**必须同时正确重写该类的 `hashCode()` 和 `equals()` 方法**，否则无法保证键的唯一性和正确查找。