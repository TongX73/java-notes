# Java 接口核心笔记

## 一、接口基本概念

### 定义与特性
- **接口(Interface)**：使用 `interface` 关键字定义
- **核心特征**：
  - **完全抽象的类**（Java 8前）
  - **行为规范契约**：定义"能做什么"，不关心"如何做"
  - **多继承机制**：类可实现多个接口
  - **不能实例化**（但可声明引用变量）
  - **默认成员为 `public static final`**（常量）
  - **默认方法为 `public abstract`**（Java 8前）

### 接口 vs 抽象类
| 特性                | 接口                     | 抽象类                 |
|---------------------|--------------------------|------------------------|
| 实例化              | ❌                       | ❌                     |
| 方法实现            | Java 8+支持              | ✔️                     |
| 多继承              | ✔️（多实现）             | ❌（单继承）           |
| 构造方法            | ❌                       | ✔️                     |
| 成员变量            | 只能是常量               | 任意类型变量           |
| 设计理念            | "能做什么" (能力)        | "是什么" (类别)       |

## 二、接口组成元素（Java 8+）

### 1. 抽象方法（核心）
```java
public interface Vehicle {
    // 传统抽象方法（隐式 public abstract）
    void start();
    void stop();
}
```

### 2. 默认方法（Java 8+）
```java
public interface Vehicle {
    // 默认方法（可被实现类继承或重写）
    default void honk() {
        System.out.println("Beep beep!");
    }
}
```

### 3. 静态方法（Java 8+）
```java
public interface Vehicle {
    // 静态方法（接口直接调用）
    static void displayType() {
        System.out.println("This is a vehicle interface");
    }
}
```

### 4. 常量字段
```java
public interface Physics {
    // 常量（隐式 public static final）
    double SPEED_OF_LIGHT = 299792458; // m/s
    double GRAVITY_CONSTANT = 6.67430e-11; // m³kg⁻¹s⁻²
}
```

### 5. 私有方法（Java 9+）
```java
public interface Logger {
    // 私有方法（接口内部复用）
    private void log(String message) {
        System.out.println("[LOG] " + message);
    }
    
    default void info(String msg) {
        log("INFO: " + msg);
    }
}
```

## 三、接口的实现与使用

### 1. 类实现接口
```java
// 实现单个接口
class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car engine started");
    }
    
    @Override
    public void stop() {
        System.out.println("Car brakes applied");
    }
}

// 实现多个接口
class FlyingCar implements Vehicle, Aircraft {
    // 必须实现所有接口的抽象方法
}
```

### 2. 接口继承接口
```java
public interface Aircraft {
    void takeOff();
}

public interface AdvancedAircraft extends Aircraft {
    void autoPilot();
}

// 实现类需实现所有方法
class Drone implements AdvancedAircraft {
    public void takeOff() { ... }
    public void autoPilot() { ... }
}
```

### 3. 接口多态
```java
public class Test {
    public static void main(String[] args) {
        // 接口引用指向实现类对象
        Vehicle v = new Car();
        v.start(); // 调用Car的实现
        
        // 方法参数使用接口类型
        operateVehicle(new Bicycle());
    }
    
    static void operateVehicle(Vehicle vehicle) {
        vehicle.start();
        vehicle.stop();
    }
}
```

## 四、Java 8+ 接口新特性详解

### 1. 默认方法冲突解决
```java
interface A {
    default void show() {
        System.out.println("A");
    }
}

interface B {
    default void show() {
        System.out.println("B");
    }
}

class C implements A, B {
    // 必须重写解决冲突
    @Override
    public void show() {
        A.super.show(); // 显式调用A的默认方法
        System.out.println("C");
    }
}
```

### 2. 静态方法实用场景
```java
public interface MathOperations {
    static int add(int a, int b) {
        return a + b;
    }
    
    static int multiply(int a, int b) {
        return a * b;
    }
}

// 使用：
int sum = MathOperations.add(5, 3); // 8
```

### 3. 私有方法封装
```java
public interface DataValidator {
    private boolean isValidFormat(String input) {
        return input != null && !input.isEmpty();
    }
    
    default boolean validateEmail(String email) {
        if (!isValidFormat(email)) return false;
        return email.contains("@");
    }
}
```

## 五、接口的设计原则与最佳实践

### 1. 接口隔离原则（ISP）
```java
// 错误设计：臃肿接口
interface Worker {
    void code();
    void test();
    void deploy();
    void cook();
}

// 正确设计：拆分接口
interface Developer {
    void code();
    void test();
}

interface DevOps {
    void deploy();
}

interface Chef {
    void cook();
}
```

### 2. 面向接口编程
```java
// 服务接口
public interface PaymentService {
    void processPayment(double amount);
}

// 实现类
class CreditCardService implements PaymentService { ... }
class PayPalService implements PaymentService { ... }

// 使用接口类型
public class CheckoutSystem {
    private PaymentService paymentService;
    
    public CheckoutSystem(PaymentService service) {
        this.paymentService = service;
    }
    
    public void checkout(double total) {
        paymentService.processPayment(total);
    }
}
```

### 3. 标记接口（空接口）
```java
// 标记接口示例
public interface Serializable { 
    // 无方法，仅作为类型标记
}

public interface Cloneable {
    // 标记对象可克隆
}
```

## 六、函数式接口与Lambda

### 1. 函数式接口定义
```java
// 单一抽象方法的接口
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
    
    // 允许有默认方法
    default void showResult(int result) {
        System.out.println("Result: " + result);
    }
}
```

### 2. Lambda表达式实现
```java
public class LambdaDemo {
    public static void main(String[] args) {
        // Lambda实现函数式接口
        Calculator add = (a, b) -> a + b;
        Calculator multiply = (x, y) -> x * y;
        
        System.out.println(add.calculate(5, 3)); // 8
        multiply.showResult(multiply.calculate(4, 5)); // Result: 20
    }
}
```

## 七、接口在Java标准库中的应用

### 1. 集合框架
```java
List<String> list = new ArrayList<>();  // List接口
Set<Integer> set = new HashSet<>();    // Set接口
Map<String, Integer> map = new HashMap<>(); // Map接口
```

### 2. 并发编程
```java
Runnable task = () -> System.out.println("Running");
ExecutorService executor = Executors.newFixedThreadPool(2);
executor.submit(task);  // 提交Runnable任务

Callable<String> callable = () -> "Result";
Future<String> future = executor.submit(callable); // 提交Callable任务
```

### 3. 比较器
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
// 使用Comparator接口
Collections.sort(names, (s1, s2) -> s1.length() - s2.length());
```

## 八、接口设计陷阱与解决方案

### 1. 常量接口反模式
```java
// 反模式：使用接口仅定义常量
interface Constants {
    int MAX_SIZE = 100;
    String APP_NAME = "MyApp";
}

// 正确方案：使用final类
public final class AppConstants {
    private AppConstants() {} // 防止实例化
    
    public static final int MAX_SIZE = 100;
    public static final String APP_NAME = "MyApp";
}
```

### 2. 接口污染问题
```java
// 问题：接口包含过多不相关方法
interface FileOperations {
    void open();
    void close();
    void delete();
    void rename();
    void compress(); // 并非所有实现类都需要
}

// 解决方案：拆分为核心接口+扩展接口
interface BasicFileOperations {
    void open();
    void close();
}

interface AdvancedFileOperations {
    void compress();
    void encrypt();
}
```

### 3. 默认方法继承冲突
```java
interface A {
    default void method() { System.out.println("A"); }
}

interface B extends A {
    @Override
    default void method() { System.out.println("B"); }
}

class C implements A, B {
    // 将使用B的默认方法
}
```

## 九、接口最佳实践指南

1. **命名规范**：
   - 接口名使用形容词（`Runnable`, `Serializable`）
   - 或名词+able（`Comparable`, `Cloneable`）
   - 避免使用"I"前缀（过时做法）

2. **版本兼容性**：
   - Java 8+：优先使用默认方法扩展接口
   ```java
   public interface Legacy {
       void oldMethod();
       
       default void newMethod() {
           // 兼容实现
       }
   }
   ```

3. **文档注释**：
   ```java
   /**
    * 定义车辆基本操作规范
    */
   public interface Vehicle {
       /**
        * 启动车辆
        * @throws EngineFailureException 当引擎启动失败时抛出
        */
       void start() throws EngineFailureException;
   }
   ```

4. **组合优于继承**：
   ```java
   class AdvancedCar implements Vehicle, EntertainmentSystem, GPSNavigator {
       // 通过组合多个接口实现复杂功能
   }
   ```

## 十、接口与设计模式

### 1. 策略模式
```java
interface SortingStrategy {
    void sort(int[] array);
}

class BubbleSort implements SortingStrategy { ... }
class QuickSort implements SortingStrategy { ... }

class Sorter {
    private SortingStrategy strategy;
    
    public void setStrategy(SortingStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void executeSort(int[] data) {
        strategy.sort(data);
    }
}
```

### 2. 工厂模式
```java
interface Shape {
    void draw();
}

class Circle implements Shape { ... }
class Square implements Shape { ... }

class ShapeFactory {
    public static Shape createShape(String type) {
        return switch(type.toLowerCase()) {
            case "circle" -> new Circle();
            case "square" -> new Square();
            default -> throw new IllegalArgumentException();
        };
    }
}
```

### 3. 观察者模式
```java
interface Observer {
    void update(String message);
}

interface Subject {
    void registerObserver(Observer o);
    void notifyObservers(String message);
}

class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    
    public void registerObserver(Observer o) {
        observers.add(o);
    }
    
    public void notifyObservers(String news) {
        for (Observer o : observers) {
            o.update(news);
        }
    }
}
```

**设计哲学**：  
接口 = **能力契约** + **行为规范**  