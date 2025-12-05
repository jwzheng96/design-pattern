# 单例模式 (Singleton Pattern)

[← 返回创建型模式](../README.md) | [返回目录](../../../README.md)

---

## 📚 目录

- [意图与动机](#意图与动机)
- [问题场景](#问题场景)
- [解决方案](#解决方案)
- [结构](#结构)
- [代码实现](#代码实现)
- [初学者指南](#初学者指南)
- [高级应用](#高级应用)
- [最佳实践](#最佳实践)
- [常见陷阱](#常见陷阱)
- [实战案例](#实战案例)
- [相关模式](#相关模式)

---

## 意图与动机

### 一句话定义

> **确保一个类只有一个实例，并提供该实例的全局访问点。**

### 🔰 初学者理解

想象一下政府部门：一个国家只有一个总统/主席，不管你从哪里获取"当前领导人"的信息，都是同一个人。

单例模式就是确保某个类的对象在整个程序中**有且仅有一个**。

### 🚀 高级理解

单例模式是一种创建型设计模式，它解决了两个问题：

1. **保证一个类只有一个实例**
   - 控制对某些共享资源（如数据库或文件）的访问

2. **为该实例提供一个全局访问节点**
   - 比全局变量更安全，不会被意外覆盖

---

## 问题场景

### 什么时候需要单例？

1. **共享资源管理**
   - 数据库连接池
   - 线程池
   - 缓存

2. **全局状态管理**
   - 配置管理器
   - 日志记录器
   - 应用程序状态

3. **协调系统操作**
   - 打印机后台处理程序
   - 文件系统
   - 设备驱动程序

### 如果不使用单例会怎样？

```cpp
// 不使用单例：可能创建多个配置管理器
ConfigManager* config1 = new ConfigManager();
config1->load("settings.json");

ConfigManager* config2 = new ConfigManager();
config2->load("settings.json");  // 重复加载！

// 修改 config1 不会影响 config2
config1->set("theme", "dark");
// config2 仍然是旧值，导致不一致
```

---

## 解决方案

### 核心思路

1. 将构造函数设为**私有**，防止外部直接创建实例
2. 创建一个**静态方法**作为获取实例的唯一入口
3. 在该方法中，首次调用时创建实例，之后返回已有实例

```
┌─────────────────────────────────────────────────────────────┐
│                     单例模式工作流程                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Client A ──┐                                              │
│              │      ┌──────────────────────┐                │
│   Client B ──┼──────►  getInstance()       │                │
│              │      │    │                 │                │
│   Client C ──┘      │    ▼                 │                │
│                     │  instance == null?   │                │
│                     │    │                 │                │
│                     │    ├── Yes ──► create new instance    │
│                     │    │                 │                │
│                     │    └── No  ──► return existing        │
│                     │                      │                │
│                     │         ▼            │                │
│                     │  ┌─────────────┐     │                │
│                     │  │  Singleton  │     │                │
│                     │  │  Instance   │◄────┘                │
│                     │  └─────────────┘                      │
│                     └──────────────────────┘                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 结构

### UML 类图

```
┌─────────────────────────────────────────┐
│              Singleton                  │
├─────────────────────────────────────────┤
│ - instance: Singleton        [static]   │
│ - data: SomeType                        │
├─────────────────────────────────────────┤
│ - Singleton()                [private]  │
│ + getInstance(): Singleton   [static]   │
│ + businessLogic(): void                 │
└─────────────────────────────────────────┘
```

### 参与者

- **Singleton**：定义一个 `getInstance()` 方法，让客户端访问唯一的实例

---

## 代码实现

### C++ 实现

#### 基础版本（懒汉式 - 非线程安全）

```cpp
// singleton_basic.cpp
#include <iostream>
#include <string>

class Singleton {
private:
    // 私有静态指针，存储唯一实例
    static Singleton* instance;
    
    // 私有构造函数，防止外部创建实例
    Singleton() {
        std::cout << "Singleton instance created." << std::endl;
    }
    
    // 禁止拷贝和赋值
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
    // 一些数据
    std::string data;

public:
    // 公共静态方法，获取唯一实例
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
    
    void setData(const std::string& value) {
        data = value;
    }
    
    std::string getData() const {
        return data;
    }
    
    void businessLogic() {
        std::cout << "Executing business logic with data: " << data << std::endl;
    }
};

// 初始化静态成员
Singleton* Singleton::instance = nullptr;

int main() {
    // 获取单例实例
    Singleton* s1 = Singleton::getInstance();
    s1->setData("Hello, Singleton!");
    
    // 再次获取，得到的是同一个实例
    Singleton* s2 = Singleton::getInstance();
    
    std::cout << "s1 data: " << s1->getData() << std::endl;
    std::cout << "s2 data: " << s2->getData() << std::endl;
    std::cout << "s1 == s2: " << (s1 == s2 ? "true" : "false") << std::endl;
    
    s2->businessLogic();
    
    return 0;
}
```

#### 线程安全版本（C++11 Meyer's Singleton）

```cpp
// singleton_thread_safe.cpp
#include <iostream>
#include <string>
#include <thread>
#include <vector>

class Singleton {
private:
    Singleton() {
        std::cout << "Singleton instance created by thread: " 
                  << std::this_thread::get_id() << std::endl;
    }
    
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
    std::string data;

public:
    // C++11 保证局部静态变量的线程安全初始化
    static Singleton& getInstance() {
        static Singleton instance;  // 线程安全的懒汉式
        return instance;
    }
    
    void setData(const std::string& value) {
        data = value;
    }
    
    std::string getData() const {
        return data;
    }
};

void threadFunction(int id) {
    Singleton& instance = Singleton::getInstance();
    std::cout << "Thread " << id << " got instance at: " << &instance << std::endl;
}

int main() {
    std::vector<std::thread> threads;
    
    // 创建多个线程同时访问单例
    for (int i = 0; i < 5; ++i) {
        threads.emplace_back(threadFunction, i);
    }
    
    for (auto& t : threads) {
        t.join();
    }
    
    std::cout << "All threads accessed the same instance!" << std::endl;
    
    return 0;
}
```

#### 饿汉式（程序启动时创建）

```cpp
// singleton_eager.cpp
#include <iostream>

class Singleton {
private:
    // 程序启动时就创建实例
    static Singleton instance;
    
    Singleton() {
        std::cout << "Singleton created at startup." << std::endl;
    }
    
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

public:
    static Singleton& getInstance() {
        return instance;
    }
    
    void doSomething() {
        std::cout << "Doing something..." << std::endl;
    }
};

// 静态成员初始化（程序启动时执行）
Singleton Singleton::instance;

int main() {
    std::cout << "Main function started." << std::endl;
    Singleton::getInstance().doSomething();
    return 0;
}
```

### Python 实现

#### 基础版本（使用 __new__）

```python
# singleton_basic.py

class Singleton:
    """
    单例模式的基础实现
    """
    _instance = None
    
    def __new__(cls):
        """
        控制实例创建过程
        __new__ 是真正创建实例的方法，在 __init__ 之前调用
        """
        if cls._instance is None:
            print("Creating new Singleton instance")
            cls._instance = super().__new__(cls)
            cls._instance.data = None  # 初始化实例变量
        return cls._instance
    
    def set_data(self, value):
        self.data = value
    
    def get_data(self):
        return self.data
    
    def business_logic(self):
        print(f"Executing business logic with data: {self.data}")


if __name__ == "__main__":
    # 测试单例
    s1 = Singleton()
    s1.set_data("Hello, Singleton!")
    
    s2 = Singleton()
    
    print(f"s1 data: {s1.get_data()}")
    print(f"s2 data: {s2.get_data()}")
    print(f"s1 is s2: {s1 is s2}")
    
    s2.business_logic()
```

#### 使用装饰器（更 Pythonic 的方式）

```python
# singleton_decorator.py

def singleton(cls):
    """
    单例装饰器
    将任何类转换为单例类
    """
    instances = {}
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance


@singleton
class Database:
    """
    数据库连接类（单例）
    """
    def __init__(self):
        print("Initializing Database connection...")
        self.connection = "Connected to database"
    
    def query(self, sql):
        print(f"Executing: {sql}")
        return f"Results for: {sql}"


@singleton
class Logger:
    """
    日志记录器（单例）
    """
    def __init__(self):
        print("Initializing Logger...")
        self.log_file = "app.log"
    
    def log(self, message):
        print(f"[LOG] {message}")


if __name__ == "__main__":
    # 测试 Database 单例
    db1 = Database()
    db2 = Database()
    print(f"db1 is db2: {db1 is db2}")
    
    print()
    
    # 测试 Logger 单例
    logger1 = Logger()
    logger2 = Logger()
    print(f"logger1 is logger2: {logger1 is logger2}")
```

#### 线程安全版本

```python
# singleton_thread_safe.py
import threading

class ThreadSafeSingleton:
    """
    线程安全的单例实现
    """
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        # 双重检查锁定
        if cls._instance is None:
            with cls._lock:
                # 再次检查，因为可能有多个线程同时通过第一次检查
                if cls._instance is None:
                    print(f"Creating instance from thread: {threading.current_thread().name}")
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        # 注意：__init__ 可能被调用多次
        pass


def access_singleton(thread_id):
    """线程函数"""
    instance = ThreadSafeSingleton()
    print(f"Thread {thread_id} got instance: {id(instance)}")


if __name__ == "__main__":
    threads = []
    
    # 创建多个线程同时访问单例
    for i in range(5):
        t = threading.Thread(target=access_singleton, args=(i,))
        threads.append(t)
    
    # 启动所有线程
    for t in threads:
        t.start()
    
    # 等待所有线程完成
    for t in threads:
        t.join()
    
    print("All threads completed!")
```

#### 使用 metaclass（最优雅的方式）

```python
# singleton_metaclass.py

class SingletonMeta(type):
    """
    单例元类
    使用元类是实现单例的最优雅方式
    """
    _instances = {}
    _lock = threading.Lock()
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            with cls._lock:
                if cls not in cls._instances:
                    instance = super().__call__(*args, **kwargs)
                    cls._instances[cls] = instance
        return cls._instances[cls]


import threading

class Configuration(metaclass=SingletonMeta):
    """
    配置管理器（使用元类实现单例）
    """
    def __init__(self):
        print("Loading configuration...")
        self.settings = {
            "theme": "dark",
            "language": "zh-CN",
            "debug": False
        }
    
    def get(self, key):
        return self.settings.get(key)
    
    def set(self, key, value):
        self.settings[key] = value


if __name__ == "__main__":
    # 创建配置实例
    config1 = Configuration()
    config1.set("theme", "light")
    
    # 获取的是同一个实例
    config2 = Configuration()
    
    print(f"config1.theme: {config1.get('theme')}")
    print(f"config2.theme: {config2.get('theme')}")
    print(f"config1 is config2: {config1 is config2}")
```

---

## 初学者指南

### 理解单例的核心

```
问题：我需要整个程序中只有一个配置管理器

┌─────────────────────────────────────────────────────────────┐
│   不使用单例                                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   模块A: config1 = Config()   ──► 实例 1                    │
│   模块B: config2 = Config()   ──► 实例 2                    │
│   模块C: config3 = Config()   ──► 实例 3                    │
│                                                             │
│   问题：三个不同的配置实例，可能状态不一致！                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│   使用单例                                                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   模块A: config1 = Config.getInstance() ─┐                  │
│   模块B: config2 = Config.getInstance() ─┼──► 同一个实例     │
│   模块C: config3 = Config.getInstance() ─┘                  │
│                                                             │
│   结果：所有模块共享同一个配置实例，状态一致！                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 常见问题解答

**Q1: 为什么要把构造函数设为私有？**
> 为了防止其他代码使用 `new` 直接创建实例。如果可以随便 `new`，就无法保证只有一个实例。

**Q2: 懒汉式和饿汉式有什么区别？**
> - **懒汉式**：第一次使用时才创建，节省资源，但需要处理线程安全
> - **饿汉式**：程序启动时就创建，简单安全，但可能造成资源浪费

**Q3: Python 的 `__new__` 和 `__init__` 有什么区别？**
> - `__new__`：真正创建实例的方法，返回一个实例
> - `__init__`：初始化实例的方法，在 `__new__` 之后调用

---

## 高级应用

### 1. 单例注册表

当你需要管理多个命名的单例时：

```cpp
// singleton_registry.cpp
#include <iostream>
#include <map>
#include <string>
#include <memory>

class SingletonRegistry {
private:
    static std::map<std::string, std::shared_ptr<void>> registry;
    
public:
    template<typename T>
    static std::shared_ptr<T> getInstance(const std::string& name) {
        auto it = registry.find(name);
        if (it == registry.end()) {
            auto instance = std::make_shared<T>();
            registry[name] = instance;
            return instance;
        }
        return std::static_pointer_cast<T>(it->second);
    }
    
    static void clear() {
        registry.clear();
    }
};

std::map<std::string, std::shared_ptr<void>> SingletonRegistry::registry;
```

```python
# singleton_registry.py

class SingletonRegistry:
    """
    单例注册表
    管理多个命名的单例实例
    """
    _instances = {}
    
    @classmethod
    def get_instance(cls, name, factory):
        """
        获取命名的单例实例
        :param name: 单例的名称
        :param factory: 创建实例的工厂函数
        """
        if name not in cls._instances:
            cls._instances[name] = factory()
        return cls._instances[name]
    
    @classmethod
    def clear(cls):
        """清除所有注册的单例"""
        cls._instances.clear()


# 使用示例
class DatabaseConnection:
    def __init__(self, host):
        self.host = host
        print(f"Connected to {host}")

# 获取不同的数据库连接单例
master_db = SingletonRegistry.get_instance(
    "master", 
    lambda: DatabaseConnection("master.db.com")
)
slave_db = SingletonRegistry.get_instance(
    "slave", 
    lambda: DatabaseConnection("slave.db.com")
)
```

### 2. 可继承的单例基类

```python
# singleton_base.py

class SingletonBase:
    """
    可继承的单例基类
    子类自动成为单例
    """
    _instances = {}
    
    def __new__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__new__(cls)
        return cls._instances[cls]


class Logger(SingletonBase):
    def __init__(self):
        if not hasattr(self, 'initialized'):
            self.log_file = "app.log"
            self.initialized = True
    
    def log(self, message):
        print(f"[LOG] {message}")


class Config(SingletonBase):
    def __init__(self):
        if not hasattr(self, 'initialized'):
            self.settings = {}
            self.initialized = True
```

### 3. 依赖注入友好的单例

```cpp
// singleton_di.cpp
#include <memory>
#include <functional>

template<typename T>
class Singleton {
private:
    static std::unique_ptr<T> instance;
    static std::function<std::unique_ptr<T>()> factory;
    
public:
    // 允许注入工厂函数（用于测试）
    static void setFactory(std::function<std::unique_ptr<T>()> f) {
        factory = f;
        instance.reset();
    }
    
    static T& getInstance() {
        if (!instance) {
            if (factory) {
                instance = factory();
            } else {
                instance = std::make_unique<T>();
            }
        }
        return *instance;
    }
    
    // 用于测试的重置方法
    static void reset() {
        instance.reset();
    }
};

template<typename T>
std::unique_ptr<T> Singleton<T>::instance = nullptr;

template<typename T>
std::function<std::unique_ptr<T>()> Singleton<T>::factory = nullptr;
```

---

## 最佳实践

### ✅ 推荐做法

1. **优先使用 C++11 的 Meyer's Singleton**
   ```cpp
   static Singleton& getInstance() {
       static Singleton instance;
       return instance;
   }
   ```

2. **Python 中优先使用模块级单例**
   ```python
   # config.py
   class _Config:
       def __init__(self):
           self.settings = {}
   
   # 模块级单例
   config = _Config()
   ```

3. **考虑使用依赖注入替代**
   - 单例虽然方便，但会增加代码耦合
   - 依赖注入可以达到类似效果，且更易测试

### ❌ 避免做法

1. **避免过度使用单例**
   - 单例是全局状态，会增加代码耦合
   - 可能导致隐藏的依赖关系

2. **避免在单例中存储可变状态**
   - 多线程环境下可能导致竞态条件

3. **避免单例继承**
   - 单例类的子类也是单例吗？语义不清

---

## 常见陷阱

### 陷阱 1：多线程竞态条件

```cpp
// ❌ 非线程安全
static Singleton* getInstance() {
    if (instance == nullptr) {        // 线程 A 和 B 可能同时到达这里
        instance = new Singleton();   // 可能创建两个实例！
    }
    return instance;
}

// ✅ 线程安全（C++11）
static Singleton& getInstance() {
    static Singleton instance;  // 线程安全
    return instance;
}
```

### 陷阱 2：Python 中 __init__ 被多次调用

```python
# ❌ 问题：__init__ 会被多次调用
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        print("Init called!")  # 每次调用 Singleton() 都会执行
        self.data = []  # 数据会被重置！

# ✅ 解决方案：使用标志位
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if self._initialized:
            return
        print("Init called!")
        self.data = []
        self._initialized = True
```

### 陷阱 3：单例销毁顺序

```cpp
// C++ 静态对象销毁顺序是不确定的
// 如果单例 A 依赖单例 B，可能在 B 销毁后 A 还在使用 B

class SingletonA {
    static SingletonA& getInstance() {
        static SingletonA instance;
        return instance;
    }
    ~SingletonA() {
        // 如果在这里使用 SingletonB，可能已经被销毁了！
        SingletonB::getInstance().doSomething();  // 危险！
    }
};
```

---

## 实战案例

### 案例 1：配置管理器

```cpp
// config_manager.cpp
#include <iostream>
#include <map>
#include <string>
#include <fstream>
#include <sstream>

class ConfigManager {
private:
    std::map<std::string, std::string> config;
    
    ConfigManager() {
        loadDefaultConfig();
    }
    
    void loadDefaultConfig() {
        config["app.name"] = "MyApp";
        config["app.version"] = "1.0.0";
        config["database.host"] = "localhost";
        config["database.port"] = "5432";
    }

public:
    static ConfigManager& getInstance() {
        static ConfigManager instance;
        return instance;
    }
    
    void loadFromFile(const std::string& filename) {
        std::ifstream file(filename);
        std::string line;
        while (std::getline(file, line)) {
            auto pos = line.find('=');
            if (pos != std::string::npos) {
                std::string key = line.substr(0, pos);
                std::string value = line.substr(pos + 1);
                config[key] = value;
            }
        }
    }
    
    std::string get(const std::string& key, 
                    const std::string& defaultValue = "") const {
        auto it = config.find(key);
        return (it != config.end()) ? it->second : defaultValue;
    }
    
    void set(const std::string& key, const std::string& value) {
        config[key] = value;
    }
    
    void printAll() const {
        for (const auto& pair : config) {
            std::cout << pair.first << " = " << pair.second << std::endl;
        }
    }
};

int main() {
    // 获取配置管理器
    auto& config = ConfigManager::getInstance();
    
    // 读取配置
    std::cout << "App Name: " << config.get("app.name") << std::endl;
    std::cout << "Version: " << config.get("app.version") << std::endl;
    
    // 修改配置
    config.set("app.debug", "true");
    
    // 另一处代码获取的是同一个实例
    auto& config2 = ConfigManager::getInstance();
    std::cout << "Debug: " << config2.get("app.debug") << std::endl;
    
    return 0;
}
```

```python
# config_manager.py
import json
import os
from typing import Any, Optional

class ConfigManager:
    """
    配置管理器单例
    支持从文件加载、获取和设置配置
    """
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if self._initialized:
            return
        
        self._config = {
            "app.name": "MyApp",
            "app.version": "1.0.0",
            "database.host": "localhost",
            "database.port": 5432,
            "debug": False
        }
        self._initialized = True
    
    def load_from_file(self, filename: str) -> None:
        """从 JSON 文件加载配置"""
        if os.path.exists(filename):
            with open(filename, 'r') as f:
                loaded_config = json.load(f)
                self._config.update(loaded_config)
    
    def get(self, key: str, default: Any = None) -> Any:
        """获取配置值"""
        return self._config.get(key, default)
    
    def set(self, key: str, value: Any) -> None:
        """设置配置值"""
        self._config[key] = value
    
    def save_to_file(self, filename: str) -> None:
        """保存配置到文件"""
        with open(filename, 'w') as f:
            json.dump(self._config, f, indent=2)
    
    def __repr__(self):
        return f"ConfigManager({self._config})"


if __name__ == "__main__":
    # 获取配置管理器
    config = ConfigManager()
    
    print(f"App Name: {config.get('app.name')}")
    print(f"Version: {config.get('app.version')}")
    
    # 修改配置
    config.set("debug", True)
    
    # 另一处代码获取的是同一个实例
    config2 = ConfigManager()
    print(f"Debug: {config2.get('debug')}")
    print(f"Same instance: {config is config2}")
```

### 案例 2：日志系统

```python
# logger.py
import datetime
import threading
from enum import Enum
from typing import Optional

class LogLevel(Enum):
    DEBUG = 1
    INFO = 2
    WARNING = 3
    ERROR = 4
    CRITICAL = 5

class Logger:
    """
    线程安全的日志单例
    """
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if self._initialized:
            return
        
        self._level = LogLevel.DEBUG
        self._file: Optional[str] = None
        self._write_lock = threading.Lock()
        self._initialized = True
    
    def set_level(self, level: LogLevel) -> None:
        """设置日志级别"""
        self._level = level
    
    def set_file(self, filename: str) -> None:
        """设置日志文件"""
        self._file = filename
    
    def _format_message(self, level: LogLevel, message: str) -> str:
        """格式化日志消息"""
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        thread_name = threading.current_thread().name
        return f"[{timestamp}] [{level.name}] [{thread_name}] {message}"
    
    def _log(self, level: LogLevel, message: str) -> None:
        """内部日志方法"""
        if level.value < self._level.value:
            return
        
        formatted = self._format_message(level, message)
        
        with self._write_lock:
            print(formatted)
            if self._file:
                with open(self._file, 'a') as f:
                    f.write(formatted + '\n')
    
    def debug(self, message: str) -> None:
        self._log(LogLevel.DEBUG, message)
    
    def info(self, message: str) -> None:
        self._log(LogLevel.INFO, message)
    
    def warning(self, message: str) -> None:
        self._log(LogLevel.WARNING, message)
    
    def error(self, message: str) -> None:
        self._log(LogLevel.ERROR, message)
    
    def critical(self, message: str) -> None:
        self._log(LogLevel.CRITICAL, message)


# 便捷函数
def get_logger() -> Logger:
    return Logger()


if __name__ == "__main__":
    # 配置日志
    logger = get_logger()
    logger.set_level(LogLevel.DEBUG)
    
    # 在不同模块中使用
    logger.debug("Debug message")
    logger.info("Application started")
    logger.warning("This is a warning")
    logger.error("An error occurred")
    
    # 验证单例
    logger2 = get_logger()
    print(f"Same instance: {logger is logger2}")
```

---

## 相关模式

| 模式 | 关系 |
|-----|------|
| **抽象工厂** | 抽象工厂类通常被实现为单例 |
| **建造者** | 建造者类可以是单例 |
| **原型** | 原型管理器通常是单例 |
| **外观** | 外观类通常被实现为单例 |

---

## 总结

### 单例模式的优缺点

| 优点 | 缺点 |
|------|------|
| 确保只有一个实例 | 违反单一职责原则（既管理自己又管理实例化） |
| 提供全局访问点 | 可能掩盖不良设计 |
| 仅在首次请求时初始化 | 多线程环境需要特殊处理 |
| | 单元测试困难 |

### 何时使用单例

✅ **适合使用的场景：**
- 数据库连接池
- 配置管理器
- 日志系统
- 缓存管理器
- 线程池

❌ **不适合使用的场景：**
- 需要频繁创建和销毁的对象
- 需要多个不同配置的实例
- 测试环境需要隔离的场景

---

[← 返回创建型模式](../README.md) | [下一章：工厂方法模式 →](../factory-method/README.md)

