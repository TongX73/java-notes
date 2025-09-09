
### **Java Lambda 表达式全面总结**

#### **一、核心思想：是什么？为什么？**

*   **是什么？**
    Lambda 表达式是一个**可传递的匿名函数**。它没有名称，但有参数列表、函数体和返回类型。
*   **为什么？**
    1.  **简洁 (Conciseness)**：极大地简化了匿名内部类的语法。
    2.  **函数式编程 (Functional Programming)**：允许将行为（函数）作为参数传递给方法，使代码更灵活、更易于表达。
    3.  **并行处理能力**：为 Java 引入的 Stream API 和并行计算提供了基础。

#### **二、语法结构 (Syntax)**

Lambda 表达式的基本语法如下：

```java
(parameters) -> expression
```
或
```java
(parameters) -> { statements; }
```

*   **`(parameters)`**：参数列表。参数类型可以**显式声明**，也可以由编译器**从上下文中推断**（通常可以省略类型）。如果没有参数，必须使用空括号 `()`。如果只有一个参数且类型可推断，括号 `()` 可以省略。
*   **`->`**：箭头操作符 (Arrow Token)，将参数列表和 Lambda 主体分开。
*   **`expression`**：如果 Lambda 主体只有一条表达式，它会计算该表达式并自动返回其结果。不需要 `return` 语句。
*   **`{ statements; }`**：如果 Lambda 主体包含多条语句，必须用花括号 `{}` 包围，并且如果需要返回值，必须显式使用 `return` 语句。

#### **三、核心概念：函数式接口 (Functional Interface)**

Lambda 表达式的类型是什么？它是一个**函数式接口**的实例。

*   **定义**：函数式接口是**只有一个抽象方法**的接口。
*   **注解**：可以使用 `@FunctionalInterface` 注解来标记一个接口。这虽然不是必须的，但它可以让编译器检查该接口是否确实只有一个抽象方法。
*   **常见函数式接口**（位于 `java.util.function` 包）：
    *   **`Supplier<T>`**：无参数，返回一个结果。 `T get()`
    *   **`Consumer<T>`**：接受一个参数，无返回。 `void accept(T t)`
    *   **`Function<T, R>`**：接受一个参数，产生一个结果。 `R apply(T t)`
    *   **`Predicate<T>`**：接受一个参数，返回一个布尔值。 `boolean test(T t)`
    *   **`Runnable`**：无参数，无返回。 `void run()`

**Lambda 的本质**：`(parameters) -> expression` 就是实现了函数式接口中那个唯一的抽象方法。

#### **四、从匿名类到 Lambda (Evolution)**

这是理解 Lambda 威力的最佳方式。

**场景**：创建一个线程并运行。

1.  **传统匿名内部类方式**：
    ```java
    new Thread(new Runnable() {
        @Override
        public void run() { // 方法名、参数列表、返回类型都是冗余信息
            System.out.println("Hello from anonymous class");
        }
    }).start();
    ```

2.  **Lambda 表达式方式**：
    ```java
    new Thread(() -> System.out.println("Hello from Lambda")).start();
    ```
    *   `Runnable` 接口只有一个抽象方法 `run()`。
    *   `run()` 方法没有参数 `()`，也没有返回值 `void`。
    *   Lambda `() -> ...` 正好完美匹配了这个方法签名。
    *   所有冗余信息（`new Runnable()`, `@Override`, `public void run()`）都被省略了，只留下了最核心的逻辑。

#### **五、方法引用 (Method References)**

它是 Lambda 表达式的一种更简洁的写法，用于直接调用已有的方法。语法是 `目标引用::方法名`。

*   **四种主要形式**：
    1.  **引用静态方法**：`ClassName::staticMethod`
        ```java
        // Lambda
        Function<String, Integer> parser1 = s -> Integer.parseInt(s);
        // 方法引用 (更简洁)
        Function<String, Integer> parser2 = Integer::parseInt;
        ```
    2.  **引用特定对象的实例方法**：`objectInstance::instanceMethod`
        ```java
        // Lambda
        Consumer<String> printer1 = s -> System.out.println(s);
        // 方法引用
        Consumer<String> printer2 = System.out::println;
        ```
    3.  **引用特定类型的任意对象的实例方法**：`ClassName::instanceMethod`
        ```java
        // Lambda：比较两个字符串长度
        Comparator<String> comp1 = (a, b) -> a.compareToIgnoreCase(b);
        // 方法引用
        Comparator<String> comp2 = String::compareToIgnoreCase;
        ```
    4.  **引用构造方法**：`ClassName::new`
        ```java
        // Lambda
        Supplier<List<String>> supplier1 = () -> new ArrayList<>();
        // 方法引用
        Supplier<List<String>> supplier2 = ArrayList::new;
        ```

#### **六、变量作用域 (Variable Scope)**

Lambda 表达式可以捕获外部作用域的变量，但有一个重要限制：** effectively final**。

*   **effectively final**：一个变量如果在初始化后其值从未被改变，那么它就被认为是“事实上的最终变量”。
*   **规则**：Lambda 表达式只能捕获**局部变量**（或参数），并且这些变量必须是 **effectively final** 的。
*   **原因**：Lambda 通常是在另一个线程中执行的，为了保证数据一致性，它捕获的必须是不可变的副本。实例变量则不受此限制，因为它们存储在堆中，其访问由 `this` 引用控制。

```java
String greeting = "Hello, "; // effectively final 变量
// greeting = "Hi, ";       // 如果这行取消注释，下面一行会编译错误
Consumer<String> consumer = name -> System.out.println(greeting + name);
consumer.accept("Alice"); // 输出 "Hello, Alice"
```

#### **七、实战示例 (Practical Examples)**

1.  **遍历集合 (forEach)**：
    ```java
    List<String> list = Arrays.asList("a", "b", "c");
    // 传统 for-loop
    for (String s : list) { System.out.println(s); }
    // Lambda + forEach
    list.forEach(s -> System.out.println(s));
    // 方法引用
    list.forEach(System.out::println);
    ```

2.  **线程 (Thread)**：
    ```java
    // 之前已演示
    new Thread(() -> {...}).start();
    ```

3.  **排序 (Sorting)**：
    ```java
    List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
    // 传统匿名Comparator
    Collections.sort(names, new Comparator<String>() {
        @Override
        public int compare(String a, String b) {
            return a.compareTo(b);
        }
    });
    // Lambda
    Collections.sort(names, (a, b) -> a.compareTo(b));
    // 或者使用 List.sort
    names.sort((a, b) -> a.compareTo(b));
    // 方法引用
    names.sort(String::compareTo);
    ```

4.  **配合 Stream API (Filter & Map)**：
    ```java
    List<String> languages = Arrays.asList("java", "python", "c", "javascript");
    // 过滤出长度大于3的字符串，并将其转为大写
    List<String> filtered = languages.stream()
                                    .filter(s -> s.length() > 3)       // Predicate
                                    .map(String::toUpperCase)          // Function
                                    .collect(Collectors.toList());     // 收集为List
    // filtered = ["JAVA", "JAVASCRIPT"]
    ```

#### **八、总结与最佳实践**

| 特性 | 描述 | 优点 |
| :--- | :--- | :--- |
| **简洁性** | 省略了匿名类的模板代码 | 代码更短，更易读 |
| **函数作为参数** | 可以将行为传递给方法 | 代码更灵活，更具表现力 |
| **并行友好** | 为 Stream API 提供基础 | 更容易编写并行代码 |

**何时使用 Lambda？**
*   当你需要创建函数式接口的实例时，优先使用 Lambda。
*   尤其是在处理集合、线程、事件监听器以及使用 Stream API 时。

**注意事项：**
*   **可读性**：有时过于复杂的 Lambda 反而会降低可读性。如果逻辑很长，或许传统的命名方法或匿名类更合适。
*   **调试**：Lambda 表达式的堆栈跟踪可能比普通方法更复杂。