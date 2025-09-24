
## **Java Stream API 笔记**

### **一、Stream 核心概念**

#### **1. 什么是 Stream？**
- Stream 是 Java 8 引入的新 API，用于处理集合数据的**声明式编程**工具
- 它不是数据结构，不存储数据，而是对数据源（集合、数组等）进行高效处理
- 类似于 SQL 语句，你只需要告诉它"做什么"，而不是"怎么做"

#### **2. Stream 的特点**
- **不会修改源数据**：Stream 操作不会改变原始数据源
- **惰性求值**：中间操作都是惰性的，只有终止操作才会真正执行
- **可消费性**：Stream 只能被消费一次，就像迭代器一样

#### **3. Stream 操作的三个步骤**
1. **创建 Stream**：从数据源获取流
2. **中间操作**：对数据进行过滤、映射、排序等处理（返回新的 Stream）
3. **终止操作**：产生最终结果或副作用

---

### **二、Stream 的创建方式**

#### **1. 从集合创建（最常用）**
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();           // 顺序流
Stream<String> parallelStream = list.parallelStream(); // 并行流
```

#### **2. 从数组创建**
```java
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);
Stream<String> partialStream = Arrays.stream(array, 1, 3); // [b, c]
```

#### **3. 使用 Stream.of() 静态方法**
```java
Stream<String> stream = Stream.of("a", "b", "c");
Stream<Integer> numberStream = Stream.of(1, 2, 3, 4, 5);
```

#### **4. 创建无限流**
```java
// 生成：接受 Supplier 函数接口
Stream<String> generated = Stream.generate(() -> "element").limit(10);

// 迭代：生成规律序列
Stream<Integer> iterated = Stream.iterate(0, n -> n + 2).limit(5); // 0, 2, 4, 6, 8

// 从 JDK 9 开始，iterate 可以添加谓词条件
Stream<Integer> iteratedWithCondition = Stream.iterate(0, n -> n < 10, n -> n + 1);
```

#### **5. 基本类型流（避免装箱拆箱开销）**
```java
IntStream intStream = IntStream.range(1, 5);        // 1, 2, 3, 4
IntStream closedIntStream = IntStream.rangeClosed(1, 5); // 1, 2, 3, 4, 5
DoubleStream doubleStream = DoubleStream.of(1.1, 2.2, 3.3);
```

#### **6. 其他创建方式**
```java
// 从文件创建
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.forEach(System.out::println);
}

// 从正则表达式创建
Stream<String> words = Pattern.compile(",").splitAsStream("a,b,c");
```

---

### **三、中间操作（Intermediate Operations）**

中间操作返回新的 Stream，可以进行链式调用。

#### **1. 过滤操作**
```java
List<String> list = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");

// filter：过滤满足条件的元素
List<String> nonEmpty = list.stream()
    .filter(s -> !s.isEmpty())
    .collect(Collectors.toList()); // ["abc", "bc", "efg", "abcd", "jkl"]

// distinct：去重
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3);
List<Integer> distinct = numbers.stream().distinct().collect(Collectors.toList()); // [1, 2, 3]

// limit：限制元素数量
List<Integer> limited = numbers.stream().limit(3).collect(Collectors.toList()); // [1, 2, 2]

// skip：跳过前n个元素
List<Integer> skipped = numbers.stream().skip(2).collect(Collectors.toList()); // [2, 3, 3, 3]
```

#### **2. 映射操作**
```java
List<String> words = Arrays.asList("Java", "Stream", "API");

// map：一对一的元素转换
List<Integer> wordLengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList()); // [4, 6, 3]

// flatMap：一对多的映射，将多个流合并为一个流
List<List<String>> listOfLists = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d")
);
List<String> flatList = listOfLists.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList()); // ["a", "b", "c", "d"]

// 更复杂的 flatMap 示例
List<String> sentences = Arrays.asList("Hello world", "Java Stream");
List<String> allWords = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList()); // ["Hello", "world", "Java", "Stream"]
```

#### **3. 排序操作**
```java
List<String> names = Arrays.asList("Tom", "Jerry", "Alice", "Bob");

// 自然排序
List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList()); // ["Alice", "Bob", "Jerry", "Tom"]

// 自定义排序
List<String> customSorted = names.stream()
    .sorted((s1, s2) -> s2.compareTo(s1)) // 倒序
    .collect(Collectors.toList()); // ["Tom", "Jerry", "Bob", "Alice"]

// 使用 Comparator 工具方法
List<String> lengthSorted = names.stream()
    .sorted(Comparator.comparing(String::length).thenComparing(Comparator.naturalOrder()))
    .collect(Collectors.toList()); // 先按长度，再按字母顺序
```

#### **4. 调试操作：peek**
```java
List<String> result = Stream.of("one", "two", "three", "four")
    .filter(e -> e.length() > 3)
    .peek(e -> System.out.println("Filtered value: " + e)) // 调试用，不要修改元素
    .map(String::toUpperCase)
    .peek(e -> System.out.println("Mapped value: " + e))
    .collect(Collectors.toList());
```

---

### **四、终止操作（Terminal Operations）**

终止操作会触发流的处理并返回结果。

#### **1. 匹配操作**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

boolean allEven = numbers.stream().allMatch(n -> n % 2 == 0); // false
boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0); // true  
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0); // true
```

#### **2. 查找操作**
```java
List<String> names = Arrays.asList("Tom", "Jerry", "Alice");

Optional<String> first = names.stream().findFirst(); // Optional["Tom"]
Optional<String> any = names.stream().findAny(); // 可能是任意元素

// 使用 Optional 安全地处理结果
first.ifPresent(System.out::println); // 如果存在则打印
String result = any.orElse("Default"); // 如果不存在返回默认值
```

#### **3. 规约操作（Reduction）**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 求和
int sum = numbers.stream().reduce(0, Integer::sum); // 15

// 最大值
Optional<Integer> max = numbers.stream().reduce(Integer::max); // Optional[5]

// 字符串连接
List<String> letters = Arrays.asList("a", "b", "c");
String concatenated = letters.stream().reduce("", String::concat); // "abc"

// 复杂规约：计算单词总长度
List<String> words = Arrays.asList("Java", "Stream", "API");
int totalLength = words.stream()
    .reduce(0, 
        (partialResult, word) -> partialResult + word.length(), // 累加器
        Integer::sum // 组合器，并行流时使用
    ); // 13
```

#### **4. 收集操作（Collection）**
```java
List<String> names = Arrays.asList("Tom", "Jerry", "Alice", "Bob");

// 转换为 List
List<String> list = names.stream().collect(Collectors.toList());

// 转换为 Set（自动去重）
Set<String> set = names.stream().collect(Collectors.toSet());

// 转换为特定集合
TreeSet<String> treeSet = names.stream()
    .collect(Collectors.toCollection(TreeSet::new));

// 转换为 Map
Map<String, Integer> nameLengthMap = names.stream()
    .collect(Collectors.toMap(
        Function.identity(), // key: 名字本身
        String::length       // value: 名字长度
    )); // {Tom=3, Jerry=5, Alice=5, Bob=3}

// 处理重复键
List<String> duplicateNames = Arrays.asList("Tom", "Jerry", "Tom");
Map<String, Integer> mapWithMerge = duplicateNames.stream()
    .collect(Collectors.toMap(
        Function.identity(),
        String::length,
        (existing, replacement) -> existing // 键冲突时保留现有值
    ));
```

#### **5. 统计操作**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 使用 Collectors 的统计方法
Long count = numbers.stream().collect(Collectors.counting()); // 5
Integer sum = numbers.stream().collect(Collectors.summingInt(Integer::intValue)); // 15
Double average = numbers.stream().collect(Collectors.averagingInt(Integer::intValue)); // 3.0
IntSummaryStatistics stats = numbers.stream().collect(Collectors.summarizingInt(Integer::intValue));
// stats: count=5, sum=15, min=1, max=5, average=3.0

// 使用基本类型流的统计（更高效）
IntSummaryStatistics intStats = numbers.stream().mapToInt(Integer::intValue).summaryStatistics();
```

#### **6. 分组和分区**
```java
List<String> names = Arrays.asList("Tom", "Jerry", "Alice", "Bob", "Anna");

// 分组：按首字母分组
Map<Character, List<String>> groupedByFirstLetter = names.stream()
    .collect(Collectors.groupingBy(name -> name.charAt(0)));
// {T=[Tom], J=[Jerry], A=[Alice, Anna], B=[Bob]}

// 多级分组
Map<Character, Map<Integer, List<String>>> multiLevelGroup = names.stream()
    .collect(Collectors.groupingBy(
        name -> name.charAt(0), // 第一级：按首字母
        Collectors.groupingBy(String::length) // 第二级：按长度
    ));

// 分区：按条件分为 true/false 两组
Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(name -> name.length() > 3));
// {false=[Tom, Bob], true=[Jerry, Alice, Anna]}

// 分组后进行下游收集
Map<Character, Long> countByFirstLetter = names.stream()
    .collect(Collectors.groupingBy(
        name -> name.charAt(0),
        Collectors.counting()
    )); // {T=1, J=1, A=2, B=1}

Map<Character, String> joinedNamesByLetter = names.stream()
    .collect(Collectors.groupingBy(
        name -> name.charAt(0),
        Collectors.joining(", ")
    )); // {T=Tom, J=Jerry, A=Alice, Anna, B=Bob}
```

#### **7. 遍历操作：forEach**
```java
List<String> names = Arrays.asList("Tom", "Jerry", "Alice");

// 遍历每个元素（注意：在并行流中顺序不确定）
names.stream().forEach(System.out::println);

// 保持 encounter order（如果流有定义顺序）
names.stream().forEachOrdered(System.out::println);
```

---

### **五、并行流（Parallel Streams）**

#### **1. 创建并行流**
```java
List<String> list = Arrays.asList("a", "b", "c");

// 方法1：从集合创建
Stream<String> parallelStream = list.parallelStream();

// 方法2：将顺序流转为并行流
Stream<String> parallel = list.stream().parallel();

// 转回顺序流
Stream<String> sequential = parallel.sequential();
```

#### **2. 并行流注意事项**
```java
// 错误示例：共享状态修改（线程不安全）
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> badResults = Collections.synchronizedList(new ArrayList<>());

numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .forEach(badResults::add); // 可能丢失元素或产生异常

// 正确做法：使用线程安全的收集器
List<Integer> goodResults = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

#### **3. 影响并行流性能的因素**
- 数据量：数据量太小可能不如顺序流
- 数据结构：ArrayList 比 LinkedList 更适合并行处理
- 操作成本：操作越耗时，并行收益越大
- 合并成本：规约操作的合并器不能太耗时

---

### **六、实战案例与最佳实践**

#### **案例1：数据处理管道**
```java
public class StreamProcessingExample {
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
            new Person("Alice", 25, "Engineering"),
            new Person("Bob", 30, "Sales"),
            new Person("Charlie", 28, "Engineering"),
            new Person("Diana", 35, "Marketing"),
            new Person("Eve", 22, "Engineering")
        );

        // 复杂查询：工程部门年龄大于25岁的员工，按年龄排序，提取姓名
        List<String> result = people.stream()
            .filter(p -> "Engineering".equals(p.getDepartment()))
            .filter(p -> p.getAge() > 25)
            .sorted(Comparator.comparing(Person::getAge))
            .map(Person::getName)
            .collect(Collectors.toList());
        
        System.out.println(result); // [Charlie]
    }
}

class Person {
    private String name;
    private int age;
    private String department;
    
    // 构造方法、getter、setter 省略
}
```

#### **案例2：数据统计与分析**
```java
public class DataAnalysisExample {
    public static void main(String[] args) {
        List<Transaction> transactions = Arrays.asList(
            new Transaction("USD", 1000.0),
            new Transaction("EUR", 1500.0),
            new Transaction("USD", 2000.0),
            new Transaction("GBP", 800.0),
            new Transaction("EUR", 1200.0)
        );

        // 按货币分组并统计总金额
        Map<String, Double> totalByCurrency = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getCurrency,
                Collectors.summingDouble(Transaction::getAmount)
            ));

        // 找出金额最大的交易
        Optional<Transaction> maxTransaction = transactions.stream()
            .max(Comparator.comparing(Transaction::getAmount));

        // 统计信息
        DoubleSummaryStatistics stats = transactions.stream()
            .mapToDouble(Transaction::getAmount)
            .summaryStatistics();

        System.out.println("按货币统计: " + totalByCurrency);
        System.out.println("最大交易: " + maxTransaction.orElse(null));
        System.out.println("统计信息: " + stats);
    }
}
```

#### **最佳实践**
1. **优先使用方法引用**让代码更简洁
2. **避免在流中修改外部状态**，保持无副作用
3. **合理使用并行流**，先测试再使用
4. **复杂操作适当换行**，保持可读性
5. **适时使用基本类型流**避免装箱开销

---

### **七、常见陷阱与性能优化**

#### **1. 流只能消费一次**
```java
Stream<String> stream = Stream.of("a", "b", "c");
stream.forEach(System.out::println); // 正常
// stream.forEach(System.out::println); // 抛出 IllegalStateException
```

#### **2. 无限流必须限制**
```java
// 错误：无限循环
// Stream.iterate(0, n -> n + 1).forEach(System.out::println);

// 正确：添加限制
Stream.iterate(0, n -> n + 1).limit(10).forEach(System.out::println);
```

#### **3. 避免在 filter 中修改状态**
```java
// 不好的做法：有副作用
List<String> result = new ArrayList<>();
list.stream()
    .filter(s -> {
        if (s.length() > 3) {
            result.add(s); // 副作用！
            return true;
        }
        return false;
    })
    .collect(Collectors.toList());

// 好的做法：无副作用
List<String> cleanResult = list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());
```