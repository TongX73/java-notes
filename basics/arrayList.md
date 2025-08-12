### Java ArrayList 集合详解笔记

---

#### **1. 基本概念**
- **定义**：`ArrayList` 是 Java 集合框架中基于动态数组实现的类，位于 `java.util` 包。
- **特点**：
  - 有序（按添加顺序存储）
  - 可重复元素
  - 允许 `null` 值
  - 非线程安全（线程安全需用 `Collections.synchronizedList` 包装）

---

#### **2. 核心特性**
| 特性                | 说明                                                                 |
|---------------------|----------------------------------------------------------------------|
| 动态扩容            | 初始容量 10，扩容公式：`newCapacity = oldCapacity + (oldCapacity >> 1)` |
| 随机访问效率高      | 基于索引访问，时间复杂度 O(1)                                         |
| 增删效率低          | 中间插入/删除需移动元素，时间复杂度 O(n)                              |
| 实现接口            | `List`, `RandomAccess`（标记支持快速随机访问）                          |

---

#### **3. 创建 ArrayList**
```java
// 基础创建
ArrayList<String> list1 = new ArrayList<>(); 

// 指定初始容量
ArrayList<Integer> list2 = new ArrayList<>(20); 

// 从其他集合初始化
List<String> names = Arrays.asList("Alice", "Bob");
ArrayList<String> list3 = new ArrayList<>(names);
```

---

#### **4. 常用方法**
| 方法                          | 功能描述                                     | 示例                                      |
|-------------------------------|---------------------------------------------|-------------------------------------------|
| `add(E e)`                    | 尾部添加元素                                | `list.add("Java");`                       |
| `add(int index, E e)`         | 指定位置插入元素                            | `list.add(0, "Python");`                  |
| `remove(int index)`           | 删除索引处元素                              | `list.remove(1);`                         |
| `remove(Object o)`            | 删除首次出现的指定元素                      | `list.remove("Java");`                    |
| `get(int index)`              | 获取索引处元素                              | `String s = list.get(0);`                 |
| `set(int index, E e)`         | 替换索引处元素                              | `list.set(0, "C++");`                     |
| `size()`                      | 返回元素数量                                | `int len = list.size();`                  |
| `contains(Object o)`          | 判断是否包含元素                            | `boolean hasJava = list.contains("Java");`|
| `clear()`                     | 清空所有元素                                | `list.clear();`                           |
| `subList(int from, int to)`   | 截取子列表（左闭右开）                      | `List<String> sub = list.subList(0, 2);`  |

---

#### **5. 遍历方式**
1. **for 循环（随机访问）**
   ```java
   for (int i = 0; i < list.size(); i++) {
       System.out.println(list.get(i));
   }
   ```

2. **增强 for 循环**
   ```java
   for (String item : list) {
       System.out.println(item);
   }
   ```

3. **迭代器 (Iterator)**
   ```java
   Iterator<String> it = list.iterator();
   while (it.hasNext()) {
       System.out.println(it.next());
   }
   ```

4. **forEach() + Lambda（Java 8+）**
   ```java
   list.forEach(item -> System.out.println(item));
   ```

---

#### **6. 注意事项**
- **容量管理**：
  - 预估数据量时指定初始容量避免多次扩容
  - 扩容成本高：`trimToSize()` 可释放多余空间
- **并发安全**：
  - 多线程环境使用 `CopyOnWriteArrayList` 或同步包装：
    ```java
    List<String> syncList = Collections.synchronizedList(new ArrayList<>());
    ```
- **元素类型**：
  - 使用泛型约束类型（避免 `ClassCastException`）
- **删除陷阱**：
  - 遍历时删除要用 `Iterator.remove()`，避免 `ConcurrentModificationException`

---

#### **7. 与数组对比**
| 场景               | 数组 (`[]`)                     | `ArrayList`                           |
|--------------------|----------------------------------|----------------------------------------|
| 长度固定           | 是（声明时确定）                | 否（动态扩容）                        |
| 内存占用           | 更小（无额外结构）              | 较大（需维护容量和结构）              |
| 功能支持           | 基础操作                        | 提供丰富方法（增删查改等）            |
| 基本类型存储       | 支持（`int[]`, `double[]` 等）  | 需装箱（`Integer`, `Double` 等）      |

---

#### **8. 示例代码**
```java
import java.util.ArrayList;

public class ArrayListDemo {
    public static void main(String[] args) {
        // 创建并添加元素
        ArrayList<String> languages = new ArrayList<>();
        languages.add("Java");
        languages.add("Python");
        languages.add(1, "C++"); // 在索引1处插入

        // 修改元素
        languages.set(2, "JavaScript");

        // 删除元素
        languages.remove("C++");

        // 遍历输出
        System.out.println("当前列表：");
        for (String lang : languages) {
            System.out.println(lang); // 输出: Java, JavaScript
        }

        // 其他操作
        System.out.println("是否包含Java: " + languages.contains("Java")); // true
        System.out.println("列表大小: " + languages.size()); // 2
    }
}
```

---

#### **9. 性能优化建议**
1. 初始化时指定容量（减少扩容次数）
2. 避免频繁在中间位置插入/删除
3. 大量删除时先用 `clear()` 再批量添加
4. 读取多、修改少时优先使用 `ArrayList`

> **提示**：`ArrayList` 是最常用的集合之一，适合读多写少的场景。需要频繁增删时考虑 `LinkedList`。