# 命令模式 (Command Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **将一个请求封装成一个对象，从而使你可以用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作。**

### 🔰 初学者理解

想象餐厅点餐：
- 你（客户端）告诉服务员（调用者）你想要什么
- 服务员把订单（命令）写在纸上
- 厨师（接收者）根据订单做菜

订单就是"命令"——它封装了你的请求，可以排队、取消、重做。

---

## 代码实现

### Python 实现

```python
# command.py
from abc import ABC, abstractmethod
from typing import List
from dataclasses import dataclass, field

# ============================================
# 命令接口
# ============================================
class Command(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass
    
    @abstractmethod
    def undo(self) -> None:
        pass


# ============================================
# 接收者 - 文本编辑器
# ============================================
class TextEditor:
    def __init__(self):
        self._text = ""
    
    def insert(self, text: str, position: int) -> None:
        self._text = self._text[:position] + text + self._text[position:]
    
    def delete(self, position: int, length: int) -> str:
        deleted = self._text[position:position + length]
        self._text = self._text[:position] + self._text[position + length:]
        return deleted
    
    def get_text(self) -> str:
        return self._text
    
    def __repr__(self):
        return f'TextEditor("{self._text}")'


# ============================================
# 具体命令
# ============================================
class InsertCommand(Command):
    def __init__(self, editor: TextEditor, text: str, position: int):
        self._editor = editor
        self._text = text
        self._position = position
    
    def execute(self) -> None:
        self._editor.insert(self._text, self._position)
    
    def undo(self) -> None:
        self._editor.delete(self._position, len(self._text))


class DeleteCommand(Command):
    def __init__(self, editor: TextEditor, position: int, length: int):
        self._editor = editor
        self._position = position
        self._length = length
        self._deleted_text = ""
    
    def execute(self) -> None:
        self._deleted_text = self._editor.delete(self._position, self._length)
    
    def undo(self) -> None:
        self._editor.insert(self._deleted_text, self._position)


# ============================================
# 调用者 - 命令管理器（支持撤销/重做）
# ============================================
class CommandManager:
    def __init__(self):
        self._history: List[Command] = []
        self._undo_stack: List[Command] = []
    
    def execute(self, command: Command) -> None:
        command.execute()
        self._history.append(command)
        self._undo_stack.clear()  # 新操作清空重做栈
    
    def undo(self) -> bool:
        if not self._history:
            return False
        command = self._history.pop()
        command.undo()
        self._undo_stack.append(command)
        return True
    
    def redo(self) -> bool:
        if not self._undo_stack:
            return False
        command = self._undo_stack.pop()
        command.execute()
        self._history.append(command)
        return True


if __name__ == "__main__":
    editor = TextEditor()
    manager = CommandManager()
    
    # 执行命令
    manager.execute(InsertCommand(editor, "Hello", 0))
    print(f"After insert 'Hello': {editor.get_text()}")
    
    manager.execute(InsertCommand(editor, " World", 5))
    print(f"After insert ' World': {editor.get_text()}")
    
    manager.execute(DeleteCommand(editor, 5, 6))
    print(f"After delete: {editor.get_text()}")
    
    # 撤销
    manager.undo()
    print(f"After undo: {editor.get_text()}")
    
    manager.undo()
    print(f"After undo: {editor.get_text()}")
    
    # 重做
    manager.redo()
    print(f"After redo: {editor.get_text()}")
```

### C++ 实现

```cpp
// command.cpp
#include <iostream>
#include <memory>
#include <stack>
#include <string>

class Light {
public:
    void on() { std::cout << "Light is ON" << std::endl; }
    void off() { std::cout << "Light is OFF" << std::endl; }
};

class Command {
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};

class LightOnCommand : public Command {
private:
    Light* light;
public:
    LightOnCommand(Light* l) : light(l) {}
    void execute() override { light->on(); }
    void undo() override { light->off(); }
};

class LightOffCommand : public Command {
private:
    Light* light;
public:
    LightOffCommand(Light* l) : light(l) {}
    void execute() override { light->off(); }
    void undo() override { light->on(); }
};

class RemoteControl {
private:
    std::stack<std::shared_ptr<Command>> history;
public:
    void executeCommand(std::shared_ptr<Command> cmd) {
        cmd->execute();
        history.push(cmd);
    }
    
    void undo() {
        if (!history.empty()) {
            history.top()->undo();
            history.pop();
        }
    }
};

int main() {
    Light light;
    RemoteControl remote;
    
    remote.executeCommand(std::make_shared<LightOnCommand>(&light));
    remote.executeCommand(std::make_shared<LightOffCommand>(&light));
    remote.undo();
    remote.undo();
    
    return 0;
}
```

---

## 实战案例：任务队列

```python
# task_queue.py
import time
from abc import ABC, abstractmethod
from queue import Queue
from threading import Thread
from typing import Any

class Task(ABC):
    @abstractmethod
    def execute(self) -> Any:
        pass

class SendEmailTask(Task):
    def __init__(self, to: str, subject: str, body: str):
        self.to = to
        self.subject = subject
        self.body = body
    
    def execute(self) -> Any:
        print(f"Sending email to {self.to}: {self.subject}")
        time.sleep(0.5)  # 模拟发送
        return True

class ProcessDataTask(Task):
    def __init__(self, data: list):
        self.data = data
    
    def execute(self) -> Any:
        print(f"Processing {len(self.data)} items...")
        result = [x * 2 for x in self.data]
        return result

class TaskQueue:
    def __init__(self):
        self._queue = Queue()
        self._running = False
    
    def add_task(self, task: Task):
        self._queue.put(task)
    
    def start(self):
        self._running = True
        Thread(target=self._process_tasks, daemon=True).start()
    
    def _process_tasks(self):
        while self._running:
            if not self._queue.empty():
                task = self._queue.get()
                result = task.execute()
                print(f"Task completed with result: {result}")
            time.sleep(0.1)


if __name__ == "__main__":
    queue = TaskQueue()
    queue.start()
    
    queue.add_task(SendEmailTask("user@example.com", "Hello", "Test email"))
    queue.add_task(ProcessDataTask([1, 2, 3, 4, 5]))
    queue.add_task(SendEmailTask("admin@example.com", "Report", "Daily report"))
    
    time.sleep(3)
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 解耦调用者与接收者 | 可能产生大量命令类 |
| 支持撤销/重做 | |
| 支持组合命令（宏命令） | |
| 支持延迟执行和队列 | |

### 何时使用

- 需要支持撤销/重做
- 需要将操作放入队列执行
- 需要支持事务
- 需要记录操作日志

---

[← 上一章：责任链模式](../chain-of-responsibility/README.md) | [下一章：迭代器模式 →](../iterator/README.md)

