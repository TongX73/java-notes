
### **Java LinkedHashSet 核心笔记**

#### **一、核心概述**

1.  **继承体系**：
    *   `LinkedHashSet` ➡️ 继承自 `HashSet` ➡️ 实现了 `Set` 接口。
    *   这意味着它拥有 `HashSet` 的所有特性（基于哈希表、唯一性、高效查询）。

2.  **底层实现**：基于 **LinkedHashMap**。
    *   正如 `HashSet` 基于 `HashMap`，`LinkedHashSet` 基于 `LinkedHashMap`。
    *   `LinkedHashSet` 的元素被存储为 `LinkedHashMap` 的 **Key**，Value 同样是一个固定的占位符对象。

3.  **核心特性**：
    *   **唯一 (Unique)**：不允许存储重复的元素。（继承自 `HashSet`）
    *   **有序 (Ordered)**：**维护元素的插入顺序**。这是它与 `HashSet` 最根本的区别。当你遍历它时，元素的顺序就是你第一次添加它们的顺序。
    *   **性能**：与 `HashSet` 相比，`LinkedHashSet` 的插入和删除性能略低，因为需要维护链表结构，但遍历性能更高。
    *   **允许 null 元素**：可以添加一个 `null` 值。
    *   **非线程安全**。

---

#### **二、源码分析与实现原理**

**1. 核心成员变量与构造方法**

`LinkedHashSet` 的实现非常取巧，它本身**没有自己独立的底层数据结构**。它通过调用父类 (`HashSet`) 的特定构造方法，让父类的 `HashMap` 实例化为一个 `LinkedHashMap`。

```java
// LinkedHashSet 的全部构造方法
public LinkedHashSet(int initialCapacity, float loadFactor) {
    // 调用HashSet的包级私有构造方法
    super(initialCapacity, loadFactor, true); // 注意第三个参数dummy为true
}

public LinkedHashSet(int initialCapacity) {
    super(initialCapacity, .75f, true); // 同样调用特定的父类构造方法
}

public LinkedHashSet() {
    super(16, .75f, true); // 默认容量16，负载因子0.75
}

public LinkedHashSet(Collection<? extends E> c) {
    super(Math.max(2*c.size(), 11), .75f, true); // 计算一个合适的初始容量
    addAll(c);
}
```

关键在于 `HashSet` 中这个特殊的、专门为 `LinkedHashSet` 准备的构造方法：

```java
// HashSet的包级私有构造方法
// dummy参数没有实际意义，只是一个标记，用来区分其他构造方法
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    // 创建的是一个LinkedHashMap，而不是HashMap！
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```
*   **结论**：`LinkedHashSet` 在创建时，其内部的 `map` 实际上是一个 `LinkedHashMap` 对象。这就是它所有魔法之源。

**2. `LinkedHashMap` 如何维护顺序？**

`LinkedHashMap` 是 `HashMap` 的子类，它在 `HashMap` “数组 + 链表/红黑树” 结构的基础上，**额外维护了一个贯穿所有条目的双向链表**。



*   这个**双向链表**定义了元素的迭代顺序。
*   默认情况下，这个顺序是**插入顺序**（`accessOrder` 参数为 `false`）。
*   每次插入一个新元素（Key），除了将其放入哈希桶中，还会将其作为一个节点添加到这个双向链表的**尾部**。
*   因此，当使用迭代器遍历时，`LinkedHashMap`（以及 `LinkedHashSet`）只需要简单地遍历这个双向链表即可，顺序就是插入的顺序。

**3. 核心方法的工作方式**

由于 `LinkedHashSet` 没有重写 `add()`, `remove()`, `contains()` 等方法，它完全继承了 `HashSet` 的实现。这些方法依然去调用 `map`（现在是 `LinkedHashMap`）的对应方法。

*   `add(E e)`: 调用 `LinkedHashMap.put(e, PRESENT)`。`LinkedHashMap` 的 `put()` 方法在完成哈希表的操作后，会额外将新元素节点添加到内部双向链表的尾部。
*   `iterator()`: 返回的是 `LinkedHashMap.keySet().iterator()`。这个迭代器是遍历内部的双向链表，所以顺序是插入顺序。
*   其他所有方法（`remove`, `contains`等）的性能特征与 `HashSet` 基本一致，都是 O(1) 时间复杂度。

---

#### **三、与 HashSet 的对比**

| 特性 | `HashSet` | `LinkedHashSet` |
| :--- | :--- | :--- |
| **底层结构** | **HashMap** | **LinkedHashMap** |
| **顺序性** | **完全无序** | **维护插入顺序** |
| **遍历性能** | 遍历时需要访问整个**哈希表数组**及其中的**链表/树**，性能较低。 | 遍历时直接遍历内部的**双向链表**，性能更高。 |
| **插入/删除性能** | **略高**。只需要操作哈希表。 | **略低**。除了操作哈希表，还需要维护双向链表（修改前后节点的指针）。 |
| **内存开销** | 较小。存储元素本身和必要的链表/树节点信息。 | **较大**。每个元素除了 `HashSet` 的开销外，在 `LinkedHashMap` 的节点中还额外包含了 **`before`** 和 **`after`** 两个指针，用于维护双向链表。 |
| **使用场景** | 绝对优先考虑**查询性能**和**内存节省**，且**完全不关心顺序**。 | 既需要 `Set` 的**去重**特性，又需要** predictable 的迭代顺序**（通常是插入顺序）。 |

---

#### **四、使用示例**

```java
import java.util.LinkedHashSet;
import java.util.Set;

public class LinkedHashSetDemo {
    public static void main(String[] args) {
        // 创建一个LinkedHashSet
        Set<String> linkedHashSet = new LinkedHashSet<>();

        // 添加元素
        linkedHashSet.add("Apple");
        linkedHashSet.add("Banana");
        linkedHashSet.add("Orange");
        linkedHashSet.add("Grape");
        linkedHashSet.add("Banana"); // 重复元素，不会被添加
        linkedHashSet.add(null); // 可以添加null

        // 遍历：顺序将是插入的顺序
        System.out.println("LinkedHashSet 遍历顺序 (插入序):");
        for (String fruit : linkedHashSet) {
            System.out.println("  " + fruit);
        }
        // 输出顺序永远是: Apple, Banana, Orange, Grape, null
        // 注意：null 也会被维护在它被插入的位置

        // 对比 HashSet
        Set<String> hashSet = new HashSet<>(linkedHashSet); // 用相同元素创建HashSet
        System.out.println("\nHashSet 遍历顺序 (无序):");
        for (String fruit : hashSet) {
            System.out.println("  " + fruit);
        }
        // 输出顺序每次都可能不同，例如: null, Apple, Grape, Orange, Banana
    }
}
```

---

#### **五、总结与使用场景**

**`LinkedHashSet` 是 `HashSet` 和 `LinkedList` 行为的一个完美结合体：**

*   它拥有 `HashSet` 的**查询效率**和**去重**特性。
*   它拥有 `LinkedList` 的**维护插入顺序**的特性。

**何时使用 LinkedHashSet？**

1.  **需要维护插入顺序的去重集合**：这是最经典的场景。例如，记录用户访问网站的独特页面，并且要求按访问先后顺序输出。
2.  **需要频繁遍历 Set**：因为遍历 `LinkedHashSet` 的性能比遍历 `HashSet` 更高（直接遍历链表 vs 扫描整个哈希表）。
3.  **缓存实现 (LRU Cache 的简单变体)**：虽然标准的 LRU 缓存需要使用 `accessOrder` 模式下的 `LinkedHashMap`，但 `LinkedHashSet` 可以用于实现需要淘汰最早插入元素的缓存策略。

**一句话总结：**
**如果你需要一个既不能重复，又要记住添加顺序的集合，请毫不犹豫地选择 `LinkedHashSet`。** 它通过内部维护一个双向链表，以微小的性能开销为代价，换来了非常有价值的顺序稳定性。