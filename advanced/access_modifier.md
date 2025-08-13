# Java 权限修饰符核心笔记

## 一、权限修饰符概览

| 修饰符       | 同类 | 同包 | 子类 | 不同包 | 适用场景 |
|-------------|------|------|------|--------|----------|
| `public`    | ✔️   | ✔️   | ✔️   | ✔️     | 完全公开的API |
| `protected` | ✔️   | ✔️   | ✔️   | ❌     | 受保护的继承结构 |
| *默认* (无修饰符) | ✔️ | ✔️   | ❌   | ❌     | 包内可见的组件 |
| `private`   | ✔️   | ❌   | ❌   | ❌     | 内部实现细节 |

> **记忆口诀**：public > protected > 默认 > private（访问范围递减）

## 二、详细访问规则

### 1. `public` - 公共访问
- **最大访问权限**
- 可修饰：类、接口、成员变量、方法、构造器
- 不同包的类需导入后才能访问
```java
public class PublicClass {
    public int publicField;
    public void publicMethod() {}
}
```

### 2. `protected` - 受保护访问
- **核心特点**：包内可见 + 子类可见
- 可修饰：成员变量、方法、构造器（不能修饰顶级类）
- 子类访问规则：
  - 同包子类：直接访问
  - 不同包子类：通过继承访问（`super`或自身实例）
```java
public class Parent {
    protected void protectedMethod() {}
}

class Child extends Parent {
    void test() {
        protectedMethod();    // ✅ 子类直接访问
        new Parent().protectedMethod(); // ❌ 不同包时编译错误
    }
}
```

### 3. *默认* (包私有) - 包级访问
- **无关键字修饰**
- 可修饰：类、接口、成员变量、方法、构造器
- 同包内自由访问，包外完全不可见
```java
class PackagePrivateClass { // 默认访问修饰
    int packagePrivateField; // 默认修饰符
    
    void packagePrivateMethod() {}
}
```

### 4. `private` - 私有访问
- **最小访问权限**
- 可修饰：成员变量、方法、构造器（不能修饰顶级类）
- 仅本类内部可见
```java
public class BankAccount {
    private double balance; // 私有字段
    
    private void deductFee() { // 私有方法
        balance -= 5.0;
    }
    
    public void deposit(double amount) {
        balance += amount;
        deductFee(); // ✅ 同类中可访问私有方法
    }
}
```

## 三、特殊应用场景

### 1. 构造器的访问控制
```java
public class Singleton {
    private static Singleton instance;
    
    // 私有构造器防止外部实例化
    private Singleton() {}
    
    public static Singleton getInstance() {
        if(instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### 2. 方法重写时的访问权限
- **重写规则**：子类方法访问权限**不能低于**父类方法
```java
class Parent {
    protected void doSomething() {}
}

class Child extends Parent {
    // ✅ 允许扩大访问范围 (protected → public)
    @Override public void doSomething() {}
    
    // ❌ 禁止缩小访问范围 (protected → 默认)
    // @Override void doSomething() {} 
}
```

### 3. 接口成员的默认权限
- 接口方法默认`public abstract`
- 接口常量默认`public static final`
```java
public interface MyInterface {
    // 等同于 public static final int MAX = 100;
    int MAX = 100;
    
    // 等同于 public abstract void execute();
    void execute();
}
```

## 四、设计原则与最佳实践

### 1. 最小权限原则
- **核心思想**：只开放必要的访问权限
- 推荐做法：
  - 字段优先用`private`
  - 方法按需选择最小权限
  - 类默认用包私有，必要时才用`public`

### 2. 封装性实现
```java
public class Employee {
    // 私有字段保护数据
    private String name;
    private double salary;
    
    // 公共访问方法
    public String getName() {
        return name;
    }
    
    // 受保护的设置方法（仅子类/同包可修改）
    protected void setSalary(double salary) {
        if(salary > 0) this.salary = salary;
    }
}
```

### 3. 包结构设计建议
```
com
└── company
    └── project
        ├── api      // 公共API (public类)
        ├── impl     // 实现包 (默认/protected类)
        ├── internal // 内部实现 (private/包私有)
        └── util     // 工具类 (public final类)
```

## 五、常见问题解析

### 1. 跨包访问问题
```java
// PackageA
public class ClassA {
    protected void methodA() {}
}

// PackageB
import PackageA.ClassA;
public class ClassB extends ClassA {
    void test() {
        methodA();       // ✅ 子类访问protected方法
        new ClassA().methodA(); // ❌ 编译错误（不同包的非子类访问）
    }
}
```

### 2. 继承时的权限冲突
```java
class Parent {
    void doWork() {} // 默认访问
}

public class Child extends Parent {
    // ❌ 编译错误：不能降低访问权限
    // private void doWork() {} 
    
    // ✅ 正确：保持或扩大权限
    public void doWork() {} 
}
```

### 3. 内部类的特殊权限
```java
public class Outer {
    private int privateField;
    
    public class Inner {
        void access() {
            privateField = 10; // ✅ 内部类可访问外部类私有成员
        }
    }
}
```

## 六、权限修饰符对比表

| 特性                | public | protected | 默认 | private |
|--------------------|--------|-----------|------|---------|
| 修饰类              | ✔️     | ❌        | ✔️   | ❌      |
| 修饰接口成员        | ✔️     | ❌        | ❌   | ❌      |
| 跨包访问            | ✔️     | ❌        | ❌   | ❌      |
| 子类访问(同包)      | ✔️     | ✔️        | ✔️   | ❌      |
| 子类访问(不同包)    | ✔️     | ✔️        | ❌   | ❌      |
| 同包非子类访问      | ✔️     | ✔️        | ✔️   | ❌      |

## 七、实战应用技巧

1. **API设计**：
   - 公共API：`public`
   - 扩展点：`protected`
   - 内部实现：`private`或默认

2. **测试支持**：
   ```java
   // 生产代码
   class PaymentProcessor {
       // 包私有方法便于测试
       boolean validateCard(String cardNo) {
           // 验证逻辑
       }
   }
   
   // 测试代码 (同包)
   class PaymentProcessorTest {
       @Test void testValidateCard() {
           PaymentProcessor p = new PaymentProcessor();
           assertTrue(p.validateCard("4111111111111111"));
       }
   }
   ```

3. **框架开发**：
   ```java
   public abstract class BaseController {
       // 子类必须实现的模板方法
       protected abstract void processRequest();
       
       // 内部辅助方法
       private void logRequest() {...}
   }
   ```

**总结原则**：  
**公有最小化，私有最大化**  
**包内协作用默认，继承扩展用保护**  # Java 权限修饰符核心笔记

## 一、权限修饰符概览

| 修饰符       | 同类 | 同包 | 子类 | 不同包 | 适用场景 |
|-------------|------|------|------|--------|----------|
| `public`    | ✔️   | ✔️   | ✔️   | ✔️     | 完全公开的API |
| `protected` | ✔️   | ✔️   | ✔️   | ❌     | 受保护的继承结构 |
| *默认* (无修饰符) | ✔️ | ✔️   | ❌   | ❌     | 包内可见的组件 |
| `private`   | ✔️   | ❌   | ❌   | ❌     | 内部实现细节 |

> **记忆口诀**：public > protected > 默认 > private（访问范围递减）

## 二、详细访问规则

### 1. `public` - 公共访问
- **最大访问权限**
- 可修饰：类、接口、成员变量、方法、构造器
- 不同包的类需导入后才能访问
```java
public class PublicClass {
    public int publicField;
    public void publicMethod() {}
}
```

### 2. `protected` - 受保护访问
- **核心特点**：包内可见 + 子类可见
- 可修饰：成员变量、方法、构造器（不能修饰顶级类）
- 子类访问规则：
  - 同包子类：直接访问
  - 不同包子类：通过继承访问（`super`或自身实例）
```java
public class Parent {
    protected void protectedMethod() {}
}

class Child extends Parent {
    void test() {
        protectedMethod();    // ✅ 子类直接访问
        new Parent().protectedMethod(); // ❌ 不同包时编译错误
    }
}
```

### 3. *默认* (包私有) - 包级访问
- **无关键字修饰**
- 可修饰：类、接口、成员变量、方法、构造器
- 同包内自由访问，包外完全不可见
```java
class PackagePrivateClass { // 默认访问修饰
    int packagePrivateField; // 默认修饰符
    
    void packagePrivateMethod() {}
}
```

### 4. `private` - 私有访问
- **最小访问权限**
- 可修饰：成员变量、方法、构造器（不能修饰顶级类）
- 仅本类内部可见
```java
public class BankAccount {
    private double balance; // 私有字段
    
    private void deductFee() { // 私有方法
        balance -= 5.0;
    }
    
    public void deposit(double amount) {
        balance += amount;
        deductFee(); // ✅ 同类中可访问私有方法
    }
}
```

## 三、特殊应用场景

### 1. 构造器的访问控制
```java
public class Singleton {
    private static Singleton instance;
    
    // 私有构造器防止外部实例化
    private Singleton() {}
    
    public static Singleton getInstance() {
        if(instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### 2. 方法重写时的访问权限
- **重写规则**：子类方法访问权限**不能低于**父类方法
```java
class Parent {
    protected void doSomething() {}
}

class Child extends Parent {
    // ✅ 允许扩大访问范围 (protected → public)
    @Override public void doSomething() {}
    
    // ❌ 禁止缩小访问范围 (protected → 默认)
    // @Override void doSomething() {} 
}
```

### 3. 接口成员的默认权限
- 接口方法默认`public abstract`
- 接口常量默认`public static final`
```java
public interface MyInterface {
    // 等同于 public static final int MAX = 100;
    int MAX = 100;
    
    // 等同于 public abstract void execute();
    void execute();
}
```

## 四、设计原则与最佳实践

### 1. 最小权限原则
- **核心思想**：只开放必要的访问权限
- 推荐做法：
  - 字段优先用`private`
  - 方法按需选择最小权限
  - 类默认用包私有，必要时才用`public`

### 2. 封装性实现
```java
public class Employee {
    // 私有字段保护数据
    private String name;
    private double salary;
    
    // 公共访问方法
    public String getName() {
        return name;
    }
    
    // 受保护的设置方法（仅子类/同包可修改）
    protected void setSalary(double salary) {
        if(salary > 0) this.salary = salary;
    }
}
```

### 3. 包结构设计建议
```
com
└── company
    └── project
        ├── api      // 公共API (public类)
        ├── impl     // 实现包 (默认/protected类)
        ├── internal // 内部实现 (private/包私有)
        └── util     // 工具类 (public final类)
```

## 五、常见问题解析

### 1. 跨包访问问题
```java
// PackageA
public class ClassA {
    protected void methodA() {}
}

// PackageB
import PackageA.ClassA;
public class ClassB extends ClassA {
    void test() {
        methodA();       // ✅ 子类访问protected方法
        new ClassA().methodA(); // ❌ 编译错误（不同包的非子类访问）
    }
}
```

### 2. 继承时的权限冲突
```java
class Parent {
    void doWork() {} // 默认访问
}

public class Child extends Parent {
    // ❌ 编译错误：不能降低访问权限
    // private void doWork() {} 
    
    // ✅ 正确：保持或扩大权限
    public void doWork() {} 
}
```

### 3. 内部类的特殊权限
```java
public class Outer {
    private int privateField;
    
    public class Inner {
        void access() {
            privateField = 10; // ✅ 内部类可访问外部类私有成员
        }
    }
}
```

## 六、权限修饰符对比表

| 特性                | public | protected | 默认 | private |
|--------------------|--------|-----------|------|---------|
| 修饰类              | ✔️     | ❌        | ✔️   | ❌      |
| 修饰接口成员        | ✔️     | ❌        | ❌   | ❌      |
| 跨包访问            | ✔️     | ❌        | ❌   | ❌      |
| 子类访问(同包)      | ✔️     | ✔️        | ✔️   | ❌      |
| 子类访问(不同包)    | ✔️     | ✔️        | ❌   | ❌      |
| 同包非子类访问      | ✔️     | ✔️        | ✔️   | ❌      |

## 七、实战应用技巧

1. **API设计**：
   - 公共API：`public`
   - 扩展点：`protected`
   - 内部实现：`private`或默认

2. **测试支持**：
   ```java
   // 生产代码
   class PaymentProcessor {
       // 包私有方法便于测试
       boolean validateCard(String cardNo) {
           // 验证逻辑
       }
   }
   
   // 测试代码 (同包)
   class PaymentProcessorTest {
       @Test void testValidateCard() {
           PaymentProcessor p = new PaymentProcessor();
           assertTrue(p.validateCard("4111111111111111"));
       }
   }
   ```

3. **框架开发**：
   ```java
   public abstract class BaseController {
       // 子类必须实现的模板方法
       protected abstract void processRequest();
       
       // 内部辅助方法
       private void logRequest() {...}
   }
   ```

**总结原则**：  
**公有最小化，私有最大化**  
**包内协作用默认，继承扩展用保护**  