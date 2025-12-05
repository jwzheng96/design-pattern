# 第零章：设计模式概述

> "设计模式是软件开发人员在软件开发过程中面临的一般问题的解决方案。"

[← 返回目录](../../README.md)

---

## 📚 本章内容

- [什么是设计模式？](#什么是设计模式)
- [为什么要学习设计模式？](#为什么要学习设计模式)
- [设计模式的分类](#设计模式的分类)
- [设计模式的六大原则](#设计模式的六大原则)
- [如何学习设计模式](#如何学习设计模式)

---

## 什么是设计模式？

### 🔰 初学者理解

想象一下，你在搭积木。有些积木组合方式特别稳固，有些组合特别灵活。久而久之，你会发现一些"经典"的组合方式，每次搭建时都会用到。

**设计模式就是软件开发中的"经典积木组合"。**

它们是经过无数开发者验证的、用于解决特定问题的代码组织方式。学会了设计模式，你就掌握了前辈们积累的智慧。

### 🚀 高级理解

设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。

**设计模式的本质是：**
- 对软件设计中普遍存在的问题所提出的解决方案
- 一种编程范式，而非具体的代码实现
- 面向对象设计原则的具体化和实际应用

### 设计模式的历史

1994年，Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides（合称 Gang of Four，简称 GoF）出版了《设计模式：可复用面向对象软件的基础》一书，首次系统性地总结了 **23 种设计模式**。

```
┌─────────────────────────────────────────────────────────────────┐
│                    设计模式的发展历程                             │
├─────────────────────────────────────────────────────────────────┤
│  1977年  Christopher Alexander 在建筑学中提出"模式"概念           │
│    ↓                                                            │
│  1987年  Kent Beck 和 Ward Cunningham 将模式思想引入软件领域       │
│    ↓                                                            │
│  1994年  GoF 出版经典著作，定义 23 种设计模式                      │
│    ↓                                                            │
│  至今    设计模式已成为软件工程的基础知识                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 为什么要学习设计模式？

### 对于初学者

1. **提高代码质量**：写出更清晰、更易维护的代码
2. **学习前人智慧**：站在巨人的肩膀上
3. **提升沟通效率**：与团队成员使用共同的"语言"
4. **面试必备**：设计模式是技术面试的常见话题

### 对于高级工程师

1. **架构设计能力**：设计模式是构建复杂系统的基石
2. **代码审查能力**：快速识别代码中的设计问题
3. **重构能力**：将混乱的代码重构为优雅的设计
4. **框架理解能力**：主流框架大量使用设计模式

### 设计模式 vs 算法

| 对比维度 | 设计模式 | 算法 |
|---------|---------|------|
| 关注点 | 代码的组织结构 | 数据的处理过程 |
| 抽象层次 | 类与对象的关系 | 具体的计算步骤 |
| 目标 | 可维护性、可扩展性 | 效率、正确性 |
| 实现方式 | 多种实现皆可 | 步骤相对固定 |

---

## 设计模式的分类

GoF 将 23 种设计模式分为三大类：

### 1. 创建型模式 (Creational Patterns)

**核心问题**：如何创建对象？

创建型模式将对象的**创建**和**使用**分离，使代码更加灵活。

```
┌─────────────────────────────────────────────────────────────┐
│                     创建型模式                               │
├─────────────────────────────────────────────────────────────┤
│  单例模式        确保一个类只有一个实例                        │
│  工厂方法模式    让子类决定创建什么对象                        │
│  抽象工厂模式    创建相关对象的家族                           │
│  建造者模式      分步骤构建复杂对象                           │
│  原型模式        通过克隆创建对象                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. 结构型模式 (Structural Patterns)

**核心问题**：如何组合类和对象？

结构型模式关注类和对象的组合，形成更大的结构。

```
┌─────────────────────────────────────────────────────────────┐
│                     结构型模式                               │
├─────────────────────────────────────────────────────────────┤
│  适配器模式      转换接口，使不兼容的类协同工作                 │
│  桥接模式        将抽象与实现分离                             │
│  组合模式        将对象组合成树形结构                         │
│  装饰器模式      动态添加功能                                 │
│  外观模式        简化复杂系统的接口                           │
│  享元模式        共享细粒度对象                              │
│  代理模式        控制对象的访问                              │
└─────────────────────────────────────────────────────────────┘
```

### 3. 行为型模式 (Behavioral Patterns)

**核心问题**：对象之间如何通信和协作？

行为型模式关注对象之间的职责分配和通信方式。

```
┌─────────────────────────────────────────────────────────────┐
│                     行为型模式                               │
├─────────────────────────────────────────────────────────────┤
│  责任链模式      让多个对象都有机会处理请求                    │
│  命令模式        将请求封装成对象                             │
│  迭代器模式      顺序访问聚合对象的元素                        │
│  中介者模式      封装对象之间的交互                           │
│  备忘录模式      保存和恢复对象状态                           │
│  观察者模式      对象状态改变时通知依赖对象                    │
│  状态模式        状态改变时改变行为                           │
│  策略模式        封装可互换的算法                             │
│  模板方法模式    定义算法骨架                                 │
│  访问者模式      为对象结构定义新操作                         │
│  解释器模式      解释语言中的句子                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 设计模式的六大原则

设计模式的基础是**面向对象设计原则**，也称为 **SOLID 原则**（加上其他重要原则）。

### 1. 单一职责原则 (Single Responsibility Principle, SRP)

> **一个类应该只有一个引起它变化的原因。**

#### 🔰 初学者理解

就像一个人专注做一件事会做得更好，一个类只负责一件事也会更容易维护。

#### 示例

```cpp
// ❌ 违反 SRP：一个类做了太多事
class Employee {
public:
    void calculatePay();      // 计算薪资
    void saveToDatabase();    // 保存到数据库
    void generateReport();    // 生成报告
};

// ✅ 遵循 SRP：每个类只做一件事
class Employee { /* 员工数据 */ };
class PayCalculator { void calculate(Employee& e); };
class EmployeeRepository { void save(Employee& e); };
class ReportGenerator { void generate(Employee& e); };
```

```python
# ❌ 违反 SRP
class Employee:
    def calculate_pay(self): pass
    def save_to_database(self): pass
    def generate_report(self): pass

# ✅ 遵循 SRP
class Employee: pass
class PayCalculator:
    def calculate(self, employee): pass
class EmployeeRepository:
    def save(self, employee): pass
class ReportGenerator:
    def generate(self, employee): pass
```

### 2. 开放封闭原则 (Open/Closed Principle, OCP)

> **软件实体应该对扩展开放，对修改封闭。**

#### 🔰 初学者理解

当需要新功能时，应该通过添加新代码来实现，而不是修改已有的代码。

#### 示例

```cpp
// ❌ 违反 OCP：添加新形状需要修改 AreaCalculator
class AreaCalculator {
public:
    double calculate(Shape* shape) {
        if (shape->type == "circle") {
            return 3.14 * shape->radius * shape->radius;
        } else if (shape->type == "rectangle") {
            return shape->width * shape->height;
        }
        // 添加新形状需要修改这里
    }
};

// ✅ 遵循 OCP：添加新形状只需创建新类
class Shape {
public:
    virtual double area() const = 0;
};

class Circle : public Shape {
public:
    double area() const override { 
        return 3.14 * radius * radius; 
    }
};

class Rectangle : public Shape {
public:
    double area() const override { 
        return width * height; 
    }
};
```

### 3. 里氏替换原则 (Liskov Substitution Principle, LSP)

> **子类对象必须能够替换其父类对象。**

#### 🔰 初学者理解

如果一个程序使用父类，那么替换成子类后，程序应该仍然正常工作。

#### 经典反例：正方形不应该继承矩形

```cpp
class Rectangle {
public:
    virtual void setWidth(int w) { width = w; }
    virtual void setHeight(int h) { height = h; }
    int area() { return width * height; }
protected:
    int width, height;
};

// ❌ 违反 LSP
class Square : public Rectangle {
public:
    void setWidth(int w) override { 
        width = height = w;  // 设置宽度时也改变高度
    }
    void setHeight(int h) override { 
        width = height = h;  // 设置高度时也改变宽度
    }
};

void testArea(Rectangle& r) {
    r.setWidth(5);
    r.setHeight(4);
    assert(r.area() == 20);  // 对于 Square 会失败！
}
```

### 4. 接口隔离原则 (Interface Segregation Principle, ISP)

> **客户端不应该被迫依赖它不使用的接口。**

#### 🔰 初学者理解

不要创建"胖接口"，要创建多个专门的小接口。

```cpp
// ❌ 违反 ISP：胖接口
class IWorker {
public:
    virtual void work() = 0;
    virtual void eat() = 0;
    virtual void sleep() = 0;
};

// 机器人不需要 eat() 和 sleep()
class Robot : public IWorker {
    void work() override { /* 工作 */ }
    void eat() override { /* 被迫实现空方法 */ }
    void sleep() override { /* 被迫实现空方法 */ }
};

// ✅ 遵循 ISP：小接口
class IWorkable { virtual void work() = 0; };
class IEatable { virtual void eat() = 0; };
class ISleepable { virtual void sleep() = 0; };

class Robot : public IWorkable {
    void work() override { /* 工作 */ }
};

class Human : public IWorkable, public IEatable, public ISleepable {
    void work() override { /* 工作 */ }
    void eat() override { /* 吃饭 */ }
    void sleep() override { /* 睡觉 */ }
};
```

### 5. 依赖倒置原则 (Dependency Inversion Principle, DIP)

> **高层模块不应该依赖低层模块，两者都应该依赖抽象。**

#### 🔰 初学者理解

不要让核心业务代码直接依赖具体的数据库、文件系统等，而是依赖抽象接口。

```cpp
// ❌ 违反 DIP：高层直接依赖低层
class MySQLDatabase {
public:
    void save(string data) { /* 保存到 MySQL */ }
};

class UserService {
private:
    MySQLDatabase db;  // 直接依赖具体实现
public:
    void saveUser(string user) {
        db.save(user);
    }
};

// ✅ 遵循 DIP：依赖抽象
class IDatabase {
public:
    virtual void save(string data) = 0;
};

class MySQLDatabase : public IDatabase {
public:
    void save(string data) override { /* 保存到 MySQL */ }
};

class UserService {
private:
    IDatabase* db;  // 依赖抽象
public:
    UserService(IDatabase* database) : db(database) {}
    void saveUser(string user) {
        db->save(user);
    }
};
```

### 6. 迪米特法则 (Law of Demeter, LoD)

> **一个对象应该对其他对象有最少的了解。**

也称为"最少知识原则"。

```cpp
// ❌ 违反 LoD：了解太多细节
class Customer {
    Wallet* wallet;
};

// 收银员直接访问钱包
void pay(Customer* c, double amount) {
    if (c->wallet->money >= amount) {
        c->wallet->money -= amount;
    }
}

// ✅ 遵循 LoD：只与直接朋友交流
class Customer {
public:
    void pay(double amount) {
        wallet.deduct(amount);
    }
};

void checkout(Customer* c, double amount) {
    c->pay(amount);  // 不需要知道钱包的存在
}
```

---

## 如何学习设计模式

### 学习路径建议

```
第一阶段：入门（1-2周）
├── 理解面向对象基础
├── 学习 SOLID 原则
└── 掌握 3-5 个常用模式
    ├── 单例模式
    ├── 工厂方法模式
    ├── 策略模式
    ├── 观察者模式
    └── 装饰器模式

第二阶段：深入（2-4周）
├── 学习剩余的设计模式
├── 理解模式之间的关系
└── 在实际项目中应用

第三阶段：精通（持续）
├── 研究框架中的设计模式应用
├── 学会模式的组合使用
└── 理解何时不使用模式
```

### 学习技巧

1. **先理解问题，再学习解决方案**
   - 每个模式都是为了解决特定问题
   - 先想想如果没有这个模式，你会怎么做

2. **动手实践**
   - 不要只看代码，要自己写
   - 尝试在不同场景下应用同一模式

3. **对比学习**
   - 相似模式放在一起比较
   - 理解它们的区别和适用场景

4. **不要过度设计**
   - 设计模式是工具，不是目标
   - 简单问题用简单方案

---

## 设计模式之间的关系

```
                    ┌─────────────┐
                    │   创建型    │
                    └──────┬──────┘
                           │
    ┌──────────────────────┼──────────────────────┐
    │                      │                      │
    ▼                      ▼                      ▼
┌─────────┐          ┌─────────┐          ┌─────────┐
│  单例   │◄─────────│ 抽象工厂 │─────────►│  原型   │
└─────────┘          └────┬────┘          └─────────┘
                          │
                          ▼
                    ┌─────────┐
                    │ 工厂方法 │
                    └────┬────┘
                         │
                         ▼
                    ┌─────────┐
                    │  建造者  │
                    └─────────┘
```

**常见的模式组合：**

- **抽象工厂 + 单例**：工厂类通常是单例
- **建造者 + 工厂方法**：建造者可以使用工厂方法创建组件
- **策略 + 工厂**：使用工厂创建策略对象
- **装饰器 + 组合**：两者都处理树形结构
- **观察者 + 中介者**：中介者可以作为观察者的实现方式

---

## 下一步

准备好开始学习了吗？让我们从创建型模式开始：

➡️ [第一章：单例模式](../01-creational/singleton/README.md)

---

[← 返回目录](../../README.md) | [下一章：单例模式 →](../01-creational/singleton/README.md)

