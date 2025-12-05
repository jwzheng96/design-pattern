# 工厂方法模式 (Factory Method Pattern)

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
- [最佳实践](#最佳实践)
- [常见陷阱](#常见陷阱)
- [实战案例](#实战案例)
- [相关模式](#相关模式)

---

## 意图与动机

### 一句话定义

> **定义一个创建对象的接口，但让子类决定实例化哪个类。**

### 🔰 初学者理解

想象一家物流公司，最初只用卡车运输。后来业务扩展，需要加入轮船运输。如果所有代码都写死用卡车，那每次加入新的运输方式都要改大量代码。

工厂方法就像是说："我需要一个运输工具，但具体是卡车还是轮船，让各个子公司（子类）自己决定。"

### 🚀 高级理解

工厂方法模式是一种创建型设计模式，提供了创建对象的接口，同时将对象的实际创建推迟到子类。这实现了：

1. **代码与具体类解耦**：核心代码只依赖产品接口，不关心具体实现
2. **遵循开闭原则**：添加新产品只需创建新的工厂子类
3. **单一职责**：将产品创建代码集中到一处

---

## 问题场景

### 场景描述

假设你正在开发一个物流管理应用。最初只支持卡车运输：

```cpp
// 最初的代码
class Truck {
public:
    void deliver() {
        std::cout << "Delivering by truck on the road" << std::endl;
    }
};

void planDelivery() {
    Truck* truck = new Truck();  // 直接创建卡车
    truck->deliver();
}
```

现在需要加入海运。问题来了：

```cpp
// ❌ 糟糕的做法：到处修改代码
void planDelivery(std::string type) {
    if (type == "truck") {
        Truck* truck = new Truck();
        truck->deliver();
    } else if (type == "ship") {
        Ship* ship = new Ship();
        ship->deliver();
    }
    // 每加入一种运输方式，就要修改这里
}
```

### 问题分析

1. **违反开闭原则**：添加新运输方式需要修改已有代码
2. **代码重复**：每个使用运输工具的地方都有类似的 if-else
3. **紧耦合**：代码直接依赖具体类

---

## 解决方案

### 核心思路

1. 定义产品接口 `Transport`
2. 创建具体产品类 `Truck`、`Ship`
3. 定义工厂接口（创建者），声明工厂方法
4. 创建具体工厂类，各自实现工厂方法返回对应产品

```
┌─────────────────────────────────────────────────────────────────────┐
│                        工厂方法模式结构                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│    Creator (抽象工厂)              Product (抽象产品)                │
│    ┌──────────────────┐           ┌──────────────────┐              │
│    │ + factoryMethod()│           │ + operation()    │              │
│    │ + someOperation()│           └────────▲─────────┘              │
│    └────────▲─────────┘                    │                        │
│             │                    ┌─────────┴─────────┐              │
│    ┌────────┴────────┐           │                   │              │
│    │                 │    ┌──────┴───────┐    ┌──────┴───────┐      │
│ ┌──┴───────┐  ┌──────┴──┐ │ConcreteProduct│    │ConcreteProduct│     │
│ │TruckCreator│  │ShipCreator│ │      A       │    │      B       │     │
│ └───────────┘  └──────────┘ └──────────────┘    └──────────────┘     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 结构

### UML 类图

```
┌─────────────────────────────────────┐
│           <<interface>>             │
│            Product                  │
├─────────────────────────────────────┤
│ + operation(): void                 │
└──────────────────▲──────────────────┘
                   │
       ┌───────────┴───────────┐
       │                       │
┌──────┴──────┐         ┌──────┴──────┐
│ ConcreteA   │         │ ConcreteB   │
├─────────────┤         ├─────────────┤
│ +operation()│         │ +operation()│
└─────────────┘         └─────────────┘

┌─────────────────────────────────────┐
│           <<abstract>>              │
│            Creator                  │
├─────────────────────────────────────┤
│ + factoryMethod(): Product          │
│ + someOperation(): void             │
└──────────────────▲──────────────────┘
                   │
       ┌───────────┴───────────┐
       │                       │
┌──────┴──────┐         ┌──────┴──────┐
│ CreatorA    │         │ CreatorB    │
├─────────────┤         ├─────────────┤
│+factoryMethod()│      │+factoryMethod()│
└─────────────┘         └─────────────┘
```

### 参与者

| 角色 | 职责 |
|------|------|
| **Product** | 声明产品接口 |
| **ConcreteProduct** | 实现产品接口 |
| **Creator** | 声明工厂方法，可以提供默认实现 |
| **ConcreteCreator** | 重写工厂方法返回具体产品 |

---

## 代码实现

### C++ 实现

```cpp
// factory_method.cpp
#include <iostream>
#include <memory>
#include <string>

// ============================================
// 产品接口
// ============================================
class Transport {
public:
    virtual ~Transport() = default;
    virtual void deliver() const = 0;
    virtual std::string getType() const = 0;
};

// ============================================
// 具体产品
// ============================================
class Truck : public Transport {
public:
    void deliver() const override {
        std::cout << "🚚 Delivering by land in a truck" << std::endl;
    }
    
    std::string getType() const override {
        return "Truck";
    }
};

class Ship : public Transport {
public:
    void deliver() const override {
        std::cout << "🚢 Delivering by sea in a ship" << std::endl;
    }
    
    std::string getType() const override {
        return "Ship";
    }
};

class Airplane : public Transport {
public:
    void deliver() const override {
        std::cout << "✈️ Delivering by air in an airplane" << std::endl;
    }
    
    std::string getType() const override {
        return "Airplane";
    }
};

// ============================================
// 工厂接口（创建者）
// ============================================
class Logistics {
public:
    virtual ~Logistics() = default;
    
    // 工厂方法 - 子类必须实现
    virtual std::unique_ptr<Transport> createTransport() const = 0;
    
    // 业务逻辑 - 使用工厂方法创建的产品
    void planDelivery() const {
        // 调用工厂方法获取产品
        auto transport = createTransport();
        
        std::cout << "Planning delivery using " << transport->getType() << std::endl;
        transport->deliver();
        std::cout << "Delivery completed!" << std::endl;
    }
};

// ============================================
// 具体工厂
// ============================================
class RoadLogistics : public Logistics {
public:
    std::unique_ptr<Transport> createTransport() const override {
        return std::make_unique<Truck>();
    }
};

class SeaLogistics : public Logistics {
public:
    std::unique_ptr<Transport> createTransport() const override {
        return std::make_unique<Ship>();
    }
};

class AirLogistics : public Logistics {
public:
    std::unique_ptr<Transport> createTransport() const override {
        return std::make_unique<Airplane>();
    }
};

// ============================================
// 客户端代码
// ============================================
void clientCode(const Logistics& logistics) {
    std::cout << "=== Starting delivery process ===" << std::endl;
    logistics.planDelivery();
    std::cout << std::endl;
}

int main() {
    std::cout << "App: Launched with RoadLogistics." << std::endl;
    RoadLogistics roadLogistics;
    clientCode(roadLogistics);
    
    std::cout << "App: Launched with SeaLogistics." << std::endl;
    SeaLogistics seaLogistics;
    clientCode(seaLogistics);
    
    std::cout << "App: Launched with AirLogistics." << std::endl;
    AirLogistics airLogistics;
    clientCode(airLogistics);
    
    return 0;
}
```

### Python 实现

```python
# factory_method.py
from abc import ABC, abstractmethod

# ============================================
# 产品接口
# ============================================
class Transport(ABC):
    """运输工具抽象基类"""
    
    @abstractmethod
    def deliver(self) -> str:
        """执行运输"""
        pass
    
    @abstractmethod
    def get_type(self) -> str:
        """获取运输类型"""
        pass


# ============================================
# 具体产品
# ============================================
class Truck(Transport):
    """卡车"""
    
    def deliver(self) -> str:
        return "🚚 Delivering by land in a truck"
    
    def get_type(self) -> str:
        return "Truck"


class Ship(Transport):
    """轮船"""
    
    def deliver(self) -> str:
        return "🚢 Delivering by sea in a ship"
    
    def get_type(self) -> str:
        return "Ship"


class Airplane(Transport):
    """飞机"""
    
    def deliver(self) -> str:
        return "✈️ Delivering by air in an airplane"
    
    def get_type(self) -> str:
        return "Airplane"


# ============================================
# 工厂接口（创建者）
# ============================================
class Logistics(ABC):
    """物流抽象基类"""
    
    @abstractmethod
    def create_transport(self) -> Transport:
        """
        工厂方法
        子类必须实现此方法来创建具体的运输工具
        """
        pass
    
    def plan_delivery(self) -> None:
        """
        业务逻辑方法
        使用工厂方法创建的产品执行业务逻辑
        """
        # 调用工厂方法获取产品
        transport = self.create_transport()
        
        print(f"Planning delivery using {transport.get_type()}")
        print(transport.deliver())
        print("Delivery completed!")


# ============================================
# 具体工厂
# ============================================
class RoadLogistics(Logistics):
    """公路物流"""
    
    def create_transport(self) -> Transport:
        return Truck()


class SeaLogistics(Logistics):
    """海运物流"""
    
    def create_transport(self) -> Transport:
        return Ship()


class AirLogistics(Logistics):
    """空运物流"""
    
    def create_transport(self) -> Transport:
        return Airplane()


# ============================================
# 客户端代码
# ============================================
def client_code(logistics: Logistics) -> None:
    """
    客户端代码只依赖 Logistics 接口
    不关心具体的运输工具类型
    """
    print("=== Starting delivery process ===")
    logistics.plan_delivery()
    print()


if __name__ == "__main__":
    print("App: Launched with RoadLogistics.")
    client_code(RoadLogistics())
    
    print("App: Launched with SeaLogistics.")
    client_code(SeaLogistics())
    
    print("App: Launched with AirLogistics.")
    client_code(AirLogistics())
```

---

## 初学者指南

### 理解工厂方法的核心

```
传统方式 vs 工厂方法

【传统方式】
┌─────────────────────────────────────────────────────────────┐
│  客户端代码                                                  │
│    │                                                        │
│    ├── if (type == "truck") { new Truck(); }               │
│    ├── if (type == "ship")  { new Ship();  }               │
│    └── if (type == "air")   { new Airplane(); }            │
│                                                             │
│  问题：每添加新类型，都要修改客户端代码                        │
└─────────────────────────────────────────────────────────────┘

【工厂方法】
┌─────────────────────────────────────────────────────────────┐
│  客户端代码                                                  │
│    │                                                        │
│    └── logistics.createTransport()  // 不关心具体类型       │
│            │                                                │
│            ▼                                                │
│  ┌────────────────────────────────────────────┐            │
│  │            Logistics (抽象)                │            │
│  │    + createTransport() : Transport         │            │
│  └────────────────────────────────────────────┘            │
│            ▲         ▲         ▲                           │
│            │         │         │                           │
│    ┌───────┴──┐ ┌────┴────┐ ┌──┴──────┐                   │
│    │ Road     │ │ Sea     │ │ Air     │                   │
│    │ Logistics│ │Logistics│ │Logistics│                   │
│    └──────────┘ └─────────┘ └─────────┘                   │
│                                                             │
│  优点：添加新类型只需创建新的工厂类，无需修改客户端代码         │
└─────────────────────────────────────────────────────────────┘
```

### 常见问题解答

**Q1: 工厂方法和简单工厂有什么区别？**

```python
# 简单工厂（不是 GoF 设计模式）
class SimpleFactory:
    @staticmethod
    def create(type: str) -> Transport:
        if type == "truck":
            return Truck()
        elif type == "ship":
            return Ship()
        # 添加新类型需要修改这里

# 工厂方法
class Logistics(ABC):
    @abstractmethod
    def create_transport(self) -> Transport:
        pass  # 子类负责创建，无需修改现有代码
```

**Q2: 什么时候用工厂方法而不是直接 new？**

当你发现：
- 需要创建的对象类型可能变化
- 创建逻辑复杂，需要集中管理
- 想要代码更加灵活、可测试

**Q3: 每个产品都需要一个工厂类吗？**

是的，这是工厂方法的特点。如果觉得类太多，可以考虑：
- 使用简单工厂（如果不需要经常扩展）
- 使用抽象工厂（如果产品有多个维度）

---

## 高级应用

### 1. 参数化工厂方法

工厂方法可以接收参数，根据参数创建不同变体：

```cpp
// parameterized_factory.cpp
class Dialog {
public:
    virtual std::unique_ptr<Button> createButton(const std::string& style = "default") const = 0;
};

class WindowsDialog : public Dialog {
public:
    std::unique_ptr<Button> createButton(const std::string& style) const override {
        if (style == "rounded") {
            return std::make_unique<WindowsRoundedButton>();
        }
        return std::make_unique<WindowsButton>();
    }
};
```

```python
# parameterized_factory.py
class Dialog(ABC):
    @abstractmethod
    def create_button(self, style: str = "default") -> Button:
        pass

class WindowsDialog(Dialog):
    def create_button(self, style: str = "default") -> Button:
        if style == "rounded":
            return WindowsRoundedButton()
        return WindowsButton()
```

### 2. 工厂方法与依赖注入

```python
# factory_with_di.py
from typing import Callable, Type

class Container:
    """简单的依赖注入容器"""
    _factories: dict[Type, Callable] = {}
    
    @classmethod
    def register(cls, interface: Type, factory: Callable):
        cls._factories[interface] = factory
    
    @classmethod
    def resolve(cls, interface: Type):
        factory = cls._factories.get(interface)
        if factory:
            return factory()
        raise ValueError(f"No factory registered for {interface}")

# 注册工厂
Container.register(Transport, lambda: Truck())

# 使用
transport = Container.resolve(Transport)
```

### 3. 泛型工厂（C++ 模板）

```cpp
// generic_factory.cpp
#include <memory>
#include <map>
#include <functional>

template<typename Base>
class GenericFactory {
private:
    std::map<std::string, std::function<std::unique_ptr<Base>()>> creators;
    
public:
    template<typename Derived>
    void registerType(const std::string& name) {
        creators[name] = []() { return std::make_unique<Derived>(); };
    }
    
    std::unique_ptr<Base> create(const std::string& name) {
        auto it = creators.find(name);
        if (it != creators.end()) {
            return it->second();
        }
        return nullptr;
    }
};

// 使用
int main() {
    GenericFactory<Transport> factory;
    factory.registerType<Truck>("truck");
    factory.registerType<Ship>("ship");
    
    auto transport = factory.create("truck");
    transport->deliver();
}
```

### 4. 工厂方法与单例结合

```python
# factory_singleton.py
class LoggerFactory:
    """工厂本身是单例"""
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def create_logger(self, logger_type: str) -> Logger:
        if logger_type == "file":
            return FileLogger()
        elif logger_type == "console":
            return ConsoleLogger()
        else:
            return NullLogger()
```

---

## 最佳实践

### ✅ 推荐做法

1. **产品必须有共同接口**
   ```cpp
   // 所有产品继承自同一接口
   class Product {
   public:
       virtual void operation() = 0;
   };
   ```

2. **工厂方法应该返回接口类型**
   ```python
   def create_transport(self) -> Transport:  # 返回抽象类型
       return Truck()  # 返回具体实现
   ```

3. **在创建者中提供有意义的默认实现**
   ```cpp
   class Logistics {
   public:
       virtual std::unique_ptr<Transport> createTransport() const {
           return std::make_unique<Truck>();  // 默认返回卡车
       }
   };
   ```

### ❌ 避免做法

1. **不要在工厂方法中使用类型判断**
   ```python
   # ❌ 这样做就失去了工厂方法的意义
   def create_transport(self, type: str) -> Transport:
       if type == "truck":
           return Truck()
       elif type == "ship":
           return Ship()
   ```

2. **不要让客户端依赖具体工厂**
   ```cpp
   // ❌ 客户端知道具体工厂类型
   RoadLogistics* logistics = new RoadLogistics();
   
   // ✅ 客户端只依赖抽象
   Logistics* logistics = getLogisticsFromConfig();
   ```

---

## 常见陷阱

### 陷阱 1：过度使用工厂方法

```python
# ❌ 不需要工厂方法的场景
class StringFactory:
    def create_string(self) -> str:
        return ""  # 简单对象不需要工厂

# ✅ 直接使用
s = ""
```

### 陷阱 2：工厂方法做了太多事情

```python
# ❌ 工厂方法不应该包含业务逻辑
class Logistics:
    def create_transport(self) -> Transport:
        transport = Truck()
        transport.configure()  # ❌ 配置应该在别处
        transport.validate()   # ❌ 验证应该在别处
        return transport

# ✅ 工厂方法只负责创建
class Logistics:
    def create_transport(self) -> Transport:
        return Truck()
    
    def configure_transport(self, transport: Transport):
        transport.configure()
```

### 陷阱 3：忘记处理资源管理

```cpp
// ❌ 内存泄漏风险
Transport* createTransport() {
    return new Truck();  // 谁负责 delete？
}

// ✅ 使用智能指针
std::unique_ptr<Transport> createTransport() {
    return std::make_unique<Truck>();
}
```

---

## 实战案例

### 案例 1：跨平台 UI 框架

```cpp
// cross_platform_ui.cpp
#include <iostream>
#include <memory>

// 产品接口
class Button {
public:
    virtual ~Button() = default;
    virtual void render() const = 0;
    virtual void onClick() const = 0;
};

class Checkbox {
public:
    virtual ~Checkbox() = default;
    virtual void render() const = 0;
    virtual void onCheck() const = 0;
};

// Windows 产品
class WindowsButton : public Button {
public:
    void render() const override {
        std::cout << "[Windows Button]" << std::endl;
    }
    void onClick() const override {
        std::cout << "Windows button clicked" << std::endl;
    }
};

// macOS 产品
class MacButton : public Button {
public:
    void render() const override {
        std::cout << "[macOS Button]" << std::endl;
    }
    void onClick() const override {
        std::cout << "macOS button clicked" << std::endl;
    }
};

// 工厂接口
class Dialog {
public:
    virtual ~Dialog() = default;
    virtual std::unique_ptr<Button> createButton() const = 0;
    
    void render() const {
        auto button = createButton();
        button->render();
    }
};

// 具体工厂
class WindowsDialog : public Dialog {
public:
    std::unique_ptr<Button> createButton() const override {
        return std::make_unique<WindowsButton>();
    }
};

class MacDialog : public Dialog {
public:
    std::unique_ptr<Button> createButton() const override {
        return std::make_unique<MacButton>();
    }
};

// 客户端
std::unique_ptr<Dialog> createDialog() {
    #ifdef _WIN32
        return std::make_unique<WindowsDialog>();
    #else
        return std::make_unique<MacDialog>();
    #endif
}

int main() {
    auto dialog = createDialog();
    dialog->render();
    return 0;
}
```

### 案例 2：文档解析器

```python
# document_parser.py
from abc import ABC, abstractmethod
from typing import List, Dict, Any
import json

# 产品接口
class Document(ABC):
    @abstractmethod
    def parse(self, content: str) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def validate(self) -> bool:
        pass

# 具体产品
class JSONDocument(Document):
    def __init__(self):
        self.data = {}
    
    def parse(self, content: str) -> Dict[str, Any]:
        self.data = json.loads(content)
        return self.data
    
    def validate(self) -> bool:
        return isinstance(self.data, dict)

class XMLDocument(Document):
    def __init__(self):
        self.data = {}
    
    def parse(self, content: str) -> Dict[str, Any]:
        # 简化的 XML 解析
        import xml.etree.ElementTree as ET
        root = ET.fromstring(content)
        self.data = {child.tag: child.text for child in root}
        return self.data
    
    def validate(self) -> bool:
        return len(self.data) > 0

class CSVDocument(Document):
    def __init__(self):
        self.data = {}
    
    def parse(self, content: str) -> Dict[str, Any]:
        lines = content.strip().split('\n')
        if len(lines) >= 2:
            headers = lines[0].split(',')
            values = lines[1].split(',')
            self.data = dict(zip(headers, values))
        return self.data
    
    def validate(self) -> bool:
        return len(self.data) > 0

# 工厂接口
class DocumentParser(ABC):
    @abstractmethod
    def create_document(self) -> Document:
        pass
    
    def parse_file(self, filepath: str) -> Dict[str, Any]:
        """模板方法：解析文件的通用流程"""
        doc = self.create_document()
        
        with open(filepath, 'r') as f:
            content = f.read()
        
        result = doc.parse(content)
        
        if not doc.validate():
            raise ValueError("Document validation failed")
        
        return result

# 具体工厂
class JSONParser(DocumentParser):
    def create_document(self) -> Document:
        return JSONDocument()

class XMLParser(DocumentParser):
    def create_document(self) -> Document:
        return XMLDocument()

class CSVParser(DocumentParser):
    def create_document(self) -> Document:
        return CSVDocument()

# 工厂选择器
def get_parser(filepath: str) -> DocumentParser:
    """根据文件扩展名选择合适的解析器"""
    if filepath.endswith('.json'):
        return JSONParser()
    elif filepath.endswith('.xml'):
        return XMLParser()
    elif filepath.endswith('.csv'):
        return CSVParser()
    else:
        raise ValueError(f"Unsupported file type: {filepath}")


if __name__ == "__main__":
    # 使用示例
    json_content = '{"name": "John", "age": "30"}'
    json_doc = JSONDocument()
    print(json_doc.parse(json_content))
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **抽象工厂** | 通常基于一组工厂方法实现 |
| **模板方法** | 工厂方法常在模板方法中被调用 |
| **原型** | 不需要子类，但需要初始化操作；工厂方法需要子类但无需初始化 |
| **单例** | 具体工厂通常是单例 |

---

## 总结

### 工厂方法的优缺点

| 优点 | 缺点 |
|------|------|
| 避免创建者与具体产品的紧耦合 | 每增加产品就需要增加工厂类 |
| 单一职责：产品创建代码集中 | 代码可能变得复杂 |
| 开闭原则：无需修改现有代码 | |
| 便于单元测试 | |

### 何时使用

✅ **适合场景：**
- 不知道代码将要使用的对象的确切类型
- 希望为库或框架的用户提供扩展方式
- 希望复用现有对象而非每次都创建新对象

❌ **不适合场景：**
- 对象创建简单，不需要封装
- 对象类型固定，不会变化
- 不需要子类化

---

[← 上一章：单例模式](../singleton/README.md) | [下一章：抽象工厂模式 →](../abstract-factory/README.md)

