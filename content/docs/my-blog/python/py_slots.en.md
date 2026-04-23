---
title: "为什么 Python `__slots__` 不只是节省内存？"
blogType: "tutorial"
weight: 3
bookToc: true
---

# 为什么 Python `__slots__` 不只是节省内存？

很多 Python 开发者都听说过 `__slots__`，知道它可以“省内存”。但如果你只把它当成一个性能小技巧，那其实低估了它的价值。

这篇文章我们从一个真实工程问题出发，系统讲清楚：

- `__slots__` 到底在做什么？
- 为什么它能节省内存？
- 除了性能，还有哪些设计上的好处？
- 和 `@dataclass` 应该怎么搭配？

---

## 从一个真实问题开始

假设你在做一个 IoT 系统，需要处理大量传感器数据：

```python
class SensorData:
    def __init__(self, device_id, temperature, humidity):
        self.device_id = device_id
        self.temperature = temperature
        self.humidity = humidity

# 模拟 100 万条数据
records = [SensorData(i, 25.0, 60.0) for i in range(1_000_000)]
```

代码很简单，但问题来了：

👉 **内存占用非常大**

为什么？

---

### 真正的罪魁祸首：`__dict__`

默认情况下，每个 Python 对象都有一个：

```python
obj.__dict__
```

它本质是一个字典，用来存储属性：

```python
s = SensorData(1, 25, 60)
print(s.__dict__)
# {'device_id': 1, 'temperature': 25, 'humidity': 60}
```

优点很明显：

✅ 可以随时加属性

```python
s.status = "OK"
```

但代价是什么😧：

* 每个对象都带一个 dict
* dict 有额外空间开销（哈希表）
* 对象越多 → 内存爆炸

---

### 如果我们使用 `__slots__`

我们改造一下：

```python
class SensorData:
    __slots__ = ['device_id', 'temperature', 'humidity']

    def __init__(self, device_id, temperature, humidity):
        self.device_id = device_id
        self.temperature = temperature
        self.humidity = humidity
```

变化很关键：

👉 **不再使用 `__dict__`**

使用 `__slots__` 后，Python 会：

- 不再创建 `__dict__`
- 用一个固定结构存储属性（类似数组）

---

## 📊 实测对比

```python
import sys
import time
import tracemalloc


class SensorData:
    def __init__(self, device_id, temperature, humidity):
        self.device_id = device_id
        self.temperature = temperature
        self.humidity = humidity


class SensorDataWithSlots:
    __slots__ = ['device_id', 'temperature', 'humidity']

    def __init__(self, device_id, temperature, humidity):
        self.device_id = device_id
        self.temperature = temperature
        self.humidity = humidity


def measure_memory_and_speed(cls, n_objects=1_000_000):
    # 启动内存跟踪
    tracemalloc.start()
    
    # 创建对象
    start_time = time.time()
    objects = [cls(i, 25, 20) for i in range(n_objects)]
    creation_time = time.time() - start_time
    
    # 测量内存
    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    
    # 测试属性访问速度
    start_time = time.time()
    for obj in objects:
        _ = obj.device_id
        _ = obj.temperature
        _ = obj.humidity
    access_time = time.time() - start_time
    
    return {
        "内存占用(MB)": peak / 1024 / 1024,
        "对象创建时间(秒)": creation_time,
        "属性访问时间(秒)": access_time
    }


def main():
    # 测试普通类
    print("测试普通类:")
    dict_results = measure_memory_and_speed(SensorData)
    for k, v in dict_results.items():
        print(f"{k}: {v:.2f}")
    
    print("\n测试使用 __slots__ 的类:")
    slots_results = measure_memory_and_speed(SensorDataWithSlots)
    for k, v in slots_results.items():
        print(f"{k}: {v:.2f}")
    
    # 计算差异百分比
    print("\n性能提升:")
    for k in dict_results:
        improvement = (dict_results[k] - slots_results[k]) / dict_results[k] * 100
        print(f"{k}: 提升 {improvement:.1f}%")

    # 展示单个对象的大小差异
    normal_obj = SensorData(1, 25, 20)
    slots_obj = SensorDataWithSlots(1, 25, 20)
    print(f"\n单个对象大小对比:")
    print(f"普通对象: {sys.getsizeof(normal_obj)} bytes")
    print(f"普通对象的__dict__: {sys.getsizeof(normal_obj.__dict__)} bytes")
    print(f"普通对象总大小: {sys.getsizeof(normal_obj) + sys.getsizeof(normal_obj.__dict__)} bytes")
    print(f"Slots对象: {sys.getsizeof(slots_obj)} bytes")
    try:
        print(f"Slots对象的__dict__: {sys.getsizeof(slots_obj.__dict__)} bytes")
    except AttributeError as e:
        print(f"Slots对象没有__dict__属性：{e}")

if __name__ == "__main__":
    main()

```

输出结果:

```python
测试普通类:
内存占用(MB): 122.49
对象创建时间(秒): 11.60
属性访问时间(秒): 0.18

测试使用 __slots__ 的类:
内存占用(MB): 91.97
对象创建时间(秒): 9.01
属性访问时间(秒): 0.19

性能提升:
内存占用(MB): 提升 24.9%
对象创建时间(秒): 提升 22.4%
属性访问时间(秒): 提升 -4.6%

单个对象大小对比:
普通对象: 48 bytes
普通对象的__dict__: 96 bytes
普通对象总大小: 144 bytes
Slots对象: 56 bytes
Slots对象没有__dict__属性：'SensorDataWithSlots' object has no attribute '__dict__'


```

---

## 性能提升来自哪里？

### 1️⃣ 内存更少

没有 `__dict__`

👉 每个对象节省几十到上百字节

---

### 2️⃣ 访问更快

普通情况：

```text
属性访问 → dict 查找 → 哈希
```

slots：

```text
属性访问 → 直接偏移量
```

👉 更像 C 结构体

---

## 不只是性能：设计层面的优势

### 1️⃣ 明确“数据结构”

```python
class User:
    __slots__ = ['id', 'name']
```

👉 等价于：

“这个类只能有这两个字段”

---

### 2️⃣ 防止拼写错误

```python
u = User()
u.nmae = "Alice"  # 直接报错
```

👉 提前发现 bug

---

### 3️⃣ 更像“类型系统”

你其实在做：

👉 限制对象结构

这在大型项目中非常重要

---

## 和 `@dataclass` 的关系

很多人会问：

👉 `dataclass` 不是已经很好了吗？

来看：

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int
```

👉 默认还是有 `__dict__`

---

### 推荐写法（Python 3.10+）

```python
@dataclass(slots=True)
class Point:
    x: int
    y: int
```

👉 最佳组合：

- 自动生成方法
- 内存优化

---

## 使用 `__slots__` 的限制

### ❌ 不能随便加属性

```python
obj.new_attr = 1  # 报错
```

---

### ❌ 某些库不兼容

比如依赖 `__dict__` 的库

---

### ❌ 调试稍微麻烦

```python
vars(obj)  # 会报错
```

---

## 什么时候该用？

推荐场景:

- 大量对象（10万+）
- 数据结构固定
- 性能敏感系统（如交易 / 实时处理）

---

不推荐:

- 需要动态属性
- 业务逻辑复杂、结构不稳定

---

## 最佳实践

👉 如果你用 Python 3.10+：

```python
@dataclass(slots=True)
class Trade:
    symbol: str
    price: float
```

---

## 总结

`__slots__` 本质不是“优化技巧”，而是：

👉 **一种数据建模方式**

它让你的类：

- 更节省内存
- 更快
- 更严格

但代价是：

👉 失去一部分灵活性

---

💡 最重要的一点：

> 先写正确的代码，再优化；
> 但当你需要优化时，`__slots__` 是一个非常值得考虑的工具。
