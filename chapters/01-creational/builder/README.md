# 建造者模式 (Builder Pattern)

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

> **将一个复杂对象的构建与其表示分离，使得同样的构建过程可以创建不同的表示。**

### 🔰 初学者理解

想象你在订制一辆汽车：
- 先选引擎类型
- 再选座椅数量  
- 然后选内饰
- 最后选颜色

每一步都是独立的，最终组合成完整的汽车。建造者模式就是把这种"分步构建"的过程形式化。

### 🚀 高级理解

建造者模式解决的核心问题：
1. **构造函数参数爆炸**：当对象有很多可选参数时
2. **复杂对象的创建逻辑**：需要执行多个步骤
3. **相同构建过程，不同表示**：用同样的步骤创建不同的对象

---

## 问题场景

### 构造函数参数爆炸

```cpp
// ❌ 太多参数，难以阅读和维护
class House {
public:
    House(int windows, int doors, int rooms, 
          bool hasGarage, bool hasSwimmingPool,
          bool hasStatues, bool hasGarden,
          int floors, std::string roofType, 
          std::string wallMaterial, /* ... */) {
        // ...
    }
};

// 调用时完全不知道每个参数是什么
House house(4, 2, 5, true, false, true, true, 2, "gable", "brick");
```

---

## 解决方案

```
┌─────────────────────────────────────────────────────────────────────┐
│                        建造者模式结构                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Director (指挥者)                                                  │
│   ┌─────────────────────────────┐                                  │
│   │ + construct()               │ ◄── 定义构建步骤的顺序            │
│   └─────────────┬───────────────┘                                  │
│                 │ uses                                              │
│                 ▼                                                   │
│   Builder (建造者接口)                                               │
│   ┌─────────────────────────────┐                                  │
│   │ + buildPartA()              │                                  │
│   │ + buildPartB()              │                                  │
│   │ + buildPartC()              │                                  │
│   │ + getResult()               │                                  │
│   └─────────────▲───────────────┘                                  │
│                 │                                                   │
│       ┌─────────┴─────────┐                                        │
│       │                   │                                        │
│  ┌────┴─────┐        ┌────┴─────┐                                  │
│  │ConcreteBuilder1│   │ConcreteBuilder2│                            │
│  └──────────┘        └──────────┘                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 结构

### 参与者

| 角色 | 职责 |
|------|------|
| **Builder** | 声明构建产品各部分的方法 |
| **ConcreteBuilder** | 实现 Builder 接口，提供具体的构建实现 |
| **Director** | 定义构建步骤的调用顺序 |
| **Product** | 被构建的复杂对象 |

---

## 代码实现

### C++ 实现

```cpp
// builder.cpp
#include <iostream>
#include <string>
#include <memory>

// ============================================
// 产品类
// ============================================
class House {
public:
    std::string foundation;
    std::string structure;
    std::string roof;
    std::string interior;
    bool hasGarage = false;
    bool hasSwimmingPool = false;
    bool hasGarden = false;
    
    void show() const {
        std::cout << "House Details:" << std::endl;
        std::cout << "  Foundation: " << foundation << std::endl;
        std::cout << "  Structure: " << structure << std::endl;
        std::cout << "  Roof: " << roof << std::endl;
        std::cout << "  Interior: " << interior << std::endl;
        std::cout << "  Garage: " << (hasGarage ? "Yes" : "No") << std::endl;
        std::cout << "  Swimming Pool: " << (hasSwimmingPool ? "Yes" : "No") << std::endl;
        std::cout << "  Garden: " << (hasGarden ? "Yes" : "No") << std::endl;
    }
};

// ============================================
// 建造者接口
// ============================================
class HouseBuilder {
public:
    virtual ~HouseBuilder() = default;
    virtual void reset() = 0;
    virtual void buildFoundation() = 0;
    virtual void buildStructure() = 0;
    virtual void buildRoof() = 0;
    virtual void buildInterior() = 0;
    virtual void addGarage() = 0;
    virtual void addSwimmingPool() = 0;
    virtual void addGarden() = 0;
    virtual std::unique_ptr<House> getResult() = 0;
};

// ============================================
// 具体建造者 - 普通房屋
// ============================================
class NormalHouseBuilder : public HouseBuilder {
private:
    std::unique_ptr<House> house;

public:
    NormalHouseBuilder() { reset(); }
    
    void reset() override {
        house = std::make_unique<House>();
    }
    
    void buildFoundation() override {
        house->foundation = "Concrete foundation";
    }
    
    void buildStructure() override {
        house->structure = "Wood and brick walls";
    }
    
    void buildRoof() override {
        house->roof = "Asphalt shingles";
    }
    
    void buildInterior() override {
        house->interior = "Standard interior finish";
    }
    
    void addGarage() override {
        house->hasGarage = true;
    }
    
    void addSwimmingPool() override {
        house->hasSwimmingPool = true;
    }
    
    void addGarden() override {
        house->hasGarden = true;
    }
    
    std::unique_ptr<House> getResult() override {
        auto result = std::move(house);
        reset();
        return result;
    }
};

// ============================================
// 具体建造者 - 豪华房屋
// ============================================
class LuxuryHouseBuilder : public HouseBuilder {
private:
    std::unique_ptr<House> house;

public:
    LuxuryHouseBuilder() { reset(); }
    
    void reset() override {
        house = std::make_unique<House>();
    }
    
    void buildFoundation() override {
        house->foundation = "Reinforced concrete with basement";
    }
    
    void buildStructure() override {
        house->structure = "Steel frame with marble walls";
    }
    
    void buildRoof() override {
        house->roof = "Slate tiles";
    }
    
    void buildInterior() override {
        house->interior = "Premium hardwood floors, designer kitchen";
    }
    
    void addGarage() override {
        house->hasGarage = true;
    }
    
    void addSwimmingPool() override {
        house->hasSwimmingPool = true;
    }
    
    void addGarden() override {
        house->hasGarden = true;
    }
    
    std::unique_ptr<House> getResult() override {
        auto result = std::move(house);
        reset();
        return result;
    }
};

// ============================================
// 指挥者
// ============================================
class Director {
public:
    // 构建简单房屋
    void constructSimpleHouse(HouseBuilder& builder) {
        builder.reset();
        builder.buildFoundation();
        builder.buildStructure();
        builder.buildRoof();
        builder.buildInterior();
    }
    
    // 构建完整房屋
    void constructFullFeaturedHouse(HouseBuilder& builder) {
        builder.reset();
        builder.buildFoundation();
        builder.buildStructure();
        builder.buildRoof();
        builder.buildInterior();
        builder.addGarage();
        builder.addSwimmingPool();
        builder.addGarden();
    }
};

int main() {
    Director director;
    
    // 使用普通建造者
    NormalHouseBuilder normalBuilder;
    director.constructFullFeaturedHouse(normalBuilder);
    auto normalHouse = normalBuilder.getResult();
    std::cout << "=== Normal House ===" << std::endl;
    normalHouse->show();
    
    std::cout << std::endl;
    
    // 使用豪华建造者
    LuxuryHouseBuilder luxuryBuilder;
    director.constructFullFeaturedHouse(luxuryBuilder);
    auto luxuryHouse = luxuryBuilder.getResult();
    std::cout << "=== Luxury House ===" << std::endl;
    luxuryHouse->show();
    
    return 0;
}
```

### Python 实现（Fluent Builder）

```python
# builder.py
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Optional

# ============================================
# 产品类
# ============================================
class Computer:
    def __init__(self):
        self.cpu: Optional[str] = None
        self.ram: Optional[str] = None
        self.storage: Optional[str] = None
        self.gpu: Optional[str] = None
        self.os: Optional[str] = None
    
    def __str__(self) -> str:
        parts = [
            f"CPU: {self.cpu}",
            f"RAM: {self.ram}",
            f"Storage: {self.storage}",
            f"GPU: {self.gpu or 'Integrated'}",
            f"OS: {self.os or 'None'}",
        ]
        return "Computer Configuration:\n  " + "\n  ".join(parts)


# ============================================
# Fluent Builder（流式建造者）
# ============================================
class ComputerBuilder:
    """流式建造者 - 支持链式调用"""
    
    def __init__(self):
        self._computer = Computer()
    
    def set_cpu(self, cpu: str) -> ComputerBuilder:
        self._computer.cpu = cpu
        return self  # 返回 self 支持链式调用
    
    def set_ram(self, ram: str) -> ComputerBuilder:
        self._computer.ram = ram
        return self
    
    def set_storage(self, storage: str) -> ComputerBuilder:
        self._computer.storage = storage
        return self
    
    def set_gpu(self, gpu: str) -> ComputerBuilder:
        self._computer.gpu = gpu
        return self
    
    def set_os(self, os: str) -> ComputerBuilder:
        self._computer.os = os
        return self
    
    def build(self) -> Computer:
        """构建并返回产品"""
        computer = self._computer
        self._computer = Computer()  # 重置建造者
        return computer


# ============================================
# 指挥者（预设配置）
# ============================================
class ComputerDirector:
    """预定义的电脑配置"""
    
    @staticmethod
    def build_gaming_pc(builder: ComputerBuilder) -> Computer:
        return (builder
            .set_cpu("Intel Core i9-13900K")
            .set_ram("32GB DDR5")
            .set_storage("2TB NVMe SSD")
            .set_gpu("NVIDIA RTX 4090")
            .set_os("Windows 11")
            .build())
    
    @staticmethod
    def build_office_pc(builder: ComputerBuilder) -> Computer:
        return (builder
            .set_cpu("Intel Core i5-13400")
            .set_ram("16GB DDR4")
            .set_storage("512GB SSD")
            .set_os("Windows 11")
            .build())
    
    @staticmethod
    def build_developer_mac(builder: ComputerBuilder) -> Computer:
        return (builder
            .set_cpu("Apple M2 Pro")
            .set_ram("32GB Unified")
            .set_storage("1TB SSD")
            .set_os("macOS Ventura")
            .build())


if __name__ == "__main__":
    builder = ComputerBuilder()
    
    # 使用指挥者构建预设配置
    print("=== Gaming PC ===")
    gaming_pc = ComputerDirector.build_gaming_pc(builder)
    print(gaming_pc)
    
    print("\n=== Office PC ===")
    office_pc = ComputerDirector.build_office_pc(builder)
    print(office_pc)
    
    print("\n=== Developer Mac ===")
    dev_mac = ComputerDirector.build_developer_mac(builder)
    print(dev_mac)
    
    # 自定义配置（不使用指挥者）
    print("\n=== Custom PC ===")
    custom_pc = (ComputerBuilder()
        .set_cpu("AMD Ryzen 9 7950X")
        .set_ram("64GB DDR5")
        .set_storage("4TB NVMe SSD")
        .set_gpu("NVIDIA RTX 4080")
        .set_os("Ubuntu 22.04")
        .build())
    print(custom_pc)
```

---

## 初学者指南

### 建造者模式 vs 构造函数

```python
# ❌ 传统构造函数方式
computer = Computer("i9", "32GB", "2TB", "RTX4090", "Win11")
# 问题：参数太多，顺序容易记错

# ✅ 建造者模式
computer = (ComputerBuilder()
    .set_cpu("i9")
    .set_ram("32GB")
    .set_storage("2TB")
    .set_gpu("RTX4090")
    .set_os("Win11")
    .build())
# 优点：清晰明了，可选参数灵活
```

### 何时使用建造者模式？

- 对象有很多可选参数
- 对象的创建需要多个步骤
- 需要创建同一对象的不同变体

---

## 高级应用

### 不可变对象建造者

```python
# immutable_builder.py
from dataclasses import dataclass
from typing import Optional

@dataclass(frozen=True)  # 不可变
class ImmutableUser:
    name: str
    email: str
    age: Optional[int] = None
    address: Optional[str] = None

class UserBuilder:
    def __init__(self):
        self._name: Optional[str] = None
        self._email: Optional[str] = None
        self._age: Optional[int] = None
        self._address: Optional[str] = None
    
    def name(self, name: str) -> 'UserBuilder':
        self._name = name
        return self
    
    def email(self, email: str) -> 'UserBuilder':
        self._email = email
        return self
    
    def age(self, age: int) -> 'UserBuilder':
        self._age = age
        return self
    
    def address(self, address: str) -> 'UserBuilder':
        self._address = address
        return self
    
    def build(self) -> ImmutableUser:
        if not self._name or not self._email:
            raise ValueError("name and email are required")
        return ImmutableUser(
            name=self._name,
            email=self._email,
            age=self._age,
            address=self._address
        )

# 使用
user = (UserBuilder()
    .name("Alice")
    .email("alice@example.com")
    .age(25)
    .build())
```

---

## 实战案例

### SQL 查询建造者

```python
# sql_builder.py
class SQLBuilder:
    """SQL 查询建造者"""
    
    def __init__(self):
        self._select = "*"
        self._from = ""
        self._where = []
        self._order_by = []
        self._limit = None
    
    def select(self, *columns: str) -> 'SQLBuilder':
        self._select = ", ".join(columns) if columns else "*"
        return self
    
    def from_table(self, table: str) -> 'SQLBuilder':
        self._from = table
        return self
    
    def where(self, condition: str) -> 'SQLBuilder':
        self._where.append(condition)
        return self
    
    def order_by(self, column: str, desc: bool = False) -> 'SQLBuilder':
        order = f"{column} DESC" if desc else column
        self._order_by.append(order)
        return self
    
    def limit(self, count: int) -> 'SQLBuilder':
        self._limit = count
        return self
    
    def build(self) -> str:
        if not self._from:
            raise ValueError("FROM clause is required")
        
        query = f"SELECT {self._select} FROM {self._from}"
        
        if self._where:
            query += " WHERE " + " AND ".join(self._where)
        
        if self._order_by:
            query += " ORDER BY " + ", ".join(self._order_by)
        
        if self._limit:
            query += f" LIMIT {self._limit}"
        
        return query


if __name__ == "__main__":
    query = (SQLBuilder()
        .select("id", "name", "email")
        .from_table("users")
        .where("age > 18")
        .where("status = 'active'")
        .order_by("created_at", desc=True)
        .limit(10)
        .build())
    
    print(query)
    # SELECT id, name, email FROM users 
    # WHERE age > 18 AND status = 'active' 
    # ORDER BY created_at DESC LIMIT 10
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **抽象工厂** | 抽象工厂创建产品族，建造者分步创建单个产品 |
| **单例** | 建造者可以实现为单例 |
| **组合** | 建造者常用于创建组合对象 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 分步构建复杂对象 | 需要创建多个类 |
| 可以复用构建代码 | 代码量增加 |
| 支持链式调用（Fluent API） | |
| 隔离复杂的构建逻辑 | |

---

[← 上一章：抽象工厂模式](../abstract-factory/README.md) | [下一章：原型模式 →](../prototype/README.md)

