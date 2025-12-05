# 策略模式 (Strategy Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。策略模式使得算法可以独立于使用它的客户端而变化。**

### 🔰 初学者理解

想象导航 App 的路线规划：
- 开车：走高速公路
- 步行：走人行道
- 骑车：走自行车道
- 公交：查找公交路线

同样是"规划路线"，不同的交通方式（策略）有不同的算法。

---

## 代码实现

### Python 实现

```python
# strategy.py
from abc import ABC, abstractmethod
from typing import List

# ============================================
# 策略接口
# ============================================
class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> bool:
        pass
    
    @abstractmethod
    def get_name(self) -> str:
        pass


# ============================================
# 具体策略
# ============================================
class CreditCardPayment(PaymentStrategy):
    def __init__(self, card_number: str, cvv: str):
        self._card_number = card_number
        self._cvv = cvv
    
    def pay(self, amount: float) -> bool:
        print(f"Paying ${amount:.2f} with Credit Card ending in {self._card_number[-4:]}")
        return True
    
    def get_name(self) -> str:
        return "Credit Card"


class PayPalPayment(PaymentStrategy):
    def __init__(self, email: str):
        self._email = email
    
    def pay(self, amount: float) -> bool:
        print(f"Paying ${amount:.2f} via PayPal ({self._email})")
        return True
    
    def get_name(self) -> str:
        return "PayPal"


class CryptoPayment(PaymentStrategy):
    def __init__(self, wallet_address: str):
        self._wallet = wallet_address
    
    def pay(self, amount: float) -> bool:
        print(f"Paying ${amount:.2f} in crypto to {self._wallet[:10]}...")
        return True
    
    def get_name(self) -> str:
        return "Cryptocurrency"


# ============================================
# 上下文
# ============================================
class ShoppingCart:
    def __init__(self):
        self._items: List[tuple] = []
        self._payment_strategy: PaymentStrategy = None
    
    def add_item(self, name: str, price: float) -> None:
        self._items.append((name, price))
    
    def get_total(self) -> float:
        return sum(price for _, price in self._items)
    
    def set_payment_strategy(self, strategy: PaymentStrategy) -> None:
        self._payment_strategy = strategy
    
    def checkout(self) -> bool:
        if not self._payment_strategy:
            print("Please select a payment method")
            return False
        
        total = self.get_total()
        print(f"\nOrder total: ${total:.2f}")
        print(f"Payment method: {self._payment_strategy.get_name()}")
        return self._payment_strategy.pay(total)


if __name__ == "__main__":
    cart = ShoppingCart()
    cart.add_item("Laptop", 999.99)
    cart.add_item("Mouse", 29.99)
    cart.add_item("Keyboard", 79.99)
    
    # 使用信用卡支付
    print("=== Credit Card Payment ===")
    cart.set_payment_strategy(CreditCardPayment("1234567890123456", "123"))
    cart.checkout()
    
    # 切换到 PayPal
    print("\n=== PayPal Payment ===")
    cart.set_payment_strategy(PayPalPayment("user@example.com"))
    cart.checkout()
    
    # 切换到加密货币
    print("\n=== Crypto Payment ===")
    cart.set_payment_strategy(CryptoPayment("0x1234abcd5678efgh"))
    cart.checkout()
```

### C++ 实现

```cpp
// strategy.cpp
#include <iostream>
#include <memory>
#include <vector>
#include <algorithm>

class SortStrategy {
public:
    virtual ~SortStrategy() = default;
    virtual void sort(std::vector<int>& data) = 0;
    virtual std::string getName() const = 0;
};

class BubbleSort : public SortStrategy {
public:
    void sort(std::vector<int>& data) override {
        for (size_t i = 0; i < data.size(); ++i) {
            for (size_t j = 0; j < data.size() - i - 1; ++j) {
                if (data[j] > data[j + 1]) {
                    std::swap(data[j], data[j + 1]);
                }
            }
        }
    }
    std::string getName() const override { return "Bubble Sort"; }
};

class QuickSort : public SortStrategy {
public:
    void sort(std::vector<int>& data) override {
        std::sort(data.begin(), data.end());
    }
    std::string getName() const override { return "Quick Sort"; }
};

class Sorter {
private:
    std::unique_ptr<SortStrategy> strategy;
public:
    void setStrategy(std::unique_ptr<SortStrategy> s) {
        strategy = std::move(s);
    }
    
    void sort(std::vector<int>& data) {
        if (strategy) {
            std::cout << "Sorting using " << strategy->getName() << std::endl;
            strategy->sort(data);
        }
    }
};

int main() {
    std::vector<int> data = {64, 34, 25, 12, 22, 11, 90};
    
    Sorter sorter;
    
    sorter.setStrategy(std::make_unique<BubbleSort>());
    sorter.sort(data);
    
    data = {64, 34, 25, 12, 22, 11, 90};
    sorter.setStrategy(std::make_unique<QuickSort>());
    sorter.sort(data);
    
    return 0;
}
```

---

## 实战案例：数据压缩

```python
# compression_strategy.py
from abc import ABC, abstractmethod
import zlib
import base64

class CompressionStrategy(ABC):
    @abstractmethod
    def compress(self, data: bytes) -> bytes:
        pass
    
    @abstractmethod
    def decompress(self, data: bytes) -> bytes:
        pass

class ZlibCompression(CompressionStrategy):
    def compress(self, data: bytes) -> bytes:
        return zlib.compress(data)
    
    def decompress(self, data: bytes) -> bytes:
        return zlib.decompress(data)

class Base64Encoding(CompressionStrategy):
    def compress(self, data: bytes) -> bytes:
        return base64.b64encode(data)
    
    def decompress(self, data: bytes) -> bytes:
        return base64.b64decode(data)

class NoCompression(CompressionStrategy):
    def compress(self, data: bytes) -> bytes:
        return data
    
    def decompress(self, data: bytes) -> bytes:
        return data

class DataProcessor:
    def __init__(self, strategy: CompressionStrategy = None):
        self._strategy = strategy or NoCompression()
    
    def set_strategy(self, strategy: CompressionStrategy):
        self._strategy = strategy
    
    def process(self, data: str) -> bytes:
        return self._strategy.compress(data.encode())
    
    def restore(self, data: bytes) -> str:
        return self._strategy.decompress(data).decode()


if __name__ == "__main__":
    original = "Hello, World! " * 100
    processor = DataProcessor()
    
    # 无压缩
    processor.set_strategy(NoCompression())
    compressed = processor.process(original)
    print(f"No compression: {len(compressed)} bytes")
    
    # Zlib 压缩
    processor.set_strategy(ZlibCompression())
    compressed = processor.process(original)
    print(f"Zlib: {len(compressed)} bytes")
    
    # 验证解压
    restored = processor.restore(compressed)
    print(f"Restored correctly: {restored == original}")
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 运行时切换算法 | 客户端需要了解不同策略 |
| 开闭原则 | 策略数量多时管理复杂 |
| 消除条件语句 | |
| 算法可复用 | |

### 何时使用

- 需要使用算法的不同变体
- 需要在运行时切换算法
- 有很多相似的类，仅行为不同

---

[← 上一章：状态模式](../state/README.md) | [下一章：模板方法模式 →](../template-method/README.md)

