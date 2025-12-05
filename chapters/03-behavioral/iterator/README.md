# 迭代器模式 (Iterator Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。**

### 🔰 初学者理解

迭代器就像图书馆管理员：你不需要知道书架如何组织，只需要说"给我下一本书"，管理员会帮你取来。

**好消息**：Python 和 C++ 已经内置了迭代器支持，但理解原理有助于自定义迭代行为。

---

## 代码实现

### Python 实现

```python
# iterator.py
from typing import Iterator, Iterable, List, Any

# ============================================
# 自定义迭代器类
# ============================================
class BookIterator:
    """书籍迭代器"""
    
    def __init__(self, books: List[str]):
        self._books = books
        self._index = 0
    
    def __iter__(self):
        return self
    
    def __next__(self) -> str:
        if self._index >= len(self._books):
            raise StopIteration
        book = self._books[self._index]
        self._index += 1
        return book


class BookShelf:
    """书架 - 可迭代对象"""
    
    def __init__(self):
        self._books: List[str] = []
    
    def add_book(self, book: str):
        self._books.append(book)
    
    def __iter__(self) -> BookIterator:
        return BookIterator(self._books)
    
    def reverse_iter(self) -> Iterator[str]:
        """返回反向迭代器"""
        return reversed(self._books)


# ============================================
# 使用生成器简化迭代器
# ============================================
class Tree:
    """二叉树节点"""
    
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
    
    def inorder(self):
        """中序遍历生成器"""
        if self.left:
            yield from self.left.inorder()
        yield self.value
        if self.right:
            yield from self.right.inorder()
    
    def preorder(self):
        """前序遍历生成器"""
        yield self.value
        if self.left:
            yield from self.left.preorder()
        if self.right:
            yield from self.right.preorder()


if __name__ == "__main__":
    # 使用书架迭代器
    shelf = BookShelf()
    shelf.add_book("Design Patterns")
    shelf.add_book("Clean Code")
    shelf.add_book("Refactoring")
    
    print("Books on shelf:")
    for book in shelf:
        print(f"  - {book}")
    
    print("\nBooks in reverse:")
    for book in shelf.reverse_iter():
        print(f"  - {book}")
    
    # 使用树迭代器
    tree = Tree(4,
        Tree(2, Tree(1), Tree(3)),
        Tree(6, Tree(5), Tree(7))
    )
    
    print("\nTree inorder:")
    print(list(tree.inorder()))
    
    print("\nTree preorder:")
    print(list(tree.preorder()))
```

### C++ 实现

```cpp
// iterator.cpp
#include <iostream>
#include <vector>
#include <string>

template<typename T>
class CustomIterator {
private:
    T* ptr;
public:
    CustomIterator(T* p) : ptr(p) {}
    
    CustomIterator& operator++() {
        ++ptr;
        return *this;
    }
    
    T& operator*() { return *ptr; }
    
    bool operator!=(const CustomIterator& other) const {
        return ptr != other.ptr;
    }
};

template<typename T>
class Container {
private:
    std::vector<T> data;

public:
    void add(const T& item) {
        data.push_back(item);
    }
    
    // 标准库风格的迭代器
    using iterator = typename std::vector<T>::iterator;
    
    iterator begin() { return data.begin(); }
    iterator end() { return data.end(); }
};

int main() {
    Container<std::string> container;
    container.add("Apple");
    container.add("Banana");
    container.add("Cherry");
    
    // 使用范围 for 循环
    for (const auto& item : container) {
        std::cout << item << std::endl;
    }
    
    return 0;
}
```

---

## Python 迭代协议

```python
# Python 迭代器协议
class MyRange:
    """自定义 range 实现"""
    
    def __init__(self, start: int, end: int, step: int = 1):
        self.start = start
        self.end = end
        self.step = step
    
    def __iter__(self):
        current = self.start
        while current < self.end:
            yield current
            current += self.step

# 使用
for i in MyRange(0, 10, 2):
    print(i)  # 0, 2, 4, 6, 8
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 简化集合接口 | 简单集合可能过度设计 |
| 支持多种遍历方式 | |
| 隐藏内部实现 | |

### 现代语言支持

- **Python**: `__iter__`, `__next__`, 生成器
- **C++**: STL 迭代器, 范围 for 循环
- **Java**: `Iterator`, `Iterable` 接口

---

[← 上一章：命令模式](../command/README.md) | [下一章：中介者模式 →](../mediator/README.md)

