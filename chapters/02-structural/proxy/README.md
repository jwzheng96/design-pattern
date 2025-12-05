# 代理模式 (Proxy Pattern)

[← 返回结构型模式](../README.md) | [返回目录](../../../README.md)

---

## 📚 目录

- [意图与动机](#意图与动机)
- [问题场景](#问题场景)
- [解决方案](#解决方案)
- [代码实现](#代码实现)
- [代理类型](#代理类型)
- [实战案例](#实战案例)
- [相关模式](#相关模式)

---

## 意图与动机

### 一句话定义

> **为其他对象提供一种代理以控制对这个对象的访问。**

### 🔰 初学者理解

想象信用卡就是现金的代理：
- 它们具有相同的接口（都可以支付）
- 信用卡控制对你银行账户的访问
- 可以添加额外功能（消费提醒、透支保护）

代理模式让你在不修改原对象的情况下，控制对它的访问。

### 🚀 高级理解

代理模式的核心价值：
- **延迟初始化**（虚拟代理）
- **访问控制**（保护代理）
- **远程对象访问**（远程代理）
- **日志/缓存**（智能代理）

---

## 问题场景

### 场景：大型图片加载

```python
# ❌ 没有代理：所有图片在初始化时就加载
class ImageViewer:
    def __init__(self):
        # 加载所有图片，即使用户可能只看其中几张
        self.images = [
            Image("huge_photo_1.jpg"),  # 100MB
            Image("huge_photo_2.jpg"),  # 150MB
            Image("huge_photo_3.jpg"),  # 200MB
        ]
        # 启动慢，内存占用大
```

---

## 解决方案

```
┌─────────────────────────────────────────────────────────────────────┐
│                        代理模式结构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Client ────► Subject (接口)                                       │
│                   ▲                                                 │
│                   │                                                 │
│           ┌───────┴───────┐                                        │
│           │               │                                        │
│       ┌───┴────┐    ┌─────┴────┐                                   │
│       │  Proxy │────►│RealSubject│                                 │
│       │        │    │          │                                   │
│       │ 控制访问│    │ 真实对象  │                                   │
│       └────────┘    └──────────┘                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 代码实现

### C++ 实现

```cpp
// proxy.cpp
#include <iostream>
#include <memory>
#include <string>
#include <unordered_map>

// ============================================
// 主题接口
// ============================================
class Image {
public:
    virtual ~Image() = default;
    virtual void display() = 0;
    virtual int getWidth() const = 0;
    virtual int getHeight() const = 0;
};

// ============================================
// 真实主题
// ============================================
class RealImage : public Image {
private:
    std::string filename;
    int width, height;
    
    void loadFromDisk() {
        std::cout << "Loading image from disk: " << filename << std::endl;
        // 模拟加载大文件
        width = 1920;
        height = 1080;
        std::cout << "Image loaded: " << width << "x" << height << std::endl;
    }

public:
    RealImage(const std::string& filename) : filename(filename) {
        loadFromDisk();  // 创建时就加载
    }
    
    void display() override {
        std::cout << "Displaying image: " << filename << std::endl;
    }
    
    int getWidth() const override { return width; }
    int getHeight() const override { return height; }
};

// ============================================
// 虚拟代理 - 延迟加载
// ============================================
class ImageProxy : public Image {
private:
    std::string filename;
    mutable std::unique_ptr<RealImage> realImage;
    
    void ensureLoaded() const {
        if (!realImage) {
            realImage = std::make_unique<RealImage>(filename);
        }
    }

public:
    ImageProxy(const std::string& filename) : filename(filename) {
        std::cout << "Created proxy for: " << filename << std::endl;
    }
    
    void display() override {
        ensureLoaded();
        realImage->display();
    }
    
    int getWidth() const override {
        ensureLoaded();
        return realImage->getWidth();
    }
    
    int getHeight() const override {
        ensureLoaded();
        return realImage->getHeight();
    }
};

// ============================================
// 保护代理 - 访问控制
// ============================================
class SecureImage : public Image {
private:
    std::unique_ptr<RealImage> realImage;
    std::string requiredRole;
    std::string currentUserRole;

public:
    SecureImage(const std::string& filename, const std::string& requiredRole)
        : realImage(std::make_unique<RealImage>(filename)),
          requiredRole(requiredRole) {}
    
    void setCurrentUserRole(const std::string& role) {
        currentUserRole = role;
    }
    
    void display() override {
        if (currentUserRole == requiredRole || currentUserRole == "admin") {
            realImage->display();
        } else {
            std::cout << "Access denied! Required role: " << requiredRole << std::endl;
        }
    }
    
    int getWidth() const override { return realImage->getWidth(); }
    int getHeight() const override { return realImage->getHeight(); }
};

int main() {
    std::cout << "=== Virtual Proxy Demo ===" << std::endl;
    
    // 创建代理不会加载图片
    std::unique_ptr<Image> proxy1 = std::make_unique<ImageProxy>("photo1.jpg");
    std::unique_ptr<Image> proxy2 = std::make_unique<ImageProxy>("photo2.jpg");
    
    std::cout << "\nFirst display (triggers loading):" << std::endl;
    proxy1->display();
    
    std::cout << "\nSecond display (already loaded):" << std::endl;
    proxy1->display();
    
    // proxy2 从未被使用，所以从未加载
    
    std::cout << "\n=== Protection Proxy Demo ===" << std::endl;
    auto secureImage = std::make_unique<SecureImage>("secret.jpg", "premium");
    
    secureImage->setCurrentUserRole("basic");
    secureImage->display();  // 被拒绝
    
    secureImage->setCurrentUserRole("premium");
    secureImage->display();  // 允许访问
    
    return 0;
}
```

### Python 实现

```python
# proxy.py
from abc import ABC, abstractmethod
from typing import Optional
import time

# ============================================
# 主题接口
# ============================================
class Database(ABC):
    @abstractmethod
    def query(self, sql: str) -> list:
        pass
    
    @abstractmethod
    def execute(self, sql: str) -> bool:
        pass


# ============================================
# 真实主题
# ============================================
class RealDatabase(Database):
    def __init__(self, host: str):
        print(f"Connecting to database at {host}...")
        time.sleep(1)  # 模拟连接延迟
        self._host = host
        print("Database connected!")
    
    def query(self, sql: str) -> list:
        print(f"Executing query: {sql}")
        return [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
    
    def execute(self, sql: str) -> bool:
        print(f"Executing: {sql}")
        return True


# ============================================
# 虚拟代理 - 延迟初始化
# ============================================
class LazyDatabase(Database):
    """延迟初始化代理"""
    
    def __init__(self, host: str):
        self._host = host
        self._real_db: Optional[RealDatabase] = None
        print("LazyDatabase proxy created (not connected yet)")
    
    def _ensure_connected(self):
        if self._real_db is None:
            self._real_db = RealDatabase(self._host)
    
    def query(self, sql: str) -> list:
        self._ensure_connected()
        return self._real_db.query(sql)
    
    def execute(self, sql: str) -> bool:
        self._ensure_connected()
        return self._real_db.execute(sql)


# ============================================
# 缓存代理
# ============================================
class CachingDatabase(Database):
    """带缓存的数据库代理"""
    
    def __init__(self, real_db: Database):
        self._real_db = real_db
        self._cache: dict = {}
    
    def query(self, sql: str) -> list:
        # 检查缓存
        if sql in self._cache:
            print(f"Cache hit for: {sql}")
            return self._cache[sql]
        
        # 缓存未命中，查询真实数据库
        print(f"Cache miss for: {sql}")
        result = self._real_db.query(sql)
        self._cache[sql] = result
        return result
    
    def execute(self, sql: str) -> bool:
        # 写操作清除缓存
        self._cache.clear()
        return self._real_db.execute(sql)


# ============================================
# 日志代理
# ============================================
class LoggingDatabase(Database):
    """带日志的数据库代理"""
    
    def __init__(self, real_db: Database):
        self._real_db = real_db
    
    def query(self, sql: str) -> list:
        print(f"[LOG] Query started: {sql}")
        start = time.time()
        result = self._real_db.query(sql)
        elapsed = time.time() - start
        print(f"[LOG] Query completed in {elapsed:.3f}s, {len(result)} rows")
        return result
    
    def execute(self, sql: str) -> bool:
        print(f"[LOG] Execute started: {sql}")
        start = time.time()
        result = self._real_db.execute(sql)
        elapsed = time.time() - start
        print(f"[LOG] Execute completed in {elapsed:.3f}s, success={result}")
        return result


# ============================================
# 访问控制代理
# ============================================
class SecureDatabase(Database):
    """带访问控制的数据库代理"""
    
    def __init__(self, real_db: Database, allowed_users: set):
        self._real_db = real_db
        self._allowed_users = allowed_users
        self._current_user: Optional[str] = None
    
    def login(self, username: str):
        self._current_user = username
    
    def _check_access(self):
        if self._current_user not in self._allowed_users:
            raise PermissionError(f"User '{self._current_user}' not authorized")
    
    def query(self, sql: str) -> list:
        self._check_access()
        return self._real_db.query(sql)
    
    def execute(self, sql: str) -> bool:
        self._check_access()
        # 额外检查危险操作
        if "DROP" in sql.upper() or "DELETE" in sql.upper():
            if self._current_user != "admin":
                raise PermissionError("Only admin can perform destructive operations")
        return self._real_db.execute(sql)


if __name__ == "__main__":
    print("=== Lazy Proxy Demo ===")
    lazy_db = LazyDatabase("localhost:5432")
    print("Proxy created, database not connected yet")
    print("\nFirst query (triggers connection):")
    lazy_db.query("SELECT * FROM users")
    
    print("\n=== Caching Proxy Demo ===")
    real_db = RealDatabase("localhost:5432")
    cached_db = CachingDatabase(real_db)
    
    print("\nFirst query (cache miss):")
    cached_db.query("SELECT * FROM users")
    
    print("\nSecond query (cache hit):")
    cached_db.query("SELECT * FROM users")
    
    print("\n=== Logging Proxy Demo ===")
    logged_db = LoggingDatabase(real_db)
    logged_db.query("SELECT * FROM users")
```

---

## 代理类型

| 类型 | 用途 | 示例 |
|------|------|------|
| **虚拟代理** | 延迟创建开销大的对象 | 图片延迟加载 |
| **保护代理** | 控制访问权限 | 权限检查 |
| **远程代理** | 表示远程对象 | RPC 客户端 |
| **缓存代理** | 缓存结果 | 数据库缓存 |
| **日志代理** | 记录请求日志 | 调试/监控 |
| **智能引用** | 引用计数、加锁 | 智能指针 |

---

## 实战案例

### Python 描述符代理

```python
# property_proxy.py
class LazyProperty:
    """延迟计算属性的代理"""
    
    def __init__(self, func):
        self.func = func
        self.name = func.__name__
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        
        # 计算值并缓存
        value = self.func(obj)
        setattr(obj, self.name, value)  # 替换描述符
        return value


class DataAnalyzer:
    def __init__(self, data):
        self._data = data
    
    @LazyProperty
    def mean(self):
        print("Computing mean...")
        return sum(self._data) / len(self._data)
    
    @LazyProperty
    def variance(self):
        print("Computing variance...")
        m = self.mean
        return sum((x - m) ** 2 for x in self._data) / len(self._data)


if __name__ == "__main__":
    analyzer = DataAnalyzer([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    
    print("Analyzer created")
    print(f"Mean: {analyzer.mean}")  # 首次计算
    print(f"Mean again: {analyzer.mean}")  # 使用缓存
    print(f"Variance: {analyzer.variance}")  # 首次计算
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **适配器** | 适配器改变接口，代理保持接口 |
| **装饰器** | 装饰器增强功能，代理控制访问 |
| **外观** | 外观简化接口，代理保持相同接口 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 控制对象访问 | 增加响应时间 |
| 延迟初始化 | 代码复杂度增加 |
| 添加额外功能 | |

---

[← 上一章：享元模式](../flyweight/README.md) | [下一部分：行为型模式 →](../../03-behavioral/README.md)

