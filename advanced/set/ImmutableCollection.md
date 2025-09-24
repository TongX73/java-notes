
### **Java 不可变集合笔记**

#### **一、不可变集合概述**

1.  **什么是不可变集合？**
    *   不可变集合是**创建后其内容不能被修改**的集合。
    *   任何试图添加、删除或修改元素的操作都会抛出 `UnsupportedOperationException`。

2.  **为什么需要不可变集合？**
    *   **线程安全**：由于不可变，多个线程可以安全地共享而不需要任何同步机制。
    *   **防御性编程**：作为方法返回值或参数时，可以防止调用方意外修改内部数据。
    *   **常量集合**：适合表示程序中的常量数据。
    *   **性能优化**：不可变对象可以被安全地缓存和重用。
    *   **简化代码**：不需要担心对象在传递过程中被修改。

---

#### **二、创建不可变集合的多种方式**

##### **1. JDK 9+ 推荐方式：`of()` 工厂方法（最佳实践）**

从 JDK 9 开始，`List`、`Set`、`Map` 接口都提供了静态的 `of()` 方法来创建不可变集合。

**特点：**
*   **真正不可变**：创建后完全不能修改。
*   **空间优化**：针对小规模集合进行了特殊优化。
*   **拒绝 `null`**：不允许包含 `null` 元素/键值。
*   **元素唯一性**：对于 `Set.of()` 和 `Map.of()` 的键，传入重复元素会抛出 `IllegalArgumentException`。

**代码示例：**
```java
// 1. 不可变List
List<String> immutableList = List.of("A", "B", "C");
System.out.println(immutableList); // [A, B, C]
// immutableList.add("D"); // 抛出 UnsupportedOperationException
// immutableList.set(0, "X"); // 抛出 UnsupportedOperationException

// 2. 不可变Set
Set<Integer> immutableSet = Set.of(1, 2, 3, 4, 5);
System.out.println(immutableSet); // 顺序可能不同，如 [3, 5, 1, 2, 4]
// Set<Integer> badSet = Set.of(1, 2, 2); // 抛出 IllegalArgumentException

// 3. 不可变Map (最多10个键值对)
Map<String, Integer> immutableMap = Map.of(
    "Alice", 25,
    "Bob", 30,
    "Charlie", 35
);
System.out.println(immutableMap); // {Charlie=35, Alice=25, Bob=30}

// 4. 超过10个键值对使用 Map.ofEntries()
Map<String, Integer> largeImmutableMap = Map.ofEntries(
    Map.entry("A", 1),
    Map.entry("B", 2),
    Map.entry("C", 3),
    Map.entry("D", 4),
    // ... 更多条目
    Map.entry("Z", 26)
);

// 5. 空集合
List<String> emptyList = List.of();
Set<Double> emptySet = Set.of();
Map<String, String> emptyMap = Map.of();
```

##### **2. JDK 8 及之前的方式：`Collections.unmodifiableXxx()`**

这是 JDK 9 之前创建不可变视图的标准方式。

**重要特点：**
*   返回的是**原集合的视图**，不是副本！
*   如果原集合被修改，这个"不可变"视图的内容也会跟着改变。
*   它只是"包装器"，真正的不可变性依赖于不保留对原集合的引用。

**代码示例：**
```java
// 1. 创建可变集合
List<String> mutableList = new ArrayList<>();
mutableList.add("A");
mutableList.add("B");

// 2. 创建不可变视图
List<String> unmodifiableList = Collections.unmodifiableList(mutableList);
System.out.println(unmodifiableList); // [A, B]

// 3. 尝试修改视图会失败
// unmodifiableList.add("C"); // 抛出 UnsupportedOperationException

// 4. 但是修改原集合会影响视图！
mutableList.add("C");
System.out.println(unmodifiableList); // [A, B, C] !!! 视图内容改变了！

// 5. 创建真正的不可变集合（防御性拷贝）
List<String> trulyImmutable = Collections.unmodifiableList(new ArrayList<>(mutableList));
// 现在即使修改 mutableList，trulyImmutable 也不会改变
```

**`Collections.unmodifiableXxx()` 方法列表：**
```java
Collections.unmodifiableCollection(Collection<? extends T> c)
Collections.unmodifiableList(List<? extends T> list)
Collections.unmodifiableSet(Set<? extends T> s)
Collections.unmodifiableMap(Map<? extends K, ? extends V> m)
Collections.unmodifiableSortedSet(SortedSet<T> s)
Collections.unmodifiableSortedMap(SortedMap<K, ? extends V> m)
```

##### **3. 使用 `Collections.emptyXxx()` 和 `Collections.singletonXxx()`**

这些方法也返回不可变集合，适用于特殊场景。

```java
// 空集合（推荐用于返回空结果，避免返回null）
List<String> emptyList = Collections.emptyList();
Set<Integer> emptySet = Collections.emptySet();
Map<String, Object> emptyMap = Collections.emptyMap();

// 单元素集合
List<String> singleList = Collections.singletonList("Only One");
Set<Double> singleSet = Collections.singleton(3.14);
Map<String, Integer> singleMap = Collections.singletonMap("key", 100);
```

##### **4. 使用 Stream API 创建（JDK 8+）**

对于需要复杂处理的情况，可以使用 Stream：

```java
// 从数组创建不可变List
List<String> immutableList = Arrays.stream(new String[]{"A", "B", "C"})
                                  .collect(Collectors.collectingAndThen(
                                      Collectors.toList(),
                                      Collections::unmodifiableList
                                  ));

// 更简洁的方式（JDK 10+）
List<String> immutableList = Arrays.asList("A", "B", "C").stream()
                                  .collect(Collectors.toUnmodifiableList());

// JDK 16+ 的 Stream.toList() 返回不可变列表（但具体实现可能因版本而异）
List<String> list = Stream.of("A", "B", "C").toList();
```

##### **5. 使用 Guava 库的不可变集合**

Google Guava 库提供了更强大的不可变集合支持：

```java
// 需要先添加Guava依赖
import com.google.common.collect.ImmutableList;
import com.google.common.collect.ImmutableSet;
import com.google.common.collect.ImmutableMap;

List<String> list = ImmutableList.of("A", "B", "C");
Set<Integer> set = ImmutableSet.of(1, 2, 3, 4, 5);
Map<String, Integer> map = ImmutableMap.of("Alice", 25, "Bob", 30);

// 使用构建器模式
List<String> complexList = ImmutableList.<String>builder()
    .add("A")
    .addAll(existingList)
    .add("Z")
    .build();
```

---

#### **三、不可变集合的特性总结**

| 特性 | 说明 |
| :--- | :--- |
| **不可修改性** | 创建后不能添加、删除、修改元素。 |
| **线程安全** | 天然线程安全，无需同步。 |
| **内存效率** | JDK 9+ 的 `of()` 方法对小集合有特殊优化。 |
| **Null 处理** | JDK 9+ 的 `of()` 方法**不允许 null**，会抛出 `NullPointerException`。 |
| **重复元素** | `Set.of()` 和 `Map.of()` 的键**不允许重复**。 |
| **序列化** | 所有不可变集合都实现了 `Serializable`。 |

---

#### **四、最佳实践与使用场景**

##### **1. 方法返回值**
```java
// 好的实践：返回不可变集合
public List<String> getConfigurations() {
    // 从数据库或配置文件读取
    List<String> configs = fetchConfigsFromDB();
    // 返回不可变副本，防止调用方修改
    return List.copyOf(configs); // JDK 10+ 的便捷方法
}

// 调用方可以安全使用，不用担心内部数据被意外修改
List<String> configs = service.getConfigurations();
// configs.add("new config"); // 编译通过但运行时报错，明确表示不允许修改
```

##### **2. 常量定义**
```java
// 使用不可变集合定义常量
public class Constants {
    public static final List<String> SUPPORTED_LANGUAGES = 
        List.of("Java", "Python", "JavaScript", "Go");
    
    public static final Map<String, Integer> DEFAULT_TIMEOUTS = 
        Map.of("connection", 5000, "read", 30000, "write", 30000);
    
    // 使用 Collections.unmodifiableXXX 包装（JDK 8）
    public static final Set<String> RESERVED_KEYWORDS = 
        Collections.unmodifiableSet(new HashSet<>(Arrays.asList("public", "private", "class")));
}
```

##### **3. 防御性编程**
```java
public class DataProcessor {
    private final List<String> data;
    
    // 构造函数中创建防御性拷贝
    public DataProcessor(List<String> inputData) {
        // 防止调用方保留引用并后续修改
        this.data = List.copyOf(inputData); // JDK 10+
        // 或 this.data = Collections.unmodifiableList(new ArrayList<>(inputData));
    }
    
    public void process() {
        // 安全地使用 data，不用担心被外部修改
        for (String item : data) {
            // 处理逻辑
        }
    }
}
```

##### **4. 选择指南**

| 场景 | 推荐方式 | 说明 |
| :--- | :--- | :--- |
| **JDK 9+ 环境，小型常量集合** | `List/Set/Map.of()` | **首选**，最简洁、最高效 |
| **JDK 8 环境** | `Collections.unmodifiableXxx()` + 拷贝 | 需要先创建副本 |
| **需要空集合** | `Collections.emptyXxx()` 或 `Xxx.of()` | 避免返回 `null` |
| **单元素集合** | `Collections.singletonXxx()` | 专用方法更清晰 |
| **复杂构建逻辑** | **Guava 的 ImmutableXxx.builder()** | 构建器模式最灵活 |
| **从现有集合转换** | `List.copyOf()` (JDK 10+) | 最简洁的转换方式 |

---

#### **五、注意事项**

1.  **`Collections.unmodifiableXxx()` 不是真正的不可变**
    *   它只是原集合的视图，必须配合防御性拷贝使用。
    *   **错误示例**：
        ```java
        List<String> original = new ArrayList<>(Arrays.asList("A", "B"));
        List<String> "immutable" = Collections.unmodifiableList(original);
        original.add("C"); // "immutable" 现在包含 [A, B, C]！
        ```
    *   **正确做法**：
        ```java
        List<String> trulyImmutable = 
            Collections.unmodifiableList(new ArrayList<>(original));
        ```

2.  **JDK 9+ 的 `of()` 方法限制**
    *   不允许 `null` 元素。
    *   `Set.of()` 和 `Map.of()` 不允许重复元素。
    *   这些限制实际上是优点，可以提前发现数据问题。

3.  **性能考虑**
    *   对于频繁修改的场景，不可变集合不适用（因为需要创建新集合）。
    *   但对于只读访问，不可变集合通常性能更好（无需同步检查）。

---

#### **六、总结**

| 方面 | 关键点 |
| :--- | :--- |
| **核心理念** | **创建后不能修改**的集合，提供安全性和可靠性。 |
| **JDK 9+ 首选** | `List.of()`, `Set.of()`, `Map.of()`, `Map.ofEntries()` |
| **JDK 8 方案** | `Collections.unmodifiableXxx(original)` + **防御性拷贝** |
| **特殊场景** | 空集合用 `emptyXxx()`，单元素用 `singletonXxx()` |
| **主要优势** | **线程安全**、**防御性编程**、**常量定义**、**代码清晰** |
| **使用场景** | 方法返回值、常量定义、配置数据、共享数据 |

**现代 Java 开发最佳实践：**
1.  **优先使用不可变集合**，除非有明确的修改需求。
2.  **在 JDK 9+ 中，总是使用 `Xxx.of()`** 来创建小型不可变集合。
3.  **避免返回 `null`**，使用空集合代替。
4.  **使用防御性拷贝**来保护内部数据不被外部修改。
5.  **将不可变思维作为默认选择**，可变作为特例。

**一句话总结：**
**不可变集合是现代 Java 开发的基石之一，它们通过禁止修改来提供线程安全和代码可靠性。JDK 9+ 的工厂方法让创建不可变集合变得异常简单，应该成为首选工具。**