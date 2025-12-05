# 状态模式 (State Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。**

### 🔰 初学者理解

想象一个音乐播放器：
- **停止状态**：点击播放 → 开始播放
- **播放状态**：点击播放 → 暂停
- **暂停状态**：点击播放 → 继续播放

同样的"播放"按钮，在不同状态下有不同的行为。

---

## 代码实现

### Python 实现

```python
# state.py
from abc import ABC, abstractmethod

# ============================================
# 状态接口
# ============================================
class State(ABC):
    @abstractmethod
    def insert_coin(self, machine: 'VendingMachine') -> None:
        pass
    
    @abstractmethod
    def eject_coin(self, machine: 'VendingMachine') -> None:
        pass
    
    @abstractmethod
    def select_product(self, machine: 'VendingMachine') -> None:
        pass
    
    @abstractmethod
    def dispense(self, machine: 'VendingMachine') -> None:
        pass


# ============================================
# 具体状态
# ============================================
class NoCoinState(State):
    def insert_coin(self, machine: 'VendingMachine') -> None:
        print("Coin inserted")
        machine.set_state(machine.has_coin_state)
    
    def eject_coin(self, machine: 'VendingMachine') -> None:
        print("No coin to eject")
    
    def select_product(self, machine: 'VendingMachine') -> None:
        print("Please insert a coin first")
    
    def dispense(self, machine: 'VendingMachine') -> None:
        print("Please insert a coin first")


class HasCoinState(State):
    def insert_coin(self, machine: 'VendingMachine') -> None:
        print("Coin already inserted")
    
    def eject_coin(self, machine: 'VendingMachine') -> None:
        print("Coin returned")
        machine.set_state(machine.no_coin_state)
    
    def select_product(self, machine: 'VendingMachine') -> None:
        print("Product selected")
        machine.set_state(machine.sold_state)
    
    def dispense(self, machine: 'VendingMachine') -> None:
        print("Please select a product first")


class SoldState(State):
    def insert_coin(self, machine: 'VendingMachine') -> None:
        print("Please wait, dispensing product")
    
    def eject_coin(self, machine: 'VendingMachine') -> None:
        print("Sorry, already dispensing")
    
    def select_product(self, machine: 'VendingMachine') -> None:
        print("Already dispensing")
    
    def dispense(self, machine: 'VendingMachine') -> None:
        print("Dispensing product...")
        machine.release_product()
        if machine.count > 0:
            machine.set_state(machine.no_coin_state)
        else:
            print("Out of products!")
            machine.set_state(machine.sold_out_state)


class SoldOutState(State):
    def insert_coin(self, machine: 'VendingMachine') -> None:
        print("Sorry, sold out")
    
    def eject_coin(self, machine: 'VendingMachine') -> None:
        print("No coin inserted")
    
    def select_product(self, machine: 'VendingMachine') -> None:
        print("Sold out")
    
    def dispense(self, machine: 'VendingMachine') -> None:
        print("Sold out")


# ============================================
# 上下文 - 自动售货机
# ============================================
class VendingMachine:
    def __init__(self, count: int):
        self.no_coin_state = NoCoinState()
        self.has_coin_state = HasCoinState()
        self.sold_state = SoldState()
        self.sold_out_state = SoldOutState()
        
        self.count = count
        self._state = self.no_coin_state if count > 0 else self.sold_out_state
    
    def set_state(self, state: State) -> None:
        self._state = state
    
    def insert_coin(self) -> None:
        self._state.insert_coin(self)
    
    def eject_coin(self) -> None:
        self._state.eject_coin(self)
    
    def select_product(self) -> None:
        self._state.select_product(self)
        self._state.dispense(self)
    
    def release_product(self) -> None:
        if self.count > 0:
            self.count -= 1
            print(f"Product released! Remaining: {self.count}")


if __name__ == "__main__":
    machine = VendingMachine(2)
    
    print("=== Test 1: Normal purchase ===")
    machine.insert_coin()
    machine.select_product()
    
    print("\n=== Test 2: Eject coin ===")
    machine.insert_coin()
    machine.eject_coin()
    
    print("\n=== Test 3: No coin ===")
    machine.select_product()
    
    print("\n=== Test 4: Last product ===")
    machine.insert_coin()
    machine.select_product()
    
    print("\n=== Test 5: Sold out ===")
    machine.insert_coin()
```

### C++ 实现

```cpp
// state.cpp
#include <iostream>
#include <memory>

class Document;

class State {
public:
    virtual ~State() = default;
    virtual void publish(Document* doc) = 0;
    virtual std::string getName() const = 0;
};

class Draft : public State {
public:
    void publish(Document* doc) override;
    std::string getName() const override { return "Draft"; }
};

class Moderation : public State {
public:
    void publish(Document* doc) override;
    std::string getName() const override { return "Moderation"; }
};

class Published : public State {
public:
    void publish(Document* doc) override {
        std::cout << "Already published" << std::endl;
    }
    std::string getName() const override { return "Published"; }
};

class Document {
private:
    std::shared_ptr<State> state;
public:
    Document() : state(std::make_shared<Draft>()) {}
    
    void setState(std::shared_ptr<State> s) { state = s; }
    
    void publish() {
        std::cout << "Current state: " << state->getName() << std::endl;
        state->publish(this);
    }
};

void Draft::publish(Document* doc) {
    std::cout << "Moving to moderation" << std::endl;
    doc->setState(std::make_shared<Moderation>());
}

void Moderation::publish(Document* doc) {
    std::cout << "Publishing document" << std::endl;
    doc->setState(std::make_shared<Published>());
}

int main() {
    Document doc;
    doc.publish();  // Draft -> Moderation
    doc.publish();  // Moderation -> Published
    doc.publish();  // Already published
    return 0;
}
```

---

## 状态 vs 策略

| 状态模式 | 策略模式 |
|---------|---------|
| 状态之间知道彼此，可以触发转换 | 策略相互独立 |
| 状态改变对客户端透明 | 客户端选择策略 |
| 替代条件语句（状态相关） | 替代条件语句（算法选择） |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 消除复杂条件语句 | 状态多时类数量增加 |
| 单一职责 | |
| 开闭原则 | |

---

[← 上一章：观察者模式](../observer/README.md) | [下一章：策略模式 →](../strategy/README.md)

