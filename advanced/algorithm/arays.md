
### **Java Arrays 工具类全面总结**

`java.util.Arrays` 类包含用于操作数组的各种方法（比如排序和搜索）。所有方法都是静态的，可以通过类名直接调用（`Arrays.methodName(...)`）。

---

### **一、核心方法分类**

#### **1. 转换与输出 (Conversion & Output)**

*   **`toString(array)`**：将一维数组转换为字符串格式，方便打印调试。
    *   **注意**：直接打印数组 (`System.out.println(arr)`) 输出的是对象的哈希码（如 `[I@1b6d3586`），而 `Arrays.toString(arr)` 输出的是直观的内容（如 `[1, 2, 3]`）。
    *   **示例**：
        ```java
        int[] arr = {1, 2, 3, 4, 5};
        System.out.println(arr); // 输出类似 [I@1b6d3586
        System.out.println(Arrays.toString(arr)); // 输出 [1, 2, 3, 4, 5]
        ```

*   **`deepToString(Object[] array)`**：将**多维数组**（如二维数组）转换为字符串格式。`toString()` 方法对多维数组会输出哈希码，所以必须用 `deepToString`。
    *   **示例**：
        ```java
        int[][] deepArray = {{1, 2}, {3, 4}};
        System.out.println(Arrays.toString(deepArray)); // 输出 [[I@1b6d3586, [I@4554617c]
        System.out.println(Arrays.deepToString(deepArray)); // 输出 [[1, 2], [3, 4]]
        ```

#### **2. 填充数组 (Filling)**

*   **`fill(array, value)`**：将指定的值填充到数组的**每一个元素**。
*   **`fill(array, fromIndex, toIndex, value)`**：将指定的值填充到数组的**指定范围** `[fromIndex, toIndex)`。
    *   **示例**：
        ```java
        int[] arr = new int[5];
        Arrays.fill(arr, 10); // arr = [10, 10, 10, 10, 10]
        Arrays.fill(arr, 1, 4, 99); // arr = [10, 99, 99, 99, 10]
        ```

#### **3. 排序 (Sorting)**

*   **`sort(array)`**：对数组按**自然升序**进行排序。
*   **`sort(array, fromIndex, toIndex)`**：对数组的指定范围排序。
*   **`sort(array, comparator)`**：使用自定义的 `Comparator` 对象对对象数组进行排序。
    *   **实现原理**：对于基本类型，使用经过优化的**快速排序 (Dual-Pivot Quicksort)**。对于对象类型，使用**归并排序 (TimSort)**，这是一种稳定排序。
    *   **示例**：
        ```java
        int[] numbers = {5, 3, 8, 1};
        Arrays.sort(numbers); // numbers = [1, 3, 5, 8]

        String[] words = {"cat", "apple", "dog", "banana"};
        Arrays.sort(words); // 按字母顺序排序: [apple, banana, cat, dog]
        // 按字符串长度排序
        Arrays.sort(words, (a, b) -> a.length() - b.length()); // [cat, dog, apple, banana]
        ```

#### **4. 二分查找 (Searching)**

*   **`binarySearch(array, key)`**：使用**二分查找算法**在**已排序的数组**中查找指定元素。
    *   **返回值**：
        *   如果找到，返回元素的**索引**。
        *   如果没找到，返回一个**负值** `(-(insertion point) - 1)`。`插入点 (insertion point)` 是指第一个大于 key 的元素的索引，如果所有元素都小于 key，则插入点为 `array.length`。
    *   **重要前提**：**数组必须是有序的！** 如果未排序，结果是未定义的。
    *   **示例**：
        ```java
        int[] sortedArr = {10, 20, 30, 40, 50};
        int index1 = Arrays.binarySearch(sortedArr, 30); // index1 = 2 (找到)
        int index2 = Arrays.binarySearch(sortedArr, 25); // index2 = -3 (未找到。插入点应为 index=2, -(-2)-1 = -3)
        ```

#### **5. 比较 (Comparison)**

*   **`equals(array1, array2)`**：比较两个一维数组是否**相等**。相等意味着元素个数相同，且对应位置的元素也相等。
*   **`deepEquals(Object[] a, Object[] b)`**：比较两个**多维数组**是否 deeply equal（深度相等）。
    *   **示例**：
        ```java
        int[] a1 = {1, 2, 3};
        int[] a2 = {1, 2, 3};
        int[] a3 = {4, 5, 6};
        boolean isEqual = Arrays.equals(a1, a2); // true
        isEqual = Arrays.equals(a1, a3); // false

        int[][] b1 = {{1, 2}, {3, 4}};
        int[][] b2 = {{1, 2}, {3, 4}};
        boolean isDeepEqual = Arrays.deepEquals(b1, b2); // true
        ```

#### **6. 复制 (Copying)**

*   **`copyOf(originalArray, newLength)`**：复制指定的数组，并指定新的长度。如果新长度大于原数组，多余的部分会用该数据类型的**默认值**填充（如 `int` 是 `0`，`Object` 是 `null`）。
*   **`copyOfRange(originalArray, fromIndex, toIndex)`**：复制原数组的指定范围 `[fromIndex, toIndex)` 到一个新数组。
    *   **底层实现**：使用 `System.arraycopy()`，这是一个 native 方法，性能很高。
    *   **示例**：
        ```java
        int[] original = {1, 2, 3, 4, 5};
        int[] copiedFull = Arrays.copyOf(original, original.length); // [1, 2, 3, 4, 5]
        int[] copiedWithPadding = Arrays.copyOf(original, 7); // [1, 2, 3, 4, 5, 0, 0]
        int[] copiedRange = Arrays.copyOfRange(original, 1, 3); // [2, 3] (toIndex 是 exclusive)
        ```

#### **7. 转换为集合 (Conversion to Collection)**

*   **`asList(T... a)`**：返回一个由指定数组支持的**固定大小的列表** (`List<T>`)。
    *   **重要特性**：
        1.  **视图**：返回的列表是数组的一个“视图”，对列表的修改会**直接反映到原数组**上，反之亦然。
        2.  **固定大小**：这个列表不支持添加 (`add`) 或删除 (`remove`) 操作，会抛出 `UnsupportedOperationException`。但可以修改 (`set`) 元素。
    *   **常用场景**：快速初始化一个列表。
    *   **示例**：
        ```java
        String[] fruits = {"Apple", "Banana", "Orange"};
        List<String> list = Arrays.asList(fruits);
        System.out.println(list); // 输出 [Apple, Banana, Orange]

        // 修改列表会影响原数组
        list.set(0, "Mango");
        System.out.println(fruits[0]); // 输出 "Mango"

        // list.add("Pear"); // 错误！抛出 UnsupportedOperationException
        // list.remove(0); // 错误！抛出 UnsupportedOperationException
        ```

#### **8. 使用并行前缀计算 (Parallel Prefix)**

*   **`parallelPrefix(array, op)`**：使用提供的二元运算符，并行地累积计算数组中的每个元素（类似于 `Stream.reduce` 但会修改原数组并记录所有中间结果）。
    *   **示例**（计算前缀和）：
        ```java
        int[] arr = {1, 2, 3, 4};
        Arrays.parallelPrefix(arr, (left, right) -> left + right);
        System.out.println(Arrays.toString(arr)); // 输出 [1, 3, 6, 10]
        // 计算过程： [1, 1+2=3, 3+3=6, 6+4=10]
        ```

#### **9. 设置并行排序的阈值**

*   **`setAll(array, generator)`**：使用生成器函数并行地设置所有数组元素的值。
*   **`parallelSetAll(array, generator)`**：功能同 `setAll`，但是并行操作，对于大数据量性能更好。
    *   **示例**：
        ```java
        int[] squares = new int[5];
        Arrays.parallelSetAll(squares, i -> i * i);
        System.out.println(Arrays.toString(squares)); // 输出 [0, 1, 4, 9, 16]
        ```

#### **10. 转换为流 (Stream API Integration)**

*   **`stream(array)`**：将数组转换为一个 `Stream`，以便使用强大的 Java 8 Stream API 进行各种操作（过滤、映射、收集等）。
    *   **示例**：
        ```java
        int[] numbers = {1, 2, 3, 4, 5};
        int sum = Arrays.stream(numbers) // 创建 IntStream
                       .filter(n -> n % 2 == 0) // 过滤偶数
                       .sum(); // 求和
        System.out.println(sum); // 输出 6 (2+4)
        ```

---

### **二、常用操作场景速查**

| 你想做什么？ | 应该使用的方法 |
| :--- | :--- |
| **快速打印数组内容** | `Arrays.toString(arr)` 或 `Arrays.deepToString(arr)` |
| **快速初始化/重置数组** | `Arrays.fill(arr, value)` |
| **对数组排序** | `Arrays.sort(arr)` |
| **在有序数组中快速查找** | `Arrays.binarySearch(arr, key)` |
| **比较两个数组内容是否一样** | `Arrays.equals(arr1, arr2)` 或 `Arrays.deepEquals(arr1, arr2)` |
| **复制一个数组或其中一部分** | `Arrays.copyOf(arr, newLength)` 或 `Arrays.copyOfRange(arr, from, to)` |
| **快速将数组转为List（只读视图）** | `Arrays.asList(arr)` |
| **使用Stream API处理数组** | `Arrays.stream(arr)` |

---

### **三、重要注意事项**

1.  **`asList()` 的陷阱**：最常出错的地方。记住它返回的 List 大小是**固定**的，源自一个数组。
2.  **`binarySearch()` 的前提**：调用前必须确保数组是**升序排序**的，否则结果错误。
3.  **范围参数**：所有带 `fromIndex` 和 `toIndex` 参数的方法，范围都是 **`[fromIndex, toIndex)`**，即**左闭右开**。
4.  **性能**：像 `sort()`, `copyOf()` 等方法底层都经过高度优化，性能比自己手写的通用算法要好，应优先使用。