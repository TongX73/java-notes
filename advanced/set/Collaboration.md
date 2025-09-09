
### **Java 单列集合顶层接口：Collection **

#### **一、Collection 接口概述**

1.  **是什么？**：它是所有单列集合（如 List, Set, Queue）的顶级接口，定义了这些集合共有的操作。
2.  **代表什么？**：代表一组对象，这些对象称为元素的集合。
3.  **主要子接口**：
    *   **`List`**：有序、可重复。
    *   **`Set`**：无序、不可重复。
    *   **`Queue`**：队列，特定操作规则。

---

#### **二、核心方法详解与代码示例**

我们将以一个最简单的 `Collection` 实现——`ArrayList` 为例来演示这些方法。

```java
// 导入包
import java.util.ArrayList;
import java.util.Collection;
import java.util.Iterator;

public class CollectionDemo {
    public static void main(String[] args) {
        // 1. 创建集合对象 (多态写法)
        Collection<String> coll = new ArrayList<>();
        System.out.println("初始集合: " + coll); // 输出: 初始集合: []
        System.out.println("==========基本操作==========");

        // 2. add(E e) 方法：添加元素
        coll.add("张三");
        coll.add("李四");
        coll.add("王五");
        coll.add("张三"); // List允许重复，所以这个"张三"会被添加进去
        System.out.println("添加元素后: " + coll); // 输出: [张三, 李四, 王五, 张三]

        // 3. remove(Object o) 方法：移除指定元素
        boolean isRemoved = coll.remove("李四");
        boolean isRemoved2 = coll.remove("赵六"); // 集合中不存在的元素
        System.out.println("移除‘李四’是否成功? " + isRemoved); // 输出: true
        System.out.println("移除‘赵六’是否成功? " + isRemoved2); // 输出: false
        System.out.println("移除后: " + coll); // 输出: [张三, 王五, 张三]

        // 4. contains(Object o) 方法：判断是否包含某元素
        boolean isContains = coll.contains("王五");
        boolean isContains2 = coll.contains("李四");
        System.out.println("包含‘王五’吗? " + isContains); // 输出: true
        System.out.println("包含‘李四’吗? " + isContains2); // 输出: false

        // 5. isEmpty() 方法：判断集合是否为空
        // 6. size() 方法：获取集合大小
        System.out.println("集合是空的吗? " + coll.isEmpty()); // 输出: false
        System.out.println("集合的大小是: " + coll.size()); // 输出: 3

        System.out.println("==========遍历操作==========");

        // 7. iterator() 方法：获取迭代器，用于遍历
        Iterator<String> it = coll.iterator();
        System.out.print("使用迭代器遍历: ");
        while (it.hasNext()) {
            String s = it.next();
            System.out.print(s + " "); // 输出: 张三 王五 张三
        }
        System.out.println(); // 换行

        // 8. 增强for循环 (foreach) 遍历
        System.out.print("使用增强for循环遍历: ");
        for (String s : coll) {
            System.out.print(s + " "); // 输出: 张三 王五 张三
        }
        System.out.println(); // 换行

        System.out.println("==========批量操作==========");

        // 创建另一个集合
        Collection<String> otherColl = new ArrayList<>();
        otherColl.add("赵六");
        otherColl.add("孙七");
        otherColl.add("王五"); // 这个元素在原集合coll中也存在

        // 9. addAll(Collection c) 方法：添加另一个集合的所有元素 (并集)
        coll.addAll(otherColl);
        System.out.println("将otherColl添加到coll后: " + coll); // 输出: [张三, 王五, 张三, 赵六, 孙七, 王五]

        // 10. containsAll(Collection c) 方法：判断是否包含另一个集合的所有元素 (子集)
        Collection<String> checkColl = new ArrayList<>();
        checkColl.add("张三");
        checkColl.add("赵六");
        System.out.println("coll包含checkColl的所有元素吗? " + coll.containsAll(checkColl)); // 输出: true

        // 11. removeAll(Collection c) 方法：移除与另一个集合共有的所有元素 (差集)
        coll.removeAll(checkColl); // 移除所有"张三"和"赵六"
        System.out.println("移除checkColl中的所有元素后: " + coll); // 输出: [王五, 孙七, 王五]

        // 12. retainAll(Collection c) 方法：保留与另一个集合共有的元素，移除其他所有元素 (交集)
        Collection<String> retainColl = new ArrayList<>();
        retainColl.add("王五");
        coll.retainAll(retainColl); // 只保留"王五"，移除"孙七"等
        System.out.println("仅保留与retainColl的交集后: " + coll); // 输出: [王五, 王五]

        System.out.println("==========转换操作==========");

        // 13. toArray() 方法：转换为Object数组
        Object[] objects = coll.toArray();
        for (Object o : objects) {
            System.out.print(o + " "); // 输出: 王五 王五
        }
        System.out.println();

        // 14. toArray(T[] a) 方法：转换为指定类型的数组 (更常用)
        String[] strings = coll.toArray(new String[0]); // 数组大小不够，会自动创建新数组
        // String[] strings = coll.toArray(new String[coll.size()]); // 数组大小刚好，则使用该数组
        System.out.println("数组第一个元素: " + strings[0]); // 输出: 王五

        System.out.println("==========清空操作==========");

        // 15. clear() 方法：清空集合
        System.out.println("清空前: " + coll); // 输出: [王五, 王五]
        coll.clear();
        System.out.println("清空后: " + coll); // 输出: []
        System.out.println("清空后集合是空的吗? " + coll.isEmpty()); // 输出: true
    }
}
```

---

#### **三、关键特性与注意事项（结合示例理解）**

1.  **泛型的使用**：`Collection<String> coll` 使用了泛型 `<String>`，这意味着这个集合只能存储 `String` 类型的对象。这提供了编译时类型安全检查。
2.  **打印集合**：直接打印集合对象（如 `System.out.println(coll)`）会输出格式良好的字符串，如 `[元素1, 元素2, ...]`，这是因为它调用了 `toString()` 方法。
3.  **`equals()` 方法的重要性**：`contains()` 和 `remove()` 等方法依赖元素的 `equals()` 方法来判断是否“相等”。对于 `String` 类，它比较的是内容；对于自定义类，你需要正确重写 `equals()` 方法，否则将使用 `Object` 类的默认实现（比较地址）。
4.  **并发修改异常**：在**迭代器遍历过程中**，如果使用**集合对象自己的 `add()`, `remove()`** 方法修改了集合结构，会抛出 `ConcurrentModificationException`。安全的做法是使用**迭代器自己的 `remove()`** 方法。
    ```java
    // 错误示例：会在运行时抛出异常
    Iterator<String> it = coll.iterator();
    while (it.hasNext()) {
        String s = it.next();
        if ("王五".equals(s)) {
            coll.remove(s); // 错误！用了集合的remove方法
        }
    }

    // 正确示例：使用迭代器的remove方法
    Iterator<String> it = coll.iterator();
    while (it.hasNext()) {
        String s = it.next();
        if ("王五".equals(s)) {
            it.remove(); // 正确！用了迭代器的remove方法
        }
    }
    ```