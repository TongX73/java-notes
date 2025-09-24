
### **Java Collections 工具类笔记**

#### **一、Collections 类概述**

1.  **它是什么？**
    *   `java.util.Collections` 是一个**工具类**，仅由静态方法组成，用于操作或返回集合。
    *   它类似于数组的工具类 `Arrays`，但服务的对象是 `Collection` 及其子类。

2.  **它的作用？**
    *   提供了一系列通用算法，如排序、查找、替换、线程同步等，避免了开发者重复造轮子。
    *   提供了创建特殊集合（如空集合、不可变集合、同步集合）的工厂方法。

3.  **注意区分：**
    *   **`Collection`**：是单列集合的**顶层接口**（如 List, Set）。
    *   **`Collections`**：是操作集合的**工具类**。

---

#### **二、排序与查找操作（针对 List）**

这些方法通常只对 `List` 有效，因为 `Set` 通常是无序的。

**1. 排序 `sort()`**
*   `static <T extends Comparable<? super T>> void sort(List<T> list)`
    *   **自然排序**：根据元素的自然顺序（`Comparable` 接口）对指定列表进行升序排序。
    *   **要求**：列表中的所有元素都必须实现 `Comparable` 接口。

*   `static <T> void sort(List<T> list, Comparator<? super T> c)`
    *   **定制排序**：根据指定的 `Comparator` 比较器产生的顺序对列表进行排序。
    *   **更灵活**：可以自定义排序规则，或者为未实现 `Comparable` 的类定义顺序。

**代码示例：**
```java
List<Integer> numbers = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5, 9));
List<String> words = new ArrayList<>(Arrays.asList("banana", "apple", "cherry"));

// 1. 自然排序
Collections.sort(numbers); // [1, 1, 3, 4, 5, 9]
Collections.sort(words);   // [apple, banana, cherry]

// 2. 定制排序：按字符串长度排序
Collections.sort(words, (s1, s2) -> s1.length() - s2.length());
// 或者使用 Comparator.comparingInt
Collections.sort(words, Comparator.comparingInt(String::length));
// 结果: [apple, banana, cherry] (假设长度相同，顺序可能保留)

// 3. 定制排序：降序
Collections.sort(numbers, Collections.reverseOrder());
// 结果: [9, 5, 4, 3, 1, 1]
```

**2. 查找 `binarySearch()`**
*   `static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key)`
*   `static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c)`
*   **前提**：列表必须是**已排序**的（通常先调用 `sort()`），否则结果未定义。
*   **返回值**：
    *   如果找到 key，返回其**索引**。
    *   如果没找到，返回一个**负值** `(-(insertion point) - 1)`。`插入点`是指第一个大于 key 的元素索引，如果所有元素都小于 key，则插入点为 `list.size()`。
    *   这个负返回值很有用，可以计算出 key 应该插入的位置：`-returnValue - 1`。

**代码示例：**
```java
List<Integer> list = new ArrayList<>(Arrays.asList(2, 4, 6, 8, 10));
Collections.sort(list); // 确保有序 [2, 4, 6, 8, 10]

int index1 = Collections.binarySearch(list, 6);
System.out.println(index1); // 输出: 2 (找到)

int index2 = Collections.binarySearch(list, 5);
System.out.println(index2); // 输出: -3 (未找到)
// 插入点计算: -(-3) - 1 = 3 - 1 = 2。意思是5应该插入在索引2的位置（即6的前面）
// 列表变为 [2, 4, 5, 6, 8, 10]
```

**3. 反转 `reverse()`**
*   `static void reverse(List<?> list)`
*   反转指定列表中元素的顺序。

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4));
Collections.reverse(list);
System.out.println(list); // 输出: [4, 3, 2, 1]
```

**4. 随机打乱 `shuffle()`**
*   `static void shuffle(List<?> list)`
*   `static void shuffle(List<?> list, Random rnd)`
*   使用默认的或指定的随机源对列表进行随机置换（洗牌）。

```java
List<Integer> cards = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
Collections.shuffle(cards);
System.out.println(cards); // 输出: [6, 8, 3, 10, 2, ...] (每次顺序随机)
```

**5. 交换 `swap()`**
*   `static void swap(List<?> list, int i, int j)`
*   交换列表中指定位置的元素。

```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
Collections.swap(list, 0, 2);
System.out.println(list); // 输出: [C, B, A]
```

**6. 填充 `fill()`**
*   `static <T> void fill(List<? super T> list, T obj)`
*   用指定的元素替换列表中的所有元素。

```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
Collections.fill(list, "X");
System.out.println(list); // 输出: [X, X, X]
```

**7. 复制 `copy()`**
*   `static <T> void copy(List<? super T> dest, List<? extends T> src)`
*   **注意**：将源列表（src）的所有元素复制到**目标列表（dest）** 中。
*   **要求**：目标列表的**长度必须大于等于源列表**，否则会抛出 `IndexOutOfBoundsException`。通常先确保 `dest` 的大小。

```java
List<String> src = Arrays.asList("A", "B", "C");
List<String> dest = new ArrayList<>(Arrays.asList("1", "2", "3", "4", "5"));
Collections.copy(dest, src);
System.out.println(dest); // 输出: [A, B, C, 4, 5] (前三个被覆盖)
```

---

#### **三、同步控制方法（线程安全）**

`ArrayList`, `HashSet`, `HashMap` 等都是**非线程安全**的。`Collections` 提供了将它们转换为**线程安全版本**的方法。其原理是**装饰器模式**，返回一个包装类，所有方法都用 `synchronized` 同步块包裹。

*   `static <T> Collection<T> synchronizedCollection(Collection<T> c)`
*   `static <T> List<T> synchronizedList(List<T> list)`
*   `static <T> Set<T> synchronizedSet(Set<T> s)`
*   `static <K,V> Map<K,V> synchronizedMap(Map<K,V> m)`
*   `static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s)`
*   `static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m)`

**代码示例与重要注意事项：**
```java
// 创建一个非线程安全的ArrayList
List<String> unsafeList = new ArrayList<>();

// 将其包装成一个线程安全的List
List<String> safeList = Collections.synchronizedList(unsafeList);

// 现在多个线程可以安全地调用 safeList 的单个方法了
safeList.add("element"); // 这个操作是原子的、线程安全的

// ---------------- 重要警告！----------------
// 即使返回的集合是线程安全的，但复合操作（迭代、条件操作等）仍然需要手动同步！
// 错误示例：
// if (!safeList.contains("a")) { // 操作1
//     safeList.add("a");        // 操作2 (这两个操作分开不是原子的)
// }

// 正确做法：在客户端代码手动同步
synchronized (safeList) { // 必须使用 safeList 作为监视器锁
    if (!safeList.contains("a")) {
        safeList.add("a");
    }
}

// 迭代器也必须在同步块中使用！
synchronized (safeList) {
    Iterator<String> it = safeList.iterator();
    while (it.hasNext()) {
        System.out.println(it.next());
    }
}
```
**在现代Java开发中，更推荐使用 `java.util.concurrent` 包下的并发集合（如 `CopyOnWriteArrayList`, `ConcurrentHashMap`），它们的性能和行为通常优于这些同步包装器。**

---

#### **四、不可变（只读）集合**

这些方法用于创建不可修改的集合视图。任何试图修改返回集合的操作都会抛出 `UnsupportedOperationException`。

*   `static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c)`
*   `static <T> List<T> unmodifiableList(List<? extends T> list)`
*   `static <T> Set<T> unmodifiableSet(Set<? extends T> s)`
*   `static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m)`
*   `static <T> SortedSet<T> unmodifiableSortedSet(SortedSet<T> s)`
*   `static <K,V> SortedMap<K,V> unmodifiableSortedMap(SortedMap<K, ? extends V> m)`

**代码示例：**
```java
List<String> originalList = new ArrayList<>();
originalList.add("A");
originalList.add("B");

// 创建一个不可修改的视图
List<String> unmodifiableList = Collections.unmodifiableList(originalList);

System.out.println(unmodifiableList.get(0)); // 可以读 "A"
// unmodifiableList.add("C"); // 抛出 UnsupportedOperationException
// unmodifiableList.set(0, "X"); // 抛出 UnsupportedOperationException

// 但是，原始集合仍然可以被修改，并且修改会反映到不可修改视图上！
originalList.add("C");
System.out.println(unmodifiableList); // 输出: [A, B, C]
```
**注意**：它返回的是原集合的一个**视图**，并非一个独立的拷贝。如果原集合改变，这个“不可变”视图的内容也会跟着变。如果需要真正的不可变副本，可以：
```java
List<String> trulyImmutableList = Collections.unmodifiableList(new ArrayList<>(originalList));
```

**JDK 9+ 的 `List.of()`, `Set.of()`, `Map.of()`**
*   从 JDK 9 开始，推荐使用这些新方法创建**真正不可变**的微型集合。
*   它们创建的集合是独立的，与任何原集合无关，且空间效率更高。
    ```java
    List<String> list = List.of("A", "B", "C");
    Set<Integer> set = Set.of(1, 2, 3);
    Map<String, Integer> map = Map.of("Key1", 1, "Key2", 2);
    // list.add("D"); // 直接抛出 UnsupportedOperationException
    ```

---

#### **五、其他实用方法**

**1. 单元素集合**
*   `static <T> Set<T> singleton(T o)`
*   `static <T> List<T> singletonList(T o)`
*   `static <K,V> Map<K,V> singletonMap(K key, V value)`
*   返回一个不可变的、只包含一个指定对象的集合。常用于需要传入集合参数但只有一个值的情况，避免创建不必要的集合对象。

```java
// 传统方式：需要创建一个ArrayList
List<String> tempList = new ArrayList<>();
tempList.add("OnlyItem");
processList(tempList);

// 优雅方式：使用singletonList
processList(Collections.singletonList("OnlyItem"));

// 从集合中移除所有等于"ToRemove"的元素
list.removeAll(Collections.singleton("ToRemove"));
```

**2. 空集合**
*   `static <T> List<T> emptyList()`
*   `static <T> Set<T> emptySet()`
*   `static <K,V> Map<K,V> emptyMap()`
*   返回一个不可变的空集合。用于需要返回空集合的方法，避免返回 `null`，从而避免客户端的空指针检查。

```java
// 不好的做法：返回null
public List<String> getData() {
    if (noData) {
        return null; // 调用方必须判空，否则可能NPE
    }
}

// 好的做法：返回空集合
public List<String> getData() {
    if (noData) {
        return Collections.emptyList(); // 调用方可以直接遍历，无需判空
    }
}
```

**3. 频率与不相交**
*   `static int frequency(Collection<?> c, Object o)`
    *   返回指定集合中等于指定对象的元素个数。
    ```java
    List<Integer> list = Arrays.asList(1, 2, 3, 2, 1, 2);
    int freq = Collections.frequency(list, 2);
    System.out.println(freq); // 输出: 3
    ```
*   `static boolean disjoint(Collection<?> c1, Collection<?> c2)`
    *   如果两个指定集合中没有相同的元素，则返回 `true`（即它们没有交集）。
    ```java
    Set<Integer> set1 = Set.of(1, 2, 3);
    Set<Integer> set2 = Set.of(4, 5, 6);
    boolean noCommonElements = Collections.disjoint(set1, set2);
    System.out.println(noCommonElements); // 输出: true
    ```

**4. 极值查找**
*   `static <T extends Object & Comparable<? super T>> T min/max(Collection<? extends T> coll)`
*   `static <T> T min/max(Collection<? extends T> coll, Comparator<? super T> comp)`
*   根据自然顺序或比较器返回集合中的最小/最大元素。

```java
List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9);
Integer min = Collections.min(numbers); // 1
Integer max = Collections.max(numbers); // 9
```

**5. 添加多个元素**
*   `static <T> boolean addAll(Collection<? super T> c, T... elements)`
*   一种便捷方法，将所有指定元素添加到指定集合中。可变参数的使用使其非常方便。

```java
List<String> list = new ArrayList<>();
// 传统方式：
list.add("A");
list.add("B");
list.add("C");

// 便捷方式：
Collections.addAll(list, "A", "B", "C");
// 或者从数组添加
String[] moreElements = {"D", "E", "F"};
Collections.addAll(list, moreElements);
```

---

#### **六、总结与最佳实践**

| 功能类别 | 核心方法 | 说明与建议 |
| :--- | :--- | :--- |
| **排序查找** | `sort`, `binarySearch` | `List` 专用，`binarySearch` 前必须先 `sort`。 |
| **线程安全** | `synchronizedXxx` | **谨慎使用**，需手动同步复合操作。优先考虑 `java.util.concurrent.*` 包。 |
| **不可变集合** | `unmodifiableXxx` | 创建防御性拷贝或只读视图。**JDK9+ 优先用 `Xxx.of()`**。 |
| **单元素/空集合** | `singletonXxx`, `emptyXxx` | **避免 `null`**，使 API 更清晰、更安全。 |
| **算法操作** | `reverse`, `shuffle`, `swap`, `fill`, `copy` | 实用的工具方法，注意 `copy` 的目标列表大小要求。 |
| **信息获取** | `frequency`, `disjoint`, `min/max` | 方便的统计和比较工具。 |

**最佳实践：**
1.  **选择正确的工具**：根据需求（是否需要排序、线程安全、不可变）选择合适的方法。
2.  **避免 `null`**：使用 `emptyXxx()` 返回空集合，而不是 `null`。
3.  **性能考量**：同步包装器有性能代价，并发包通常是更好选择。不可变视图并非副本。
4.  **`binarySearch` 的前提**：牢记使用前必须排序，并理解其返回值的含义。
5.  **拥抱新特性**：在 JDK 9+ 环境中，优先使用 `List.of()`, `Set.of()`, `Map.of()` 来创建小型不可变集合。

**一句话总结：**
**`Collections` 是操作 Java 集合的万能工具箱，提供了从排序、线程安全到不可变视图等一系列强大功能，熟练掌握它能极大提升开发效率和代码质量。**