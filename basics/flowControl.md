# Java 流程控制全面解析

## 一、分支结构

### 1. if 语句

```java
// 基本形式
if (条件) {
    // 条件为 true 时执行
}

// if-else 结构
if (score >= 60) {
    System.out.println("及格");
} else {
    System.out.println("不及格");
}

// 多条件分支
if (score >= 90) {
    System.out.println("优秀");
} else if (score >= 80) {
    System.out.println("良好");
} else if (score >= 70) {
    System.out.println("中等");
} else {
    System.out.println("需努力");
}
```

### 2. switch 语句（JDK 14+ 增强）

```java
// 传统形式
int day = 3;
switch (day) {
    case 1:
        System.out.println("周一");
        break;
    case 2:
        System.out.println("周二");
        break;
    default:
        System.out.println("其他");
}

// JDK 14+ 箭头表达式
String season = switch (month) {
    case 12, 1, 2 -> "冬季";
    case 3, 4, 5 -> "春季";
    case 6, 7, 8 -> "夏季";
    case 9, 10, 11 -> "秋季";
    default -> "无效月份";
};

// yield 返回值
String description = switch (fruit) {
    case "apple" -> "苹果";
    case "banana" -> {
        String color = "黄色";
        yield color + "的香蕉"; // yield 返回值
    }
    default -> "未知水果";
};
```

## 二、循环结构

### 1. for 循环

```java
// 标准 for 循环
for (int i = 0; i < 5; i++) {
    System.out.println("第" + i + "次循环");
}

// 增强 for 循环（foreach）
String[] fruits = {"苹果", "香蕉", "橙子"};
for (String fruit : fruits) {
    System.out.println(fruit);
}

List<Book> books = getBookList();
for (Book b : books) {
    System.out.println(b.getTitle() + " - " + b.getAuthor());
}
```

### 2. while 循环

```java
// 先判断后执行
int count = 5;
while (count > 0) {
    System.out.println(count);
    count--;
}

// 输入验证循环
Scanner sc = new Scanner(System.in);
int input;
while (true) {
    System.out.print("请输入1-10之间的数: ");
    input = sc.nextInt();
    if (input >= 1 && input <= 10) {
        break;
    }
    System.out.println("输入无效，请重新输入！");
}
```

### 3. do-while 循环

```java
// 先执行后判断（至少执行一次）
int num;
do {
    System.out.print("请输入正数: ");
    num = sc.nextInt();
} while (num <= 0);
```

## 三、循环控制语句

### 1. break 与 continue

```java
// break 终止整个循环
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break; // 当 i=5 时终止循环
    }
    System.out.println(i);
}

// continue 跳过当前迭代
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) {
        continue; // 跳过偶数
    }
    System.out.println(i);
}
```

### 2. 带标签的跳转

```java
// 多层循环控制
outer: for (int i = 0; i < 3; i++) {
    inner: for (int j = 0; j < 3; j++) {
        if (i == 1 && j == 1) {
            break outer; // 跳出外层循环
        }
        System.out.println(i + "," + j);
    }
}
```

## 四、集合遍历与判断

### 1. 增强 for 循环（foreach）详解

```java
// 遍历数组
int[] numbers = {1, 2, 3, 4, 5};
for (int num : numbers) {
    System.out.print(num + " ");
}

// 遍历集合
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
for (String name : names) {
    System.out.println(name);
}

// 遍历Map
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 90);
scores.put("Bob", 85);

for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

### 2. isEmpty() 方法详解

```java
// 字符串判空
String text = "";
if (text.isEmpty()) {
    System.out.println("字符串为空");
}

// 集合判空
List<String> list = new ArrayList<>();
if (list.isEmpty()) {
    System.out.println("集合为空");
}

// Map判空
Map<String, Integer> map = new HashMap<>();
if (map.isEmpty()) {
    System.out.println("Map为空");
}

// 安全判空方法
public boolean isNullOrEmpty(String str) {
    return str == null || str.isEmpty();
}

public boolean isNullOrEmpty(Collection<?> collection) {
    return collection == null || collection.isEmpty();
}
```

### 3. 集合遍历与判空结合

```java
List<Book> books = getBooksFromDatabase();

// 安全处理空集合
if (books == null || books.isEmpty()) {
    System.out.println("没有找到书籍");
} else {
    System.out.println("找到" + books.size() + "本书:");
    
    // 使用增强for循环遍历
    for (Book b : books) {
        System.out.println("- " + b.getTitle());
    }
}
```

## 五、流程控制最佳实践

### 1. 避免嵌套过深

```java
// 不佳：嵌套过深
if (condition1) {
    if (condition2) {
        if (condition3) {
            // 代码
        }
    }
}

// 改进：提前返回
if (!condition1) return;
if (!condition2) return;
if (!condition3) return;
// 主逻辑代码
```

### 2. 循环优化技巧

```java
// 缓存集合大小（避免每次调用size()）
List<String> items = getLargeList();
int size = items.size();
for (int i = 0; i < size; i++) {
    // 处理逻辑
}

// 避免在循环内创建对象
StringBuilder result = new StringBuilder();
for (String item : items) {
    result.append(item); // 比每次创建新字符串高效
}
```

### 3. 使用 switch 表达式（JDK 14+）

```java
// 传统switch
String dayType;
switch (dayOfWeek) {
    case "MON":
    case "TUE":
    case "WED":
    case "THU":
    case "FRI":
        dayType = "工作日";
        break;
    case "SAT":
    case "SUN":
        dayType = "周末";
        break;
    default:
        dayType = "未知";
}

// JDK 14+ switch表达式
String dayType = switch (dayOfWeek) {
    case "MON", "TUE", "WED", "THU", "FRI" -> "工作日";
    case "SAT", "SUN" -> "周末";
    default -> "未知";
};
```

## 六、流程控制常见问题

### 1. 无限循环预防

```java
// 设置安全计数器
int maxAttempts = 5;
int attempts = 0;
while (true) {
    attempts++;
    if (attempts > maxAttempts) {
        System.out.println("超过最大尝试次数");
        break;
    }
    // 业务逻辑
}
```

### 2. 浮点数比较问题

```java
// 避免直接比较浮点数
double a = 0.1 + 0.2;
double b = 0.3;

// 错误方式
if (a == b) { 
    // 可能不会执行
}

// 正确方式：使用容差
final double EPSILON = 1e-10;
if (Math.abs(a - b) < EPSILON) {
    System.out.println("a ≈ b");
}
```

### 3. 增强 for 循环的限制

```java
List<String> names = new ArrayList<>(Arrays.asList("Alice", "Bob", "Charlie"));

// 错误：在遍历中删除元素
for (String name : names) {
    if (name.equals("Bob")) {
        names.remove(name); // 抛出ConcurrentModificationException
    }
}

// 正确：使用迭代器
Iterator<String> iterator = names.iterator();
while (iterator.hasNext()) {
    String name = iterator.next();
    if (name.equals("Bob")) {
        iterator.remove(); // 安全删除
    }
}
```

## 七、流程控制总结表

| 结构类型 | 关键字/方法 | 使用场景 | 注意事项 |
|---------|------------|---------|---------|
| **分支结构** | if-else | 条件判断 | 避免嵌套过深 |
|          | switch-case | 多值匹配 | 注意break穿透 |
| **循环结构** | for | 已知循环次数 | 控制变量作用域 |
|          | 增强for | 遍历集合/数组 | 不能修改集合结构 |
|          | while | 条件循环 | 防止无限循环 |
|          | do-while | 至少执行一次 | 条件在循环后检查 |
| **循环控制** | break | 终止循环 | 支持标签跳转 |
|          | continue | 跳过当前迭代 | 谨慎使用 |
| **集合操作** | isEmpty() | 检查空集合 | 需先判空再调用 |
|          | for(Item item : collection) | 简化遍历 | 只读操作安全 |
