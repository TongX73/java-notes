### StringBuilder 深度总结  

---

#### **一、基本概念**  
**定义**：`StringBuilder` 是可变的字符串容器，用于高效操作字符串。  
**核心特性**：  
- **可变性**：内容可修改（与 `String` 不可变相对）  
- **高性能**：避免频繁创建新对象，特别适合字符串拼接和修改  
- **非线程安全**：单线程环境下性能优于 `StringBuffer`  

**类继承关系**：  
```
Object
  ↳ AbstractStringBuilder
      ↳ StringBuilder
```

---

#### **二、构造方法**  
| 方法                         | 说明                          | 示例                                  |
|------------------------------|-------------------------------|---------------------------------------|
| `StringBuilder()`            | 创建初始容量 16 的空对象       | `StringBuilder sb = new StringBuilder();` |
| `StringBuilder(int capacity)`| 指定初始容量                  | `StringBuilder sb = new StringBuilder(100);` |
| `StringBuilder(String str)`  | 用指定字符串初始化            | `StringBuilder sb = new StringBuilder("Hello");` |

---

#### **三、核心操作方法**  
##### **1. 追加内容（append）**  
```java
// 支持所有基本类型和对象
sb.append("Java");       // → "Java"
sb.append(11);           // → "Java11"
sb.append('!');          // → "Java11!"
sb.append(true);         // → "Java11!true"
```

##### **2. 插入内容（insert）**  
```java
sb.insert(0, 2023);      // → "2023Java11!true"
sb.insert(4, " ");       // → "2023 Java11!true"
```

##### **3. 删除内容**  
```java
sb.delete(4, 9);         // 删除[4,9) → "202311!true"
sb.deleteCharAt(0);      // 删除首字符 → "02311!true"
```

##### **4. 替换内容（replace）**  
```java
sb.replace(3, 6, "Python"); // [3,6)替换 → "023Python!true"
```

##### **5. 反转字符串（reverse）**  
```java
sb.reverse();             // → "eurt!nohtyP320"
```

---

#### **四、信息获取方法**  
| 方法                     | 说明                     | 示例                         |
|--------------------------|--------------------------|------------------------------|
| `length()`               | 当前字符数量             | `int len = sb.length();`     |
| `charAt(int index)`      | 获取指定位置字符         | `char c = sb.charAt(3);`     |
| `indexOf(String str)`    | 查找子串首次位置         | `int pos = sb.indexOf("!");` |
| `substring(int start)`   | 截取从 start 到末尾      | `String s = sb.substring(5);`|
| `substring(s,e)`         | 截取 [s,e) 子串          | `String s = sb.substring(2,5);` |

---

#### **五、容量控制方法**  
| 方法                     | 说明                     | 示例                         |
|--------------------------|--------------------------|------------------------------|
| `capacity()`             | 返回当前容量             | `int cap = sb.capacity();`   |
| `ensureCapacity(int min)`| 确保最小容量             | `sb.ensureCapacity(100);`    |
| `setLength(int newLen)`  | 设置字符序列长度         | `sb.setLength(10);`          |
| `trimToSize()`           | 释放多余空间             | `sb.trimToSize();`           |

---

#### **六、其他实用方法**  
| 方法                     | 说明                     | 示例                         |
|--------------------------|--------------------------|------------------------------|
| `setCharAt(int index, char ch)` | 设置指定位置字符 | `sb.setCharAt(0, 'A');`      |
| `toString()`             | 转为不可变 String        | `String result = sb.toString();` |
| `codePointAt(int index)` | 获取索引处 Unicode 代码点| `int cp = sb.codePointAt(1);`|

---

### **七、最佳实践与性能优化**  
#### **1. 预分配容量（关键优化）**  
```java
// 预估最终字符串长度（避免多次扩容）
StringBuilder sb = new StringBuilder(1000); 
for (int i = 0; i < 100; i++) {
    sb.append(i);
}
```

#### **2. 链式调用**  
```java
String result = new StringBuilder()
    .append("Name: ").append(name)
    .append(", Age: ").append(age)
    .toString();
```

#### **3. 与 String 的转换**  
```java
// String → StringBuilder
StringBuilder sb = new StringBuilder(str);

// StringBuilder → String
String result = sb.toString();
```

#### **4. 重置内容**  
```java
// 方法1：创建新对象（小对象推荐）
sb = new StringBuilder();

// 方法2：setLength(0)（重用大容量对象）
sb.setLength(0);  // 保留原容量
```

---

### **八、与 String/StringBuffer 对比**  
| 特性                | StringBuilder       | StringBuffer         | String          |
|---------------------|--------------------|----------------------|-----------------|
| **可变性**          | 可变               | 可变                 | 不可变          |
| **线程安全**        | ❌ 非安全          | ✅ 安全（同步方法）   | ✅ 安全         |
| **性能**            | ⭐⭐⭐ 最高         | ⭐⭐ 中等             | ⭐ 最低         |
| **内存开销**        | 低（预分配）       | 中（同步开销）       | 高（频繁创建）  |
| **适用场景**        | 单线程字符串操作   | 多线程字符串操作     | 常量字符串      |

---

### **九、常见陷阱**  
#### **1. 线程安全问题**  
```java
// ❌ 多线程环境错误使用
StringBuilder sb = new StringBuilder();
ExecutorService executor = Executors.newFixedThreadPool(10);
for (int i = 0; i < 1000; i++) {
    executor.submit(() -> sb.append("a"));
}
// 结果可能少于 1000

// ✅ 多线程解决方案
StringBuffer safeSb = new StringBuffer();  // 同步方法保证安全
```

#### **2. 未预分配导致频繁扩容**  
```java
// ❌ 默认容量16，频繁扩容（16→34→70→106...）
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);  // 多次扩容影响性能
}

// ✅ 预分配足够空间
StringBuilder sb = new StringBuilder(20000);
```

#### **3. 误用 substring 导致内存泄漏**  
```java
StringBuilder sb = new StringBuilder(10000);
sb.append(/* 大量数据 */);
String small = sb.substring(0, 10);  // 小字符串引用大char[]

// ✅ 正确截取（创建新数组）
String safeSmall = new String(sb.substring(0, 10));
```

---

### **十、性能测试对比**  
```java
// 测试代码：拼接 10 万次字符串
long start = System.currentTimeMillis();

// 方式1：String +（最慢）
String s = "";
for (int i = 0; i < 100_000; i++) {
    s += i;  // 每次循环创建新对象
}

// 方式2：StringBuilder（最快）
StringBuilder sb = new StringBuilder(200_000);
for (int i = 0; i < 100_000; i++) {
    sb.append(i);
}
String result = sb.toString();

// 结果对比（示例）：
// String +：   约 5 秒
// StringBuilder：约 5 毫秒
```

> **性能差异可达 1000 倍！**

---

### 终极使用原则  
> 🔹 **单线程操作字符串 → 首选 StringBuilder**  
> 🔹 **循环拼接字符串 → 必须 StringBuilder**  
> 🔹 **预分配容量 → 性能提升关键**  
> 🔹 **多线程环境 → 改用 StringBuffer**