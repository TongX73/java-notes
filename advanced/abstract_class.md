# Java 抽象类与抽象方法核心笔记

## 一、抽象类基本概念

### 定义与特性
- **抽象类**：使用 `abstract` 修饰的类
- **核心特征**：
  - **不能被实例化**（不能创建对象）
  - **可包含抽象方法和具体方法**
  - **必须被继承**才能使用
  - 可以包含**构造方法**（供子类调用）
  - 可以定义**成员变量**（实例/静态）

### 适用场景
1. 作为多个相关类的**通用模板**
2. 定义**部分实现**，强制子类完成特定功能
3. 实现**模板方法模式**

## 二、抽象方法基本概念

### 定义与特性
- **抽象方法**：使用 `abstract` 修饰的方法
- **核心特征**：
  - **只有声明，没有实现**（无方法体）
  - 必须以分号 `;` 结束
  - **必须存在于抽象类中**（接口除外）
  - **子类必须重写**（除非子类也是抽象类）

### 语法格式
```java
// 抽象方法声明
[访问修饰符] abstract 返回类型 方法名(参数列表);
```

## 三、抽象类与抽象方法的关系

| 特性             | 抽象类 | 抽象方法 |
|------------------|--------|----------|
| `abstract`修饰符 | ✔️      | ✔️        |
| 可被继承         | ✔️      | ❌        |
| 可包含具体实现   | ✔️      | ❌        |
| 可包含构造方法   | ✔️      | ❌        |
| 可实例化         | ❌      | ❌        |

## 四、抽象类的组成元素

```java
public abstract class Animal {
    // 1. 成员变量
    protected String species;
    
    // 2. 构造方法（子类通过super调用）
    public Animal(String species) {
        this.species = species;
    }
    
    // 3. 抽象方法（子类必须实现）
    public abstract void makeSound();
    
    // 4. 具体方法（子类可直接继承或重写）
    public void breathe() {
        System.out.println(species + " is breathing");
    }
    
    // 5. 静态方法
    public static void displayType() {
        System.out.println("This is an Animal class");
    }
    
    // 6. final方法（不可重写）
    public final void live() {
        System.out.println("All animals live");
    }
}
```

## 五、继承与实现规则

### 1. 子类继承要求
- **必须实现**父类所有抽象方法
- 否则子类也必须声明为 `abstract`

### 2. 代码示例
```java
abstract class Bird extends Animal {
    public Bird() {
        super("Bird");
    }
    
    // 实现部分抽象方法
    @Override
    public void makeSound() {
        System.out.println("Chirping");
    }
    
    // 新增抽象方法
    public abstract void fly();
}

class Eagle extends Bird {
    // 必须实现所有未实现的抽象方法
    @Override
    public void fly() {
        System.out.println("Soaring high");
    }
}
```

## 六、抽象类 vs 接口（Java 8+）

| 特性                | 抽象类                     | 接口                     |
|---------------------|---------------------------|--------------------------|
| 方法实现            | 可包含具体方法             | default/static方法实现   |
| 成员变量            | 任意类型                  | 只能是常量               |
| 构造方法            | ✔️                        | ❌                       |
| 单继承/多实现       | 单继承                    | 多实现                   |
| 访问修饰符          | 无限制                    | 默认public               |
| 设计理念            | "是什么" (is-a关系)       | "能做什么" (has-a能力)   |
| 状态维护            | 可维护对象状态             | 无状态                  |

## 七、抽象类的设计优势

### 1. 模板方法模式
```java
abstract class ReportGenerator {
    // 模板方法（final防止子类修改算法结构）
    public final void generateReport() {
        collectData();
        processData();
        formatReport();
    }
    
    // 具体方法
    private void collectData() {
        System.out.println("Collecting data...");
    }
    
    // 抽象方法（子类实现）
    protected abstract void processData();
    protected abstract void formatReport();
}

class PDFReportGenerator extends ReportGenerator {
    @Override
    protected void processData() {
        System.out.println("Processing data for PDF");
    }
    
    @Override
    protected void formatReport() {
        System.out.println("Formatting PDF report");
    }
}
```

### 2. 代码复用与扩展
- **复用**：共享通用方法实现
- **扩展**：子类只需关注差异部分

### 3. 强制规范
- 确保子类实现核心功能
- 统一类体系的行为接口

## 八、使用注意事项

### 1. 构造方法调用链
```java
abstract class Shape {
    private String color;
    
    public Shape(String color) {
        this.color = color;
        System.out.println("Shape constructor");
    }
}

class Circle extends Shape {
    private double radius;
    
    public Circle(String color, double radius) {
        super(color); // 必须调用父类构造器
        this.radius = radius;
    }
}
```

### 2. 抽象方法的限制
- **不能**：private、static、final、synchronized、native
- **合法组合**：public/protected + abstract

### 3. 设计原则
- **抽象层次**：不超过2层（避免过度设计）
- **抽象粒度**：适中（3-5个抽象方法为宜）
- **命名规范**：AbstractXxx 或 BaseXxx

## 九、典型应用案例

### 1. Java集合框架
```java
AbstractList, AbstractSet, AbstractMap
```

### 2. Servlet API
```java
HttpServlet (提供service()模板方法)
```

### 3. GUI开发
```java
JComponent (Swing组件基类)
```

## 十、常见错误及解决

### 1. 尝试实例化抽象类
```java
Animal animal = new Animal(); // ❌ 编译错误
// 正确：通过具体子类实例化
Animal animal = new Dog();    // ✅
```

### 2. 未实现所有抽象方法
```java
abstract class A {
    abstract void m1();
    abstract void m2();
}

class B extends A { // ❌ 编译错误
    void m1() {}   // 缺少m2()实现
}
```

### 3. 错误的方法修饰符组合
```java
abstract class C {
    private abstract void method(); // ❌ 非法组合
    static abstract void method2(); // ❌ 非法组合
}
```

## 十一、最佳实践指南

1. **模板方法模式**：将不变行为移到抽象类
2. **钩子方法**：提供可选扩展点
   ```java
   abstract class Game {
       abstract void initialize();
       abstract void startPlay();
       abstract void endPlay();
       
       // 钩子方法（可选重写）
       boolean allowReplay() {
           return false;
       }
   }
   ```
3. **工厂方法**：抽象类中定义创建接口
   ```java
   abstract class Dialog {
       abstract Button createButton();
       
       void render() {
           Button okButton = createButton();
           okButton.onClick(/*...*/);
       }
   }
   ```

**设计哲学**：  
抽象类 = **部分实现** + **行为规范**  
抽象方法 = **契约声明** + **实现延迟**  