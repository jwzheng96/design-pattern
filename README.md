# 🎨 设计模式完全指南

> 一本面向 C++ 和 Python 开发者的设计模式权威教程

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

## 📖 关于本书

本书是一本系统性介绍 **23 种经典设计模式** 的完整教程，适合从初学者到高级工程师的所有开发者。每个设计模式都包含：

- 🎯 **核心概念**：用通俗易懂的语言解释模式的本质
- 📊 **UML 类图**：可视化展示模式结构
- 💻 **双语代码**：C++ 和 Python 的完整实现
- 🔰 **初学者指南**：循序渐进的入门讲解
- 🚀 **高级应用**：深入探讨高级用法和最佳实践
- ⚠️ **常见陷阱**：避免常见错误
- 🌍 **实战案例**：真实世界中的应用场景

---

## 📚 目录

### 第零章：引言
- [设计模式概述](chapters/00-introduction/README.md)

### 第一部分：创建型模式 (Creational Patterns)

创建型模式关注**对象的创建机制**，将对象的创建与使用分离。

| 模式 | 难度 | 简介 |
|------|------|------|
| [单例模式 (Singleton)](chapters/01-creational/singleton/README.md) | ⭐ | 确保一个类只有一个实例 |
| [工厂方法模式 (Factory Method)](chapters/01-creational/factory-method/README.md) | ⭐⭐ | 定义创建对象的接口，让子类决定实例化哪个类 |
| [抽象工厂模式 (Abstract Factory)](chapters/01-creational/abstract-factory/README.md) | ⭐⭐⭐ | 创建相关对象的家族，而无需指定具体类 |
| [建造者模式 (Builder)](chapters/01-creational/builder/README.md) | ⭐⭐ | 分步骤构建复杂对象 |
| [原型模式 (Prototype)](chapters/01-creational/prototype/README.md) | ⭐⭐ | 通过复制现有对象来创建新对象 |

### 第二部分：结构型模式 (Structural Patterns)

结构型模式关注**类和对象的组合**，形成更大的结构。

| 模式 | 难度 | 简介 |
|------|------|------|
| [适配器模式 (Adapter)](chapters/02-structural/adapter/README.md) | ⭐⭐ | 让不兼容的接口能够协同工作 |
| [桥接模式 (Bridge)](chapters/02-structural/bridge/README.md) | ⭐⭐⭐ | 将抽象与实现分离，使两者可以独立变化 |
| [组合模式 (Composite)](chapters/02-structural/composite/README.md) | ⭐⭐ | 将对象组合成树形结构，统一处理单个对象和组合对象 |
| [装饰器模式 (Decorator)](chapters/02-structural/decorator/README.md) | ⭐⭐ | 动态地给对象添加额外职责 |
| [外观模式 (Facade)](chapters/02-structural/facade/README.md) | ⭐ | 为复杂子系统提供简化接口 |
| [享元模式 (Flyweight)](chapters/02-structural/flyweight/README.md) | ⭐⭐⭐ | 通过共享来高效支持大量细粒度对象 |
| [代理模式 (Proxy)](chapters/02-structural/proxy/README.md) | ⭐⭐ | 为另一个对象提供代理以控制访问 |

### 第三部分：行为型模式 (Behavioral Patterns)

行为型模式关注**对象之间的通信**，以及职责的分配。

| 模式 | 难度 | 简介 |
|------|------|------|
| [责任链模式 (Chain of Responsibility)](chapters/03-behavioral/chain-of-responsibility/README.md) | ⭐⭐ | 让多个对象都有机会处理请求 |
| [命令模式 (Command)](chapters/03-behavioral/command/README.md) | ⭐⭐ | 将请求封装成对象 |
| [迭代器模式 (Iterator)](chapters/03-behavioral/iterator/README.md) | ⭐⭐ | 顺序访问聚合对象中的元素 |
| [中介者模式 (Mediator)](chapters/03-behavioral/mediator/README.md) | ⭐⭐⭐ | 用中介对象封装一系列对象交互 |
| [备忘录模式 (Memento)](chapters/03-behavioral/memento/README.md) | ⭐⭐ | 在不破坏封装的前提下保存和恢复对象状态 |
| [观察者模式 (Observer)](chapters/03-behavioral/observer/README.md) | ⭐⭐ | 对象状态改变时通知所有依赖对象 |
| [状态模式 (State)](chapters/03-behavioral/state/README.md) | ⭐⭐⭐ | 允许对象在内部状态改变时改变行为 |
| [策略模式 (Strategy)](chapters/03-behavioral/strategy/README.md) | ⭐⭐ | 定义算法族，使它们可以互相替换 |
| [模板方法模式 (Template Method)](chapters/03-behavioral/template-method/README.md) | ⭐⭐ | 定义算法骨架，将某些步骤延迟到子类 |
| [访问者模式 (Visitor)](chapters/03-behavioral/visitor/README.md) | ⭐⭐⭐ | 为对象结构中的元素定义新操作 |
| [解释器模式 (Interpreter)](chapters/03-behavioral/interpreter/README.md) | ⭐⭐⭐ | 为语言创建解释器 |

---

## 🎯 如何使用本书

### 对于初学者

1. 首先阅读[设计模式概述](chapters/00-introduction/README.md)
2. 从简单的模式开始（如单例、工厂方法、外观）
3. 运行示例代码，动手修改并观察结果
4. 在自己的项目中尝试应用

### 对于有经验的开发者

1. 直接跳转到感兴趣的模式
2. 重点关注"高级应用"和"最佳实践"部分
3. 比较不同模式的适用场景
4. 研究模式之间的组合使用

---

## 🛠️ 运行代码

### C++ 代码

```bash
# 编译单个文件
g++ -std=c++17 -o output chapters/01-creational/singleton/cpp/singleton.cpp

# 运行
./output
```

### Python 代码

```bash
# 直接运行
python3 chapters/01-creational/singleton/python/singleton.py
```

---

## 📁 项目结构

```
design-pattern/
├── README.md                          # 本文件
├── chapters/
│   ├── 00-introduction/               # 引言
│   ├── 01-creational/                 # 创建型模式
│   │   ├── singleton/
│   │   ├── factory-method/
│   │   ├── abstract-factory/
│   │   ├── builder/
│   │   └── prototype/
│   ├── 02-structural/                 # 结构型模式
│   │   ├── adapter/
│   │   ├── bridge/
│   │   ├── composite/
│   │   ├── decorator/
│   │   ├── facade/
│   │   ├── flyweight/
│   │   └── proxy/
│   └── 03-behavioral/                 # 行为型模式
│       ├── chain-of-responsibility/
│       ├── command/
│       ├── iterator/
│       ├── mediator/
│       ├── memento/
│       ├── observer/
│       ├── state/
│       ├── strategy/
│       ├── template-method/
│       ├── visitor/
│       └── interpreter/
└── assets/                            # 图片资源
```

---

## 🔗 参考资料

- 《设计模式：可复用面向对象软件的基础》- GoF 经典著作
- 《Head First 设计模式》- 更加通俗易懂的入门读物
- [Refactoring Guru](https://refactoring.guru/design-patterns) - 优秀的在线资源

---

## 📝 贡献

欢迎提交 Issue 和 Pull Request！

---

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

---

<p align="center">
  <b>⭐ 如果这个项目对你有帮助，请给一个 Star！⭐</b>
</p>

