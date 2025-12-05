# 抽象工厂模式 (Abstract Factory Pattern)

[← 返回创建型模式](../README.md) | [返回目录](../../../README.md)

---

## 📚 目录

- [意图与动机](#意图与动机)
- [问题场景](#问题场景)
- [解决方案](#解决方案)
- [结构](#结构)
- [代码实现](#代码实现)
- [初学者指南](#初学者指南)
- [高级应用](#高级应用)
- [实战案例](#实战案例)
- [相关模式](#相关模式)

---

## 意图与动机

### 一句话定义

> **提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。**

### 🔰 初学者理解

想象你在开一家家具店，你可以销售"现代风格"或"维多利亚风格"的家具。每种风格都包含：沙发、椅子、茶几等。

你不会把现代沙发和维多利亚椅子搭配销售——它们应该是配套的。抽象工厂就像是"风格套餐"，确保创建的所有对象都属于同一风格。

### 🚀 高级理解

抽象工厂是工厂方法的升级版：
- **工厂方法**：创建一种产品
- **抽象工厂**：创建一系列相关产品（产品族）

核心价值：
1. 确保产品之间的兼容性
2. 将产品的创建代码与使用代码分离
3. 支持产品族的切换

---

## 问题场景

### 场景：跨平台 UI 组件库

你需要开发一个跨平台的 GUI 应用，支持 Windows 和 macOS。每个平台都有自己风格的按钮、复选框、文本框等。

```
Windows 风格: [Button] [☑ Checkbox] [TextBox]
macOS 风格:   (Button) (☑ Checkbox) (TextBox)
```

问题：如何确保同一个应用中所有组件都是同一风格？

```cpp
// ❌ 糟糕的做法：混用不同风格
Button* btn = new WindowsButton();
Checkbox* cb = new MacCheckbox();  // 风格不一致！
```

---

## 解决方案

```
┌─────────────────────────────────────────────────────────────────────┐
│                        抽象工厂模式结构                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   AbstractFactory                                                   │
│   ┌────────────────────────────────┐                               │
│   │ + createButton(): Button       │                               │
│   │ + createCheckbox(): Checkbox   │                               │
│   └───────────────▲────────────────┘                               │
│                   │                                                 │
│       ┌───────────┴───────────┐                                    │
│       │                       │                                    │
│  ┌────┴─────────┐      ┌──────┴───────┐                           │
│  │WindowsFactory│      │  MacFactory  │                           │
│  └──────────────┘      └──────────────┘                           │
│         │                     │                                    │
│         ▼                     ▼                                    │
│  WindowsButton           MacButton                                 │
│  WindowsCheckbox         MacCheckbox                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 结构

### UML 类图

```
     ┌──────────────────────┐
     │  AbstractFactory     │
     ├──────────────────────┤
     │+createProductA()     │
     │+createProductB()     │
     └──────────▲───────────┘
                │
     ┌──────────┴──────────┐
     │                     │
┌────┴────────┐     ┌──────┴──────┐
│ConcreteFactory1│  │ConcreteFactory2│
└─────────────┘     └──────────────┘

┌────────────────┐  ┌────────────────┐
│ AbstractProductA│  │ AbstractProductB│
└───────▲────────┘  └───────▲────────┘
        │                   │
   ┌────┴────┐         ┌────┴────┐
   │ProductA1│         │ProductB1│
   │ProductA2│         │ProductB2│
   └─────────┘         └─────────┘
```

---

## 代码实现

### C++ 实现

```cpp
// abstract_factory.cpp
#include <iostream>
#include <memory>
#include <string>

// ============================================
// 抽象产品 A - 按钮
// ============================================
class Button {
public:
    virtual ~Button() = default;
    virtual void render() const = 0;
    virtual void onClick() const = 0;
};

// ============================================
// 抽象产品 B - 复选框
// ============================================
class Checkbox {
public:
    virtual ~Checkbox() = default;
    virtual void render() const = 0;
    virtual void onCheck() const = 0;
};

// ============================================
// Windows 风格产品
// ============================================
class WindowsButton : public Button {
public:
    void render() const override {
        std::cout << "Rendering Windows button: [  OK  ]" << std::endl;
    }
    void onClick() const override {
        std::cout << "Windows button clicked!" << std::endl;
    }
};

class WindowsCheckbox : public Checkbox {
public:
    void render() const override {
        std::cout << "Rendering Windows checkbox: [☑]" << std::endl;
    }
    void onCheck() const override {
        std::cout << "Windows checkbox checked!" << std::endl;
    }
};

// ============================================
// macOS 风格产品
// ============================================
class MacButton : public Button {
public:
    void render() const override {
        std::cout << "Rendering macOS button: (  OK  )" << std::endl;
    }
    void onClick() const override {
        std::cout << "macOS button clicked!" << std::endl;
    }
};

class MacCheckbox : public Checkbox {
public:
    void render() const override {
        std::cout << "Rendering macOS checkbox: (☑)" << std::endl;
    }
    void onCheck() const override {
        std::cout << "macOS checkbox checked!" << std::endl;
    }
};

// ============================================
// 抽象工厂
// ============================================
class GUIFactory {
public:
    virtual ~GUIFactory() = default;
    virtual std::unique_ptr<Button> createButton() const = 0;
    virtual std::unique_ptr<Checkbox> createCheckbox() const = 0;
};

// ============================================
// 具体工厂
// ============================================
class WindowsFactory : public GUIFactory {
public:
    std::unique_ptr<Button> createButton() const override {
        return std::make_unique<WindowsButton>();
    }
    std::unique_ptr<Checkbox> createCheckbox() const override {
        return std::make_unique<WindowsCheckbox>();
    }
};

class MacFactory : public GUIFactory {
public:
    std::unique_ptr<Button> createButton() const override {
        return std::make_unique<MacButton>();
    }
    std::unique_ptr<Checkbox> createCheckbox() const override {
        return std::make_unique<MacCheckbox>();
    }
};

// ============================================
// 客户端代码
// ============================================
class Application {
private:
    std::unique_ptr<Button> button;
    std::unique_ptr<Checkbox> checkbox;

public:
    Application(const GUIFactory& factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }
    
    void render() const {
        button->render();
        checkbox->render();
    }
    
    void interact() const {
        button->onClick();
        checkbox->onCheck();
    }
};

int main() {
    std::cout << "=== Windows Application ===" << std::endl;
    WindowsFactory windowsFactory;
    Application windowsApp(windowsFactory);
    windowsApp.render();
    windowsApp.interact();
    
    std::cout << "\n=== macOS Application ===" << std::endl;
    MacFactory macFactory;
    Application macApp(macFactory);
    macApp.render();
    macApp.interact();
    
    return 0;
}
```

### Python 实现

```python
# abstract_factory.py
from abc import ABC, abstractmethod

# ============================================
# 抽象产品
# ============================================
class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass
    
    @abstractmethod
    def on_click(self) -> str:
        pass

class Checkbox(ABC):
    @abstractmethod
    def render(self) -> str:
        pass
    
    @abstractmethod
    def on_check(self) -> str:
        pass

# ============================================
# Windows 风格产品
# ============================================
class WindowsButton(Button):
    def render(self) -> str:
        return "Rendering Windows button: [  OK  ]"
    
    def on_click(self) -> str:
        return "Windows button clicked!"

class WindowsCheckbox(Checkbox):
    def render(self) -> str:
        return "Rendering Windows checkbox: [☑]"
    
    def on_check(self) -> str:
        return "Windows checkbox checked!"

# ============================================
# macOS 风格产品
# ============================================
class MacButton(Button):
    def render(self) -> str:
        return "Rendering macOS button: (  OK  )"
    
    def on_click(self) -> str:
        return "macOS button clicked!"

class MacCheckbox(Checkbox):
    def render(self) -> str:
        return "Rendering macOS checkbox: (☑)"
    
    def on_check(self) -> str:
        return "macOS checkbox checked!"

# ============================================
# 抽象工厂
# ============================================
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass
    
    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass

# ============================================
# 具体工厂
# ============================================
class WindowsFactory(GUIFactory):
    def create_button(self) -> Button:
        return WindowsButton()
    
    def create_checkbox(self) -> Checkbox:
        return WindowsCheckbox()

class MacFactory(GUIFactory):
    def create_button(self) -> Button:
        return MacButton()
    
    def create_checkbox(self) -> Checkbox:
        return MacCheckbox()

# ============================================
# 客户端代码
# ============================================
class Application:
    def __init__(self, factory: GUIFactory):
        self.button = factory.create_button()
        self.checkbox = factory.create_checkbox()
    
    def render(self):
        print(self.button.render())
        print(self.checkbox.render())
    
    def interact(self):
        print(self.button.on_click())
        print(self.checkbox.on_check())


if __name__ == "__main__":
    print("=== Windows Application ===")
    windows_app = Application(WindowsFactory())
    windows_app.render()
    windows_app.interact()
    
    print("\n=== macOS Application ===")
    mac_app = Application(MacFactory())
    mac_app.render()
    mac_app.interact()
```

---

## 初学者指南

### 理解抽象工厂 vs 工厂方法

```
【工厂方法】创建单一产品
┌─────────────────────────────────────┐
│  ButtonFactory                      │
│    └── createButton() → Button      │
│                                     │
│  只关心创建一种产品                   │
└─────────────────────────────────────┘

【抽象工厂】创建产品族
┌─────────────────────────────────────┐
│  GUIFactory                         │
│    ├── createButton() → Button      │
│    ├── createCheckbox() → Checkbox  │
│    └── createTextBox() → TextBox    │
│                                     │
│  创建一组配套的产品                   │
└─────────────────────────────────────┘
```

### 何时选择抽象工厂？

当你需要：
1. 创建一组相关的产品
2. 确保产品之间的兼容性
3. 支持在运行时切换整个产品族

---

## 高级应用

### 工厂注册表

```python
# factory_registry.py
class FactoryRegistry:
    """工厂注册表 - 动态注册和获取工厂"""
    _factories: dict = {}
    
    @classmethod
    def register(cls, name: str, factory: GUIFactory):
        cls._factories[name] = factory
    
    @classmethod
    def get_factory(cls, name: str) -> GUIFactory:
        return cls._factories.get(name)

# 使用
FactoryRegistry.register("windows", WindowsFactory())
FactoryRegistry.register("mac", MacFactory())

factory = FactoryRegistry.get_factory("windows")
```

---

## 实战案例

### 数据库访问层

```python
# database_factory.py
from abc import ABC, abstractmethod

class Connection(ABC):
    @abstractmethod
    def connect(self, connection_string: str): pass

class Command(ABC):
    @abstractmethod
    def execute(self, query: str): pass

# MySQL 实现
class MySQLConnection(Connection):
    def connect(self, connection_string: str):
        print(f"Connecting to MySQL: {connection_string}")

class MySQLCommand(Command):
    def execute(self, query: str):
        print(f"Executing MySQL query: {query}")

# PostgreSQL 实现
class PostgreSQLConnection(Connection):
    def connect(self, connection_string: str):
        print(f"Connecting to PostgreSQL: {connection_string}")

class PostgreSQLCommand(Command):
    def execute(self, query: str):
        print(f"Executing PostgreSQL query: {query}")

# 抽象工厂
class DatabaseFactory(ABC):
    @abstractmethod
    def create_connection(self) -> Connection: pass
    
    @abstractmethod
    def create_command(self) -> Command: pass

class MySQLFactory(DatabaseFactory):
    def create_connection(self) -> Connection:
        return MySQLConnection()
    
    def create_command(self) -> Command:
        return MySQLCommand()

class PostgreSQLFactory(DatabaseFactory):
    def create_connection(self) -> Connection:
        return PostgreSQLConnection()
    
    def create_command(self) -> Command:
        return PostgreSQLCommand()
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **工厂方法** | 抽象工厂的基础，创建单一产品 |
| **单例** | 抽象工厂通常实现为单例 |
| **建造者** | 建造者关注分步构建，抽象工厂关注产品族 |
| **原型** | 可以用原型实现抽象工厂 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 确保产品兼容性 | 添加新产品类型困难 |
| 单一职责 | 代码可能变得复杂 |
| 开闭原则 | |
| 避免紧耦合 | |

---

[← 上一章：工厂方法模式](../factory-method/README.md) | [下一章：建造者模式 →](../builder/README.md)

