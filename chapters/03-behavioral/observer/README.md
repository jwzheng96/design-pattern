# 观察者模式 (Observer Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知并被自动更新。**

### 🔰 初学者理解

就像订阅 YouTube 频道：
- 你订阅了某个 UP 主
- 当他发布新视频时，你会收到通知
- 你可以随时取消订阅

这是最常用的设计模式之一，也叫"发布-订阅"模式。

---

## 代码实现

### Python 实现

```python
# observer.py
from abc import ABC, abstractmethod
from typing import List, Dict, Any

# ============================================
# 观察者接口
# ============================================
class Observer(ABC):
    @abstractmethod
    def update(self, subject: 'Subject', *args, **kwargs) -> None:
        pass


# ============================================
# 主题（被观察者）
# ============================================
class Subject:
    def __init__(self):
        self._observers: List[Observer] = []
    
    def attach(self, observer: Observer) -> None:
        if observer not in self._observers:
            self._observers.append(observer)
    
    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)
    
    def notify(self, *args, **kwargs) -> None:
        for observer in self._observers:
            observer.update(self, *args, **kwargs)


# ============================================
# 具体主题 - 股票价格
# ============================================
class Stock(Subject):
    def __init__(self, symbol: str, price: float):
        super().__init__()
        self._symbol = symbol
        self._price = price
    
    @property
    def symbol(self) -> str:
        return self._symbol
    
    @property
    def price(self) -> float:
        return self._price
    
    @price.setter
    def price(self, value: float) -> None:
        old_price = self._price
        self._price = value
        self.notify(old_price=old_price, new_price=value)


# ============================================
# 具体观察者
# ============================================
class Investor(Observer):
    def __init__(self, name: str):
        self._name = name
    
    def update(self, subject: Subject, **kwargs) -> None:
        if isinstance(subject, Stock):
            print(f"[{self._name}] Stock {subject.symbol}: "
                  f"${kwargs['old_price']:.2f} -> ${kwargs['new_price']:.2f}")


class TradingBot(Observer):
    def __init__(self, name: str, buy_threshold: float, sell_threshold: float):
        self._name = name
        self._buy_threshold = buy_threshold
        self._sell_threshold = sell_threshold
    
    def update(self, subject: Subject, **kwargs) -> None:
        if isinstance(subject, Stock):
            new_price = kwargs['new_price']
            if new_price < self._buy_threshold:
                print(f"[{self._name}] BUY signal for {subject.symbol} at ${new_price:.2f}")
            elif new_price > self._sell_threshold:
                print(f"[{self._name}] SELL signal for {subject.symbol} at ${new_price:.2f}")


if __name__ == "__main__":
    # 创建股票
    apple = Stock("AAPL", 150.0)
    
    # 创建观察者
    investor1 = Investor("Alice")
    investor2 = Investor("Bob")
    bot = TradingBot("AutoTrader", buy_threshold=140, sell_threshold=160)
    
    # 订阅
    apple.attach(investor1)
    apple.attach(investor2)
    apple.attach(bot)
    
    # 价格变化
    print("--- Price changes ---")
    apple.price = 155.0
    apple.price = 165.0  # 触发卖出信号
    apple.price = 135.0  # 触发买入信号
    
    # 取消订阅
    print("\n--- Bob unsubscribes ---")
    apple.detach(investor2)
    apple.price = 145.0
```

### C++ 实现

```cpp
// observer.cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <memory>

class Subject;

class Observer {
public:
    virtual ~Observer() = default;
    virtual void update(Subject* subject) = 0;
};

class Subject {
protected:
    std::vector<Observer*> observers;
public:
    virtual ~Subject() = default;
    
    void attach(Observer* observer) {
        observers.push_back(observer);
    }
    
    void detach(Observer* observer) {
        observers.erase(
            std::remove(observers.begin(), observers.end(), observer),
            observers.end()
        );
    }
    
    void notify() {
        for (auto* observer : observers) {
            observer->update(this);
        }
    }
};

class WeatherStation : public Subject {
private:
    float temperature;
    float humidity;
public:
    void setMeasurements(float temp, float hum) {
        temperature = temp;
        humidity = hum;
        notify();
    }
    
    float getTemperature() const { return temperature; }
    float getHumidity() const { return humidity; }
};

class Display : public Observer {
private:
    std::string name;
public:
    Display(const std::string& n) : name(n) {}
    
    void update(Subject* subject) override {
        auto* weather = dynamic_cast<WeatherStation*>(subject);
        if (weather) {
            std::cout << "[" << name << "] Temperature: " << weather->getTemperature()
                      << "°C, Humidity: " << weather->getHumidity() << "%" << std::endl;
        }
    }
};

int main() {
    WeatherStation station;
    
    Display phoneDisplay("Phone");
    Display windowDisplay("Window");
    
    station.attach(&phoneDisplay);
    station.attach(&windowDisplay);
    
    station.setMeasurements(25.5, 65);
    station.setMeasurements(28.0, 70);
    
    station.detach(&windowDisplay);
    station.setMeasurements(22.0, 55);
    
    return 0;
}
```

---

## 实战案例：事件系统

```python
# event_system.py
from typing import Callable, Dict, List, Any

class EventEmitter:
    """通用事件发射器"""
    
    def __init__(self):
        self._listeners: Dict[str, List[Callable]] = {}
    
    def on(self, event: str, callback: Callable) -> 'EventEmitter':
        """订阅事件"""
        if event not in self._listeners:
            self._listeners[event] = []
        self._listeners[event].append(callback)
        return self
    
    def off(self, event: str, callback: Callable) -> 'EventEmitter':
        """取消订阅"""
        if event in self._listeners:
            self._listeners[event].remove(callback)
        return self
    
    def emit(self, event: str, *args, **kwargs) -> None:
        """发射事件"""
        if event in self._listeners:
            for callback in self._listeners[event]:
                callback(*args, **kwargs)
    
    def once(self, event: str, callback: Callable) -> 'EventEmitter':
        """订阅一次性事件"""
        def wrapper(*args, **kwargs):
            callback(*args, **kwargs)
            self.off(event, wrapper)
        return self.on(event, wrapper)


class User:
    def __init__(self, name: str):
        self.name = name
        self.events = EventEmitter()
    
    def login(self):
        print(f"{self.name} logged in")
        self.events.emit("login", user=self)
    
    def logout(self):
        print(f"{self.name} logged out")
        self.events.emit("logout", user=self)


if __name__ == "__main__":
    user = User("Alice")
    
    # 订阅事件
    user.events.on("login", lambda user: print(f"  -> Welcome back, {user.name}!"))
    user.events.on("login", lambda user: print(f"  -> Loading preferences..."))
    user.events.on("logout", lambda user: print(f"  -> Goodbye, {user.name}!"))
    
    # 触发事件
    user.login()
    user.logout()
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 松耦合 | 通知顺序不确定 |
| 支持广播通信 | 可能导致意外更新 |
| 符合开闭原则 | 观察者过多影响性能 |

### 何时使用

- 一个对象状态改变需要通知其他对象
- 不知道有多少对象需要被通知
- 对象间需要松耦合

---

[← 上一章：备忘录模式](../memento/README.md) | [下一章：状态模式 →](../state/README.md)

