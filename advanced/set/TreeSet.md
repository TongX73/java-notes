
### **Java TreeSet 核心笔记**

#### **一、核心概述**

1.  **实现接口**：实现了 `NavigableSet` 接口（继承自 `SortedSet` 接口），因此支持一系列导航方法。
2.  **底层实现**：基于 **TreeMap** （与 `HashSet`/`HashMap` 的关系如出一辙）。
    *   `TreeSet` 的元素被存储为 `TreeMap` 的 **Key**。
    *   `TreeMap` 的 **Value** 同样统一指向一个静态的、无意义的 `Object` 对象（`PRESENT`）。
3.  **核心特性**：
    *   **唯一 (Unique)**：不允许存储重复的元素。（继承自 `Set`）
    *   **有序 (Ordered)**：**对元素进行排序**。默认使用元素的**自然顺序**（如 Integer 从小到大，String 按字典序），或者根据创建时提供的 `Comparator` 进行排序。这是它与 `HashSet` 和 `LinkedHashSet` 最根本的区别。
    *   **性能**：增、删、查等操作的时间复杂度为 **O(log n)**。比基于哈希表的 `HashSet` (O(1)) 慢，但比 `List` (O(n)) 快。
    *   **不允许 null 元素**：在 JDK7+ 中，如果使用自然排序，添加 `null` 会抛出 `NullPointerException`。

---

#### **二、源码分析与实现原理**

**1. 核心成员变量**

```java
// 底层的TreeMap，负责存储和排序
private transient NavigableMap<E,Object> m; // 注意类型是NavigableMap

// 固定的假值，作为TreeMap中每个Key对应的Value
private static final Object PRESENT = new Object();
```

**2. 构造方法**

所有构造方法本质上都是在初始化内部的 `TreeMap`。
```java
// 默认构造方法：使用元素的自然顺序 (Comparable)
public TreeSet() {
    this(new TreeMap<E,Object>());
}

// 指定一个比较器 (Comparator) 来自定义排序规则
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

// 通过一个集合来创建，使用自然顺序
public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c); // 调用addAll方法添加所有元素
}

// 通过一个SortedSet来创建，继承其排序规则
public TreeSet(SortedSet<E> s) {
    this(s.comparator()); // 使用传入SortedSet的比较器
    addAll(s);
}

// 包级私有构造方法，用于基于NavigableMap创建（主要用于子集视图）
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}
```

**3. 核心方法是如何工作的？**

**a. `add(E e)` - 添加元素**
这是理解 `TreeSet` 如何保证**唯一性**和**有序性**的关键。
```java
public boolean add(E e) {
    return m.put(e, PRESENT) == null;
}
```
*   **工作原理**：
    1.  调用 `m.put(e, PRESENT)` 尝试将元素 `e` 作为 Key 放入底层 `TreeMap`。
    2.  `TreeMap` 的 `put()` 方法会做以下事情：
        *   **排序/比较**：根据构造时指定的 `Comparator` 或者元素自身的 `Comparable` 实现，将新元素 `e` 与树中已有的节点进行比较，找到其应该存放的位置。
        *   **判断是否重复**：在比较寻找位置的过程中，如果发现一个已存在的 Key `k` 满足 `comparator.compare(e, k) == 0`（或 `e.compareTo(k) == 0`），则判定为重复元素。
        *   **插入或替换**：
            *   如果是新元素，则在找到的位置创建一个新的树节点，并返回 `null`。
            *   如果是重复元素，则用新 Value (`PRESENT`) 覆盖旧 Value，并**返回旧的 Value**（也就是 `PRESENT`）。
    3.  `TreeSet.add()` 根据 `m.put()` 的返回值判断：
        *   如果返回 `null`，说明是**新元素**，添加成功，返回 `true`。
        *   如果返回的不是 `null`（返回的是 `PRESENT`），说明是**重复元素**，添加失败，返回 `false`。

**结论**：`TreeSet` 的**去重机制**和**排序机制完全依赖于 `TreeMap`**，后者又依赖于 `Comparator` 或 `Comparable` 的 `compare()` 方法。**它不依赖 `equals()` 和 `hashCode()`**（但 `compareTo()` 的逻辑通常应与 `equals()` 一致）。

**b. 导航方法 (Navigation Methods)**
由于实现了 `NavigableSet`，`TreeSet` 提供了强大的导航功能，这些是其独有的。
```java
// 返回小于给定元素的最大元素
E lower(E e);
// 返回小于等于给定元素的最大元素
E floor(E e);
// 返回大于等于给定元素的最小元素
E ceiling(E e);
// 返回大于给定元素的最小元素
E higher(E e);

// 获取第一个（最小的）元素
E first();
// 获取最后一个（最大的）元素
E last();

// 获取并移除第一个元素
E pollFirst();
// 获取并移除最后一个元素
E pollLast();

// 获取子集 [fromElement, toElement)
SortedSet<E> subSet(E fromElement, E toElement);
// 获取子集 [fromElement, ∞)
SortedSet<E> tailSet(E fromElement);
// 获取子集 (-∞, toElement)
SortedSet<E> headSet(E toElement);
// NavigableSet版本的方法，可以指定是否包含边界
NavigableSet<E> subSet(E fromElement, boolean fromInclusive, E toElement, boolean toInclusive);
```

---

#### **三、排序规则：Comparable vs. Comparator**

这是使用 `TreeSet`（以及 `TreeMap`）的**核心**。

**1. 自然排序 (Natural Ordering) - Comparable**
*   如果 `TreeSet` 是无参构造的，它要求集合中的元素必须实现 `java.lang.Comparable` 接口。
*   `Comparable` 接口有一个方法：`int compareTo(T o)`。
*   元素通过实现 `compareTo` 方法来定义自己的“自然顺序”。
*   **示例**：`String`, `Integer`, `Double` 等包装类都实现了 `Comparable` 接口。

```java
// 自定义类实现Comparable接口
public class Student implements Comparable<Student> {
    private String name;
    private int age;

    @Override
    public int compareTo(Student other) {
        // 先按年龄升序，年龄相同再按姓名升序
        int ageCompare = Integer.compare(this.age, other.age);
        return (ageCompare != 0) ? ageCompare : this.name.compareTo(other.name);
    }
    // ... getters, setters, toString ...
}

// 使用
TreeSet<Student> treeSet = new TreeSet<>(); // 无参构造，使用自然顺序
treeSet.add(new Student("Alice", 25));
treeSet.add(new Student("Bob", 20));
// 遍历时顺序将是: Bob(20), Alice(25)
```

**2. 定制排序 (Custom Ordering) - Comparator**
*   如果 `TreeSet` 是通过传入 `Comparator` 参数构造的，则元素不需要实现 `Comparable` 接口。
*   `Comparator` 是一个函数式接口，有一个方法：`int compare(T o1, T o2)`。
*   它允许你**在集合外部定义多种不同的排序规则**，非常灵活。

```java
// 1. 创建一个按姓名降序的比较器
Comparator<Student> byNameDesc = new Comparator<Student>() {
    @Override
    public int compare(Student s1, Student s2) {
        return s2.getName().compareTo(s1.getName()); // 降序
    }
};

// 2. 使用Lambda表达式（更简洁）
Comparator<Student> byNameDesc = (s1, s2) -> s2.getName().compareTo(s1.getName());

TreeSet<Student> treeSet = new TreeSet<>(byNameDesc); // 传入Comparator
treeSet.add(new Student("Alice", 25));
treeSet.add(new Student("Bob", 20));
// 遍历时顺序将是: Bob, Alice (按名字降序)
```

**3. 规则与异常**
*   如果元素没有实现 `Comparable`，并且在创建 `TreeSet` 时也**没有提供 `Comparator`**，那么尝试添加元素时会抛出 `ClassCastException`。
*   `compareTo` 或 `compare` 方法的返回值必须与 `equals()` 方法保持逻辑上的一致（即 `x.compareTo(y)==0` 应该与 `x.equals(y)` 同真同假），否则会导致 `Set` 的行为违背“唯一性”原则。

---

#### **四、与 HashSet/LinkedHashSet 的对比**

| 特性 | `HashSet` | `LinkedHashSet` | `TreeSet` |
| :--- | :--- | :--- | :--- |
| **底层结构** | **哈希表** (HashMap) | **哈希表 + 链表** (LinkedHashMap) | **红黑树** (TreeMap) |
| **顺序性** | **无序** | **插入顺序** | **排序顺序** (自然或定制) |
| **`add`/`remove`/`contains` 性能** | **O(1)** (平均) | **O(1)** (平均，略慢于HashSet) | **O(log n)** |
| **允许 `null`** | **是** (1个) | **是** (1个) | **否** (JDK7+) |
| **元素要求** | 需正确重写 `hashCode()` 和 `equals()` | 需正确重写 `hashCode()` 和 `equals()` | 需实现 `Comparable` **或** 提供 `Comparator` |
| **使用场景** | 最通用的Set，追求最高性能，不关心顺序 | 需要维护插入顺序的去重集合 | **需要排序**的去重集合，或需要范围查询、导航操作 |

---

#### **五、使用示例**

```java
import java.util.TreeSet;
import java.util.Comparator;

public class TreeSetDemo {
    public static void main(String[] args) {
        // 1. 自然排序示例 (Integer)
        TreeSet<Integer> nums = new TreeSet<>();
        nums.add(5);
        nums.add(2);
        nums.add(8);
        nums.add(1);
        System.out.println("自然排序 (Integer): " + nums); // 输出: [1, 2, 5, 8]

        // 2. 导航方法示例
        System.out.println("大于5的最小元素: " + nums.higher(5));   // 8
        System.out.println("小于等于5的最大元素: " + nums.floor(5)); // 5
        System.out.println("第一个元素: " + nums.first());          // 1
        System.out.println("最后一个元素: " + nums.last());         // 8
        System.out.println("子集 [2, 5): " + nums.subSet(2, 5));   // [2] (左闭右开)

        // 3. 定制排序示例 (String - 按长度排序，长度相同按字典序)
        TreeSet<String> words = new TreeSet<>(
            Comparator.comparingInt(String::length) // 先按长度比
                .thenComparing(Comparator.naturalOrder()) // 长度相同再按自然顺序比
        );
        words.add("apple");
        words.add("banana");
        words.add("cat");
        words.add("dog");
        words.add("elephant");
        System.out.println("按长度排序: " + words);
        // 输出: [cat, dog, apple, banana, elephant]
    }
}
```

---

#### **六、总结与使用场景**

**`TreeSet` 的核心价值在于其“排序”和“导航”能力。**

**何时使用 TreeSet？**

1.  **需要一个始终处于排序状态的去重集合**：这是最核心的场景。例如，需要实时显示一个排行榜（分数从高到低排序）。
2.  **需要频繁进行范围查询和导航操作**：例如，查找比某个元素大/小的所有元素，或者获取某个范围内的子集。
3.  **对集合的遍历顺序有特定的排序要求**，并且这个要求不是插入顺序。

**性能考量：**
*   如果你**不需要排序**，只是需要**去重和快速查找**，请优先选择 `HashSet`，它的性能远高于 `TreeSet`。
*   如果你需要**维护插入顺序**，请选择 `LinkedHashSet`。
*   只有在**必须排序**或者**需要范围查询**时，才选择 `TreeSet`。

**一句话总结：**
**`TreeSet` 是一个基于红黑树实现的、元素唯一且始终有序的集合。** 它的排序规则由元素的 `Comparable` 自然顺序或构造时提供的 `Comparator` 定制规则决定，并提供了强大的导航方法来操作排序后的数据。