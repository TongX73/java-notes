# Java 包（Package）核心笔记

## 一、包的基本概念

### 什么是包？
- **逻辑容器**：用于组织相关的类和接口
- **命名空间**：解决类名冲突问题
- **访问控制**：提供包级别的访问权限控制

### 包的作用
1. 避免命名冲突
2. 提供访问保护
3. 更好地组织项目结构
4. 方便类的查找和使用

## 二、包的使用语法

### 包声明
```java
// 必须是文件的第一行代码（注释除外）
package com.example.project;
```

### 导入包
```java
// 导入单个类
import java.util.ArrayList;

// 导入整个包
import java.util.*;

// 静态导入（JDK 5+）
import static java.lang.Math.PI;
import static java.lang.Math.pow;
```

## 三、包与目录结构

### 关键规则
- **包名 = 目录路径**
- 包名使用**小写字母**（约定）
- 使用**逆域名**命名法（如：com.company.project）

### 目录结构示例
```
项目根目录
├── src
│   ├── com
│   │   ├── example
│   │   │   ├── model
│   │   │   │   ├── User.java    // package com.example.model;
│   │   │   ├── util
│   │   │   │   ├── StringUtils.java // package com.example.util;
```

## 四、访问权限与包

### 访问修饰符总结
| 修饰符      | 同类 | 同包 | 子类 | 不同包 |
|------------|------|------|------|--------|
| `public`   | ✔️   | ✔️   | ✔️   | ✔️     |
| `protected`| ✔️   | ✔️   | ✔️   | ❌     |
| 默认(无修饰符)| ✔️ | ✔️   | ❌   | ❌     |
| `private`  | ✔️   | ❌   | ❌   | ❌     |

> **包访问权限**：没有修饰符的类/成员只能在同包内访问

## 五、常用Java内置包

| 包名              | 主要功能                     | 常用类示例                 |
|-------------------|----------------------------|--------------------------|
| `java.lang`       | Java核心类库               | String, System, Math     |
| `java.util`       | 工具类和集合框架           | ArrayList, Scanner, Date |
| `java.io`         | 输入输出操作               | File, InputStream        |
| `java.net`        | 网络编程                   | Socket, URL              |
| `java.sql`        | 数据库操作                 | Connection, Statement    |
| `java.awt`/`javax.swing` | GUI开发                | JFrame, JButton          |

## 六、包的最佳实践

1. **命名规范**：
   - 使用公司域名倒序（如：com.google.common）
   - 全部小写字母
   - 避免使用Java保留字

2. **分包原则**：
   ```plaintext
   com
   └── company
       └── project
           ├── controller  // 控制器层
           ├── service     // 业务逻辑层
           ├── dao        // 数据访问层
           ├── model      // 数据模型
           └── util       // 工具类
   ```

3. **导入建议**：
   - 避免使用`import package.*;`（通配符导入）
   - 优先导入特定类
   - 静态导入仅用于常量或常用静态方法

## 七、CLASSPATH环境变量

### 作用：
- 告诉JVM在哪里查找类文件
- 包含目录路径和JAR文件路径

### 设置方法：
```bash
# Windows
set CLASSPATH=.;C:\myproject\bin;C:\lib\mylib.jar

# Linux/macOS
export CLASSPATH=.:/home/user/project/bin:/lib/mylib.jar
```

## 八、JAR文件（Java Archive）

### 主要特点：
- 基于ZIP格式的打包文件
- 包含类文件和相关资源
- 包含META-INF/MANIFEST.MF描述文件

### 常用命令：
```bash
# 创建JAR文件
jar cvf myapp.jar com/

# 查看JAR内容
jar tf myapp.jar

# 提取JAR文件
jar xvf myapp.jar

# 运行可执行JAR
java -jar myapp.jar
```

## 九、常见问题解决

1. **"找不到类"错误**：
   - 检查包声明是否正确
   - 确认类文件在正确目录
   - 检查CLASSPATH设置

2. **访问权限错误**：
   - 检查类/成员的访问修饰符
   - 确认是否在同一个包中

3. **命名冲突**：
   - 使用完全限定名：`java.util.Date utilDate = new java.util.Date();`
   - 避免通配符导入

## 十、包的设计原则

1. **高内聚**：包内元素高度相关
2. **低耦合**：包之间依赖最小化
3. **稳定依赖**：依赖方向指向更稳定的包
4. **稳定抽象**：稳定包应包含抽象接口

> 包是Java项目的基本组织单元，良好的包结构设计能显著提高代码的可维护性和可读性。在实际开发中，结合Maven/Gradle等构建工具可以更高效地管理包依赖。