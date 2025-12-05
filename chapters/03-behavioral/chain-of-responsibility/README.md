# 责任链模式 (Chain of Responsibility Pattern)

[← 返回行为型模式](../README.md) | [返回目录](../../../README.md)

---

## 意图与动机

### 一句话定义

> **使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。**

### 🔰 初学者理解

想象公司的请假审批流程：
- 请假 1 天 → 组长审批
- 请假 3 天 → 经理审批
- 请假 7 天 → 总监审批
- 请假 30 天 → CEO 审批

每个审批人要么处理请求，要么传给下一级。

---

## 代码实现

### Python 实现

```python
# chain_of_responsibility.py
from abc import ABC, abstractmethod
from typing import Optional
from dataclasses import dataclass

@dataclass
class LeaveRequest:
    employee: str
    days: int
    reason: str

class Handler(ABC):
    """处理者抽象类"""
    
    def __init__(self):
        self._next_handler: Optional[Handler] = None
    
    def set_next(self, handler: 'Handler') -> 'Handler':
        self._next_handler = handler
        return handler  # 支持链式调用
    
    @abstractmethod
    def handle(self, request: LeaveRequest) -> str:
        pass
    
    def pass_to_next(self, request: LeaveRequest) -> str:
        if self._next_handler:
            return self._next_handler.handle(request)
        return "Request reached end of chain without being handled"


class TeamLeader(Handler):
    def handle(self, request: LeaveRequest) -> str:
        if request.days <= 1:
            return f"TeamLeader approved {request.employee}'s {request.days}-day leave"
        return self.pass_to_next(request)


class Manager(Handler):
    def handle(self, request: LeaveRequest) -> str:
        if request.days <= 3:
            return f"Manager approved {request.employee}'s {request.days}-day leave"
        return self.pass_to_next(request)


class Director(Handler):
    def handle(self, request: LeaveRequest) -> str:
        if request.days <= 7:
            return f"Director approved {request.employee}'s {request.days}-day leave"
        return self.pass_to_next(request)


class CEO(Handler):
    def handle(self, request: LeaveRequest) -> str:
        if request.days <= 30:
            return f"CEO approved {request.employee}'s {request.days}-day leave"
        return f"CEO rejected {request.employee}'s leave (too many days)"


if __name__ == "__main__":
    # 构建责任链
    team_leader = TeamLeader()
    manager = Manager()
    director = Director()
    ceo = CEO()
    
    team_leader.set_next(manager).set_next(director).set_next(ceo)
    
    # 测试不同请求
    requests = [
        LeaveRequest("Alice", 1, "Personal"),
        LeaveRequest("Bob", 3, "Sick"),
        LeaveRequest("Charlie", 7, "Vacation"),
        LeaveRequest("David", 20, "Travel"),
        LeaveRequest("Eve", 45, "Sabbatical"),
    ]
    
    for req in requests:
        result = team_leader.handle(req)
        print(f"{req.employee} ({req.days} days): {result}")
```

### C++ 实现

```cpp
// chain_of_responsibility.cpp
#include <iostream>
#include <memory>
#include <string>

struct Request {
    std::string type;
    int value;
};

class Handler {
protected:
    std::shared_ptr<Handler> next;

public:
    virtual ~Handler() = default;
    
    std::shared_ptr<Handler> setNext(std::shared_ptr<Handler> handler) {
        next = handler;
        return handler;
    }
    
    virtual std::string handle(const Request& req) {
        if (next) {
            return next->handle(req);
        }
        return "Request not handled";
    }
};

class ConcreteHandlerA : public Handler {
public:
    std::string handle(const Request& req) override {
        if (req.type == "A") {
            return "Handler A processed: " + req.type;
        }
        return Handler::handle(req);
    }
};

class ConcreteHandlerB : public Handler {
public:
    std::string handle(const Request& req) override {
        if (req.type == "B") {
            return "Handler B processed: " + req.type;
        }
        return Handler::handle(req);
    }
};

int main() {
    auto handlerA = std::make_shared<ConcreteHandlerA>();
    auto handlerB = std::make_shared<ConcreteHandlerB>();
    
    handlerA->setNext(handlerB);
    
    std::cout << handlerA->handle({"A", 1}) << std::endl;
    std::cout << handlerA->handle({"B", 2}) << std::endl;
    std::cout << handlerA->handle({"C", 3}) << std::endl;
    
    return 0;
}
```

---

## 实战案例：HTTP 中间件

```python
# middleware_chain.py
from abc import ABC, abstractmethod
from typing import Dict, Any, Callable

class Request:
    def __init__(self, path: str, method: str, headers: Dict = None, body: Any = None):
        self.path = path
        self.method = method
        self.headers = headers or {}
        self.body = body
        self.user = None

class Response:
    def __init__(self, status: int = 200, body: Any = None):
        self.status = status
        self.body = body

class Middleware(ABC):
    def __init__(self):
        self._next: 'Middleware' = None
    
    def set_next(self, middleware: 'Middleware') -> 'Middleware':
        self._next = middleware
        return middleware
    
    @abstractmethod
    def handle(self, request: Request) -> Response:
        pass
    
    def next(self, request: Request) -> Response:
        if self._next:
            return self._next.handle(request)
        return Response(200, {"message": "OK"})

class LoggingMiddleware(Middleware):
    def handle(self, request: Request) -> Response:
        print(f"[LOG] {request.method} {request.path}")
        response = self.next(request)
        print(f"[LOG] Response: {response.status}")
        return response

class AuthMiddleware(Middleware):
    def handle(self, request: Request) -> Response:
        token = request.headers.get("Authorization")
        if not token:
            return Response(401, {"error": "Unauthorized"})
        request.user = "authenticated_user"
        return self.next(request)

class RateLimitMiddleware(Middleware):
    def __init__(self, limit: int = 100):
        super().__init__()
        self._limit = limit
        self._count = 0
    
    def handle(self, request: Request) -> Response:
        self._count += 1
        if self._count > self._limit:
            return Response(429, {"error": "Rate limit exceeded"})
        return self.next(request)


if __name__ == "__main__":
    # 构建中间件链
    logging = LoggingMiddleware()
    auth = AuthMiddleware()
    rate_limit = RateLimitMiddleware(limit=10)
    
    logging.set_next(auth).set_next(rate_limit)
    
    # 测试请求
    req = Request("/api/users", "GET", {"Authorization": "Bearer token"})
    response = logging.handle(req)
    print(f"Final response: {response.status}")
```

---

## 总结

### 优缺点

| 优点 | 缺点 |
|------|------|
| 降低耦合度 | 不保证请求被处理 |
| 灵活添加/删除处理者 | 可能形成很长的链 |
| 单一职责 | 调试困难 |

### 何时使用

- 多个对象可以处理同一请求
- 不明确指定接收者
- 需要动态指定处理者集合

---

[← 返回行为型模式](../README.md) | [下一章：命令模式 →](../command/README.md)

