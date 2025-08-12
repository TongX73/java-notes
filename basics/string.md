### 整合版 String 类深度总结  

---

#### **一、基本概念**  
**定义**：`java.lang.String` 类代表字符串，Java 程序中的所有字符串字面量（如 `"abc"`）都是该类的对象。  
**核心特性**：  
- **不可变性**：对象创建后内容不可更改（修改操作均返回新对象）  
- **字符串常量池**：JVM 优化重复字符串的内存管理机制  
- **Unicode 支持**：UTF-16 编码（每个字符占 2 字节）  

---

#### **二、创建 String 对象的 5 种方式**  
| 方式                     | 代码示例                                                                 | 说明                                                                 |
|--------------------------|--------------------------------------------------------------------------|----------------------------------------------------------------------|
| **1. 字面量赋值**        | `String s1 = "Java";`                                                    | 优先使用常量池，最高效                                               |
| **2. 字符数组转换**      | `char[] chars = {'J','a','v','a'};`<br>`String s2 = new String(chars);`  | 完整数组转换                                                         |
| **3. 字符数组子集**      | `char[] data = {'A','B','C','D'};`<br>`String s3 = new String(data, 1, 2);` → `"BC"` | 指定起始位置和长度                                                 |
| **4. 字节数组转换**      | `byte[] bytes = {65,66,67};`<br>`String s4 = new String(bytes);`         | 使用平台默认编码（不推荐）                                           |
| **5. 指定编码转换**      | `byte[] utf8Bytes = ...;`<br>`String s5 = new String(utf8Bytes, StandardCharsets.UTF_8);` | **推荐方式**，避免乱码                                               |

> 📌 **弃用警示**：`new String()` 无参构造在 Java 9+ 已弃用，用 `""` 替代

---

#### **三、内存原理与常量池**  
```java
String s1 = "Java";      // 常量池创建
String s2 = "Java";      // 复用常量池（s1 == s2 → true）
String s3 = new String("Java");  // 堆中新对象（s1 == s3 → false）
```
**内存模型图示**：  
```
┌───────────┐       ┌─────────────────┐
│ 栈内存     │       │ 堆内存           │
├───────────┤       ├─────────────────┤
│ s1:0x100 ├──────►│ 常量池(0x100)    │
│ s2:0x100 ├──────┼─► "Java"         │
│ s3:0x200 ├──────►│ 普通对象(0x200)  │
└───────────┘       │   "Java"        │
                    └─────────────────┘
```
**手动入池**：  
```java
String s4 = new String("Python").intern();  // 强制放入常量池
```

---

#### **四、字符串比较**  
| 方法                     | 功能描述                          | 示例                                  | 结果  |
|--------------------------|-----------------------------------|---------------------------------------|-------|
| `==`                     | 比较**内存地址**                  | `"Java" == new String("Java")`        | false |
| `equals()`               | 比较**内容**（区分大小写）        | `"java".equals("Java")`               | false |
| `equalsIgnoreCase()`     | 比较内容（忽略大小写）            | `"java".equalsIgnoreCase("Java")`     | true  |
| `compareTo()`            | 字典序比较（返回差值）            | `"A".compareTo("B")`                  | -1    |

> ⚠️ **关键结论**：比较字符串内容必须用 `equals()` 而非 `==`

---

#### **五、核心方法大全**  
##### **1. 基础操作**  
| 方法               | 示例                          | 结果            |
|--------------------|-------------------------------|-----------------|
| `length()`         | `"Java".length()`             | 4               |
| `isEmpty()`        | `"".isEmpty()`                | true            |
| `charAt(int)`      | `"Java".charAt(1)`            | 'a'             |

##### **2. 查找操作**  
| 方法               | 示例                          | 结果            |
|--------------------|-------------------------------|-----------------|
| `indexOf()`        | `"Java".indexOf("va")`        | 2               |
| `lastIndexOf()`    | `"aabaa".lastIndexOf("aa")`   | 3               |
| `contains()`       | `"Java".contains("av")`       | true            |
| `startsWith()`     | `"Hello".startsWith("He")`    | true            |

##### **3. 截取/分割**  
| 方法               | 示例                          | 结果               |
|--------------------|-------------------------------|--------------------|
| `substring(int)`   | `"Java".substring(1)`         | "ava"              |
| `substring(s,e)`   | `"Java".substring(1,3)`       | "av"               |
| `split()`          | `"a,b,c".split(",")`          | ["a","b","c"]      |

##### **4. 修改操作**（返回新字符串）  
| 方法               | 示例                          | 结果               |
|--------------------|-------------------------------|--------------------|
| `concat()`         | `"Hello".concat("World")`     | "HelloWorld"       |
| `replace()`        | `"Java".replace('a','o')`     | "Jovo"             |
| `toUpperCase()`    | `"java".toUpperCase()`        | "JAVA"             |
| `trim()`           | `" Java ".trim()`             | "Java"             |

##### **5. 类型转换**  
| 方法               | 示例                          | 结果               |
|--------------------|-------------------------------|--------------------|
| `toCharArray()`    | `"Java".toCharArray()`        | ['J','a','v','a']  |
| `valueOf()`        | `String.valueOf(3.14)`        | "3.14"             |

---

#### **六、字符串遍历**  
```java
// 1. for循环 + charAt()
for (int i = 0; i < str.length(); i++) {
    char c = str.charAt(i);  // 依次获取每个字符
}

// 2. 增强for循环（转字符数组）
for (char c : str.toCharArray()) {
    System.out.println(c);
}

// 3. Java 8+ Lambda
str.chars().forEach(ch -> System.out.println((char)ch));
```

---

#### **七、新特性（Java 11+）**  
| 方法               | 示例                          | 结果               |
|--------------------|-------------------------------|--------------------|
| `isBlank()`        | `" \u3000 ".isBlank()`        | true（含Unicode空白）|
| `strip()`          | `" Java\u3000".strip()`       | "Java"             |
| `repeat(int)`      | `"Ab".repeat(3)`              | "AbAbAb"           |
| `lines()`          | `"A\nB".lines().count()`      | 2                  |

**文本块（Java 15+）**：  
```java
String json = """
    {
        "name": "Alice",
        "age": 28
    }
    """;
```

---

#### **八、最佳实践与陷阱规避**  
##### ✅ **推荐做法**  
1. **拼接优化**  
   ```java
   // 编译器优化为 StringBuilder
   String result = "A" + "B" + "C";  

   // 循环拼接用 StringBuilder
   StringBuilder sb = new StringBuilder();
   for (int i = 0; i < 100; i++) {
       sb.append(i);
   }
   ```

2. **敏感数据保护**  
   ```java
   char[] password = {'s','e','c','r','e','t'};
   // 使用后清除
   Arrays.fill(password, '\0'); 
   ```

##### ⚠️ **常见陷阱**  
1. **编码不一致乱码**  
   ```java
   // 错误：依赖系统默认编码
   String s = new String(bytes);  

   // 正确：显式指定编码
   String s = new String(bytes, "UTF-8");
   ```

2. **幻想修改字符串**  
   ```java
   String s = "hello";
   s.toUpperCase();  // ❌ 原字符串不变
   s = s.toUpperCase();  // ✅ 重新赋值
   ```

3. **正则特殊字符未转义**  
   ```java
   "file.txt".replaceAll(".", "-");  // ❌ 输出 "--------"
   "file.txt".replaceAll("\\.", "-"); // ✅ 输出 "file-txt"
   ```

---

### 终极总结口诀  
> 🔹 **字符串不可变，操作返回新串**  
> 🔹 **比较内容用 equals，地址比较用 ==**  
> 🔹 **拼接多用 StringBuilder，循环避免 + 号连**  
> 🔹 **编码转换需指定，常量池里性能先**