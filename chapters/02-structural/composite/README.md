# 组合模式 (Composite Pattern)

[← 返回结构型模式](../README.md) | [返回目录](../../../README.md)

---

## 📚 目录

- [意图与动机](#意图与动机)
- [问题场景](#问题场景)
- [解决方案](#解决方案)
- [结构](#结构)
- [代码实现](#代码实现)
- [初学者指南](#初学者指南)
- [实战案例](#实战案例)
- [相关模式](#相关模式)

---

## 意图与动机

### 一句话定义

> **将对象组合成树形结构以表示"部分-整体"的层次结构，使客户端对单个对象和组合对象的使用具有一致性。**

### 🔰 初学者理解

想象文件系统：
- 文件夹可以包含文件和其他文件夹
- 你可以对文件夹执行"获取大小"操作，它会递归计算所有内容的大小
- 你也可以对单个文件执行同样的操作

组合模式让你**统一处理**单个元素和元素集合。

### 🚀 高级理解

组合模式适用于：
- 树形结构的数据
- 需要统一对待个体和集合
- 递归结构的处理

---

## 问题场景

### 场景：图形编辑器中的形状组合

```python
# ❌ 没有组合模式：需要区分处理
def calculate_area(item):
    if isinstance(item, Circle):
        return math.pi * item.radius ** 2
    elif isinstance(item, Rectangle):
        return item.width * item.height
    elif isinstance(item, Group):
        # 组合需要特殊处理
        total = 0
        for child in item.children:
            total += calculate_area(child)  # 递归
        return total
```

---

## 解决方案

```
┌─────────────────────────────────────────────────────────────────────┐
│                        组合模式结构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                        Component (组件)                              │
│                    ┌─────────────────────┐                          │
│                    │ + operation()       │                          │
│                    └──────────▲──────────┘                          │
│                               │                                     │
│                ┌──────────────┴──────────────┐                      │
│                │                             │                      │
│         ┌──────┴──────┐              ┌───────┴───────┐              │
│         │    Leaf     │              │   Composite   │              │
│         │ (叶子节点)   │              │  (组合节点)    │              │
│         │+ operation()│              │+ operation()  │              │
│         └─────────────┘              │+ add(c)       │              │
│                                      │+ remove(c)   │              │
│                                      │+ getChild(i) │              │
│                                      └───────┬──────┘              │
│                                              │                      │
│                                              ▼                      │
│                                      包含多个 Component              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 结构

### 参与者

| 角色 | 职责 |
|------|------|
| **Component** | 声明接口，定义默认行为 |
| **Leaf** | 表示叶子节点，没有子节点 |
| **Composite** | 定义子节点行为，存储子组件 |

---

## 代码实现

### C++ 实现

```cpp
// composite.cpp
#include <iostream>
#include <vector>
#include <memory>
#include <string>
#include <algorithm>

// ============================================
// 组件接口
// ============================================
class FileSystemComponent {
public:
    virtual ~FileSystemComponent() = default;
    virtual std::string getName() const = 0;
    virtual int getSize() const = 0;
    virtual void display(int indent = 0) const = 0;
    
    // 组合专用方法（默认实现）
    virtual void add(std::shared_ptr<FileSystemComponent>) {
        throw std::runtime_error("Cannot add to a leaf");
    }
    virtual void remove(std::shared_ptr<FileSystemComponent>) {
        throw std::runtime_error("Cannot remove from a leaf");
    }
};

// ============================================
// 叶子节点 - 文件
// ============================================
class File : public FileSystemComponent {
private:
    std::string name;
    int size;

public:
    File(const std::string& name, int size) : name(name), size(size) {}
    
    std::string getName() const override { return name; }
    int getSize() const override { return size; }
    
    void display(int indent = 0) const override {
        std::cout << std::string(indent, ' ') << "📄 " << name 
                  << " (" << size << " bytes)" << std::endl;
    }
};

// ============================================
// 组合节点 - 文件夹
// ============================================
class Directory : public FileSystemComponent {
private:
    std::string name;
    std::vector<std::shared_ptr<FileSystemComponent>> children;

public:
    Directory(const std::string& name) : name(name) {}
    
    std::string getName() const override { return name; }
    
    int getSize() const override {
        int total = 0;
        for (const auto& child : children) {
            total += child->getSize();
        }
        return total;
    }
    
    void display(int indent = 0) const override {
        std::cout << std::string(indent, ' ') << "📁 " << name 
                  << " (" << getSize() << " bytes)" << std::endl;
        for (const auto& child : children) {
            child->display(indent + 2);
        }
    }
    
    void add(std::shared_ptr<FileSystemComponent> component) override {
        children.push_back(component);
    }
    
    void remove(std::shared_ptr<FileSystemComponent> component) override {
        children.erase(
            std::remove(children.begin(), children.end(), component),
            children.end()
        );
    }
};

int main() {
    // 创建文件
    auto file1 = std::make_shared<File>("readme.txt", 100);
    auto file2 = std::make_shared<File>("main.cpp", 500);
    auto file3 = std::make_shared<File>("utils.cpp", 300);
    auto file4 = std::make_shared<File>("test.cpp", 200);
    
    // 创建目录结构
    auto root = std::make_shared<Directory>("project");
    auto src = std::make_shared<Directory>("src");
    auto tests = std::make_shared<Directory>("tests");
    
    // 组装结构
    root->add(file1);
    root->add(src);
    root->add(tests);
    
    src->add(file2);
    src->add(file3);
    
    tests->add(file4);
    
    // 显示整个结构
    std::cout << "=== File System Structure ===" << std::endl;
    root->display();
    
    std::cout << "\n=== Total Size ===" << std::endl;
    std::cout << "Project total: " << root->getSize() << " bytes" << std::endl;
    
    return 0;
}
```

### Python 实现

```python
# composite.py
from abc import ABC, abstractmethod
from typing import List

# ============================================
# 组件接口
# ============================================
class Graphic(ABC):
    """图形组件基类"""
    
    @abstractmethod
    def draw(self) -> None:
        pass
    
    @abstractmethod
    def get_bounds(self) -> tuple:
        """返回 (x, y, width, height)"""
        pass

# ============================================
# 叶子节点
# ============================================
class Circle(Graphic):
    def __init__(self, x: int, y: int, radius: int):
        self.x = x
        self.y = y
        self.radius = radius
    
    def draw(self) -> None:
        print(f"Drawing Circle at ({self.x}, {self.y}) with radius {self.radius}")
    
    def get_bounds(self) -> tuple:
        return (self.x - self.radius, self.y - self.radius, 
                self.radius * 2, self.radius * 2)


class Rectangle(Graphic):
    def __init__(self, x: int, y: int, width: int, height: int):
        self.x = x
        self.y = y
        self.width = width
        self.height = height
    
    def draw(self) -> None:
        print(f"Drawing Rectangle at ({self.x}, {self.y}), "
              f"size {self.width}x{self.height}")
    
    def get_bounds(self) -> tuple:
        return (self.x, self.y, self.width, self.height)


# ============================================
# 组合节点
# ============================================
class GraphicGroup(Graphic):
    """图形组（可以包含其他图形或组）"""
    
    def __init__(self, name: str = "Group"):
        self.name = name
        self._children: List[Graphic] = []
    
    def add(self, graphic: Graphic) -> None:
        self._children.append(graphic)
    
    def remove(self, graphic: Graphic) -> None:
        self._children.remove(graphic)
    
    def draw(self) -> None:
        print(f"Drawing {self.name}:")
        for child in self._children:
            child.draw()
    
    def get_bounds(self) -> tuple:
        if not self._children:
            return (0, 0, 0, 0)
        
        min_x = min_y = float('inf')
        max_x = max_y = float('-inf')
        
        for child in self._children:
            x, y, w, h = child.get_bounds()
            min_x = min(min_x, x)
            min_y = min(min_y, y)
            max_x = max(max_x, x + w)
            max_y = max(max_y, y + h)
        
        return (min_x, min_y, max_x - min_x, max_y - min_y)


if __name__ == "__main__":
    # 创建基本图形
    circle1 = Circle(100, 100, 50)
    circle2 = Circle(200, 200, 30)
    rect1 = Rectangle(50, 50, 100, 80)
    rect2 = Rectangle(150, 150, 60, 40)
    
    # 创建图形组
    group1 = GraphicGroup("Circles Group")
    group1.add(circle1)
    group1.add(circle2)
    
    group2 = GraphicGroup("Rectangles Group")
    group2.add(rect1)
    group2.add(rect2)
    
    # 创建主画布（包含两个组）
    canvas = GraphicGroup("Canvas")
    canvas.add(group1)
    canvas.add(group2)
    
    # 绘制整个画布
    print("=== Drawing Canvas ===")
    canvas.draw()
    
    # 获取边界
    print(f"\n=== Bounds ===")
    print(f"Canvas bounds: {canvas.get_bounds()}")
    print(f"Circles group bounds: {group1.get_bounds()}")
```

---

## 初学者指南

### 组合模式的核心

```
【没有组合模式】
Client 需要区分处理不同类型：
├── if (item is File) → 处理文件
├── if (item is Directory) → 遍历处理
└── 代码复杂，难以扩展

【有组合模式】
Client 统一处理所有组件：
└── item.operation()  // 无论是叶子还是组合，接口相同
```

### 透明式 vs 安全式

| 类型 | 特点 | 优缺点 |
|------|------|--------|
| 透明式 | Component 中声明所有方法 | 叶子节点需要空实现 |
| 安全式 | 只在 Composite 中声明 add/remove | 客户端需要类型判断 |

---

## 实战案例

### 组织结构

```python
# organization.py
from abc import ABC, abstractmethod
from typing import List

class Employee(ABC):
    @abstractmethod
    def get_salary(self) -> float:
        pass
    
    @abstractmethod
    def show(self, indent: int = 0) -> None:
        pass

class Developer(Employee):
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary
    
    def get_salary(self) -> float:
        return self.salary
    
    def show(self, indent: int = 0) -> None:
        print(" " * indent + f"👨‍💻 {self.name}: ${self.salary}")

class Manager(Employee):
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary
        self._subordinates: List[Employee] = []
    
    def add(self, employee: Employee) -> None:
        self._subordinates.append(employee)
    
    def remove(self, employee: Employee) -> None:
        self._subordinates.remove(employee)
    
    def get_salary(self) -> float:
        total = self.salary
        for sub in self._subordinates:
            total += sub.get_salary()
        return total
    
    def show(self, indent: int = 0) -> None:
        print(" " * indent + f"👔 {self.name}: ${self.salary} (Team total: ${self.get_salary()})")
        for sub in self._subordinates:
            sub.show(indent + 2)


if __name__ == "__main__":
    # 构建组织结构
    ceo = Manager("CEO", 100000)
    
    cto = Manager("CTO", 80000)
    dev1 = Developer("Alice", 50000)
    dev2 = Developer("Bob", 55000)
    cto.add(dev1)
    cto.add(dev2)
    
    cfo = Manager("CFO", 75000)
    accountant = Developer("Charlie", 45000)
    cfo.add(accountant)
    
    ceo.add(cto)
    ceo.add(cfo)
    
    print("=== Organization Structure ===")
    ceo.show()
    print(f"\nTotal company salary: ${ceo.get_salary()}")
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **迭代器** | 常用于遍历组合结构 |
| **访问者** | 可以对组合结构执行操作 |
| **装饰器** | 装饰器类似组合但只有一个子组件 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 统一处理单个和组合对象 | 设计过于通用可能难以限制组件类型 |
| 容易添加新组件类型 | |
| 简化客户端代码 | |

---

[← 上一章：桥接模式](../bridge/README.md) | [下一章：装饰器模式 →](../decorator/README.md)

