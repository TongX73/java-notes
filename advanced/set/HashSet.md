
### **Java HashSet 核心笔记**

#### **一、核心概述**

1.  **实现接口**：`Set`
2.  **底层实现**：基于 **HashMap** （非常重要！）
    *   `HashSet` 的几乎所有操作都是通过调用 `HashMap` 的方法来实现的。
    *   `HashSet` 的元素被存储为 `HashMap` 的 **Key**。
    *   `HashMap` 的 **Value** 则统一指向一个静态的、无意义的 `Object` 对象（`PRESENT`），相当于一个占位符。
3.  **核心特性**：
    *   **无序 (Unordered)**：不保证元素的顺序（即插入顺序和遍历顺序不一致），并且顺序可能会随时间（如扩容时）而变化。
    *   **唯一 (Unique)**：不允许存储重复的元素。
    *   **允许 null 元素**：可以添加一个 `null` 值。
    *   **非线程安全 (Not Thread-Safe)**：多个线程同时修改需要外部同步。

---

#### **二、源码分析与实现原理**

**1. 核心成员变量**

```java
// 底层使用HashMap来存储元素
private transient HashMap<E, Object> map;

// 一个静态的、不变的假值，作为HashMap中每个Key对应的Value
private static final Object PRESENT = new Object();
```

**2. 构造方法**

所有构造方法本质上都是在初始化内部的 `HashMap`。
```java
// 默认构造方法：创建一个初始容量为16，负载因子为0.75的HashMap
public HashSet() {
    map = new HashMap<>();
}

// 指定初始容量
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

// 指定初始容量和负载因子
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

// 通过一个集合来创建HashSet
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c); // 调用addAll方法添加所有元素
}
```

**3. 核心方法是如何工作的？**

**a. `add(E e)` - 添加元素**
这是理解 `HashSet` 如何保证**唯一性**的关键。
```java
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```
*   **工作原理**：
    1.  调用 `map.put(e, PRESENT)` 尝试将元素 `e` 作为 Key 放入底层 HashMap。
    2.  HashMap 的 `put()` 方法会做以下事情：
        *   计算元素 `e` 的 **哈希值** (`hashCode()`)。
        *   根据哈希值找到对应的数组桶（Bucket）。
        *   **判断是否重复**：
            *   如果该桶为空，直接放入新节点，返回 `null`。
            *   如果该桶不为空，遍历桶内的链表或红黑树，用 `equals()` 方法比较 `e` 和已存在的 Key。
            *   如果找到 `equals()` 为 `true` 的 Key，说明是重复元素，则用新 Value (`PRESENT`) 覆盖旧 Value，并**返回旧的 Value**（也就是 `PRESENT`）。
            *   如果没有找到，则创建新节点放入，返回 `null`。
    3.  `HashSet.add()` 根据 `map.put()` 的返回值判断：
        *   如果返回 `null`，说明是**新元素**，添加成功，返回 `true`。
        *   如果返回的不是 `null`（返回的是 `PRESENT`），说明是**重复元素**，添加失败，返回 `false`。

**结论**：`HashSet` 的**去重机制完全依赖于 `HashMap` 的 Key 的唯一性**，而后者又依赖于对象的 `hashCode()` 和 `equals()` 方法。

**b. `remove(Object o)` - 删除元素**
```java
public boolean remove(Object o) {
    return map.remove(o) == PRESENT;
}
```
*   调用 `map.remove(o)` 删除 Key 为 `o` 的映射。
*   如果该 Key 存在，`map.remove()` 会返回其对应的 Value（也就是 `PRESENT`），此时 `remove` 方法返回 `true`。
*   如果 Key 不存在，返回 `null`，此时 `remove` 方法返回 `false`。

**c. `contains(Object o)` - 判断包含**
```java
public boolean contains(Object o) {
    return map.containsKey(o);
}
```
*   直接调用 `map.containsKey(o)`，其内部同样使用 `hashCode()` 和 `equals()` 进行判断。

**d. `iterator()` - 遍历**
```java
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
```
*   返回的是底层 `HashMap` 的 **KeySet 的迭代器**。因此，遍历 `HashSet` 就是遍历 `HashMap` 的所有 Key。
*   由于 `HashMap` 的桶数组结构，遍历顺序是“先从上到下遍历数组，再在每个桶里从左到右遍历链表/树”，所以是**无序的**。

**e. `size()`, `isEmpty()`, `clear()`**
*   所有这些方法都是直接委托给内部的 `map` 对象处理。
    ```java
    public int size() {
        return map.size();
    }
    public boolean isEmpty() {
        return map.isEmpty();
    }
    public void clear() {
        map.clear();
    }
    ```

---

#### **三、`hashCode()` 和 `equals()` 的重要性**

这是使用 `HashSet`（以及 `HashMap`）的**重中之重**。

**1. 规则（必须遵守）：**
*   如果两个对象通过 `equals()` 方法比较是相等的，那么它们的 `hashCode()` 返回值**必须也相等**。
*   如果两个对象的 `hashCode()` 相等，它们通过 `equals()` 比较**不一定相等**（会发生哈希冲突，此时会以链表形式存储）。

**2. 如果违反规则（未正确重写这两个方法）：**
*   **后果**：无法正确判断元素是否重复，`HashSet` 的“唯一性”将失效。
*   **示例**：
    ```java
    public class Student {
        private String name;
        private int age;

        // 构造方法、getter/setter省略...

        // 错误示例：只重写了equals，没重写hashCode
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Student student = (Student) o;
            return age == student.age && Objects.equals(name, student.name);
        }

        // 没有重写 hashCode()!
        // 默认使用Object的hashCode()，根据内存地址计算，每个对象都不同
    }

    public class Test {
        public static void main(String[] args) {
            Set<Student> set = new HashSet<>();
            Student s1 = new Student("Alice", 20);
            Student s2 = new Student("Alice", 20); // 逻辑上相等的对象

            set.add(s1);
            set.add(s2); // 因为s1和s2的hashCode()不同，HashSet会认为它们是两个不同的对象，都能添加成功！

            System.out.println(set.size()); // 输出 2，而不是预期的1！
        }
    }
    ```

**3. 正确做法：**
*   总是同时重写 `equals()` 和 `hashCode()` 方法。
*   可以使用 IDEA 等IDE自动生成，确保两者逻辑一致。
    ```java
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return age == student.age && Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        // 使用Objects.hash()方法，传入equals方法中使用的所有字段
        return Objects.hash(name, age);
    }
    ```

---

#### **四、性能特点与使用注意事项**

1.  **性能**：
    *   `add()`, `remove()`, `contains()`, `size()` 等操作的时间复杂度在理想情况下是 **O(1)**。
    *   性能高度依赖于**哈希函数**的质量。一个好的 `hashCode()` 方法应该让元素均匀分布在各桶中，避免过多的哈希冲突。
    *   在最坏情况下（所有元素哈希冲突，都放入同一个桶形成长链表），这些操作的时间复杂度会退化为 **O(n)**。JDK8之后，当链表长度超过一定阈值（8）时，会将其转换为**红黑树**，使最坏情况下的时间复杂度优化为 **O(log n)**。

2.  **初始容量 (Initial Capacity) 和负载因子 (Load Factor)**
    *   **初始容量**：创建 `HashSet` 时底层数组的大小。默认为 **16**。
    *   **负载因子**：决定数组何时扩容的比率。默认为 **0.75**。
    *   **扩容机制**：当元素数量超过 `容量 * 负载因子` 时，数组会进行扩容（大约翻倍），并重新计算所有元素的位置（Rehash）。这是一个相对耗时的操作。
    *   **优化**：如果能预估元素数量，最好通过构造方法指定一个较大的初始容量，避免频繁扩容。

3.  **线程安全**
    *   `HashSet` 不是线程安全的。
    *   如果需要在多线程环境下使用，可以使用 `Collections.synchronizedSet()` 进行包装：
        ```java
        Set<String> synSet = Collections.synchronizedSet(new HashSet<>());
        ```
    *   或者使用 `ConcurrentHashMap` 支持的 `ConcurrentHashSet`（但JDK没有直接提供，可以用 `ConcurrentHashMap.newKeySet()` 创建）。

---

#### **五、总结**

| 特性 | 说明 |
| :--- | :--- |
| **底层实现** | **基于 HashMap**，元素是 Key，Value 是固定的 `PRESENT` 对象。 |
| **核心特性** | **无序、唯一**。 |
| **去重机制** | 完全依赖元素的 **`hashCode()`** 和 **`equals()`** 方法。**必须正确重写**。 |
| **性能** | 增、删、查操作平均时间复杂度为 **O(1)**。 |
| **允许元素** | 允许一个 `null` 元素。 |
| **线程安全** | **否**。 |
| **选择原因** | 当你需要**快速查找、插入、删除**，并且**不关心顺序**、**需要去重**时，首选 `HashSet`。 |

**一句话总结：**
`HashSet` 是一个披着 Set 外衣的 `HashMap`。它的所有行为都可以用 `HashMap` 的 Key 的行为来解释。理解 `HashSet` 的关键就在于理解 `HashMap` 的 Key 是如何通过哈希表实现快速查找和去重的。