# 备忘录模式 (Memento Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，以便以后可以将该对象恢复到原先保存的状态。**

### 🔰 初学者理解

就像游戏存档：
- 你可以在关键点保存游戏状态
- 如果失败了，可以读取存档回到之前的状态
- 存档文件不需要知道游戏的内部实现细节

---

## 代码实现

### Python 实现

```python
# memento.py
from dataclasses import dataclass
from typing import List
from datetime import datetime
from copy import deepcopy

# ============================================
# 备忘录 - 存储状态
# ============================================
@dataclass
class EditorMemento:
    """编辑器状态备忘录"""
    content: str
    cursor_position: int
    timestamp: datetime
    
    def get_description(self) -> str:
        preview = self.content[:20] + "..." if len(self.content) > 20 else self.content
        return f"[{self.timestamp.strftime('%H:%M:%S')}] '{preview}'"


# ============================================
# 原发器 - 创建和恢复备忘录
# ============================================
class TextEditor:
    """文本编辑器（原发器）"""
    
    def __init__(self):
        self._content = ""
        self._cursor_position = 0
    
    def type(self, text: str) -> None:
        self._content = (self._content[:self._cursor_position] + 
                        text + 
                        self._content[self._cursor_position:])
        self._cursor_position += len(text)
    
    def delete(self, count: int = 1) -> None:
        if self._cursor_position >= count:
            self._content = (self._content[:self._cursor_position - count] + 
                           self._content[self._cursor_position:])
            self._cursor_position -= count
    
    def move_cursor(self, position: int) -> None:
        self._cursor_position = max(0, min(position, len(self._content)))
    
    def get_content(self) -> str:
        return self._content
    
    # 备忘录相关方法
    def save(self) -> EditorMemento:
        """创建备忘录"""
        return EditorMemento(
            content=self._content,
            cursor_position=self._cursor_position,
            timestamp=datetime.now()
        )
    
    def restore(self, memento: EditorMemento) -> None:
        """从备忘录恢复"""
        self._content = memento.content
        self._cursor_position = memento.cursor_position


# ============================================
# 管理者 - 管理备忘录
# ============================================
class History:
    """历史记录管理器（管理者）"""
    
    def __init__(self, editor: TextEditor):
        self._editor = editor
        self._history: List[EditorMemento] = []
        self._current = -1
    
    def backup(self) -> None:
        """保存当前状态"""
        # 如果在历史中间，删除之后的记录
        self._history = self._history[:self._current + 1]
        self._history.append(self._editor.save())
        self._current += 1
    
    def undo(self) -> bool:
        """撤销"""
        if self._current <= 0:
            return False
        self._current -= 1
        self._editor.restore(self._history[self._current])
        return True
    
    def redo(self) -> bool:
        """重做"""
        if self._current >= len(self._history) - 1:
            return False
        self._current += 1
        self._editor.restore(self._history[self._current])
        return True
    
    def show_history(self) -> None:
        """显示历史记录"""
        print("History:")
        for i, memento in enumerate(self._history):
            marker = " <-- current" if i == self._current else ""
            print(f"  {i}: {memento.get_description()}{marker}")


if __name__ == "__main__":
    editor = TextEditor()
    history = History(editor)
    
    # 初始状态
    history.backup()
    
    # 编辑操作
    editor.type("Hello")
    history.backup()
    print(f"Content: '{editor.get_content()}'")
    
    editor.type(" World")
    history.backup()
    print(f"Content: '{editor.get_content()}'")
    
    editor.type("!")
    history.backup()
    print(f"Content: '{editor.get_content()}'")
    
    # 显示历史
    history.show_history()
    
    # 撤销
    print("\n--- Undo ---")
    history.undo()
    print(f"Content: '{editor.get_content()}'")
    
    history.undo()
    print(f"Content: '{editor.get_content()}'")
    
    # 重做
    print("\n--- Redo ---")
    history.redo()
    print(f"Content: '{editor.get_content()}'")
```

### C++ 实现

```cpp
// memento.cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>

class Memento {
private:
    std::string state;
    friend class Originator;  // 只有原发器可以访问
    
    Memento(const std::string& s) : state(s) {}
    std::string getState() const { return state; }
};

class Originator {
private:
    std::string state;
public:
    void setState(const std::string& s) {
        state = s;
        std::cout << "State set to: " << state << std::endl;
    }
    
    std::string getState() const { return state; }
    
    std::shared_ptr<Memento> save() {
        return std::shared_ptr<Memento>(new Memento(state));
    }
    
    void restore(std::shared_ptr<Memento> memento) {
        state = memento->getState();
        std::cout << "State restored to: " << state << std::endl;
    }
};

class Caretaker {
private:
    std::vector<std::shared_ptr<Memento>> history;
    Originator* originator;
public:
    Caretaker(Originator* o) : originator(o) {}
    
    void backup() {
        history.push_back(originator->save());
    }
    
    void undo() {
        if (history.empty()) return;
        auto memento = history.back();
        history.pop_back();
        originator->restore(memento);
    }
};

int main() {
    Originator originator;
    Caretaker caretaker(&originator);
    
    originator.setState("State 1");
    caretaker.backup();
    
    originator.setState("State 2");
    caretaker.backup();
    
    originator.setState("State 3");
    
    std::cout << "\n--- Undo ---\n";
    caretaker.undo();
    caretaker.undo();
    
    return 0;
}
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 保持封装边界 | 可能消耗大量内存 |
| 简化原发器 | 管理者需要跟踪生命周期 |

### 何时使用

- 需要实现撤销/重做功能
- 需要保存对象快照
- 直接访问对象状态会暴露实现细节

---

[← 上一章：中介者模式](../mediator/README.md) | [下一章：观察者模式 →](../observer/README.md)

