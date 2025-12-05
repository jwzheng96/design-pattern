# 中介者模式 (Mediator Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散。**

### 🔰 初学者理解

想象机场塔台：
- 飞机不直接相互通信
- 所有通信通过塔台（中介者）协调
- 塔台知道所有飞机的状态，确保安全

中介者模式将网状通信变为星型通信，降低复杂度。

---

## 代码实现

### Python 实现

```python
# mediator.py
from abc import ABC, abstractmethod
from typing import List

# ============================================
# 中介者接口
# ============================================
class ChatMediator(ABC):
    @abstractmethod
    def send_message(self, message: str, sender: 'User') -> None:
        pass
    
    @abstractmethod
    def add_user(self, user: 'User') -> None:
        pass


# ============================================
# 具体中介者 - 聊天室
# ============================================
class ChatRoom(ChatMediator):
    def __init__(self, name: str):
        self.name = name
        self._users: List['User'] = []
    
    def add_user(self, user: 'User') -> None:
        self._users.append(user)
        print(f"[{self.name}] {user.name} joined the chat")
    
    def send_message(self, message: str, sender: 'User') -> None:
        print(f"[{self.name}] {sender.name}: {message}")
        for user in self._users:
            if user != sender:
                user.receive(message, sender.name)


# ============================================
# 同事类 - 用户
# ============================================
class User:
    def __init__(self, name: str, mediator: ChatMediator):
        self.name = name
        self._mediator = mediator
        mediator.add_user(self)
    
    def send(self, message: str) -> None:
        self._mediator.send_message(message, self)
    
    def receive(self, message: str, sender: str) -> None:
        print(f"  [{self.name} received from {sender}]: {message}")


if __name__ == "__main__":
    # 创建聊天室（中介者）
    room = ChatRoom("General")
    
    # 创建用户
    alice = User("Alice", room)
    bob = User("Bob", room)
    charlie = User("Charlie", room)
    
    # 发送消息（通过中介者）
    alice.send("Hello everyone!")
    bob.send("Hi Alice!")
```

### C++ 实现

```cpp
// mediator.cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>

class User;

class Mediator {
public:
    virtual ~Mediator() = default;
    virtual void sendMessage(const std::string& msg, User* sender) = 0;
    virtual void addUser(std::shared_ptr<User> user) = 0;
};

class User {
protected:
    std::string name;
    Mediator* mediator;
public:
    User(const std::string& n, Mediator* m) : name(n), mediator(m) {}
    virtual ~User() = default;
    
    std::string getName() const { return name; }
    
    void send(const std::string& message) {
        mediator->sendMessage(message, this);
    }
    
    virtual void receive(const std::string& message, const std::string& sender) {
        std::cout << "  [" << name << " received from " << sender << "]: " 
                  << message << std::endl;
    }
};

class ChatRoom : public Mediator {
private:
    std::vector<std::shared_ptr<User>> users;
public:
    void addUser(std::shared_ptr<User> user) override {
        users.push_back(user);
        std::cout << user->getName() << " joined" << std::endl;
    }
    
    void sendMessage(const std::string& msg, User* sender) override {
        std::cout << sender->getName() << ": " << msg << std::endl;
        for (auto& user : users) {
            if (user.get() != sender) {
                user->receive(msg, sender->getName());
            }
        }
    }
};

int main() {
    ChatRoom room;
    
    auto alice = std::make_shared<User>("Alice", &room);
    auto bob = std::make_shared<User>("Bob", &room);
    
    room.addUser(alice);
    room.addUser(bob);
    
    alice->send("Hello!");
    bob->send("Hi there!");
    
    return 0;
}
```

---

## 实战案例：GUI 组件协调

```python
# gui_mediator.py
from abc import ABC, abstractmethod

class DialogMediator(ABC):
    @abstractmethod
    def notify(self, sender: 'Component', event: str) -> None:
        pass

class Component:
    def __init__(self, mediator: DialogMediator = None):
        self._mediator = mediator
    
    def set_mediator(self, mediator: DialogMediator):
        self._mediator = mediator

class Button(Component):
    def click(self):
        print("Button clicked")
        if self._mediator:
            self._mediator.notify(self, "click")

class TextBox(Component):
    def __init__(self, mediator: DialogMediator = None):
        super().__init__(mediator)
        self._text = ""
    
    def set_text(self, text: str):
        self._text = text
        if self._mediator:
            self._mediator.notify(self, "text_changed")
    
    def get_text(self) -> str:
        return self._text

class Checkbox(Component):
    def __init__(self, mediator: DialogMediator = None):
        super().__init__(mediator)
        self._checked = False
    
    def check(self):
        self._checked = not self._checked
        if self._mediator:
            self._mediator.notify(self, "check")
    
    def is_checked(self) -> bool:
        return self._checked

class LoginDialog(DialogMediator):
    def __init__(self):
        self.username = TextBox(self)
        self.password = TextBox(self)
        self.remember_me = Checkbox(self)
        self.login_button = Button(self)
    
    def notify(self, sender: Component, event: str) -> None:
        if sender == self.login_button and event == "click":
            self._handle_login()
        elif event == "text_changed":
            self._validate_form()
    
    def _validate_form(self):
        valid = len(self.username.get_text()) > 0 and len(self.password.get_text()) > 0
        print(f"Form valid: {valid}")
    
    def _handle_login(self):
        print(f"Logging in as: {self.username.get_text()}")
        if self.remember_me.is_checked():
            print("Will remember login")


if __name__ == "__main__":
    dialog = LoginDialog()
    
    dialog.username.set_text("user@example.com")
    dialog.password.set_text("password123")
    dialog.remember_me.check()
    dialog.login_button.click()
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 减少组件间依赖 | 中介者可能变得复杂 |
| 易于复用组件 | |
| 简化对象协议 | |

### 中介者 vs 观察者

| 中介者 | 观察者 |
|--------|--------|
| 双向通信 | 单向通知 |
| 集中控制 | 分散订阅 |
| 组件无需知道彼此 | 观察者知道被观察者 |

---

[← 上一章：迭代器模式](../iterator/README.md) | [下一章：备忘录模式 →](../memento/README.md)

