# 解释器模式 (Interpreter Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。**

### 🔰 初学者理解

想象计算器：
- 输入：`3 + 5 * 2`
- 计算器需要理解这个表达式
- 然后按照数学规则计算出结果

解释器模式就是为特定语言（如数学表达式、正则表达式）构建解释执行器。

---

## 代码实现

### Python 实现

```python
# interpreter.py
from abc import ABC, abstractmethod
from typing import Dict

# ============================================
# 抽象表达式
# ============================================
class Expression(ABC):
    @abstractmethod
    def interpret(self, context: Dict[str, int]) -> int:
        pass


# ============================================
# 终结符表达式 - 数字
# ============================================
class NumberExpression(Expression):
    def __init__(self, number: int):
        self._number = number
    
    def interpret(self, context: Dict[str, int]) -> int:
        return self._number


# ============================================
# 终结符表达式 - 变量
# ============================================
class VariableExpression(Expression):
    def __init__(self, name: str):
        self._name = name
    
    def interpret(self, context: Dict[str, int]) -> int:
        return context.get(self._name, 0)


# ============================================
# 非终结符表达式 - 运算符
# ============================================
class AddExpression(Expression):
    def __init__(self, left: Expression, right: Expression):
        self._left = left
        self._right = right
    
    def interpret(self, context: Dict[str, int]) -> int:
        return self._left.interpret(context) + self._right.interpret(context)


class SubtractExpression(Expression):
    def __init__(self, left: Expression, right: Expression):
        self._left = left
        self._right = right
    
    def interpret(self, context: Dict[str, int]) -> int:
        return self._left.interpret(context) - self._right.interpret(context)


class MultiplyExpression(Expression):
    def __init__(self, left: Expression, right: Expression):
        self._left = left
        self._right = right
    
    def interpret(self, context: Dict[str, int]) -> int:
        return self._left.interpret(context) * self._right.interpret(context)


# ============================================
# 简单解析器
# ============================================
class Parser:
    """简单的表达式解析器（仅支持 + - * 和变量）"""
    
    def __init__(self, expression: str):
        self._tokens = expression.replace('(', ' ( ').replace(')', ' ) ').split()
        self._pos = 0
    
    def parse(self) -> Expression:
        return self._parse_expression()
    
    def _parse_expression(self) -> Expression:
        left = self._parse_term()
        
        while self._pos < len(self._tokens) and self._tokens[self._pos] in ['+', '-']:
            op = self._tokens[self._pos]
            self._pos += 1
            right = self._parse_term()
            
            if op == '+':
                left = AddExpression(left, right)
            else:
                left = SubtractExpression(left, right)
        
        return left
    
    def _parse_term(self) -> Expression:
        left = self._parse_factor()
        
        while self._pos < len(self._tokens) and self._tokens[self._pos] == '*':
            self._pos += 1
            right = self._parse_factor()
            left = MultiplyExpression(left, right)
        
        return left
    
    def _parse_factor(self) -> Expression:
        token = self._tokens[self._pos]
        self._pos += 1
        
        if token == '(':
            expr = self._parse_expression()
            self._pos += 1  # skip ')'
            return expr
        elif token.isdigit() or (token[0] == '-' and token[1:].isdigit()):
            return NumberExpression(int(token))
        else:
            return VariableExpression(token)


if __name__ == "__main__":
    # 上下文（变量值）
    context = {"x": 10, "y": 5}
    
    # 测试表达式
    expressions = [
        "3 + 5",
        "3 + 5 * 2",
        "x + y",
        "x * 2 + y",
        "( x + y ) * 2",
    ]
    
    for expr_str in expressions:
        parser = Parser(expr_str)
        expr = parser.parse()
        result = expr.interpret(context)
        print(f"{expr_str} = {result}")
```

### C++ 实现

```cpp
// interpreter.cpp
#include <iostream>
#include <string>
#include <map>
#include <memory>

class Context {
public:
    std::map<std::string, int> variables;
};

class Expression {
public:
    virtual ~Expression() = default;
    virtual int interpret(Context& context) = 0;
};

class NumberExpression : public Expression {
private:
    int number;
public:
    NumberExpression(int n) : number(n) {}
    int interpret(Context&) override { return number; }
};

class VariableExpression : public Expression {
private:
    std::string name;
public:
    VariableExpression(const std::string& n) : name(n) {}
    int interpret(Context& context) override {
        return context.variables[name];
    }
};

class AddExpression : public Expression {
private:
    std::shared_ptr<Expression> left, right;
public:
    AddExpression(std::shared_ptr<Expression> l, std::shared_ptr<Expression> r)
        : left(l), right(r) {}
    
    int interpret(Context& context) override {
        return left->interpret(context) + right->interpret(context);
    }
};

int main() {
    Context context;
    context.variables["x"] = 10;
    context.variables["y"] = 5;
    
    // 构建表达式：x + y
    auto x = std::make_shared<VariableExpression>("x");
    auto y = std::make_shared<VariableExpression>("y");
    auto expr = std::make_shared<AddExpression>(x, y);
    
    std::cout << "x + y = " << expr->interpret(context) << std::endl;
    
    return 0;
}
```

---

## 实战案例：布尔表达式

```python
# boolean_interpreter.py
from abc import ABC, abstractmethod

class BoolExpression(ABC):
    @abstractmethod
    def interpret(self, context: dict) -> bool:
        pass

class Constant(BoolExpression):
    def __init__(self, value: bool):
        self._value = value
    
    def interpret(self, context: dict) -> bool:
        return self._value

class Variable(BoolExpression):
    def __init__(self, name: str):
        self._name = name
    
    def interpret(self, context: dict) -> bool:
        return context.get(self._name, False)

class And(BoolExpression):
    def __init__(self, left: BoolExpression, right: BoolExpression):
        self._left = left
        self._right = right
    
    def interpret(self, context: dict) -> bool:
        return self._left.interpret(context) and self._right.interpret(context)

class Or(BoolExpression):
    def __init__(self, left: BoolExpression, right: BoolExpression):
        self._left = left
        self._right = right
    
    def interpret(self, context: dict) -> bool:
        return self._left.interpret(context) or self._right.interpret(context)

class Not(BoolExpression):
    def __init__(self, expr: BoolExpression):
        self._expr = expr
    
    def interpret(self, context: dict) -> bool:
        return not self._expr.interpret(context)


if __name__ == "__main__":
    # (A AND B) OR (NOT C)
    expr = Or(
        And(Variable("A"), Variable("B")),
        Not(Variable("C"))
    )
    
    contexts = [
        {"A": True, "B": True, "C": False},
        {"A": True, "B": False, "C": False},
        {"A": False, "B": False, "C": True},
    ]
    
    for ctx in contexts:
        result = expr.interpret(ctx)
        print(f"{ctx} => {result}")
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 易于改变和扩展文法 | 复杂文法难以维护 |
| 易于实现文法 | 效率可能较低 |

### 何时使用

- 语法简单
- 效率不是主要关注点
- 需要解释领域特定语言（DSL）

### 替代方案

对于复杂语法，考虑使用：
- 解析器生成器（ANTLR、Bison）
- 正则表达式
- 状态机

---

[← 上一章：访问者模式](../visitor/README.md) | [返回目录 →](../../../README.md)

