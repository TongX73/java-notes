
### **Java 泛型 (Generics) 超详细笔记**

#### **一、泛型概述：为什么需要泛型？**

**1. 核心目的：**
*   **类型安全 (Type Safety)**：在编译期检查类型是否正确，将运行时错误（ClassCastException）转变为编译期错误。
*   **消除强制类型转换 (Eliminate Casts)**：使代码更简洁、清晰，避免大量的强制类型转换。

**2. 没有泛型的问题（示例）：**
```java
// 1. 一个可以存放任何对象的容器
List list = new ArrayList();
list.add("Hello");
list.add(100); // 编译没问题，但逻辑上可能不一致

// 2. 取出时需要强制转换，容易出错
String str = (String) list.get(1); // 运行时抛出 ClassCastException!
```

**3. 使用泛型的优势（示例）：**
```java
// 1. 指定这个容器只能存放String类型
List<String> list = new ArrayList<>();
list.add("Hello");
// list.add(100); // 编译错误！编译器直接报错，提前发现问题
list.add("World");

// 2. 取出时无需强制转换
String str = list.get(0); // 安全、简洁
```

---

#### **二、泛型类 (Generic Class)**

**1. 定义：** 在类名后面声明一个或多个类型参数（如 `<T>`, `<K,V>`）。

**2. 语法：**
```java
// T, E, K, V 是常见的形式类型参数名，你可以使用任何标识符
// T - Type
// E - Element
// K - Key
// V - Value
public class ClassName<T> {
    // 在类中，T 可以作为一个类型来使用
    private T field;

    public void setField(T value) {
        this.field = value;
    }

    public T getField() {
        return this.field;
    }
}
```

**3. 使用示例：**
```java
// 定义一个简单的“盒子”泛型类
public class Box<T> {
    private T content;

    public Box(T content) {
        this.content = content;
    }

    public T getContent() {
        return content;
    }

    public void setContent(T content) {
        this.content = content;
    }
}

// 测试
public class Test {
    public static void main(String[] args) {
        // 创建一个存放String的Box
        Box<String> stringBox = new Box<>("Secret Message");
        String message = stringBox.getContent(); // 无需转换

        // 创建一个存放Integer的Box
        Box<Integer> integerBox = new Box<>(100);
        int number = integerBox.getContent(); // 自动拆箱
    }
}
```

---

#### **三、泛型接口 (Generic Interface)**

**1. 定义：** 与泛型类类似，在接口名后声明类型参数。

**2. 语法：**
```java
public interface InterfaceName<T> {
    void method(T param);
    T otherMethod();
}
```

**3. 使用示例：**
```java
// 定义一个泛型“生成器”接口
public interface Generator<T> {
    T next(); // 生成一个T类型的对象
}

// 实现泛型接口时，可以传入具体类型
public class StringGenerator implements Generator<String> {
    @Override
    public String next() {
        return "Generated String";
    }
}

// 实现泛型接口时，也可以继续保持泛型
public class BoxGenerator<T> implements Generator<Box<T>> {
    @Override
    public Box<T> next() {
        return new Box<>(null); // 实际中会根据逻辑生成Box
    }
}
```

---

#### **四、泛型方法 (Generic Method)**

**1. 定义：** 在方法上独立地声明自己的类型参数，**与所在类是否是泛型类无关**。

**2. 语法：** 类型参数放在方法返回类型之前。
```java
// <T> 是类型参数声明，表示这是一个泛型方法
// void/String/T 是返回类型
// (T t) 是参数
public <T> void methodName(T t) { ... }
public <T> T methodName(T t) { ... }
```

**3. 使用示例：**
```java
public class GenericMethodDemo {

    // 一个简单的泛型方法：打印任何类型的内容
    public <T> void printContent(T content) {
        System.out.println("Content: " + content);
    }

    // 一个更复杂的泛型方法：将数组元素转换为List
    public static <E> List<E> arrayToList(E[] array) {
        List<E> list = new ArrayList<>();
        for (E element : array) {
            list.add(element);
        }
        return list;
    }

    public static void main(String[] args) {
        GenericMethodDemo demo = new GenericMethodDemo();
        demo.printContent("Hello"); // T 被推断为 String
        demo.printContent(123);     // T 被推断为 Integer

        String[] stringArray = {"A", "B", "C"};
        Integer[] intArray = {1, 2, 3};

        // 编译器根据传入的数组类型自动推断 E 的类型
        List<String> stringList = arrayToList(stringArray); // E -> String
        List<Integer> intList = arrayToList(intArray);      // E -> Integer
    }
}
```

**关键点：**
*   泛型方法的类型参数`<T>`是在**调用方法时**被确定的，而不是在实例化类时。
*   静态方法无法使用类的泛型参数，但如果需要泛型，必须将自己定义为泛型方法。
    ```java
    public class Utils {
        // 正确：静态泛型方法
        public static <T> T doSomething(T input) { ... }

        // 错误：静态方法不能使用类的泛型T
        // public static T staticMethod(T t) { ... }
    }
    ```

---

#### **五、类型通配符 (Wildcards) `?`**

用于在**使用泛型**时表示未知类型，提供更大的灵活性。通常用于方法的参数类型。

**1. 无界通配符 `<?>`**
*   **意义**：表示可以匹配任何类型。例如 `List<?>` 是所有泛型List的父类。
*   **特点**：只能从中读取数据（读取的元素是Object类型），不能向其写入数据（`null`除外）。
*   **使用场景**：你只关心集合的操作，而不关心其元素类型。
    ```java
    // 打印任何List的所有元素
    public void printList(List<?> list) {
        for (Object elem : list) { // 读取出的元素只能是Object类型
            System.out.println(elem);
        }
        // list.add(new Object()); // 编译错误！不能添加任何元素（除了null）
        list.add(null); // 这是唯一允许的add操作
    }
    ```

**2. 上界通配符 `<? extends T>`**
*   **意义**：表示未知类型，但它是 `T` 或 `T` 的**子类**。
*   **特点**：**生产者 (Producer)**，主要用于**安全地读取**。可以读取为 `T` 类型。不能写入（`null`除外）。
    ```java
    // 接收一个包含Number或其子类(Integer, Double等)的List
    public void processNumbers(List<? extends Number> list) {
        for (Number num : list) { // 安全地读取为Number
            System.out.println(num.doubleValue());
        }
        // list.add(new Integer(1)); // 编译错误！不能添加
    }
    ```

**3. 下界通配符 `<? super T>`**
*   **意义**：表示未知类型，但它是 `T` 或 `T` 的**父类**。
*   **特点**：**消费者 (Consumer)**，主要用于**安全地写入**。可以写入 `T` 及其子类的对象。读取出的元素只能是 `Object` 类型。
    ```java
    // 接收一个包含Integer或其父类(Number, Object)的List
    public void addIntegers(List<? super Integer> list) {
        list.add(new Integer(100)); // 安全地写入Integer
        list.add(200);              // 自动装箱，同样安全
        // Number num = list.get(0); // 编译错误！读取出的类型不确定
        Object obj = list.get(0);   // 只能读取为Object
    }
    ```

**PECS 原则 (Producer-Extends, Consumer-Super)：**
*   如果你需要一个**提供**（生产）`T` 对象的泛型容器，使用 `<? extends T>`。
*   如果你需要一个**接收**（消费）`T` 对象的泛型容器，使用 `<? super T>`。

---

#### **六、类型擦除 (Type Erasure) - 重要概念**

**1. 本质：** Java 泛型是在**编译期**实现的。编译器在编译后会将所有的泛型类型信息（如 `<String>`）擦除，替换为它们的**原始类型**（Raw Type，通常是 `Object`）或**上界**，并在必要的地方插入强制类型转换。

**2. 示例：**
```java
// 编译前 (源代码)
List<String> list = new ArrayList<>();
list.add("Hi");
String s = list.get(0);

// 编译后 (实际运行的字节码等效于)
List list = new ArrayList(); // 类型参数 <String> 被擦除
list.add("Hi");
String s = (String) list.get(0); // 编译器插入了强制转换
```

**3. 带来的限制：**
*   **不能使用基本类型**作为泛型参数：`List<int> list;` 是错误的，必须使用包装类 `List<Integer>`。
*   **不能实例化类型参数**：`new T()`、`new T[...]`、`T.class` 都是错误的。
*   **不能用于静态上下文**：类的静态变量或静态方法不能引用类的类型参数。
*   **instanceof 检查**：运行时无法检查泛型类型，如 `if (list instanceof ArrayList<String>)` 是错误的。

---

#### **七、总结与对比**

| 概念 | 语法示例 | 说明 | 主要用途 |
| :--- | :--- | :--- | :--- |
| **泛型类** | `class Box<T> { ... }` | 定义类时声明类型参数 | 创建可重用的、类型安全的组件（如`ArrayList<T>`） |
| **泛型接口** | `interface GenInt<T> { ... }` | 定义接口时声明类型参数 | 定义通用的操作契约（如`Comparable<T>`） |
| **泛型方法** | `<T> T method(T t) { ... }` | 在方法上独立声明类型参数 | 编写类型安全的通用工具方法（如`Arrays.asList()`） |
| **无界通配符** | `List<?>` | 匹配任何类型 | 编写完全不依赖元素类型的通用代码 |
| **上界通配符** | `List<? extends Number>` | 匹配Number及其子类 | **从泛型集合中读取**（生产者） |
| **下界通配符** | `List<? super Integer>` | 匹配Integer及其父类 | **向泛型集合中写入**（消费者） |

**核心思想：**
泛型通过在编码阶段提供**类型约束**，极大地提高了代码的安全性、可读性和重用性。理解类型擦除是理解泛型诸多限制的关键，而掌握通配符和 PECS 原则则是写出高级、灵活泛型代码的必经之路。