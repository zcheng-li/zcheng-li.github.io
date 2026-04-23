---
title: "什么是 Python @dataclass ?"
blogType: "tutorial"
weight: 2
bookToc: true
---

# 什么是 Python `@dataclass` ?

---

## 引言

如果你写过这样的类：

```python
class Trade:
    def __init__(self, symbol, price, quantity):
        self.symbol = symbol
        self.price = price
        self.quantity = quantity
```

你可能觉得：

👉 “这不就很正常吗？”

但实际上，这类代码在项目中会大量重复，而且还存在一些明显问题：

- 需要手写 `__init__`
- 打印对象时可读性差
- 没有自动比较、排序能力
- 容易出现重复样板代码

这正是 `@dataclass` 诞生的意义。

---

## 什么是 dataclass？

`dataclass` 是 Python 3.7 引入的标准库工具，用于：

👉 **快速定义以“数据”为核心的类**

它可以自动生成：

- `__init__`
- `__repr__`
- `__eq__`

从而显著减少重复代码。

---

## 从传统写法到 dataclass

### 传统写法

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

打印：

```python
print(User("Alice", 20))
```

输出：

```text
<__main__.User object at 0x...>
```

👉 几乎不可读

---

### dataclass 写法

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
```

输出：

```text
User(name='Alice', age=20)
```

👉 自动生成 `__repr__`，可读性显著提升

---

## dataclass 的核心价值

### 1️⃣ 减少样板代码

无需再手写：

- `__init__`
- `__str__ / __repr__`
- 比较函数

---

### 2️⃣ 更清晰的数据建模

```python
@dataclass
class Order:
    id: int
    price: float
```

👉 类结构一目了然

---

### 3️⃣ 自动支持比较

```python
@dataclass
class Point:
    x: int
    y: int

Point(1,2) == Point(1,2)  # True
```

---

## 进阶用法

### 默认值

```python
@dataclass
class Config:
    host: str = "localhost"
    port: int = 8080
```

---

### 可变对象陷阱

❌ 错误写法：

```python
@dataclass
class A:
    items: list = []
```

运行之后发现，在 `items` 属性那行会报错：

```python
ValueError: mutable default <class 'list'> for field items is not allowed: use default_factory

```

大概的意思就是，`list` 作为一种可变的类型（引用类型，会有被其他对象意外修改的风险），不能直接作为默认值，需要用工厂方法来产生默认值。

其他字符串，数值，布尔类型的数据则没有这个问题。

我们只要定义个函数来产生此默认值即可。

✅ 正确写法：

```python
from dataclasses import field

def gen_items():
    return ["1", "2"]

@dataclass
class A:
    items: list = field(default_factory=gen_items)
```

👉 每个实例独立

---

### 隐藏敏感信息 `repr`

我们打印对象信息的时候，有时执行打印其中几个属性的信息，涉及敏感信息的属性不希望打印出来。

```python
from dataclasses import dataclass, field

@dataclass
class User:
    name: str
    password: str = field(repr=False)

if __name__ == "__main__":
    user = User("zcheng", "123456")
    print(user)
```

👉 避免敏感信息泄露

---

### 不可变对象 `frozen`

```python
@dataclass(frozen=True)
class Config:
    host: str
    port: int
```

👉 属性不可修改，更安全

---

### 初始化后处理

```python
@dataclass
class User:
    age: int

    def __post_init__(self):
        if self.age < 0:
            raise ValueError("年龄不能为负")
```

---

### 与 dict / tuple 转换

```python
from dataclasses import asdict, astuple

user = User("Alice", 20)

print(asdict(user))
print(astuple(user))
```

👉 在数据传输 / API / ML pipeline 中非常实用

---


## 什么时候使用 dataclass？

推荐场景

- 数据结构类（DTO / config / record）
- 数据处理 / 机器学习 pipeline
- API 数据对象
- 配置管理

---

不推荐场景

- 行为复杂的类（大量业务逻辑）
- 需要动态添加属性的类

---

## 最佳实践

```python
@dataclass(slots=True, frozen=True)
class Trade:
    symbol: str
    price: float
```

---

## 总结

`dataclass` 的本质不仅是语法简化，更是一种：

👉 **结构化数据建模方式**

它带来的提升：

- 更少代码
- 更高可读性
- 更安全的数据结构

---

💡 一句话总结：

> 如果一个类“只是用来存数据”，那它几乎一定应该是 dataclass。

---

## 📎 原文地址

👉 https://www.cnblogs.com/wang_yb/p/18077397