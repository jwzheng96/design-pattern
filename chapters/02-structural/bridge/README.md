# 桥接模式 (Bridge Pattern)

[← 返回结构型模式](../README.md) | [返回目录](../../../README.md)

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

> **将抽象部分与它的实现部分分离，使它们都可以独立地变化。**

### 🔰 初学者理解

想象遥控器和电视的关系：
- 你可以有不同品牌的遥控器（基础、高级、语音控制）
- 你也可以有不同品牌的电视（索尼、三星、LG）

遥控器和电视可以**独立变化**——任何遥控器都能控制任何电视。桥接模式就是把这种"独立变化"的设计应用到代码中。

### 🚀 高级理解

桥接模式解决**类爆炸**问题：当有两个或更多维度的变化时，使用继承会导致类的数量呈指数级增长。

关键洞察：**优先使用组合而非继承**。

---

## 问题场景

### 场景：图形绘制系统

你需要支持不同形状（圆形、方形）和不同渲染方式（矢量、光栅）。

使用继承会产生类爆炸：

```
                    Shape
                      │
        ┌─────────────┼─────────────┐
        │             │             │
      Circle       Square       Triangle
        │             │             │
    ┌───┴───┐     ┌───┴───┐    ┌───┴───┐
VectorCircle  RasterCircle  VectorSquare ...

类数量 = 形状数 × 渲染方式数
3形状 × 2渲染 = 6个类
添加1种渲染方式 → 9个类
```

---

## 解决方案

将**形状**和**渲染方式**分离成两个独立的层次：

```
┌─────────────────────────────────────────────────────────────────────┐
│                        桥接模式结构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Abstraction (形状)                Implementation (渲染器)          │
│   ┌──────────────────┐             ┌──────────────────┐            │
│   │ Shape            │─────────────►│ Renderer         │            │
│   │ - renderer       │  "桥"       │ + renderCircle() │            │
│   │ + draw()         │             │ + renderRect()   │            │
│   └────────▲─────────┘             └────────▲─────────┘            │
│            │                                │                      │
│     ┌──────┴──────┐                  ┌──────┴──────┐               │
│     │             │                  │             │               │
│  ┌──┴───┐    ┌────┴───┐         ┌────┴───┐   ┌─────┴────┐         │
│  │Circle│    │Rectangle│        │ Vector │   │  Raster  │         │
│  └──────┘    └────────┘         │Renderer│   │ Renderer │         │
│                                  └────────┘   └──────────┘         │
│                                                                     │
│   类数量 = 形状数 + 渲染方式数 = 2 + 2 = 4                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 结构

### UML 类图

```
┌──────────────────────┐          ┌──────────────────────┐
│    Abstraction       │          │   Implementor        │
├──────────────────────┤          ├──────────────────────┤
│ - impl: Implementor  │◆────────►│ + operationImpl()    │
│ + operation()        │          └──────────▲───────────┘
└──────────▲───────────┘                     │
           │                          ┌──────┴──────┐
           │                          │             │
┌──────────┴───────────┐   ┌─────────┴──────┐  ┌───┴─────────┐
│ RefinedAbstraction   │   │ConcreteImplA   │  │ConcreteImplB│
└──────────────────────┘   └────────────────┘  └─────────────┘
```

### 参与者

| 角色 | 职责 |
|------|------|
| **Abstraction** | 定义抽象类的接口，持有实现者的引用 |
| **RefinedAbstraction** | 扩展抽象类 |
| **Implementor** | 定义实现类的接口 |
| **ConcreteImplementor** | 具体实现 |

---

## 代码实现

### C++ 实现

```cpp
// bridge.cpp
#include <iostream>
#include <memory>
#include <string>

// ============================================
// 实现者接口
// ============================================
class Renderer {
public:
    virtual ~Renderer() = default;
    virtual void renderCircle(float x, float y, float radius) = 0;
    virtual void renderRectangle(float x, float y, float width, float height) = 0;
};

// ============================================
// 具体实现者
// ============================================
class VectorRenderer : public Renderer {
public:
    void renderCircle(float x, float y, float radius) override {
        std::cout << "Drawing circle as vector at (" << x << "," << y 
                  << ") with radius " << radius << std::endl;
    }
    
    void renderRectangle(float x, float y, float width, float height) override {
        std::cout << "Drawing rectangle as vector at (" << x << "," << y 
                  << ") " << width << "x" << height << std::endl;
    }
};

class RasterRenderer : public Renderer {
public:
    void renderCircle(float x, float y, float radius) override {
        std::cout << "Drawing pixels for circle at (" << x << "," << y 
                  << ") with radius " << radius << std::endl;
    }
    
    void renderRectangle(float x, float y, float width, float height) override {
        std::cout << "Drawing pixels for rectangle at (" << x << "," << y 
                  << ") " << width << "x" << height << std::endl;
    }
};

// ============================================
// 抽象类
// ============================================
class Shape {
protected:
    std::shared_ptr<Renderer> renderer;

public:
    Shape(std::shared_ptr<Renderer> r) : renderer(r) {}
    virtual ~Shape() = default;
    virtual void draw() = 0;
    virtual void resize(float factor) = 0;
};

// ============================================
// 精炼抽象类
// ============================================
class Circle : public Shape {
private:
    float x, y, radius;

public:
    Circle(std::shared_ptr<Renderer> r, float x, float y, float radius)
        : Shape(r), x(x), y(y), radius(radius) {}
    
    void draw() override {
        renderer->renderCircle(x, y, radius);
    }
    
    void resize(float factor) override {
        radius *= factor;
    }
};

class Rectangle : public Shape {
private:
    float x, y, width, height;

public:
    Rectangle(std::shared_ptr<Renderer> r, float x, float y, float w, float h)
        : Shape(r), x(x), y(y), width(w), height(h) {}
    
    void draw() override {
        renderer->renderRectangle(x, y, width, height);
    }
    
    void resize(float factor) override {
        width *= factor;
        height *= factor;
    }
};

int main() {
    auto vectorRenderer = std::make_shared<VectorRenderer>();
    auto rasterRenderer = std::make_shared<RasterRenderer>();
    
    // 矢量圆形
    Circle vectorCircle(vectorRenderer, 5, 10, 15);
    vectorCircle.draw();
    
    // 光栅圆形
    Circle rasterCircle(rasterRenderer, 5, 10, 15);
    rasterCircle.draw();
    
    // 矢量矩形
    Rectangle vectorRect(vectorRenderer, 0, 0, 100, 50);
    vectorRect.draw();
    
    // 光栅矩形
    Rectangle rasterRect(rasterRenderer, 0, 0, 100, 50);
    rasterRect.draw();
    
    return 0;
}
```

### Python 实现

```python
# bridge.py
from abc import ABC, abstractmethod

# ============================================
# 实现者接口
# ============================================
class Device(ABC):
    """设备接口（实现者）"""
    
    @abstractmethod
    def is_enabled(self) -> bool:
        pass
    
    @abstractmethod
    def enable(self):
        pass
    
    @abstractmethod
    def disable(self):
        pass
    
    @abstractmethod
    def get_volume(self) -> int:
        pass
    
    @abstractmethod
    def set_volume(self, volume: int):
        pass
    
    @abstractmethod
    def get_channel(self) -> int:
        pass
    
    @abstractmethod
    def set_channel(self, channel: int):
        pass


# ============================================
# 具体实现者
# ============================================
class TV(Device):
    def __init__(self):
        self._on = False
        self._volume = 30
        self._channel = 1
    
    def is_enabled(self) -> bool:
        return self._on
    
    def enable(self):
        self._on = True
        print("TV: Power ON")
    
    def disable(self):
        self._on = False
        print("TV: Power OFF")
    
    def get_volume(self) -> int:
        return self._volume
    
    def set_volume(self, volume: int):
        self._volume = max(0, min(100, volume))
        print(f"TV: Volume set to {self._volume}")
    
    def get_channel(self) -> int:
        return self._channel
    
    def set_channel(self, channel: int):
        self._channel = channel
        print(f"TV: Channel set to {self._channel}")


class Radio(Device):
    def __init__(self):
        self._on = False
        self._volume = 20
        self._channel = 88
    
    def is_enabled(self) -> bool:
        return self._on
    
    def enable(self):
        self._on = True
        print("Radio: Power ON")
    
    def disable(self):
        self._on = False
        print("Radio: Power OFF")
    
    def get_volume(self) -> int:
        return self._volume
    
    def set_volume(self, volume: int):
        self._volume = max(0, min(100, volume))
        print(f"Radio: Volume set to {self._volume}")
    
    def get_channel(self) -> int:
        return self._channel
    
    def set_channel(self, channel: int):
        self._channel = channel
        print(f"Radio: Frequency set to {self._channel} FM")


# ============================================
# 抽象类
# ============================================
class RemoteControl:
    """遥控器（抽象类）"""
    
    def __init__(self, device: Device):
        self._device = device  # 桥接到设备
    
    def toggle_power(self):
        if self._device.is_enabled():
            self._device.disable()
        else:
            self._device.enable()
    
    def volume_up(self):
        self._device.set_volume(self._device.get_volume() + 10)
    
    def volume_down(self):
        self._device.set_volume(self._device.get_volume() - 10)
    
    def channel_up(self):
        self._device.set_channel(self._device.get_channel() + 1)
    
    def channel_down(self):
        self._device.set_channel(self._device.get_channel() - 1)


# ============================================
# 精炼抽象类
# ============================================
class AdvancedRemoteControl(RemoteControl):
    """高级遥控器（扩展功能）"""
    
    def mute(self):
        print("Muting device")
        self._device.set_volume(0)
    
    def set_channel_direct(self, channel: int):
        print(f"Setting channel directly to {channel}")
        self._device.set_channel(channel)


if __name__ == "__main__":
    # 创建设备
    tv = TV()
    radio = Radio()
    
    # 基本遥控器控制电视
    print("=== Basic Remote + TV ===")
    basic_remote = RemoteControl(tv)
    basic_remote.toggle_power()
    basic_remote.volume_up()
    basic_remote.channel_up()
    
    print("\n=== Advanced Remote + Radio ===")
    advanced_remote = AdvancedRemoteControl(radio)
    advanced_remote.toggle_power()
    advanced_remote.set_channel_direct(101)
    advanced_remote.mute()
```

---

## 初学者指南

### 桥接 vs 继承

```
【继承方式】类爆炸问题
                    Shape
                      │
        ┌─────────────┴─────────────┐
   VectorShape              RasterShape
        │                        │
    ┌───┴───┐                ┌───┴───┐
VectorCircle VectorRect  RasterCircle RasterRect

问题：每增加一个维度，类数量翻倍

【桥接方式】组合优于继承
Shape ◆────────────► Renderer
  │                      │
Circle Rectangle    Vector Raster

优点：两个维度独立变化，类数量 = n + m
```

### 何时使用桥接？

- 有多个独立变化的维度
- 想避免类爆炸
- 需要在运行时切换实现

---

## 实战案例

### 数据库驱动抽象

```python
# database_bridge.py
from abc import ABC, abstractmethod
from typing import List, Dict, Any

# 实现者：数据库驱动
class DatabaseDriver(ABC):
    @abstractmethod
    def connect(self, config: Dict): pass
    
    @abstractmethod
    def execute(self, query: str) -> List: pass
    
    @abstractmethod
    def close(self): pass

class MySQLDriver(DatabaseDriver):
    def connect(self, config: Dict):
        print(f"MySQL: Connecting to {config['host']}")
    
    def execute(self, query: str) -> List:
        print(f"MySQL: Executing {query}")
        return []
    
    def close(self):
        print("MySQL: Connection closed")

class PostgreSQLDriver(DatabaseDriver):
    def connect(self, config: Dict):
        print(f"PostgreSQL: Connecting to {config['host']}")
    
    def execute(self, query: str) -> List:
        print(f"PostgreSQL: Executing {query}")
        return []
    
    def close(self):
        print("PostgreSQL: Connection closed")

# 抽象：数据访问层
class DataAccessLayer:
    def __init__(self, driver: DatabaseDriver):
        self._driver = driver
    
    def find_all(self, table: str) -> List:
        return self._driver.execute(f"SELECT * FROM {table}")
    
    def find_by_id(self, table: str, id: int) -> Dict:
        results = self._driver.execute(f"SELECT * FROM {table} WHERE id = {id}")
        return results[0] if results else None

# 精炼抽象：带缓存的数据访问层
class CachedDataAccessLayer(DataAccessLayer):
    def __init__(self, driver: DatabaseDriver):
        super().__init__(driver)
        self._cache: Dict = {}
    
    def find_by_id(self, table: str, id: int) -> Dict:
        cache_key = f"{table}:{id}"
        if cache_key in self._cache:
            print(f"Cache hit for {cache_key}")
            return self._cache[cache_key]
        
        result = super().find_by_id(table, id)
        self._cache[cache_key] = result
        return result
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **适配器** | 适配器连接不兼容接口，桥接连接抽象与实现 |
| **抽象工厂** | 可以用抽象工厂创建桥接模式中的具体实现 |
| **策略** | 策略模式改变对象行为，桥接改变对象结构 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 分离抽象与实现 | 增加代码复杂度 |
| 提高可扩展性 | 需要正确识别两个变化维度 |
| 运行时切换实现 | |
| 避免类爆炸 | |

---

[← 上一章：适配器模式](../adapter/README.md) | [下一章：组合模式 →](../composite/README.md)

