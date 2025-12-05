# 适配器模式 (Adapter Pattern)

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

> **将一个类的接口转换成客户端所期望的另一种接口，使原本不兼容的类可以一起工作。**

### 🔰 初学者理解

想象你从美国带回一个电器，但中国的插座和美国的不一样。你需要一个**转换插头**（适配器），让美国电器能用中国插座。

适配器模式就是代码世界的"转换插头"——让不兼容的接口能够协同工作。

### 🚀 高级理解

适配器模式用于解决**接口不兼容**问题：
- 整合旧系统与新系统
- 使用第三方库时接口不匹配
- 统一多个不同接口的类

---

## 问题场景

### 场景：集成第三方支付

你的系统使用统一的支付接口：

```python
class PaymentProcessor:
    def pay(self, amount: float) -> bool:
        pass
```

但第三方支付 SDK 的接口完全不同：

```python
class AlipaySDK:
    def send_payment(self, order_id: str, amount_cents: int, callback_url: str):
        pass
```

问题：如何让 `AlipaySDK` 符合 `PaymentProcessor` 接口？

---

## 解决方案

```
┌─────────────────────────────────────────────────────────────────────┐
│                        适配器模式结构                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Client ────► Target Interface                                     │
│                     ▲                                               │
│                     │                                               │
│                ┌────┴────┐                                          │
│                │ Adapter │ ────► Adaptee (第三方类)                  │
│                └─────────┘                                          │
│                                                                     │
│   适配器实现目标接口，内部调用被适配者的方法                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 结构

### 两种实现方式

1. **对象适配器**（组合）：适配器持有被适配者的实例
2. **类适配器**（继承）：适配器同时继承目标接口和被适配者

### 对象适配器 UML

```
┌────────────────┐       ┌────────────────┐
│    Client      │──────►│    Target      │
└────────────────┘       │  + request()   │
                         └───────▲────────┘
                                 │
                         ┌───────┴────────┐
                         │    Adapter     │
                         │  + request()   │───────┐
                         └────────────────┘       │
                                                  ▼
                                          ┌──────────────────┐
                                          │     Adaptee      │
                                          │+ specificRequest()│
                                          └──────────────────┘
```

---

## 代码实现

### C++ 实现

```cpp
// adapter.cpp
#include <iostream>
#include <string>
#include <memory>

// ============================================
// 目标接口 - 客户端期望的接口
// ============================================
class MediaPlayer {
public:
    virtual ~MediaPlayer() = default;
    virtual void play(const std::string& filename) = 0;
};

// ============================================
// 被适配者 - 已存在的类，接口不兼容
// ============================================
class AdvancedVideoPlayer {
public:
    void playVlc(const std::string& filename) {
        std::cout << "Playing VLC file: " << filename << std::endl;
    }
    
    void playMp4(const std::string& filename) {
        std::cout << "Playing MP4 file: " << filename << std::endl;
    }
};

class AudioPlayer {
public:
    void playMp3(const std::string& filename) {
        std::cout << "Playing MP3 audio: " << filename << std::endl;
    }
    
    void playWav(const std::string& filename) {
        std::cout << "Playing WAV audio: " << filename << std::endl;
    }
};

// ============================================
// 适配器 - 将不兼容的接口转换为目标接口
// ============================================
class MediaAdapter : public MediaPlayer {
private:
    std::unique_ptr<AdvancedVideoPlayer> videoPlayer;
    std::unique_ptr<AudioPlayer> audioPlayer;
    
    std::string getExtension(const std::string& filename) {
        size_t pos = filename.rfind('.');
        if (pos != std::string::npos) {
            return filename.substr(pos + 1);
        }
        return "";
    }

public:
    MediaAdapter() 
        : videoPlayer(std::make_unique<AdvancedVideoPlayer>()),
          audioPlayer(std::make_unique<AudioPlayer>()) {}
    
    void play(const std::string& filename) override {
        std::string ext = getExtension(filename);
        
        if (ext == "vlc") {
            videoPlayer->playVlc(filename);
        } else if (ext == "mp4") {
            videoPlayer->playMp4(filename);
        } else if (ext == "mp3") {
            audioPlayer->playMp3(filename);
        } else if (ext == "wav") {
            audioPlayer->playWav(filename);
        } else {
            std::cout << "Unsupported format: " << ext << std::endl;
        }
    }
};

// ============================================
// 客户端代码
// ============================================
void playMedia(MediaPlayer& player, const std::string& filename) {
    player.play(filename);
}

int main() {
    MediaAdapter adapter;
    
    playMedia(adapter, "movie.mp4");
    playMedia(adapter, "video.vlc");
    playMedia(adapter, "song.mp3");
    playMedia(adapter, "sound.wav");
    playMedia(adapter, "file.unknown");
    
    return 0;
}
```

### Python 实现

```python
# adapter.py
from abc import ABC, abstractmethod

# ============================================
# 目标接口
# ============================================
class Target(ABC):
    """客户端期望的接口"""
    
    @abstractmethod
    def request(self) -> str:
        pass


# ============================================
# 被适配者
# ============================================
class Adaptee:
    """已存在的类，接口与 Target 不兼容"""
    
    def specific_request(self) -> str:
        return "Adaptee's specific behavior"


# ============================================
# 适配器（对象适配器）
# ============================================
class Adapter(Target):
    """
    对象适配器：通过组合包装被适配者
    """
    
    def __init__(self, adaptee: Adaptee):
        self._adaptee = adaptee
    
    def request(self) -> str:
        # 转换调用
        result = self._adaptee.specific_request()
        return f"Adapter: (TRANSLATED) {result}"


# ============================================
# 实际案例：支付系统适配
# ============================================
class PaymentProcessor(ABC):
    """目标接口：统一的支付处理器"""
    
    @abstractmethod
    def pay(self, amount: float) -> bool:
        pass
    
    @abstractmethod
    def refund(self, amount: float) -> bool:
        pass


class AlipaySDK:
    """被适配者：支付宝SDK（第三方）"""
    
    def create_payment(self, amount_cents: int, order_id: str) -> dict:
        print(f"Alipay: Creating payment for {amount_cents} cents, order: {order_id}")
        return {"status": "success", "transaction_id": "ALI123"}
    
    def create_refund(self, amount_cents: int, transaction_id: str) -> dict:
        print(f"Alipay: Refunding {amount_cents} cents for transaction: {transaction_id}")
        return {"status": "success"}


class WechatPaySDK:
    """被适配者：微信支付SDK（第三方）"""
    
    def send_payment(self, total_fee: int, out_trade_no: str) -> str:
        print(f"WechatPay: Sending payment of {total_fee} for order: {out_trade_no}")
        return "WX_PAYMENT_SUCCESS"
    
    def request_refund(self, refund_fee: int, out_refund_no: str) -> str:
        print(f"WechatPay: Requesting refund of {refund_fee}")
        return "WX_REFUND_SUCCESS"


class AlipayAdapter(PaymentProcessor):
    """支付宝适配器"""
    
    def __init__(self):
        self._sdk = AlipaySDK()
        self._last_transaction_id = None
    
    def pay(self, amount: float) -> bool:
        # 转换：元 -> 分
        amount_cents = int(amount * 100)
        order_id = f"ORDER_{id(self)}"
        
        result = self._sdk.create_payment(amount_cents, order_id)
        self._last_transaction_id = result.get("transaction_id")
        return result.get("status") == "success"
    
    def refund(self, amount: float) -> bool:
        if not self._last_transaction_id:
            return False
        
        amount_cents = int(amount * 100)
        result = self._sdk.create_refund(amount_cents, self._last_transaction_id)
        return result.get("status") == "success"


class WechatPayAdapter(PaymentProcessor):
    """微信支付适配器"""
    
    def __init__(self):
        self._sdk = WechatPaySDK()
    
    def pay(self, amount: float) -> bool:
        total_fee = int(amount * 100)
        out_trade_no = f"WX_ORDER_{id(self)}"
        
        result = self._sdk.send_payment(total_fee, out_trade_no)
        return "SUCCESS" in result
    
    def refund(self, amount: float) -> bool:
        refund_fee = int(amount * 100)
        out_refund_no = f"WX_REFUND_{id(self)}"
        
        result = self._sdk.request_refund(refund_fee, out_refund_no)
        return "SUCCESS" in result


def process_order(processor: PaymentProcessor, amount: float):
    """客户端代码：只依赖目标接口"""
    print(f"\n--- Processing payment of ${amount} ---")
    if processor.pay(amount):
        print("Payment successful!")
    else:
        print("Payment failed!")


if __name__ == "__main__":
    # 使用支付宝
    alipay = AlipayAdapter()
    process_order(alipay, 99.99)
    
    # 使用微信支付
    wechat = WechatPayAdapter()
    process_order(wechat, 199.99)
```

---

## 初学者指南

### 理解适配器的本质

```
┌─────────────────────────────────────────────────────────────┐
│   没有适配器                                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Client ──X──► Adaptee                                     │
│                                                             │
│   接口不匹配，无法通信                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│   有适配器                                                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Client ────► Adapter ────► Adaptee                        │
│           目标接口     翻译       原有接口                    │
│                                                             │
│   适配器在中间做翻译，让双方能够通信                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 对象适配器 vs 类适配器

| 特性 | 对象适配器 | 类适配器 |
|------|-----------|---------|
| 实现方式 | 组合 | 多重继承 |
| 灵活性 | 可适配子类 | 只能适配特定类 |
| 语言支持 | 所有语言 | 需支持多重继承 |
| 推荐程度 | ⭐⭐⭐ 推荐 | ⚠️ 谨慎使用 |

---

## 高级应用

### 双向适配器

```python
# bidirectional_adapter.py

class NewSystem:
    def process(self, data: str) -> str:
        return f"NewSystem processed: {data}"

class LegacySystem:
    def handle(self, info: str) -> str:
        return f"LegacySystem handled: {info}"

class BidirectionalAdapter(NewSystem, LegacySystem):
    """双向适配器：两个系统可以互相调用"""
    
    def __init__(self):
        self._new_system = NewSystem()
        self._legacy_system = LegacySystem()
    
    def process(self, data: str) -> str:
        # 新系统调用 -> 转发到新系统
        return self._new_system.process(data)
    
    def handle(self, info: str) -> str:
        # 遗留系统调用 -> 转发到遗留系统
        return self._legacy_system.handle(info)
    
    def new_to_legacy(self, data: str) -> str:
        # 新系统格式 -> 遗留系统格式
        result = self._new_system.process(data)
        return self._legacy_system.handle(result)
    
    def legacy_to_new(self, info: str) -> str:
        # 遗留系统格式 -> 新系统格式
        result = self._legacy_system.handle(info)
        return self._new_system.process(result)
```

---

## 实战案例

### 日志框架适配

```python
# logger_adapter.py
import logging
from abc import ABC, abstractmethod
from datetime import datetime

class Logger(ABC):
    """目标接口：统一的日志接口"""
    
    @abstractmethod
    def info(self, message: str): pass
    
    @abstractmethod
    def error(self, message: str): pass
    
    @abstractmethod
    def debug(self, message: str): pass


class PythonLoggingAdapter(Logger):
    """适配 Python 标准库 logging"""
    
    def __init__(self, name: str = "app"):
        self._logger = logging.getLogger(name)
        self._logger.setLevel(logging.DEBUG)
        handler = logging.StreamHandler()
        handler.setFormatter(logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        ))
        self._logger.addHandler(handler)
    
    def info(self, message: str):
        self._logger.info(message)
    
    def error(self, message: str):
        self._logger.error(message)
    
    def debug(self, message: str):
        self._logger.debug(message)


class SimpleConsoleLogger:
    """第三方简单日志器"""
    
    def write_log(self, level: str, msg: str):
        print(f"[{datetime.now()}] {level.upper()}: {msg}")


class SimpleLoggerAdapter(Logger):
    """适配第三方简单日志器"""
    
    def __init__(self):
        self._logger = SimpleConsoleLogger()
    
    def info(self, message: str):
        self._logger.write_log("info", message)
    
    def error(self, message: str):
        self._logger.write_log("error", message)
    
    def debug(self, message: str):
        self._logger.write_log("debug", message)


def application_logic(logger: Logger):
    """应用程序只依赖统一的 Logger 接口"""
    logger.info("Application started")
    logger.debug("Processing data...")
    logger.error("Something went wrong!")


if __name__ == "__main__":
    print("=== Using Python Logging ===")
    application_logic(PythonLoggingAdapter())
    
    print("\n=== Using Simple Logger ===")
    application_logic(SimpleLoggerAdapter())
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **桥接** | 桥接分离抽象与实现，适配器让不兼容接口协作 |
| **装饰器** | 装饰器增强功能但保持接口，适配器转换接口 |
| **代理** | 代理保持相同接口，适配器转换接口 |
| **外观** | 外观定义新接口，适配器复用现有接口 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 单一职责：接口转换代码分离 | 增加代码复杂度 |
| 开闭原则：不修改现有代码 | 有时直接修改更简单 |
| 复用现有类 | |

### 何时使用

✅ **适合场景：**
- 使用第三方库但接口不匹配
- 整合遗留系统
- 统一多个不同接口

❌ **不适合场景：**
- 可以直接修改源代码
- 接口差异太大，适配成本高

---

[← 返回结构型模式](../README.md) | [下一章：桥接模式 →](../bridge/README.md)

