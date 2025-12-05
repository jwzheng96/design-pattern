# 访问者模式 (Visitor Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。**

### 🔰 初学者理解

想象保险公司的销售员：
- 访问住宅区：推销家庭保险
- 访问银行：推销商业保险
- 访问咖啡店：推销小商户保险

销售员（访问者）对不同类型的建筑（元素）有不同的行为。要添加新的销售策略，只需添加新的访问者，无需修改建筑类。

---

## 代码实现

### Python 实现

```python
# visitor.py
from abc import ABC, abstractmethod
from typing import List

# ============================================
# 访问者接口
# ============================================
class Visitor(ABC):
    @abstractmethod
    def visit_circle(self, circle: 'Circle') -> str:
        pass
    
    @abstractmethod
    def visit_rectangle(self, rectangle: 'Rectangle') -> str:
        pass
    
    @abstractmethod
    def visit_compound(self, compound: 'CompoundShape') -> str:
        pass


# ============================================
# 元素接口
# ============================================
class Shape(ABC):
    @abstractmethod
    def accept(self, visitor: Visitor) -> str:
        pass


# ============================================
# 具体元素
# ============================================
class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius
    
    def accept(self, visitor: Visitor) -> str:
        return visitor.visit_circle(self)


class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height
    
    def accept(self, visitor: Visitor) -> str:
        return visitor.visit_rectangle(self)


class CompoundShape(Shape):
    def __init__(self):
        self.children: List[Shape] = []
    
    def add(self, shape: Shape) -> None:
        self.children.append(shape)
    
    def accept(self, visitor: Visitor) -> str:
        return visitor.visit_compound(self)


# ============================================
# 具体访问者
# ============================================
class AreaCalculator(Visitor):
    """计算面积"""
    
    def visit_circle(self, circle: Circle) -> str:
        area = 3.14159 * circle.radius ** 2
        return f"Circle area: {area:.2f}"
    
    def visit_rectangle(self, rectangle: Rectangle) -> str:
        area = rectangle.width * rectangle.height
        return f"Rectangle area: {area:.2f}"
    
    def visit_compound(self, compound: CompoundShape) -> str:
        total = 0
        for child in compound.children:
            result = child.accept(self)
            # 从结果字符串中提取数值（简化处理）
            total += float(result.split(': ')[1])
        return f"Compound area: {total:.2f}"


class XMLExporter(Visitor):
    """导出为 XML"""
    
    def visit_circle(self, circle: Circle) -> str:
        return f'<circle radius="{circle.radius}"/>'
    
    def visit_rectangle(self, rectangle: Rectangle) -> str:
        return f'<rectangle width="{rectangle.width}" height="{rectangle.height}"/>'
    
    def visit_compound(self, compound: CompoundShape) -> str:
        children_xml = '\n'.join(
            '  ' + child.accept(self) for child in compound.children
        )
        return f'<compound>\n{children_xml}\n</compound>'


class JSONExporter(Visitor):
    """导出为 JSON"""
    
    def visit_circle(self, circle: Circle) -> str:
        return f'{{"type": "circle", "radius": {circle.radius}}}'
    
    def visit_rectangle(self, rectangle: Rectangle) -> str:
        return f'{{"type": "rectangle", "width": {rectangle.width}, "height": {rectangle.height}}}'
    
    def visit_compound(self, compound: CompoundShape) -> str:
        children_json = ', '.join(
            child.accept(self) for child in compound.children
        )
        return f'{{"type": "compound", "children": [{children_json}]}}'


if __name__ == "__main__":
    # 创建图形结构
    compound = CompoundShape()
    compound.add(Circle(5))
    compound.add(Rectangle(3, 4))
    compound.add(Circle(2))
    
    # 使用不同的访问者
    print("=== Area Calculation ===")
    area_calc = AreaCalculator()
    print(compound.accept(area_calc))
    
    print("\n=== XML Export ===")
    xml_exporter = XMLExporter()
    print(compound.accept(xml_exporter))
    
    print("\n=== JSON Export ===")
    json_exporter = JSONExporter()
    print(compound.accept(json_exporter))
```

### C++ 实现

```cpp
// visitor.cpp
#include <iostream>
#include <vector>
#include <memory>

class Circle;
class Rectangle;

class Visitor {
public:
    virtual ~Visitor() = default;
    virtual void visitCircle(Circle* circle) = 0;
    virtual void visitRectangle(Rectangle* rect) = 0;
};

class Shape {
public:
    virtual ~Shape() = default;
    virtual void accept(Visitor* visitor) = 0;
};

class Circle : public Shape {
public:
    double radius;
    Circle(double r) : radius(r) {}
    
    void accept(Visitor* visitor) override {
        visitor->visitCircle(this);
    }
};

class Rectangle : public Shape {
public:
    double width, height;
    Rectangle(double w, double h) : width(w), height(h) {}
    
    void accept(Visitor* visitor) override {
        visitor->visitRectangle(this);
    }
};

class AreaVisitor : public Visitor {
public:
    double totalArea = 0;
    
    void visitCircle(Circle* circle) override {
        double area = 3.14159 * circle->radius * circle->radius;
        totalArea += area;
        std::cout << "Circle area: " << area << std::endl;
    }
    
    void visitRectangle(Rectangle* rect) override {
        double area = rect->width * rect->height;
        totalArea += area;
        std::cout << "Rectangle area: " << area << std::endl;
    }
};

int main() {
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(5));
    shapes.push_back(std::make_unique<Rectangle>(3, 4));
    shapes.push_back(std::make_unique<Circle>(2));
    
    AreaVisitor areaVisitor;
    for (auto& shape : shapes) {
        shape->accept(&areaVisitor);
    }
    std::cout << "Total area: " << areaVisitor.totalArea << std::endl;
    
    return 0;
}
```

---

## 双分派

访问者模式使用"双分派"技术：

1. 第一次分派：`shape.accept(visitor)` - 根据 shape 类型
2. 第二次分派：`visitor.visitCircle(this)` - 根据 visitor 类型

```
shape.accept(visitor)
      │
      ▼
Circle.accept() ───► visitor.visitCircle(this)
      或                    │
Rectangle.accept() ───► visitor.visitRectangle(this)
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 添加新操作容易 | 添加新元素困难 |
| 相关操作集中 | 可能破坏封装 |
| 累积状态 | |

### 何时使用

- 需要对复杂对象结构执行多种不相关操作
- 对象结构稳定，但经常需要添加新操作
- 需要遍历不同类型的对象并执行特定操作

---

[← 上一章：模板方法模式](../template-method/README.md) | [下一章：解释器模式 →](../interpreter/README.md)

