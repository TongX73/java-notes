好的！Java 中的多态是面向对象编程（OOP）的三大核心特性之一（封装、继承、多态），它极大地提升了代码的灵活性、可扩展性和可维护性。以下是关键知识点的总结笔记：

# Java 多态 (Polymorphism) 核心笔记

**核心概念：**
*   **定义：** 多态指“同一个行为（方法调用）”具有“多种不同表现形式”的能力。
*   **核心机制：** **方法重写 + 父类引用指向子类对象**。
*   **关键口诀：** **编译看左边，运行看右边。**
    *   **编译时：** 编译器检查**引用变量类型（左边）** 中是否有被调用的方法声明（方法签名）。
    *   **运行时：** JVM 实际执行的是**引用变量实际指向的对象（右边）** 的类中的方法实现（如果该方法被重写）。

## 一、实现多态的必要条件

1.  **继承：** 必须存在类之间的继承关系（`extends`）。
2.  **重写：** 子类必须对父类中的方法进行**重写**。没有重写，多态就失去了“不同表现”的意义。
3.  **向上转型：** 父类（或接口）的引用变量必须指向子类对象。
    ```java
    Parent p = new Child(); // 向上转型，这是多态的核心体现
    ```

## 二、多态的核心体现：方法调用

```java
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() { // 方法重写 (必要条件2)
        System.out.println("Dog barks: Woof!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() { // 方法重写 (必要条件2)
        System.out.println("Cat meows: Meow!");
    }
}

public class PolymorphismDemo {
    public static void main(String[] args) {
        // 向上转型 (必要条件3)
        Animal animal1 = new Dog(); // Animal引用指向Dog对象
        Animal animal2 = new Cat(); // Animal引用指向Cat对象

        // 多态方法调用 (核心体现)
        animal1.makeSound(); // 输出: Dog barks: Woof! (运行时看右边 - Dog对象)
        animal2.makeSound(); // 输出: Cat meows: Meow! (运行时看右边 - Cat对象)

        // 直接调用
        Animal realAnimal = new Animal();
        realAnimal.makeSound(); // 输出: Animal makes a sound
    }
}
```

**关键解释 (`animal1.makeSound()`):**
1.  **编译时 (`animal1.makeSound()`):** 编译器检查 `animal1` 的声明类型 `Animal`。`Animal` 类中有 `makeSound()` 方法声明，所以编译通过。
2.  **运行时 (`animal1.makeSound()`):** JVM 查看 `animal1` 实际指向的对象类型 `Dog`。`Dog` 类重写了 `makeSound()` 方法，因此执行 `Dog` 类中的 `makeSound()` 方法。

## 三、多态的类型

1.  **编译时多态 (静态多态 / 方法重载 Overload):**
    *   在**编译时**根据**方法签名（方法名 + 参数列表）** 确定调用哪个方法。
    *   发生在**同一个类**中。
    *   示例：同一个类中有多个 `add(int, int)`, `add(double, double)`, `add(String, String)` 方法。

2.  **运行时多态 (动态多态 / 方法重写 Override):**
    *   在**运行时**根据**实际对象的类型**确定调用哪个重写的方法。
    *   发生在**具有继承关系的不同类**中。
    *   **这就是我们通常所说的“Java多态”的核心。** 上面 `Animal/Dog/Cat` 的例子就是运行时多态。

## 四、多态的优点

1.  **代码可扩展性：** 添加新的子类时，无需修改基于父类编写的通用代码（例如，遍历 `Animal` 列表调用 `makeSound()`）。
2.  **代码可维护性：** 将通用的行为定义在父类，具体实现分散到子类，结构清晰。
3.  **接口灵活性：** 允许程序以统一的方式处理不同子类的对象（“一个接口，多种实现”）。
4.  **消除类型耦合：** 方法参数或集合可以声明为父类类型，实际传入或存储各种子类对象。

## 五、注意事项和要点

1.  **成员变量没有多态性：**
    *   成员变量的访问取决于**引用变量的声明类型（编译时类型）**。
    ```java
    class Parent { int x = 10; }
    class Child extends Parent { int x = 20; }

    Parent p = new Child();
    System.out.println(p.x); // 输出: 10 (看左边 - Parent的x)
    ```

2.  **静态方法没有多态性：**
    *   静态方法的调用也取决于**引用变量的声明类型（编译时类型）**。
    *   静态方法属于类，不能被重写，只能被隐藏（子类定义同名同参的静态方法）。
    ```java
    class Parent { static void method() { System.out.println("Parent static"); } }
    class Child extends Parent { static void method() { System.out.println("Child static"); } }

    Parent p = new Child();
    p.method(); // 输出: Parent static (看左边 - Parent的method)
    ```

3.  **`private`/`final`/`static` 方法不能被重写：** 因此这些方法不具有多态性。调用它们依赖于编译时类型。

4.  **向下转型 (`Downcasting`) 与 `instanceof`:**
    *   将父类引用转回成子类引用。
    *   **必须显式进行 (`(Child) parentRef`)**
    *   **存在风险：** 如果父类引用实际指向的对象不是目标子类类型（或其子类），会抛出 `ClassCastException`。
    *   **安全做法：** 转型前使用 `instanceof` 运算符检查。
    ```java
    Animal animal = new Dog();
    // 向下转型 (有风险)
    if (animal instanceof Dog) {
        Dog myDog = (Dog) animal; // 安全向下转型
        myDog.fetch(); // 调用Dog特有的方法
    }
    ```

5.  **构造器与多态：**
    *   构造器隐式是 `static` 的（虽然没有 `static` 关键字），因此不具有多态性。
    *   子类构造器调用（显式或隐式）父类构造器时，调用的是父类自身的构造器，不会调用被子类重写的父类方法（如果父类构造器调用了可被重写的方法，此时子类状态可能未完全初始化，可能导致意外行为，需谨慎）。

## 六、多态与接口

*   接口是实现多态的另一种强大方式。
*   一个类可以实现多个接口 (`implements`)。
*   接口引用变量可以指向实现了该接口的任何类的对象。
*   方法调用规则与类继承的多态完全相同：编译看接口声明，运行时看实际对象的方法实现。
*   接口多态更强调“能做什么”，而不是“是什么”，提供了更大的灵活性。

```java
interface Shape {
    double getArea();
}

class Circle implements Shape {
    @Override
    public double getArea() { /* ... */ }
}

class Rectangle implements Shape {
    @Override
    public double getArea() { /* ... */ }
}

public class InterfacePoly {
    public static void main(String[] args) {
        Shape s1 = new Circle();
        Shape s2 = new Rectangle();

        System.out.println(s1.getArea()); // 调用Circle的实现
        System.out.println(s2.getArea()); // 调用Rectangle的实现
    }
}
```

**总结图示:**

```
           Animal (父类/接口)
             |  makeSound()
            / \
           /   \
         Dog   Cat (子类/实现类)
makeSound()    makeSound()
(重写)         (重写)

Animal a = new Dog(); // 向上转型
a.makeSound(); // -> 实际执行 Dog.makeSound() (运行时多态)
```

牢记 **“编译看左边，运行看右边”** 这句口诀。