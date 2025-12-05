# 外观模式 (Facade Pattern)

[← 返回结构型模式](../README.md) | [返回目录](../../../README.md)

---

## 📚 目录

- [意图与动机](#意图与动机)
- [问题场景](#问题场景)
- [解决方案](#解决方案)
- [代码实现](#代码实现)
- [初学者指南](#初学者指南)
- [实战案例](#实战案例)
- [相关模式](#相关模式)

---

## 意图与动机

### 一句话定义

> **为子系统中的一组接口提供一个统一的高层接口，使子系统更加容易使用。**

### 🔰 初学者理解

想象你去酒店前台：
- 你只需要说"我要办理入住"
- 前台帮你处理：登记、分配房间、制作房卡、安排行李搬运...

前台就是一个"外观"，把复杂的内部流程简化为一个简单的接口。

### 🚀 高级理解

外观模式的核心价值：
- **简化接口**：隐藏子系统复杂性
- **解耦客户端与子系统**
- **分层设计**：为不同层提供入口点

---

## 问题场景

### 场景：视频转换器

视频转换涉及多个复杂步骤：

```python
# ❌ 没有外观：客户端需要了解所有细节
def convert_video(filename, format):
    video_file = VideoFile(filename)
    codec = CodecFactory.extract(video_file)
    
    if codec is MP4Codec:
        source_codec = MP4CompressionCodec()
    else:
        source_codec = OggCompressionCodec()
    
    audio = AudioMixer().fix(video_file)
    result = BitrateReader.read(video_file, source_codec)
    result = BitrateReader.convert(result, target_codec)
    
    return VideoFile(result)

# 客户端需要了解 VideoFile, CodecFactory, AudioMixer, BitrateReader...
```

---

## 解决方案

```
┌─────────────────────────────────────────────────────────────────────┐
│                        外观模式结构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Client ────────────► Facade ◄────────────────────┐               │
│                          │                         │               │
│                          │ delegates to            │               │
│                          ▼                         │               │
│              ┌─────────────────────────────────────┼────┐          │
│              │        Subsystem                    │    │          │
│              │  ┌───────┐  ┌───────┐  ┌───────┐   │    │          │
│              │  │ Class │  │ Class │  │ Class │   │    │          │
│              │  │   A   │  │   B   │  │   C   │   │    │          │
│              │  └───────┘  └───────┘  └───────┘   │    │          │
│              └────────────────────────────────────────┘           │
│                                                                     │
│   外观提供简单接口，内部协调子系统组件                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 代码实现

### C++ 实现

```cpp
// facade.cpp
#include <iostream>
#include <string>
#include <memory>

// ============================================
// 子系统类
// ============================================
class CPU {
public:
    void freeze() { 
        std::cout << "CPU: Freezing..." << std::endl; 
    }
    void jump(long address) { 
        std::cout << "CPU: Jumping to " << address << std::endl; 
    }
    void execute() { 
        std::cout << "CPU: Executing..." << std::endl; 
    }
};

class Memory {
public:
    void load(long address, const std::string& data) {
        std::cout << "Memory: Loading '" << data << "' at " << address << std::endl;
    }
};

class HardDrive {
public:
    std::string read(long address, int size) {
        std::cout << "HardDrive: Reading " << size << " bytes from " << address << std::endl;
        return "boot_data";
    }
};

// ============================================
// 外观类
// ============================================
class ComputerFacade {
private:
    std::unique_ptr<CPU> cpu;
    std::unique_ptr<Memory> memory;
    std::unique_ptr<HardDrive> hardDrive;

public:
    ComputerFacade() 
        : cpu(std::make_unique<CPU>()),
          memory(std::make_unique<Memory>()),
          hardDrive(std::make_unique<HardDrive>()) {}
    
    void start() {
        std::cout << "=== Starting Computer ===" << std::endl;
        cpu->freeze();
        memory->load(0x00, hardDrive->read(0x00, 1024));
        cpu->jump(0x00);
        cpu->execute();
        std::cout << "=== Computer Started ===" << std::endl;
    }
    
    void shutdown() {
        std::cout << "=== Shutting Down ===" << std::endl;
        // 关机流程...
    }
};

int main() {
    // 客户端只需要与外观交互
    ComputerFacade computer;
    computer.start();
    
    return 0;
}
```

### Python 实现

```python
# facade.py

# ============================================
# 子系统类
# ============================================
class VideoFile:
    def __init__(self, filename: str):
        self.filename = filename
        self.codec_type = filename.split('.')[-1]
        print(f"VideoFile: Loaded {filename}")

class CodecFactory:
    @staticmethod
    def extract(file: VideoFile) -> str:
        print(f"CodecFactory: Extracting codec from {file.filename}")
        return file.codec_type

class MPEG4CompressionCodec:
    def compress(self, data: str) -> str:
        print("MPEG4Codec: Compressing...")
        return f"[MPEG4]{data}"

class OggCompressionCodec:
    def compress(self, data: str) -> str:
        print("OggCodec: Compressing...")
        return f"[OGG]{data}"

class AudioMixer:
    def fix(self, video: VideoFile) -> str:
        print(f"AudioMixer: Fixing audio for {video.filename}")
        return "fixed_audio"

class BitrateReader:
    @staticmethod
    def read(video: VideoFile) -> str:
        print(f"BitrateReader: Reading bitrate from {video.filename}")
        return "video_data"
    
    @staticmethod
    def convert(data: str, codec) -> str:
        print("BitrateReader: Converting bitrate...")
        return codec.compress(data)

# ============================================
# 外观类
# ============================================
class VideoConverter:
    """
    视频转换外观
    将复杂的视频转换流程封装为简单接口
    """
    
    def convert(self, filename: str, target_format: str) -> str:
        """
        转换视频格式
        
        Args:
            filename: 源文件名
            target_format: 目标格式 ('mp4' 或 'ogg')
        
        Returns:
            转换后的文件名
        """
        print(f"\n=== Converting {filename} to {target_format} ===")
        
        # 步骤 1: 加载视频文件
        video = VideoFile(filename)
        
        # 步骤 2: 提取编解码器
        source_codec = CodecFactory.extract(video)
        
        # 步骤 3: 选择目标编解码器
        if target_format == "mp4":
            target_codec = MPEG4CompressionCodec()
        else:
            target_codec = OggCompressionCodec()
        
        # 步骤 4: 处理音频
        audio = AudioMixer().fix(video)
        
        # 步骤 5: 读取并转换
        video_data = BitrateReader.read(video)
        result = BitrateReader.convert(video_data, target_codec)
        
        # 步骤 6: 生成输出文件
        output_file = filename.rsplit('.', 1)[0] + '.' + target_format
        
        print(f"=== Conversion Complete: {output_file} ===\n")
        return output_file


if __name__ == "__main__":
    # 客户端代码非常简单
    converter = VideoConverter()
    
    converter.convert("funny_video.avi", "mp4")
    converter.convert("movie.mov", "ogg")
```

---

## 初学者指南

### 外观模式的本质

```
【没有外观】客户端需要了解所有细节
┌────────────────────────────────────────┐
│  Client                                │
│    │                                   │
│    ├── uses ──► SubsystemA            │
│    ├── uses ──► SubsystemB            │
│    ├── uses ──► SubsystemC            │
│    └── knows all the details...       │
└────────────────────────────────────────┘

【有外观】客户端只需了解外观
┌────────────────────────────────────────┐
│  Client                                │
│    │                                   │
│    └── uses ──► Facade                │
│                   │                    │
│                   ├──► SubsystemA     │
│                   ├──► SubsystemB     │
│                   └──► SubsystemC     │
└────────────────────────────────────────┘
```

### 外观 vs 适配器

| 外观 | 适配器 |
|------|--------|
| 简化接口 | 转换接口 |
| 定义新接口 | 复用现有接口 |
| 处理多个类 | 通常处理一个类 |

---

## 实战案例

### 复杂 API 的封装

```python
# api_facade.py
import json
from typing import Dict, List, Optional
from dataclasses import dataclass

# 模拟复杂的底层 API
class HttpClient:
    def get(self, url: str, headers: Dict = None) -> str:
        print(f"HTTP GET: {url}")
        return '{"status": "ok"}'
    
    def post(self, url: str, data: Dict, headers: Dict = None) -> str:
        print(f"HTTP POST: {url}")
        return '{"id": 123}'

class TokenManager:
    def __init__(self):
        self._token = None
    
    def get_token(self, api_key: str, secret: str) -> str:
        print("TokenManager: Getting access token...")
        self._token = "access_token_123"
        return self._token
    
    def refresh_token(self) -> str:
        print("TokenManager: Refreshing token...")
        return self._token

class ResponseParser:
    @staticmethod
    def parse(response: str) -> Dict:
        return json.loads(response)

class ErrorHandler:
    @staticmethod
    def handle(response: Dict) -> None:
        if response.get("error"):
            raise Exception(response["error"])

# ============================================
# 外观
# ============================================
@dataclass
class User:
    id: int
    name: str
    email: str

class UserAPI:
    """
    用户 API 外观
    简化复杂的 HTTP 调用流程
    """
    
    def __init__(self, api_key: str, secret: str):
        self._client = HttpClient()
        self._token_manager = TokenManager()
        self._parser = ResponseParser()
        
        # 自动获取 token
        self._token = self._token_manager.get_token(api_key, secret)
    
    def _get_headers(self) -> Dict:
        return {"Authorization": f"Bearer {self._token}"}
    
    def get_user(self, user_id: int) -> Optional[User]:
        """获取用户信息"""
        response = self._client.get(
            f"/api/users/{user_id}",
            headers=self._get_headers()
        )
        data = self._parser.parse(response)
        ErrorHandler.handle(data)
        
        return User(id=user_id, name="John", email="john@example.com")
    
    def create_user(self, name: str, email: str) -> User:
        """创建用户"""
        response = self._client.post(
            "/api/users",
            data={"name": name, "email": email},
            headers=self._get_headers()
        )
        data = self._parser.parse(response)
        ErrorHandler.handle(data)
        
        return User(id=data["id"], name=name, email=email)
    
    def list_users(self) -> List[User]:
        """获取用户列表"""
        response = self._client.get(
            "/api/users",
            headers=self._get_headers()
        )
        data = self._parser.parse(response)
        ErrorHandler.handle(data)
        
        return [User(id=1, name="John", email="john@example.com")]


if __name__ == "__main__":
    # 使用外观，一行代码完成初始化
    api = UserAPI(api_key="my_key", secret="my_secret")
    
    # 简单的方法调用
    user = api.get_user(123)
    print(f"Got user: {user}")
    
    new_user = api.create_user("Alice", "alice@example.com")
    print(f"Created user: {new_user}")
```

---

## 相关模式

| 模式 | 关系 |
|------|------|
| **适配器** | 适配器改变接口，外观简化接口 |
| **中介者** | 中介者抽象对象间通信，外观简化子系统访问 |
| **单例** | 外观通常是单例 |

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 简化客户端代码 | 外观可能成为"上帝对象" |
| 减少依赖 | 可能引入不必要的间接层 |
| 分层清晰 | |

### 何时使用

- 需要简化复杂子系统的访问
- 需要分层设计
- 需要为库提供简单接口

---

[← 上一章：装饰器模式](../decorator/README.md) | [下一章：享元模式 →](../flyweight/README.md)

