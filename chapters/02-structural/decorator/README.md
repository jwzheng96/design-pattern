# 装饰器模式 (Decorator Pattern)

[← 返回结构型模式](../README.md) | [返回目录](../../../README.md)

---

## 📚 目录

- [意图与动机](#意图与动机)
- [问题场景](#问题场景)
- [解决方案](#解决方案)
- [结构](#结构)
- [代码实现](#代码实现)
- [初学者指南](#初学者指南)
- [高级应用](#高级应用)
- [实战案例](#实战案例)
- [相关模式](#相关模式)

---

## 意图与动机

### 一句话定义

> **动态地给一个对象添加一些额外的职责，就增加功能来说，装饰器模式比生成子类更为灵活。**

### 🔰 初学者理解

想象咖啡店：
- 基础咖啡（浓缩咖啡）
- 可以加奶 → 拿铁
- 可以加巧克力 → 摩卡
- 可以加奶 + 巧克力 → 摩卡拿铁

每种配料都是一个"装饰器"，可以**自由组合**来增强基础咖啡。

### 🚀 高级理解

装饰器模式通过**包装**对象来扩展其行为：
- 保持原有接口不变
- 可以在运行时动态添加功能
- 避免继承带来的类爆炸问题

---

## 问题场景

### 场景：消息通知系统

你需要一个通知系统，支持多种渠道（邮件、短信、Slack）和多种处理（加密、压缩、日志）。

使用继承会产生大量子类：
- EmailNotifier
- SMSNotifier
- EncryptedEmailNotifier
- CompressedEncryptedEmailNotifier
- LoggedCompressedEncryptedSMSNotifier
- ...

---

## 解决方案

```
┌─────────────────────────────────────────────────────────────────────┐
│                        装饰器模式结构                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                      Component                                       │
│                    ┌───────────┐                                    │
│                    │+operation()│                                    │
│                    └─────▲─────┘                                    │
│                          │                                          │
│           ┌──────────────┴──────────────┐                          │
│           │                             │                          │
│    ┌──────┴──────┐            ┌─────────┴─────────┐                │
│    │Concrete     │            │    Decorator      │                │
│    │Component    │            │- component        │                │
│    │+operation() │            │+operation()       │◆───────┐       │
│    └─────────────┘            └─────────▲─────────┘        │       │
│                                         │                  │       │
│                               ┌─────────┴─────────┐        │       │
│                               │                   │        │       │
│                        ┌──────┴─────┐      ┌──────┴─────┐  │       │
│                        │DecoratorA  │      │DecoratorB  │  │       │
│                        │+operation()│      │+operation()│──┘       │
│                        └────────────┘      └────────────┘          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 代码实现

### C++ 实现

```cpp
// decorator.cpp
#include <iostream>
#include <memory>
#include <string>

// ============================================
// 组件接口
// ============================================
class DataSource {
public:
    virtual ~DataSource() = default;
    virtual void writeData(const std::string& data) = 0;
    virtual std::string readData() = 0;
};

// ============================================
// 具体组件
// ============================================
class FileDataSource : public DataSource {
private:
    std::string filename;
    std::string content;  // 模拟文件内容

public:
    FileDataSource(const std::string& filename) : filename(filename) {}
    
    void writeData(const std::string& data) override {
        content = data;
        std::cout << "Writing to file " << filename << ": " << data << std::endl;
    }
    
    std::string readData() override {
        std::cout << "Reading from file " << filename << std::endl;
        return content;
    }
};

// ============================================
// 装饰器基类
// ============================================
class DataSourceDecorator : public DataSource {
protected:
    std::shared_ptr<DataSource> wrappee;

public:
    DataSourceDecorator(std::shared_ptr<DataSource> source) : wrappee(source) {}
    
    void writeData(const std::string& data) override {
        wrappee->writeData(data);
    }
    
    std::string readData() override {
        return wrappee->readData();
    }
};

// ============================================
// 具体装饰器 - 加密
// ============================================
class EncryptionDecorator : public DataSourceDecorator {
public:
    using DataSourceDecorator::DataSourceDecorator;
    
    void writeData(const std::string& data) override {
        std::string encrypted = encrypt(data);
        wrappee->writeData(encrypted);
    }
    
    std::string readData() override {
        std::string data = wrappee->readData();
        return decrypt(data);
    }

private:
    std::string encrypt(const std::string& data) {
        // 简单的加密：每个字符 +1
        std::string result = data;
        for (char& c : result) {
            c = c + 1;
        }
        std::cout << "Encrypted: " << data << " -> " << result << std::endl;
        return result;
    }
    
    std::string decrypt(const std::string& data) {
        std::string result = data;
        for (char& c : result) {
            c = c - 1;
        }
        std::cout << "Decrypted: " << data << " -> " << result << std::endl;
        return result;
    }
};

// ============================================
// 具体装饰器 - 压缩
// ============================================
class CompressionDecorator : public DataSourceDecorator {
public:
    using DataSourceDecorator::DataSourceDecorator;
    
    void writeData(const std::string& data) override {
        std::string compressed = compress(data);
        wrappee->writeData(compressed);
    }
    
    std::string readData() override {
        std::string data = wrappee->readData();
        return decompress(data);
    }

private:
    std::string compress(const std::string& data) {
        // 简单模拟压缩
        std::cout << "Compressing data..." << std::endl;
        return "[COMPRESSED]" + data;
    }
    
    std::string decompress(const std::string& data) {
        std::cout << "Decompressing data..." << std::endl;
        if (data.substr(0, 12) == "[COMPRESSED]") {
            return data.substr(12);
        }
        return data;
    }
};

int main() {
    // 基本文件数据源
    std::cout << "=== Simple File ===" << std::endl;
    auto source = std::make_shared<FileDataSource>("data.txt");
    source->writeData("Hello World");
    std::cout << "Read: " << source->readData() << std::endl;
    
    // 加密文件数据源
    std::cout << "\n=== Encrypted File ===" << std::endl;
    auto encryptedSource = std::make_shared<EncryptionDecorator>(
        std::make_shared<FileDataSource>("encrypted.txt")
    );
    encryptedSource->writeData("Secret Message");
    std::cout << "Read: " << encryptedSource->readData() << std::endl;
    
    // 压缩 + 加密文件数据源
    std::cout << "\n=== Compressed + Encrypted File ===" << std::endl;
    auto fullSource = std::make_shared<CompressionDecorator>(
        std::make_shared<EncryptionDecorator>(
            std::make_shared<FileDataSource>("secure.txt")
        )
    );
    fullSource->writeData("Top Secret Data");
    std::cout << "Read: " << fullSource->readData() << std::endl;
    
    return 0;
}
```

### Python 实现

```python
# decorator.py
from abc import ABC, abstractmethod

# ============================================
# 组件接口
# ============================================
class Coffee(ABC):
    """咖啡接口"""
    
    @abstractmethod
    def get_description(self) -> str:
        pass
    
    @abstractmethod
    def get_cost(self) -> float:
        pass


# ============================================
# 具体组件
# ============================================
class Espresso(Coffee):
    def get_description(self) -> str:
        return "Espresso"
    
    def get_cost(self) -> float:
        return 2.00


class HouseBlend(Coffee):
    def get_description(self) -> str:
        return "House Blend Coffee"
    
    def get_cost(self) -> float:
        return 1.50


# ============================================
# 装饰器基类
# ============================================
class CoffeeDecorator(Coffee, ABC):
    """咖啡装饰器基类"""
    
    def __init__(self, coffee: Coffee):
        self._coffee = coffee
    
    @abstractmethod
    def get_description(self) -> str:
        pass
    
    @abstractmethod
    def get_cost(self) -> float:
        pass


# ============================================
# 具体装饰器
# ============================================
class Milk(CoffeeDecorator):
    def get_description(self) -> str:
        return self._coffee.get_description() + ", Milk"
    
    def get_cost(self) -> float:
        return self._coffee.get_cost() + 0.50


class Mocha(CoffeeDecorator):
    def get_description(self) -> str:
        return self._coffee.get_description() + ", Mocha"
    
    def get_cost(self) -> float:
        return self._coffee.get_cost() + 0.70


class Whip(CoffeeDecorator):
    def get_description(self) -> str:
        return self._coffee.get_description() + ", Whip"
    
    def get_cost(self) -> float:
        return self._coffee.get_cost() + 0.30


class Vanilla(CoffeeDecorator):
    def get_description(self) -> str:
        return self._coffee.get_description() + ", Vanilla"
    
    def get_cost(self) -> float:
        return self._coffee.get_cost() + 0.40


def print_coffee(coffee: Coffee):
    print(f"{coffee.get_description()}: ${coffee.get_cost():.2f}")


if __name__ == "__main__":
    # 基础浓缩咖啡
    espresso = Espresso()
    print_coffee(espresso)
    
    # 加奶的拿铁
    latte = Milk(Espresso())
    print_coffee(latte)
    
    # 双份摩卡加奶油
    mocha = Whip(Mocha(Mocha(HouseBlend())))
    print_coffee(mocha)
    
    # 全料咖啡
    deluxe = Vanilla(Whip(Mocha(Milk(Espresso()))))
    print_coffee(deluxe)
```

---

## 初学者指南

### 装饰器 vs 继承

```
【继承方式】静态、类爆炸
Coffee
├── CoffeeWithMilk
├── CoffeeWithMocha
├── CoffeeWithMilkAndMocha
├── CoffeeWithMilkAndMochaAndWhip
└── ... (组合爆炸)

【装饰器方式】动态、灵活
Milk(Mocha(Whip(Espresso())))
       ↑      ↑      ↑
    装饰器可以任意组合和嵌套
```

### Python 函数装饰器

Python 有内置的装饰器语法，这是装饰器模式的语言级支持：

```python
# 函数装饰器
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@log_calls
def say_hello(name):
    print(f"Hello, {name}!")

say_hello("World")
# 输出:
# Calling say_hello
# Hello, World!
# Finished say_hello
```

---

## 高级应用

### 类装饰器

```python
# class_decorator.py
import functools
import time
from typing import Callable, Any

def timer(func: Callable) -> Callable:
    """计时装饰器"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs) -> Any:
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

def retry(times: int = 3, delay: float = 1.0):
    """重试装饰器"""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            last_exception = None
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    print(f"Attempt {attempt + 1} failed: {e}")
                    if attempt < times - 1:
                        time.sleep(delay)
            raise last_exception
        return wrapper
    return decorator

def cache(func: Callable) -> Callable:
    """缓存装饰器"""
    cached = {}
    
    @functools.wraps(func)
    def wrapper(*args) -> Any:
        if args in cached:
            print(f"Cache hit for {args}")
            return cached[args]
        result = func(*args)
        cached[args] = result
        return result
    return wrapper


@timer
@cache
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)


if __name__ == "__main__":
    result = fibonacci(30)
    print(f"Result: {result}")
```

---

## 实战案例

### HTTP 中间件

```python
# http_middleware.py
from abc import ABC, abstractmethod
from typing import Dict, Any, Callable
from datetime import datetime

# 请求和响应
class Request:
    def __init__(self, method: str, path: str, headers: Dict = None, body: Any = None):
        self.method = method
        self.path = path
        self.headers = headers or {}
        self.body = body

class Response:
    def __init__(self, status: int = 200, body: Any = None, headers: Dict = None):
        self.status = status
        self.body = body
        self.headers = headers or {}

# 处理器接口
class Handler(ABC):
    @abstractmethod
    def handle(self, request: Request) -> Response:
        pass

# 基础处理器
class BaseHandler(Handler):
    def handle(self, request: Request) -> Response:
        return Response(200, {"message": f"Handled {request.method} {request.path}"})

# 中间件装饰器
class Middleware(Handler):
    def __init__(self, handler: Handler):
        self._handler = handler

class LoggingMiddleware(Middleware):
    def handle(self, request: Request) -> Response:
        print(f"[{datetime.now()}] {request.method} {request.path}")
        response = self._handler.handle(request)
        print(f"[{datetime.now()}] Response: {response.status}")
        return response

class AuthMiddleware(Middleware):
    def handle(self, request: Request) -> Response:
        token = request.headers.get("Authorization")
        if not token:
            return Response(401, {"error": "Unauthorized"})
        if not self._validate_token(token):
            return Response(403, {"error": "Forbidden"})
        return self._handler.handle(request)
    
    def _validate_token(self, token: str) -> bool:
        return token == "valid-token"

class RateLimitMiddleware(Middleware):
    def __init__(self, handler: Handler, max_requests: int = 10):
        super().__init__(handler)
        self._max_requests = max_requests
        self._request_count = 0
    
    def handle(self, request: Request) -> Response:
        self._request_count += 1
        if self._request_count > self._max_requests:
            return Response(429, {"error": "Too Many Requests"})
        return self._handler.handle(request)


if __name__ == "__main__":
    # 构建中间件链
    handler = LoggingMiddleware(
        AuthMiddleware(
            RateLimitMiddleware(
                BaseHandler(),
                max_requests=5
            )
        )
    )
    
    # 测试请求
    request = Request("GET", "/api/users", headers={"Authorization": "valid-token"})
    response = handler.handle(request)
    print(f"Response: {response.body}")
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **适配器** | 适配器改变接口，装饰器增强功能但保持接口 |
| **组合** | 组合是树形结构，装饰器是链式结构 |
| **策略** | 策略改变对象内部，装饰器改变对象外部 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 比继承更灵活 | 会产生很多小对象 |
| 动态添加功能 | 装饰器顺序可能影响结果 |
| 单一职责 | 调试时栈可能很深 |

---

[← 上一章：组合模式](../composite/README.md) | [下一章：外观模式 →](../facade/README.md)

