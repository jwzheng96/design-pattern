# 原型模式 (Prototype Pattern)

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

> **用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。**

### 🔰 初学者理解

想象你在玩游戏，需要创建 100 个相似的敌人。与其每次都从头设置各种属性，不如先创建一个"模板敌人"，然后复制它 100 次，再微调每个副本。

原型模式就是**克隆**：先有一个原型对象，然后复制它来创建新对象。

### 🚀 高级理解

原型模式特别适用于：
1. **创建成本高**的对象（需要读数据库、做复杂计算）
2. **避免构建复杂的类层次结构**
3. **运行时动态配置**对象类型

---

## 问题场景

### 场景：图形编辑器中的复制功能

```cpp
// ❌ 问题：不知道对象的具体类型，如何复制？
Shape* copyShape(Shape* shape) {
    // shape 可能是 Circle、Rectangle、Triangle...
    // 不知道它的具体类型，无法 new
    
    // 糟糕的解决方案：类型判断
    if (dynamic_cast<Circle*>(shape)) {
        return new Circle(*dynamic_cast<Circle*>(shape));
    } else if (dynamic_cast<Rectangle*>(shape)) {
        return new Rectangle(*dynamic_cast<Rectangle*>(shape));
    }
    // 每添加新形状就要修改这里...
}
```

---

## 解决方案

让每个对象自己负责克隆自己：

```
┌─────────────────────────────────────────────────────────────────────┐
│                        原型模式结构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Prototype (原型接口)                                               │
│   ┌─────────────────────────────┐                                  │
│   │ + clone(): Prototype        │ ◄── 克隆方法                      │
│   └─────────────▲───────────────┘                                  │
│                 │                                                   │
│       ┌─────────┴─────────┬─────────────────┐                      │
│       │                   │                 │                      │
│  ┌────┴─────┐       ┌─────┴────┐     ┌──────┴────┐                │
│  │  Circle  │       │Rectangle │     │ Triangle  │                │
│  │ +clone() │       │ +clone() │     │ +clone()  │                │
│  └──────────┘       └──────────┘     └───────────┘                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 结构

### 参与者

| 角色 | 职责 |
|------|------|
| **Prototype** | 声明克隆方法的接口 |
| **ConcretePrototype** | 实现克隆方法 |
| **Client** | 通过调用原型的克隆方法来创建新对象 |

---

## 代码实现

### C++ 实现

```cpp
// prototype.cpp
#include <iostream>
#include <memory>
#include <string>
#include <unordered_map>

// ============================================
// 原型接口
// ============================================
class Shape {
public:
    virtual ~Shape() = default;
    virtual std::unique_ptr<Shape> clone() const = 0;
    virtual void draw() const = 0;
    virtual std::string getInfo() const = 0;
    
    // 公共属性
    std::string color = "black";
    int x = 0, y = 0;
};

// ============================================
// 具体原型 - 圆形
// ============================================
class Circle : public Shape {
public:
    int radius = 10;
    
    Circle() = default;
    
    // 拷贝构造函数
    Circle(const Circle& other) {
        this->color = other.color;
        this->x = other.x;
        this->y = other.y;
        this->radius = other.radius;
    }
    
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(*this);
    }
    
    void draw() const override {
        std::cout << "Drawing Circle at (" << x << "," << y 
                  << ") with radius " << radius 
                  << ", color: " << color << std::endl;
    }
    
    std::string getInfo() const override {
        return "Circle(r=" + std::to_string(radius) + ")";
    }
};

// ============================================
// 具体原型 - 矩形
// ============================================
class Rectangle : public Shape {
public:
    int width = 20;
    int height = 10;
    
    Rectangle() = default;
    
    Rectangle(const Rectangle& other) {
        this->color = other.color;
        this->x = other.x;
        this->y = other.y;
        this->width = other.width;
        this->height = other.height;
    }
    
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Rectangle>(*this);
    }
    
    void draw() const override {
        std::cout << "Drawing Rectangle at (" << x << "," << y 
                  << ") " << width << "x" << height
                  << ", color: " << color << std::endl;
    }
    
    std::string getInfo() const override {
        return "Rectangle(" + std::to_string(width) + "x" 
               + std::to_string(height) + ")";
    }
};

// ============================================
// 原型注册表
// ============================================
class ShapeRegistry {
private:
    std::unordered_map<std::string, std::unique_ptr<Shape>> prototypes;

public:
    void registerPrototype(const std::string& name, std::unique_ptr<Shape> prototype) {
        prototypes[name] = std::move(prototype);
    }
    
    std::unique_ptr<Shape> create(const std::string& name) const {
        auto it = prototypes.find(name);
        if (it != prototypes.end()) {
            return it->second->clone();
        }
        return nullptr;
    }
};

int main() {
    // 创建原型注册表并注册原型
    ShapeRegistry registry;
    
    auto redCircle = std::make_unique<Circle>();
    redCircle->color = "red";
    redCircle->radius = 15;
    registry.registerPrototype("red_circle", std::move(redCircle));
    
    auto blueRect = std::make_unique<Rectangle>();
    blueRect->color = "blue";
    blueRect->width = 30;
    blueRect->height = 20;
    registry.registerPrototype("blue_rect", std::move(blueRect));
    
    // 通过克隆创建新对象
    auto shape1 = registry.create("red_circle");
    shape1->x = 100;
    shape1->y = 100;
    shape1->draw();
    
    auto shape2 = registry.create("red_circle");
    shape2->x = 200;
    shape2->y = 200;
    shape2->draw();
    
    auto shape3 = registry.create("blue_rect");
    shape3->x = 50;
    shape3->y = 50;
    shape3->draw();
    
    return 0;
}
```

### Python 实现

```python
# prototype.py
import copy
from abc import ABC, abstractmethod
from typing import Dict

# ============================================
# 原型接口
# ============================================
class Prototype(ABC):
    @abstractmethod
    def clone(self) -> 'Prototype':
        pass

# ============================================
# 具体原型 - 形状基类
# ============================================
class Shape(Prototype):
    def __init__(self):
        self.x: int = 0
        self.y: int = 0
        self.color: str = "black"
    
    @abstractmethod
    def draw(self) -> None:
        pass
    
    def clone(self) -> 'Shape':
        """默认使用深拷贝"""
        return copy.deepcopy(self)

# ============================================
# 具体原型 - 圆形
# ============================================
class Circle(Shape):
    def __init__(self, radius: int = 10):
        super().__init__()
        self.radius = radius
    
    def draw(self) -> None:
        print(f"Drawing Circle at ({self.x},{self.y}) "
              f"with radius {self.radius}, color: {self.color}")
    
    def __repr__(self):
        return f"Circle(r={self.radius}, color={self.color})"

# ============================================
# 具体原型 - 矩形
# ============================================
class Rectangle(Shape):
    def __init__(self, width: int = 20, height: int = 10):
        super().__init__()
        self.width = width
        self.height = height
    
    def draw(self) -> None:
        print(f"Drawing Rectangle at ({self.x},{self.y}) "
              f"{self.width}x{self.height}, color: {self.color}")
    
    def __repr__(self):
        return f"Rectangle({self.width}x{self.height}, color={self.color})"

# ============================================
# 原型注册表
# ============================================
class ShapeRegistry:
    """原型注册表 - 管理预定义的原型"""
    
    def __init__(self):
        self._prototypes: Dict[str, Shape] = {}
    
    def register(self, name: str, prototype: Shape) -> None:
        """注册原型"""
        self._prototypes[name] = prototype
    
    def unregister(self, name: str) -> None:
        """移除原型"""
        del self._prototypes[name]
    
    def create(self, name: str) -> Shape:
        """通过克隆创建新对象"""
        prototype = self._prototypes.get(name)
        if prototype:
            return prototype.clone()
        raise ValueError(f"Unknown prototype: {name}")


if __name__ == "__main__":
    # 创建并配置原型
    registry = ShapeRegistry()
    
    red_circle = Circle(radius=15)
    red_circle.color = "red"
    registry.register("red_circle", red_circle)
    
    blue_rect = Rectangle(width=30, height=20)
    blue_rect.color = "blue"
    registry.register("blue_rect", blue_rect)
    
    # 通过克隆创建新对象
    shape1 = registry.create("red_circle")
    shape1.x, shape1.y = 100, 100
    shape1.draw()
    
    shape2 = registry.create("red_circle")
    shape2.x, shape2.y = 200, 200
    shape2.draw()
    
    shape3 = registry.create("blue_rect")
    shape3.x, shape3.y = 50, 50
    shape3.draw()
    
    # 验证是不同的对象
    print(f"\nshape1 is shape2: {shape1 is shape2}")  # False
    print(f"shape1: {shape1}")
    print(f"shape2: {shape2}")
```

---

## 初学者指南

### 深拷贝 vs 浅拷贝

```python
# 浅拷贝 vs 深拷贝的区别
import copy

class Nested:
    def __init__(self):
        self.items = [1, 2, 3]  # 可变对象

original = Nested()

# 浅拷贝：只复制第一层，嵌套对象共享
shallow = copy.copy(original)
shallow.items.append(4)
print(original.items)  # [1, 2, 3, 4] - 原对象也被修改！

# 深拷贝：递归复制所有层级
original2 = Nested()
deep = copy.deepcopy(original2)
deep.items.append(4)
print(original2.items)  # [1, 2, 3] - 原对象不受影响
```

### 何时使用原型模式？

- 创建对象成本高（如需要从数据库加载）
- 需要创建大量相似对象
- 对象类型在运行时才确定

---

## 高级应用

### 带有原型管理器的游戏对象

```python
# game_prototype.py
import copy
from typing import Dict, Any

class GameObject:
    """游戏对象原型"""
    
    def __init__(self, name: str):
        self.name = name
        self.position = [0, 0, 0]
        self.health = 100
        self.attributes: Dict[str, Any] = {}
    
    def clone(self) -> 'GameObject':
        return copy.deepcopy(self)


class GameObjectManager:
    """游戏对象原型管理器"""
    
    _prototypes: Dict[str, GameObject] = {}
    
    @classmethod
    def load_prototypes(cls):
        """从配置加载所有原型"""
        # 敌人原型
        goblin = GameObject("Goblin")
        goblin.health = 30
        goblin.attributes = {"damage": 5, "speed": 2}
        cls._prototypes["goblin"] = goblin
        
        orc = GameObject("Orc")
        orc.health = 80
        orc.attributes = {"damage": 15, "speed": 1}
        cls._prototypes["orc"] = orc
        
        dragon = GameObject("Dragon")
        dragon.health = 500
        dragon.attributes = {"damage": 50, "speed": 3, "can_fly": True}
        cls._prototypes["dragon"] = dragon
    
    @classmethod
    def spawn(cls, prototype_name: str, position: list) -> GameObject:
        """在指定位置生成对象"""
        prototype = cls._prototypes.get(prototype_name)
        if not prototype:
            raise ValueError(f"Unknown prototype: {prototype_name}")
        
        obj = prototype.clone()
        obj.position = position
        return obj


if __name__ == "__main__":
    GameObjectManager.load_prototypes()
    
    # 生成多个敌人
    enemies = [
        GameObjectManager.spawn("goblin", [10, 0, 20]),
        GameObjectManager.spawn("goblin", [15, 0, 25]),
        GameObjectManager.spawn("orc", [30, 0, 40]),
        GameObjectManager.spawn("dragon", [100, 50, 100]),
    ]
    
    for enemy in enemies:
        print(f"{enemy.name} at {enemy.position}, HP: {enemy.health}")
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **工厂方法** | 工厂方法基于继承，原型基于克隆 |
| **抽象工厂** | 可以用原型实现抽象工厂 |
| **备忘录** | 可以用原型保存对象状态快照 |
| **组合** | 使用原型可以方便地复制复杂的组合结构 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 可以克隆对象而无需与其具体类耦合 | 克隆包含循环引用的复杂对象可能困难 |
| 避免重复的初始化代码 | 需要为每个类实现克隆方法 |
| 更方便地生成复杂对象 | |
| 可以用原型管理预设对象 | |

---

[← 上一章：建造者模式](../builder/README.md) | [下一部分：结构型模式 →](../../02-structural/README.md)

