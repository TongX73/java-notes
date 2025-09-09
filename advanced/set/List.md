
### **Java List 接口**

#### **一、List 接口概述**

1.  **特点 (与 Collection 的区别)**:
    *   **有序 (Ordered)**: 元素存储和取出的顺序一致，每个元素都有对应的索引（从 0 开始）。
    *   **可重复 (Duplicates allowed)**: 可以存储重复的元素。
    *   **带索引**: 因此有了很多基于索引的操作方法，这是 `List` 最大的特色。

2.  **常用实现类**:
    *   **`ArrayList`** (最常用): 底层是**可扩容的动态数组**。**查询快，增删慢**。
    *   **`LinkedList`**: 底层是**双向链表**。**增删快，查询慢**。还实现了 `Deque` 接口，可作为队列或栈使用。
    *   **`Vector`** (古老，已过时): 类似 `ArrayList`，但是线程安全的，效率较低。

---

#### **二、List 特有方法（基于索引）**

这些方法是 `Collection` 接口没有的，是 `List` 的核心。

| 方法签名 | 返回值 | 功能描述 |
| :--- | :--- | :--- |
| `void add(int index, E element)` | `void` | **在指定索引位置插入**元素，原位置及后面的元素后移。 |
| `E remove(int index)` | `E` | **移除指定索引位置**的元素，并返回被移除的元素。 |
| `E set(int index, E element)` | `E` | **修改指定索引位置**的元素，返回被修改的旧元素。 |
| `E get(int index)` | `E` | **获取指定索引位置**的元素。 |
| `int indexOf(Object o)` | `int` | 返回指定元素在集合中**第一次出现**的索引，不存在则返回 -1。 |
| `int lastIndexOf(Object o)` | `int` | 返回指定元素在集合中**最后一次出现**的索引，不存在则返回 -1。 |
| `ListIterator<E> listIterator()` | `ListIterator<E>` | 返回一个功能更强大的**列表迭代器**。 |

**代码示例：**

```java
import java.util.ArrayList;
import java.util.List;

public class ListDemo {
    public static void main(String[] args) {
        // 1. 创建List集合对象
        List<String> list = new ArrayList<>();
        
        // 2. 添加元素 (继承自Collection)
        list.add("Apple");
        list.add("Banana");
        list.add("Orange");
        System.out.println("初始列表: " + list); // [Apple, Banana, Orange]

        // 3. void add(int index, E element): 在索引1处插入元素
        list.add(1, "Grape");
        System.out.println("在索引1插入Grape后: " + list); // [Apple, Grape, Banana, Orange]

        // 4. E remove(int index): 移除索引2的元素
        String removedElement = list.remove(2);
        System.out.println("被移除的元素是: " + removedElement); // Banana
        System.out.println("移除后: " + list); // [Apple, Grape, Orange]

        // 5. E set(int index, E element): 将索引1的元素替换掉
        String oldElement = list.set(1, "Pear");
        System.out.println("被替换的旧元素是: " + oldElement); // Grape
        System.out.println("替换后: " + list); // [Apple, Pear, Orange]

        // 6. E get(int index): 获取索引0的元素
        String element = list.get(0);
        System.out.println("索引0的元素是: " + element); // Apple

        // 7. int indexOf(Object o) / lastIndexOf
        list.add("Apple"); // 添加一个重复元素
        System.out.println("当前列表: " + list); // [Apple, Pear, Orange, Apple]
        System.out.println("Apple第一次出现的索引: " + list.indexOf("Apple")); // 0
        System.out.println("Apple最后一次出现的索引: " + list.lastIndexOf("Apple")); // 3
        System.out.println("Mango的索引 (不存在): " + list.indexOf("Mango")); // -1
    }
}
```

---

#### **三、List 的五种遍历方式**

`List` 作为 `Collection` 的子接口，自然支持其所有遍历方式，并因其“有序”的特性，多了一种**独有的遍历方式**。

**假设我们有一个 List:**
```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");
```

**方式 1: 普通 for 循环 (独有，最常用)**
*   **原理**: 利用 `List` 的索引，通过 `get(int index)` 方法获取元素。
*   **优点**: 可以在循环中拿到索引，方便进行与索引相关的操作。
*   **缺点**: 对于 `LinkedList` 这种链表结构，`get(int index)` 效率很低。

```java
for (int i = 0; i < list.size(); i++) {
    String s = list.get(i);
    System.out.println("索引_" + i + ": " + s);
}
// 输出:
// 索引_0: A
// 索引_1: B
// 索引_2: C
```

**方式 2: 迭代器 (Iterator)**
*   **原理**: 使用 `Iterator` 接口进行遍历，这是 `Collection` 体系最通用的方式。
*   **优点**: 标准、安全，可以在遍历时用 `iterator.remove()` 移除元素。
*   **缺点**: 只能单向遍历，无法获取索引。

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    System.out.println(s);
    // it.remove(); // 安全移除当前元素
}
```

**方式 3: 增强 for 循环 (foreach)**
*   **原理**: JDK5 的语法糖，内部其实也是使用 `Iterator`。
*   **优点**: 写法极其简洁。
*   **缺点**: **不能在遍历时直接通过集合的 `add/remove` 修改结构**，否则会抛出 `ConcurrentModificationException`。无法获取索引。

```java
for (String s : list) {
    System.out.println(s);
}
```

**方式 4: ListIterator (列表迭代器，独有)**
*   **原理**: `Iterator` 的子接口，专为 `List` 设计，支持**双向遍历**和**在遍历过程中增删改元素**。
*   **优点**: 功能强大，可以向前/向后遍历，可以`add(E e)`、`set(E e)`。
*   **缺点**: 略复杂。

```java
// 获取列表迭代器，初始时光标在开头
ListIterator<String> lit = list.listIterator();

// 正向遍历
while (lit.hasNext()) {
    String s = lit.next();
    System.out.println("正向: " + s);
    if ("B".equals(s)) {
        lit.set("BBB"); // 修改当前元素
        lit.add("BB");  // 在当前元素后添加新元素
    }
}

// 反向遍历 (此时光标已在末尾)
System.out.println("---反向遍历---");
while (lit.hasPrevious()) {
    String s = lit.previous();
    System.out.println("反向: " + s);
}
```

**方式 5: Lambda 表达式 (JDK8+)**
*   **原理**: 通过 `Iterable` 接口的 `forEach(Consumer action)` 方法遍历。
*   **优点**: 代码极其简洁。
*   **缺点**: 无法在遍历中修改结构，无法获取索引（除非使用外部计数器）。

```java
list.forEach(s -> System.out.println(s));
// 或使用方法引用
list.forEach(System.out::println);
```

---

#### **四、总结与选择建议**

| 遍历方式 | 是否独有 | 优点 | 缺点 | 适用场景 |
| :--- | :--- | :--- | :--- | :--- |
| **普通 for 循环** | **是** | 可操作索引 | 对 `LinkedList` 效率低 | **`ArrayList`** 遍历或需要索引时 |
| **迭代器 (Iterator)** | 否 | 通用、安全移除元素 | 只能单向，无索引 | 需要安全移除元素时 |
| **增强 for 循环** | 否 | 写法最简洁 | 无法修改结构，无索引 | 简单遍历所有元素 |
| **列表迭代器 (ListIterator)** | **是** | 可双向遍历、修改、添加 | 代码稍复杂 | 需要双向操作或复杂修改 |
| **Lambda forEach** | 否 | 代码极简 (JDK8+) | 无法修改结构，无索引 | 简单遍历所有元素 |

**核心思想：**
*   **`List` 的核心是索引**，所以 `get(index)`, `add(index, e)`, `remove(index)` 等方法非常重要。
*   选择遍历方式时，**如果需要索引，就用普通 for 循环**；如果只是简单查看所有元素，用**增强 for 循环**或 **Lambda**；如果需要边遍历边复杂修改，用**迭代器**或**列表迭代器**。