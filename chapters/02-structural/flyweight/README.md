# 享元模式 (Flyweight Pattern)

[← 返回结构型模式](../README.md) | [返回目录](../../../README.md)

---

## 📚 目录

- [意图与动机](#意图与动机)
- [问题场景](#问题场景)
- [解决方案](#解决方案)
- [代码实现](#代码实现)
- [初学者指南](#初学者指南)
- [实战案例](#实战案例)
- [相关模式](#相关模式)

---

## 意图与动机

### 一句话定义

> **运用共享技术有效地支持大量细粒度的对象。**

### 🔰 初学者理解

想象一个游戏中的森林场景，有成千上万棵树。每棵树都有：
- **相同的部分**：树的模型、纹理、颜色
- **不同的部分**：位置、大小、旋转角度

如果每棵树都存储完整的模型数据，内存会爆炸。享元模式把相同的部分**共享**，只存储不同的部分。

### 🚀 高级理解

享元模式的核心概念：
- **内在状态 (Intrinsic)**：可共享的、不变的状态
- **外在状态 (Extrinsic)**：不可共享的、随环境变化的状态

通过分离内在和外在状态，大量对象可以共享内在部分，显著减少内存使用。

---

## 问题场景

### 场景：文本编辑器

编辑器需要渲染大量字符。如果每个字符都是独立对象：

```python
class Character:
    def __init__(self, char, font, size, color, bold, italic, x, y):
        self.char = char      # 字符本身
        self.font = font      # 字体（可能很大）
        self.size = size      # 字号
        self.color = color    # 颜色
        self.bold = bold
        self.italic = italic
        self.x = x            # 位置 x
        self.y = y            # 位置 y

# 一篇 10000 字符的文档
# = 10000 个 Character 对象
# = 大量重复的字体、颜色信息
```

---

## 解决方案

```
┌─────────────────────────────────────────────────────────────────────┐
│                        享元模式结构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   FlyweightFactory                                                  │
│   ┌─────────────────────────────┐                                  │
│   │ - flyweights: Map           │                                  │
│   │ + getFlyweight(key)         │──────► 返回已有或创建新的享元     │
│   └──────────────┬──────────────┘                                  │
│                  │ creates                                          │
│                  ▼                                                  │
│            ┌───────────┐                                           │
│            │ Flyweight │ (共享的内在状态)                           │
│            │+ operation│                                            │
│            │ (extrinsic)│ ◄── 外在状态作为参数传入                   │
│            └───────────┘                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 代码实现

### C++ 实现

```cpp
// flyweight.cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <memory>
#include <vector>

// ============================================
// 享元类 - 存储内在状态
// ============================================
class TreeType {
private:
    std::string name;
    std::string color;
    std::string texture;  // 纹理数据（假设很大）

public:
    TreeType(const std::string& name, const std::string& color, 
             const std::string& texture)
        : name(name), color(color), texture(texture) {
        std::cout << "Creating tree type: " << name << std::endl;
    }
    
    // 操作使用外在状态（位置）
    void draw(int x, int y) const {
        std::cout << "Drawing " << name << " tree at (" << x << "," << y 
                  << ") with color " << color << std::endl;
    }
    
    std::string getName() const { return name; }
};

// ============================================
// 享元工厂
// ============================================
class TreeFactory {
private:
    std::unordered_map<std::string, std::shared_ptr<TreeType>> treeTypes;
    
    std::string getKey(const std::string& name, const std::string& color, 
                       const std::string& texture) {
        return name + "_" + color + "_" + texture;
    }

public:
    std::shared_ptr<TreeType> getTreeType(const std::string& name, 
                                          const std::string& color,
                                          const std::string& texture) {
        std::string key = getKey(name, color, texture);
        
        auto it = treeTypes.find(key);
        if (it == treeTypes.end()) {
            // 创建新的享元
            auto type = std::make_shared<TreeType>(name, color, texture);
            treeTypes[key] = type;
            return type;
        }
        
        // 返回已有享元
        return it->second;
    }
    
    size_t getTypeCount() const {
        return treeTypes.size();
    }
};

// ============================================
// 使用享元的具体对象 - 存储外在状态
// ============================================
class Tree {
private:
    int x, y;  // 外在状态：位置
    std::shared_ptr<TreeType> type;  // 引用共享的享元

public:
    Tree(int x, int y, std::shared_ptr<TreeType> type)
        : x(x), y(y), type(type) {}
    
    void draw() const {
        type->draw(x, y);
    }
};

// ============================================
// 森林类 - 管理大量树
// ============================================
class Forest {
private:
    std::vector<Tree> trees;
    TreeFactory factory;

public:
    void plantTree(int x, int y, const std::string& name,
                   const std::string& color, const std::string& texture) {
        auto type = factory.getTreeType(name, color, texture);
        trees.emplace_back(x, y, type);
    }
    
    void draw() const {
        std::cout << "\n=== Drawing Forest ===" << std::endl;
        for (const auto& tree : trees) {
            tree.draw();
        }
    }
    
    void printStats() const {
        std::cout << "\n=== Forest Stats ===" << std::endl;
        std::cout << "Total trees: " << trees.size() << std::endl;
        std::cout << "Tree types: " << factory.getTypeCount() << std::endl;
        std::cout << "Memory saved by sharing: ~" 
                  << (trees.size() - factory.getTypeCount()) * 100 
                  << " bytes (estimated)" << std::endl;
    }
};

int main() {
    Forest forest;
    
    // 种植大量树，但只有少数几种类型
    forest.plantTree(10, 20, "Oak", "Green", "oak_texture.png");
    forest.plantTree(30, 40, "Oak", "Green", "oak_texture.png");
    forest.plantTree(50, 60, "Pine", "DarkGreen", "pine_texture.png");
    forest.plantTree(70, 80, "Oak", "Green", "oak_texture.png");
    forest.plantTree(90, 100, "Birch", "LightGreen", "birch_texture.png");
    forest.plantTree(110, 120, "Pine", "DarkGreen", "pine_texture.png");
    
    forest.draw();
    forest.printStats();
    
    return 0;
}
```

### Python 实现

```python
# flyweight.py
from typing import Dict
from dataclasses import dataclass

# ============================================
# 享元类 - 内在状态
# ============================================
class CharacterStyle:
    """
    字符样式享元
    存储可共享的格式信息
    """
    
    def __init__(self, font: str, size: int, color: str, bold: bool = False):
        self.font = font
        self.size = size
        self.color = color
        self.bold = bold
        print(f"Creating style: {font} {size}pt {color} {'bold' if bold else ''}")
    
    def render(self, char: str, x: int, y: int):
        """渲染字符，位置作为外在状态传入"""
        style = f"{self.font} {self.size}pt {self.color}"
        if self.bold:
            style += " bold"
        print(f"Rendering '{char}' at ({x},{y}) with style: {style}")


# ============================================
# 享元工厂
# ============================================
class StyleFactory:
    """
    样式工厂
    缓存并复用相同的样式对象
    """
    
    _styles: Dict[str, CharacterStyle] = {}
    
    @classmethod
    def get_style(cls, font: str, size: int, color: str, bold: bool = False) -> CharacterStyle:
        key = f"{font}_{size}_{color}_{bold}"
        
        if key not in cls._styles:
            cls._styles[key] = CharacterStyle(font, size, color, bold)
        
        return cls._styles[key]
    
    @classmethod
    def get_style_count(cls) -> int:
        return len(cls._styles)


# ============================================
# 使用享元的对象 - 外在状态
# ============================================
@dataclass
class Character:
    """
    字符对象
    只存储字符本身和位置（外在状态）
    样式通过享元共享
    """
    char: str
    x: int
    y: int
    style: CharacterStyle
    
    def render(self):
        self.style.render(self.char, self.x, self.y)


# ============================================
# 文档类
# ============================================
class Document:
    def __init__(self):
        self._characters: list[Character] = []
    
    def add_character(self, char: str, x: int, y: int, 
                      font: str, size: int, color: str, bold: bool = False):
        # 获取（或创建）共享的样式
        style = StyleFactory.get_style(font, size, color, bold)
        # 创建字符，引用共享样式
        self._characters.append(Character(char, x, y, style))
    
    def render(self):
        print("\n=== Rendering Document ===")
        for char in self._characters:
            char.render()
    
    def print_stats(self):
        print("\n=== Document Stats ===")
        print(f"Total characters: {len(self._characters)}")
        print(f"Unique styles: {StyleFactory.get_style_count()}")
        
        # 计算节省的内存（估算）
        # 假设每个样式对象 100 bytes，每个字符引用 8 bytes
        without_flyweight = len(self._characters) * 100
        with_flyweight = StyleFactory.get_style_count() * 100 + len(self._characters) * 8
        saved = without_flyweight - with_flyweight
        
        print(f"Memory without flyweight: ~{without_flyweight} bytes")
        print(f"Memory with flyweight: ~{with_flyweight} bytes")
        print(f"Memory saved: ~{saved} bytes ({saved/without_flyweight*100:.1f}%)")


if __name__ == "__main__":
    doc = Document()
    
    # 添加一些字符，很多使用相同的样式
    text = "Hello World!"
    x = 0
    for char in text:
        if char.isupper():
            doc.add_character(char, x, 0, "Arial", 14, "Blue", bold=True)
        else:
            doc.add_character(char, x, 0, "Arial", 12, "Black")
        x += 10
    
    # 添加更多相同样式的字符
    text2 = "This is a test."
    x = 0
    for char in text2:
        doc.add_character(char, x, 20, "Arial", 12, "Black")
        x += 10
    
    doc.render()
    doc.print_stats()
```

---

## 初学者指南

### 内在状态 vs 外在状态

```
┌─────────────────────────────────────────────────────────────────┐
│   Tree 对象示例                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   内在状态（共享）           外在状态（独立）                     │
│   ┌─────────────────┐       ┌──────────────┐                   │
│   │ TreeType        │       │ Tree         │                   │
│   │ - name          │ ◄──── │ - x          │                   │
│   │ - color         │       │ - y          │                   │
│   │ - texture       │       │ - type (ref) │                   │
│   │ (可能很大)       │       │ (很小)       │                   │
│   └─────────────────┘       └──────────────┘                   │
│                                                                 │
│   1000棵树 × 1个TreeType = 节省大量内存                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 何时使用享元模式？

- 应用程序需要大量相似对象
- 对象的大部分状态可以外部化
- 移除外在状态后，可以用较少的共享对象替代大量对象
- 应用程序不依赖于对象标识（因为共享）

---

## 实战案例

### 游戏粒子系统

```python
# particle_system.py
from dataclasses import dataclass
from typing import Dict, List
import random

# 享元：粒子类型
class ParticleType:
    def __init__(self, name: str, texture: str, color: str, size: float):
        self.name = name
        self.texture = texture  # 纹理数据可能很大
        self.color = color
        self.size = size
        print(f"Loading particle type: {name}")
    
    def render(self, x: float, y: float, alpha: float):
        print(f"  {self.name} at ({x:.1f}, {y:.1f}) alpha={alpha:.2f}")

# 享元工厂
class ParticleTypeFactory:
    _types: Dict[str, ParticleType] = {}
    
    @classmethod
    def get_type(cls, name: str, texture: str, color: str, size: float) -> ParticleType:
        key = f"{name}_{texture}_{color}_{size}"
        if key not in cls._types:
            cls._types[key] = ParticleType(name, texture, color, size)
        return cls._types[key]

# 粒子（外在状态）
@dataclass
class Particle:
    x: float
    y: float
    velocity_x: float
    velocity_y: float
    alpha: float
    type: ParticleType
    
    def update(self, dt: float):
        self.x += self.velocity_x * dt
        self.y += self.velocity_y * dt
        self.alpha -= 0.1 * dt
    
    def render(self):
        self.type.render(self.x, self.y, self.alpha)
    
    def is_alive(self) -> bool:
        return self.alpha > 0

# 粒子系统
class ParticleSystem:
    def __init__(self):
        self._particles: List[Particle] = []
    
    def emit(self, x: float, y: float, particle_name: str, 
             texture: str, color: str, size: float, count: int = 10):
        particle_type = ParticleTypeFactory.get_type(particle_name, texture, color, size)
        
        for _ in range(count):
            particle = Particle(
                x=x + random.uniform(-5, 5),
                y=y + random.uniform(-5, 5),
                velocity_x=random.uniform(-10, 10),
                velocity_y=random.uniform(-20, -5),
                alpha=1.0,
                type=particle_type
            )
            self._particles.append(particle)
    
    def update(self, dt: float):
        for p in self._particles:
            p.update(dt)
        self._particles = [p for p in self._particles if p.is_alive()]
    
    def render(self):
        print(f"\nRendering {len(self._particles)} particles:")
        for p in self._particles[:5]:  # 只显示前5个
            p.render()
        if len(self._particles) > 5:
            print(f"  ... and {len(self._particles) - 5} more")


if __name__ == "__main__":
    system = ParticleSystem()
    
    # 发射大量粒子，但只有少数几种类型
    system.emit(100, 100, "Fire", "fire.png", "Orange", 2.0, count=50)
    system.emit(200, 100, "Smoke", "smoke.png", "Gray", 3.0, count=30)
    system.emit(150, 100, "Fire", "fire.png", "Orange", 2.0, count=50)  # 复用 Fire 类型
    
    system.render()
    
    print(f"\nTotal particles: {len(system._particles)}")
    print(f"Particle types created: {len(ParticleTypeFactory._types)}")
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **组合** | 享元常作为组合模式中的叶节点 |
| **单例** | 享元工厂通常是单例 |
| **状态/策略** | 可以将策略对象实现为享元 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 大量节省内存 | 增加代码复杂度 |
| | 需要分离内在/外在状态 |
| | 享元不能有独立状态 |

---

[← 上一章：外观模式](../facade/README.md) | [下一章：代理模式 →](../proxy/README.md)

