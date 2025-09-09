
### **ArrayList vs. LinkedList 源码分析与总结**

#### **一、核心概览**

| 特性 | ArrayList | LinkedList |
| :--- | :--- | :--- |
| **底层数据结构** | **动态数组** (`Object[] elementData`) | **双向链表** (`Node<E> first`, `Node<E> last`) |
| **核心特性** | 动态扩容的数组 | 每个元素都是一个节点，通过指针连接 |
| **随机访问效率** | **极高 (O(1))**。直接通过索引 `(index)` 计算内存偏移量。 | **极低 (O(n))**。需要从头部或尾部遍历到指定索引。 |
| **头部插入/删除** | **低 (O(n))**。需要移动后续所有元素。 | **极高 (O(1))**。只需修改指针。 |
| **尾部插入/删除** | **高 (O(1))** *。在未扩容的情况下，直接赋值。扩容时则为 O(n)。 | **极高 (O(1))**。只需修改指针。 |
| **中间插入/删除** | **低 (O(n))**。需要移动后续所有元素。 | **平均 O(n)**。需要遍历找到位置，但修改本身是 O(1)。 |
| **内存占用** | 较小。只存储元素本身和数组对象开销。会有**预留容量**。 | 较大。每个元素 (`Node`) 都消耗额外内存存储前后节点的指针。 |
| **数据局部性** | **好**。数组元素在内存中是连续存储的，利于CPU缓存。 | **差**。链表节点在内存中是分散的，容易导致缓存未命中。 |

** * 注解**: `ArrayList` 的 `add(E e)` (尾部添加) 摊销（Amortized）时间复杂度为 O(1)。虽然偶尔会发生耗时的扩容，但均摊到每次操作上，成本是常量级的。

---

#### **二、源码深度解析**

##### **1. ArrayList 源码要点**

**a. 核心成员变量**
```java
// 存储元素的底层数组
transient Object[] elementData; // 非private，方便内部类访问

// 集合中实际元素的个数
private int size;
```
*   `elementData.length` 是数组的**容量**，总是 >= `size`。
*   `transient` 关键字修饰是因为 ArrayList 有自己的自定义序列化逻辑（只序列化实际元素，不序列化整个空数组）。

**b. 构造方法**
*   `new ArrayList()`: 初始 `elementData` 是一个**空数组** `{}`。在第一次 `add()` 时才会扩容到**默认容量 10**。 (**懒加载**优化，避免创建后不使用的空间浪费)。
*   `new ArrayList(int initialCapacity)`: 直接创建指定容量的数组。
*   `new ArrayList(Collection<? extends E> c)`: 将集合`c`toArray()`后的数据拷贝给`elementData`。

**c. 扩容机制 (核心)**
这是 `ArrayList` 性能的关键。发生在 `add()` 操作时，如果 `size + 1 > elementData.length`。

```java
// 这是内部方法，确保数组至少有minCapacity的容量
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 如果是无参构造创建的空数组，取默认容量10和所需容量的最大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++; // 结构性修改计数器，用于迭代器快速失败机制
    if (minCapacity - elementData.length > 0)
        grow(minCapacity); // 真正扩容的方法
}

private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); // 新容量 = 旧容量 * 1.5
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity); // 创建新数组并拷贝数据
}
```
*   **关键点**: 每次扩容约 **1.5 倍**。`Arrays.copyOf()` 是耗时的根源，因为它涉及创建新数组和大量数据拷贝。

**d. 插入与删除**
*   `add(int index, E element)`: 使用 `System.arraycopy()` 将 `index` 后面的所有元素向后移动一位，然后在空位插入。**拷贝操作是 O(n)**。
*   `remove(int index)`: 同样，使用 `System.arraycopy()` 将 `index` 后面的元素向前移动一位。**拷贝操作是 O(n)**。

**e. 序列化**
*   自定义了 `writeObject` 和 `readObject` 方法，只序列化实际存储的元素 (`size` 个)，而不是整个 `elementData` 数组，节省空间和流量。

---

##### **2. LinkedList 源码要点**

**a. 核心数据结构 - 节点 (Node)**
```java
private static class Node<E> {
    E item;        // 存储的元素数据
    Node<E> next;  // 指向下一个节点的引用
    Node<E> prev;  // 指向上一个节点的引用
    // 构造方法...
}
```
*   每个元素都被封装在一个 `Node` 对象中，这就是它内存开销更大的原因。

**b. 核心成员变量**
```java
transient int size = 0;    // 元素个数
transient Node<E> first;   // 指向第一个节点的指针
transient Node<E> last;    // 指向最后一个节点的指针
```

**c. 插入操作 (核心)**
*   `linkFirst(E e)`, `linkLast(E e)`: 在头部或尾部插入新节点，只需修改几个指针，**时间复杂度是 O(1)**。
    ```java
    // 在尾部插入的核心逻辑
    void linkLast(E e) {
        final Node<E> l = last;        // 记录原尾节点
        final Node<E> newNode = new Node<>(l, e, null); // 新节点的prev指向原尾节点
        last = newNode;                // 更新尾指针指向新节点
        if (l == null)                 // 如果原链表为空
            first = newNode;           // 新节点也是头节点
        else
            l.next = newNode;          // 原尾节点的next指向新节点
        size++;                        // 大小增加
        modCount++;                    // 修改计数增加
    }
    ```
*   `add(int index, E element)`: 先**遍历**找到索引位置的节点 (`node(index)` 方法，效率 O(n))，然后在其前面插入新节点（修改指针，效率 O(1)）。**综合复杂度为 O(n)**。

**d. 删除操作**
*   同理，`removeFirst()`, `removeLast()` 是 O(1)。
*   `remove(int index)` 需要先遍历找到节点 (O(n))，再修改指针断开连接 (O(1))。

**e. 获取操作**
*   `get(int index)`: 使用 `node(int index)` 方法遍历链表。
    ```java
    // 优化：索引在前半部分从头部开始找，在后半部分从尾部开始找
    Node<E> node(int index) {
        if (index < (size >> 1)) { // 索引小于size的一半
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else { // 索引大于等于size的一半
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
    ```
    尽管有优化，但时间复杂度仍然是 **O(n)**。

**f. 它还是 Deque**
*   `LinkedList` 实现了 `Deque` 接口，所以它天然拥有双端队列的方法，如 `offer()`, `poll()`, `peek()`, `push()`, `pop()` 等。**可以用作栈或队列**。

---

#### **三、实战选择指南**

**毫不犹豫选择 `ArrayList` 的场景 ( > 90% )：**
*   **频繁按索引访问元素**（`get`, `set`）。
*   主要进行**尾部添加** (`add(E e)`) 和删除操作。
*   元素数量可以大致预估，通过构造方法指定初始容量**避免频繁扩容**。
*   追求更小的内存占用和更好的遍历性能（CPU缓存友好）。

**考虑选择 `LinkedList` 的场景：**
*   需要频繁在**集合头部或中间**进行**插入和删除**操作。例如实现一个**队列 (Queue)** 或 **栈 (Stack)**（虽然 `ArrayDeque` 通常比它更好）。
*   无法预知数据量，且非常担心 `ArrayList` 扩容带来的性能损耗。
*   应用程序对内存使用不敏感。

**一个重要的误区纠正：**
> “`ArrayList` 查询快，增删慢；`LinkedList` 增删快，查询慢。”
这个说法**不够准确**。
*   `LinkedList` 只有在**头部增删**才是真正的“快”(O(1))。如果是根据索引在中间进行增删，因为需要先遍历找到位置 (O(n))，其**综合效率可能比 `ArrayList` 更差**（`ArrayList` 的移动是内存连续的块操作，而链表的遍历是离散的指针跳转）。

#### **四、总结**

| 方面 | ArrayList | LinkedList |
| :--- | :--- | :--- |
| **本质** | **可扩容的动态数组** | **双向链表** |
| **核心操作成本** | **扩容 (拷贝数据)** 和 **中间增删 (移动数据)** | **按索引查找 (遍历)** |
| **迭代器性能** | **非常高**，顺序访问数组，CPU缓存友好。 | 高，但每次 `next()` 是指针跳转，不如数组连续。 |
| **使用场景** | **高效的“随机访问”列表** | **高效的频繁头尾操作的数据结构**（如队列、栈） |

**最终建议：**
**默认首选 `ArrayList`**。它对于绝大多数场景都是最高效、最节省内存的选择。只有在你有非常明确的、频繁在非尾部位置进行增删的需求，并且经过测试证明 `ArrayList`  indeed 成为性能瓶颈时，才去考虑使用 `LinkedList`。