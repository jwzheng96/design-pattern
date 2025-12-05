# 模板方法模式 (Template Method Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **定义一个操作中的算法骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。**

### 🔰 初学者理解

想象冲泡饮料的步骤：
1. 烧水
2. 冲泡（茶叶/咖啡粉）
3. 倒入杯子
4. 添加调料（柠檬/糖和牛奶）

步骤 1 和 3 是固定的，步骤 2 和 4 因饮料不同而不同。这个"固定的流程 + 可变的步骤"就是模板方法。

---

## 代码实现

### Python 实现

```python
# template_method.py
from abc import ABC, abstractmethod

# ============================================
# 抽象类 - 定义模板方法
# ============================================
class DataMiner(ABC):
    """数据挖掘模板"""
    
    def mine(self, path: str) -> None:
        """
        模板方法 - 定义算法骨架
        """
        file = self.open_file(path)
        raw_data = self.extract_data(file)
        data = self.parse_data(raw_data)
        analysis = self.analyze_data(data)
        self.send_report(analysis)
        self.close_file(file)
    
    # 抽象方法 - 子类必须实现
    @abstractmethod
    def open_file(self, path: str):
        pass
    
    @abstractmethod
    def extract_data(self, file) -> str:
        pass
    
    @abstractmethod
    def parse_data(self, raw_data: str) -> list:
        pass
    
    # 具体方法 - 所有子类共享
    def analyze_data(self, data: list) -> dict:
        print(f"Analyzing {len(data)} records...")
        return {"count": len(data), "data": data}
    
    def send_report(self, analysis: dict) -> None:
        print(f"Report: {analysis['count']} records processed")
    
    # 钩子方法 - 子类可选择覆盖
    def close_file(self, file) -> None:
        print("Closing file...")


# ============================================
# 具体类
# ============================================
class CSVDataMiner(DataMiner):
    def open_file(self, path: str):
        print(f"Opening CSV file: {path}")
        return f"csv_file:{path}"
    
    def extract_data(self, file) -> str:
        print("Extracting data from CSV...")
        return "name,age\nAlice,30\nBob,25"
    
    def parse_data(self, raw_data: str) -> list:
        lines = raw_data.strip().split('\n')
        headers = lines[0].split(',')
        result = []
        for line in lines[1:]:
            values = line.split(',')
            result.append(dict(zip(headers, values)))
        print(f"Parsed {len(result)} CSV records")
        return result


class JSONDataMiner(DataMiner):
    def open_file(self, path: str):
        print(f"Opening JSON file: {path}")
        return f"json_file:{path}"
    
    def extract_data(self, file) -> str:
        print("Extracting data from JSON...")
        return '[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]'
    
    def parse_data(self, raw_data: str) -> list:
        import json
        result = json.loads(raw_data)
        print(f"Parsed {len(result)} JSON records")
        return result


class PDFDataMiner(DataMiner):
    def open_file(self, path: str):
        print(f"Opening PDF file: {path}")
        return f"pdf_file:{path}"
    
    def extract_data(self, file) -> str:
        print("Extracting text from PDF (using OCR if needed)...")
        return "Alice: 30 years old\nBob: 25 years old"
    
    def parse_data(self, raw_data: str) -> list:
        lines = raw_data.strip().split('\n')
        result = []
        for line in lines:
            parts = line.split(': ')
            if len(parts) == 2:
                name = parts[0]
                age = parts[1].split()[0]
                result.append({"name": name, "age": age})
        print(f"Parsed {len(result)} PDF records")
        return result


if __name__ == "__main__":
    print("=== CSV Mining ===")
    csv_miner = CSVDataMiner()
    csv_miner.mine("data.csv")
    
    print("\n=== JSON Mining ===")
    json_miner = JSONDataMiner()
    json_miner.mine("data.json")
    
    print("\n=== PDF Mining ===")
    pdf_miner = PDFDataMiner()
    pdf_miner.mine("data.pdf")
```

### C++ 实现

```cpp
// template_method.cpp
#include <iostream>
#include <string>

class GameAI {
public:
    // 模板方法
    void turn() {
        collectResources();
        buildStructures();
        buildUnits();
        attack();
    }
    
    // 具体方法
    void collectResources() {
        std::cout << "Collecting resources from nearby sources" << std::endl;
    }
    
    // 抽象方法
    virtual void buildStructures() = 0;
    virtual void buildUnits() = 0;
    
    // 钩子方法
    virtual void attack() {
        std::cout << "Attacking enemy base" << std::endl;
    }
    
    virtual ~GameAI() = default;
};

class OrcsAI : public GameAI {
public:
    void buildStructures() override {
        std::cout << "Building orc huts and barracks" << std::endl;
    }
    
    void buildUnits() override {
        std::cout << "Training orc warriors and shamans" << std::endl;
    }
    
    void attack() override {
        std::cout << "WAAAGH! Orcs charging into battle!" << std::endl;
    }
};

class HumansAI : public GameAI {
public:
    void buildStructures() override {
        std::cout << "Building castle and church" << std::endl;
    }
    
    void buildUnits() override {
        std::cout << "Training knights and archers" << std::endl;
    }
};

int main() {
    std::cout << "=== Orcs Turn ===" << std::endl;
    OrcsAI orcs;
    orcs.turn();
    
    std::cout << "\n=== Humans Turn ===" << std::endl;
    HumansAI humans;
    humans.turn();
    
    return 0;
}
```

---

## 钩子方法

钩子是可选的步骤，子类可以选择是否覆盖：

```python
class BaseClass(ABC):
    def template_method(self):
        self.step1()
        self.step2()
        if self.hook():  # 钩子：控制是否执行 step3
            self.step3()
    
    @abstractmethod
    def step1(self): pass
    
    @abstractmethod
    def step2(self): pass
    
    def step3(self):
        print("Optional step 3")
    
    def hook(self) -> bool:
        """钩子方法 - 子类可选择覆盖"""
        return True
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 代码复用 | 算法骨架固定，灵活性受限 |
| 反向控制结构 | 子类多时难以维护 |
| 只需覆盖部分方法 | |

### 模板方法 vs 策略

| 模板方法 | 策略 |
|---------|------|
| 基于继承 | 基于组合 |
| 在类级别改变算法 | 在对象级别改变算法 |
| 编译时确定 | 运行时可切换 |

---

[← 上一章：策略模式](../strategy/README.md) | [下一章：访问者模式 →](../visitor/README.md)

