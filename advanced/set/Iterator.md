
### **Java 迭代器 (Iterator) 超详细笔记**

#### **一、迭代器概述：为什么需要它？**

1.  **什么是迭代器？**
    *   迭代器是一个对象，它提供了一种标准的方法来**顺序访问一个集合中的各个元素**，而不需要暴露该集合的内部表示（如数组索引、链表指针等）。
    *   它实现了 **Iterator 设计模式**，将集合的遍历行为与集合本身分离开来。

2.  **为什么需要迭代器？**
    *   **提供统一的遍历方式**：无论集合底层是数组（`ArrayList`）、链表（`LinkedList`）还是哈希表（`HashSet`, `HashMap`的key/entry），我们都可以用同一套 `hasNext()`/`next()` 方法来遍历。这是**多态**的极致体现。
    *   **安全地移除元素**：在遍历集合的同时，安全地移除当前元素是**迭代器独有的功能**，其他遍历方式（如增强for循环）直接操作集合会导致异常。

---

#### **二、Iterator 接口的核心方法**

`java.util.Iterator<E>` 接口非常简洁，主要定义了三个方法：

| 方法签名 | 返回值 | 功能描述 |
| :--- | :--- | :--- |
| `boolean hasNext()` | `boolean` | **判断是否还有下一个元素**。如果仍有元素可以迭代，则返回 `true`。 |
| `E next()` | `E` | **返回迭代的下一个元素**。如果没有下一个元素还调用此方法，会抛出 `NoSuchElementException`。 |
| `void remove()` | `void` | **从底层集合中移除迭代器返回的最后一个元素**（可选操作）。此方法只能在每次调用 `next()` 后调用**一次**。如果在调用 `next()` 之前调用，或不曾调用 `next()`，或上次调用 `next()` 后已经调用过 `remove()`，则会抛出 `IllegalStateException`。 |

**`default void forEachRemaining(Consumer<? super E> action)`** (JDK 8+)
*   一个默认方法，对剩余的元素执行给定的操作，直到所有元素都被处理或操作引发异常。
*   `action` - 对每个元素执行的操作。

---

#### **三、迭代器的基本使用步骤**

使用迭代器遍历集合遵循一个固定的“套路”：

1.  **获取迭代器对象**：通过调用集合的 `iterator()` 方法。
2.  **循环判断**：使用 `hasNext()` 方法检查是否还有元素。
3.  **获取元素**：在循环体内使用 `next()` 方法获取下一个元素。
4.  **(可选) 操作元素**：处理获取到的元素，或在必要时使用 `remove()` 方法移除。

**代码示例：遍历 ArrayList**
```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorBasicDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");

        // 1. 通过集合的iterator()方法获取迭代器对象
        Iterator<String> iterator = list.iterator();

        // 2. 使用 while 循环进行遍历
        while (iterator.hasNext()) { // 步骤2: 判断是否有下一个元素
            // 步骤3: 获取下一个元素
            String fruit = iterator.next();
            System.out.println(fruit);

            // 示例：在遍历时安全地移除特定元素
            if ("Banana".equals(fruit)) {
                iterator.remove(); // 步骤4 (可选): 安全移除当前元素
                // list.remove(fruit); // 错误！这会引发ConcurrentModificationException
            }
        }

        System.out.println("移除Banana后的列表: " + list); // 输出: [Apple, Cherry]
    }
}
```

---

#### **四、原理探秘：迭代器如何工作？**

1.  **迭代器对象的状态**：
    *   迭代器对象内部会维护一个指向当前集合元素的“**光标 (cursor)**”。
    *   初始时，光标位于第一个元素**之前**。
    *   每调用一次 `next()` 方法，光标就向后移动一位，并返回它刚越过的那个元素。

    

2.  **`modCount` 与快速失败 (Fail-Fast) 机制**：
    *   这是理解 `ConcurrentModificationException` 的关键。
    *   每个集合（如 `ArrayList`）内部都有一个 `modCount`（modification count）变量，用来记录集合**结构性修改**的次数（结构性修改是指改变集合大小的操作，如 `add`, `remove`, `clear`）。
    *   当你获取一个迭代器时，迭代器会记录下当前集合的 `modCount` 值（`expectedModCount = modCount`）。
    *   在迭代器每次调用 `next()`, `remove()` 等方法时，都会检查 `expectedModCount == modCount`。
    *   **如果在迭代过程中，程序直接用集合的方法（而不是迭代器的 `remove()`）对集合进行了结构性修改，集合的 `modCount` 值就会增加，导致 `expectedModCount != modCount`**。此时迭代器就会立即抛出 `ConcurrentModificationException`，而不是等到迭代结束才暴露问题，这就是“快速失败”。
    *   **迭代器自己的 `remove()` 方法会在删除元素后，同步更新 `expectedModCount` 的值**，使其与集合的 `modCount` 保持一致，所以它是安全的。

---

#### **五、迭代器 vs. 增强 for 循环 (foreach)**

*   **增强 for 循环本质上是迭代器的语法糖**。编译器在编译时会把增强 for 循环转换为使用迭代器的代码。
    ```java
    // 你写的代码
    for (String fruit : list) {
        System.out.println(fruit);
    }

    // 编译器编译后的等效代码
    Iterator<String> it = list.iterator();
    while (it.hasNext()) {
        String fruit = it.next();
        System.out.println(fruit);
    }
    ```
*   **结论**：增强 for 循环在遍历过程中**同样不能直接调用集合的 `add`, `remove` 等方法**，否则也会触发快速失败机制，抛出 `ConcurrentModificationException`。

| 特性 | **迭代器 (Iterator)** | **增强 for 循环 (foreach)** |
| :--- | :--- | :--- |
| **语法** | 稍显繁琐 | **极其简洁** |
| **移除元素** | **可以安全地使用 `iterator.remove()`** | **不能**在遍历中移除元素 |
| **底层** | 直接使用 | 是迭代器的语法糖 |
| **适用场景** | 需要**在遍历时删除元素** | **只读遍历**，代码简洁为首要需求 |

---

#### **六、ListIterator：更强大的子接口**

`ListIterator` 是 `Iterator` 的子接口，专为 `List` 设计，提供了**双向遍历**和更多操作。

| 额外方法 | 功能描述 |
| :--- | :--- |
| `boolean hasPrevious()` | 如果此列表迭代器在相反方向遍历列表时具有更多元素，则返回 true。 |
| `E previous()` | 返回列表中的上一个元素。 |
| `int nextIndex()` | 返回对 next 的后续调用所返回元素的索引。 |
| `int previousIndex()` | 返回对 previous 的后续调用所返回元素的索引。 |
| `void set(E e)` | **替换**迭代器返回的**最后一个元素**。 |
| `void add(E e)` | **插入**一个元素到列表中（紧跟 next 返回的元素之后，或 previous 返回的元素之前）。 |

**代码示例：**
```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
ListIterator<String> lit = list.listIterator();

// 正向遍历
while (lit.hasNext()) {
    System.out.print(lit.next() + " "); // 输出: A B C
}
System.out.println();

// 反向遍历
while (lit.hasPrevious()) {
    String element = lit.previous();
    System.out.print(element + " "); // 输出: C B A
    if ("B".equals(element)) {
        lit.set("B-Replaced"); // 将"B"替换为"B-Replaced"
        lit.add("New-Element"); // 在"B"之前插入一个新元素
    }
}
System.out.println("\n最终列表: " + list);
// 输出: 最终列表: [A, New-Element, B-Replaced, C]
```

---

#### **七、总结与最佳实践**

1.  **核心思想**：迭代器提供了一种**统一、安全**的集合遍历方式，是集合框架的“通用语言”。
2.  **使用步骤**：`iterator()` -> `while(hasNext())` -> `next()` -> (操作)。
3.  **核心优势**：**`remove()` 方法是其独有的、安全的元素移除方式**。
4.  **快速失败机制**：理解 `ConcurrentModificationException` 是如何产生的——**迭代过程中直接用集合方法修改结构**。
5.  **选择建议**：
    *   如果只是简单遍历，使用**增强 for 循环**，代码更简洁。
    *   如果需要在遍历中**删除元素**，**必须使用迭代器**及其 `remove()` 方法。
    *   如果需要**双向遍历**或**修改/添加元素**，使用 `ListIterator`。

**一句话总结：**
**迭代器是遍历集合的标准工具。当你的遍历操作不仅仅是“查看”，还需要“编辑”（尤其是删除）时，它就是你的不二之选。** 永远记住：用迭代器遍历时，要用迭代器自己的方法去修改集合。