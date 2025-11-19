---
sys:
  pageId: "2a81f336-f350-806a-bec1-e6e7e69b677e"
  createdTime: "2025-11-11T06:27:00.000Z"
  lastEditedTime: "2025-11-11T06:32:00.000Z"
  propFilepath: "posts/python-decorator/index.md"
title: "Python Decorator"
date: "2025-11-11T00:00:00Z"
description: ""
tags:
  - "🏷️Python"
categories:
  - "⌨️軟體開發"
author: "Harry Yang"
draft: false
url: "posts/python-decorator"
section: "posts"
---

# What are decorators


經常能在 Python 程式碼中看到 function 定義的前一行有`@`，這是用來修飾函數的語法，稱作為裝飾子(decorator)。
裝飾子的功能是在原有的物件上加上其他功能，而不會修改原有物件的定義。
當有一連串物件需要做到相同的修飾時，decorator能大幅簡化編寫流程，讓程式碼更易讀並減少重複性。


```python
@my_decorator # this declare func is decorated by my_decorator
def func(param):
    return res
```


而在Python語言中能做到這樣的功能，是因為在 Python 中，function 是屬於 First-class Citizen，
故稱作 First-class function (一級函數、頭等函數)，就是 function 也可以當成參數傳遞並執行。


在計算機科學中，如果一個編程語言中的實體（如函數、物件等）滿足以下條件，則被稱為一等公民：

- **可以作為參數傳遞給函數**。
- **可以賦值給變量或存儲在數據結構中**。
- **可以作為函數的返回值**。
- **可以在執行期(runtine)被創建，無需在設計期全部寫出**。

回到裝飾子的介紹，裝飾子利用 Python function 是一級函數的特性，將原有的 function 當作引數傳入，並增加一些功能再回傳。


而`@my_decorator`的寫法是一種 syntactic sugar(語法糖、語法糖衣)，能進一步減少重複編寫代碼。
在使用這種寫法前，原本的修飾方法會是`func = my_decorator("func")`。
這個範例是將原有的 sum 方法進行修飾，在執行前印出"Calling my_decorator”。
達到不修改原有方法的定義卻可以增加函式輸出的內容。


```python
def my_decorator(func):
    # take all the parameters from origin function
    def wrapper(*args, **kwargs):
        # do something before `sum`
        print("Calling my_decorator")
        # run the origin function
        res = func(*args, **kwargs)
        return res
    return wrapper

# decorate sum() without syntax candy
def sum(int:a, int:b=10) -> int:
		return a+b
sum = my_decorator(sum)

# decorate sum() with syntax candy
@my_decorator
def sum(int:a, int:b=10) -> int:
		return a+b
```

除此之外，修飾子也能帶有參數。這時需要在外層再包一個定義，將 decorator 的參數帶入。

```python
def my_decorator(str:msg):
		def decorator(func):
		    # take all the parameters from origin function
		    def wrapper(*args, **kwargs):
		        # do something before `sum`
		        print(f"Calling my_decorator with message: {msg}")
		        # run the origin function
		        res = func(*args, **kwargs)
		        return res
		    return wrapper
		 return decorator

@my_decorator('Hello World')
def sum(int:a, int:b=10) -> int:
		return a+b

sum(5)
# >> Calling my_decorator with message: Hello World
# >> 15
```

裝飾子是一個非常方便的設計模式，經常用來幫助程式的開發。

**1. 日誌記錄(Logging):**

使用裝飾子可以在函數執行前後記錄相關資訊，例如時間、輸入參數、輸出結果等，方便除錯和監控。

**2. 性能測試(Performance Testing):**

裝飾子可以計算函數的執行時間，幫助開發者找出性能瓶頸，優化程式碼。

**3. 權限驗證(Authorization):**

在需要保護的函數上使用裝飾器，可以在函數執行前進行權限驗證，確保只有授權的使用者才能訪問。

**4. 緩存(Caching):**

裝飾子可以將函數的輸出結果緩存起來，避免重複計算，提高執行效率。

**5. 函數呼叫次數限制(Rate Limiting):**

裝飾子可以限制函數的呼叫次數，例如防止惡意攻擊或過度使用資源。

# Decorator的進階概念

## 先後順序 chain of decorators

一個物件能被多個裝飾子修飾，且修飾存在先後順序。函數將被裝飾子由內而外包裹、由外而內執行。

```python
def deco1(func):
    def warp_1():
        print("deco1")
        func()
        print("deco1 end")
    return warp_1

def deco2(func):
    def warp_2():
        print("deco2")
        func()
        print("deco2 end")
    return warp_2

@deco1
@deco2
def foo():
    print("foo")
# equivalent to foo = deco1(deco2(foo))

foo()
# >> deco1
# >> deco2
# >> foo
# >> deco2 end
# >> deco1 end
```

由ChatGPT提供的呼叫流程圖

```python
foo()           → warp_1()
  ↓             → print("deco1")
  ↓             → warp_2()
     ↓          → print("deco2")
     ↓          → foo()
        ↓       → print("foo")
     ↑          → print("deco2 end")
  ↑             → print("deco1 end")
```

## functools.wrap

另外 decorator 的作用是以一個函式作為參數，然後丟回一個新的函式。這會改變被包裝的函式的名字與 doc string。在產出工作日誌時可能需要特別注意，不過可以透過使用 functools.wrap 再包裝一次來修正。

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        res = func(*args, **kwargs)
        return res
    return wrapper

@my_decorator
def greet():
    print("Hello! World")

print(greet.__name__)  
# expected a 'greet' output while using @functools.wraps
# output will be 'wrapper' without using @functools.wraps

```

## Class Decorator

- Use Class as decorator to add metadata to functions

	```python
	class CallCounter:
	    def __init__(self, function):
	        self.function = function
	        self.count = 0

	    def __call__(self, *args, **kwargs):
	        self.count += 1
	        print(f"Function {self.function.__name__} has been called {self.count} times.")
	        return self.function(*args, **kwargs)

	@CallCounter
	def say_hello():
	    print("Hello!")

	say_hello()
	say_hello()
	# Output:
	# Function say_hello has been called 1 times.
	# Hello!
	# Function say_hello has been called 2 times.
	# Hello!
	```

- Use decorator to track or control class object

	經常使用的範例包含為所有 class function 加上 log 紀錄、管理控制 class 的權限、紀錄 class 的實例來進行資源管理、registry 註冊器(自動註冊 class 到 dictplugin 架構、自動化加載)。


	```python
	def track_instances(cls):
	    cls._instances = []
	    original_init = cls.__init__

	    def new_init(self, *args, **kwargs):
	        cls._instances.append(self)
	        original_init(self, *args, **kwargs)

	    cls.__init__ = new_init
	    return cls

	@track_instances
	class User:
	    def __init__(self, name):
	        self.name = name

	User("Alice")
	User("Bob")

	print(User._instances)  # [<User object>, <User object>]

	```

## 重試機制@retry

當函式發生錯誤時，自動重新執行（retry），對於「可能暫時性失敗」的操作非常有用，例如：

- 呼叫不穩定的 API
- 資料庫連線
- 下載網頁、爬蟲
- 操作外部資源（如 GCP、AWS、S3、FTP）

在 Python 中可以透過 tenacity 套件，使用已經定義好的 decorator 來幫助開發，輕鬆且方便的達成重試的功能。
基本使用可以指定重複的次數，以及每次重試需要等待的時間

```python
from tenacity import retry, stop_after_attempt, wait_fixed

@retry(stop=stop_after_attempt(3), wait=wait_fixed(2))
def risky_function():
    print("Trying...")
    raise ValueError("Temporary error")
```

在更進階的且常用的參數包含

| 類別           | 引數名稱                                          | 說明                  |
| ------------ | --------------------------------------------- | ------------------- |
| **停止條件**     | `stop=stop_after_attempt(n)`                  | 執行最多 n 次（包含第一次）     |
|              | `stop=stop_after_delay(t)`                    | 最多花 t 秒             |
| **等待時間**     | `wait=wait_fixed(t)`                          | 每次重試都等固定時間 t 秒      |
|              | `wait=wait_random(min, max)`                  | 每次重試等隨機時間           |
|              | `wait=wait_exponential(multiplier=1, max=60)` | 指數回退，常用於 API        |
| **retry 條件** | `retry=retry_if_exception_type()`             | 碰到特定 exception 才重試  |
|              | `retry=retry_if_result(lambda r: r is None)`  | 結果不符合條件也重試          |
| **回呼**       | `before=before_log(logger, DEBUG)`            | 每次執行前記 log          |
|              | `after=after_log(logger, DEBUG)`              | 每次執行後記 log          |
| **其他**       | `reraise=True`                                | 重試完仍失敗，**拋出例外**（常用） |


# Log decorator 實作

了解 Python 的 decorator 技巧後，接著來示範如何實作一個 log decorator 來幫助紀錄及管理開發及應用的過程。

- Decorate functions

```python
import os
import logging
from typing import Callable
from functools import wraps

def get_logger(name: str) -> logging:
    """ """
    logger = logging.getLogger(f"{name}")
    handler = logging.StreamHandler()
    formatter = logging.Formatter(
        "time: %(asctime)s | funcName: %(funcName)s | line: %(lineno)d | level: %(levelname)s | message:{%(message)s}"
    )
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    logger.setLevel(logging.DEBUG)
    return logger

def log(logger: str) -> Callable:
    """ """
    def exception_handler(func: Callable):
        @wraps(func)
        def inner_function(*args, **kwargs):
            filename = os.path.basename(func.__code__.co_filename)
            try:
                res = func(*args, **kwargs)
                return res
            except Exception as e:
                logger.error(
                    f"[ERROR]==> File: {filename} | Function: {func.__name__} | MSG: {e}"
                )
        return inner_function
    return exception_handler

logger = get_logger(__name__)
```

- Decorate the hole class

```python
import os
import logging
from functools import wraps
import inspect

def get_logger(name: str) -> logging:
    """ """
    logger = logging.getLogger(f"{name}")
    handler = logging.StreamHandler()
    formatter = logging.Formatter(
        "time: %(asctime)s | funcName: %(funcName)s | line: %(lineno)d | level: %(levelname)s | message:{%(message)s}"
    )
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    logger.setLevel(logging.DEBUG)
    return logger

def log_all_methods(logger, exclude=None):
    """
    decorate the all class func with logging,
    and can exclude specific methods.
    """
    if exclude is None:
        exclude = []

    def decorate(cls):
        for name, method in inspect.getmembers(cls, predicate=inspect.isfunction):
            if name.startswith("__") or name in exclude:
                continue

            original_method = method

            @wraps(method)
            def wrapped(self, *args, __method=original_method, **kwargs):
                filename = os.path.basename(__method.__code__.co_filename)
                try:
                    result = __method(self, *args, **kwargs)
                    return result
                except Exception as e:
                    logger.error(
                        f"[ERROR]==> File: {filename} | Function: {__method.__name__} | MSG: {e}"
                    )

            setattr(cls, name, wrapped)
        return cls

    return decorate

```

---

# 附錄

**參考來源**

- [https://wayne-blog.com/2023-03-16/python-decorator-tutorial/](https://wayne-blog.com/2023-03-16/python-decorator-tutorial/)
- [https://ankitbko.github.io/blog/2021/04/logging-in-python/](https://ankitbko.github.io/blog/2021/04/logging-in-python/)
- [https://medium.com/citycoddee/python進階技巧-3-神奇又美好的-decorator-嗷嗚-6559edc87bc0](https://medium.com/citycoddee/python%E9%80%B2%E9%9A%8E%E6%8A%80%E5%B7%A7-3-%E7%A5%9E%E5%A5%87%E5%8F%88%E7%BE%8E%E5%A5%BD%E7%9A%84-decorator-%E5%97%B7%E5%97%9A-6559edc87bc0)
- [https://medium.com/程式愛好者/什麼是裝飾器-decorator-a344ac5b47c0](https://medium.com/%E7%A8%8B%E5%BC%8F%E6%84%9B%E5%A5%BD%E8%80%85/%E4%BB%80%E9%BA%BC%E6%98%AF%E8%A3%9D%E9%A3%BE%E5%99%A8-decorator-a344ac5b47c0)
- [https://medium.com/@manikolbe/python-decorators-for-data-engineering-5-real-world-use-cases-24a919d417a1](https://medium.com/@manikolbe/python-decorators-for-data-engineering-5-real-world-use-cases-24a919d417a1)
- [https://www.datacamp.com/tutorial/decorators-python](https://www.datacamp.com/tutorial/decorators-python)
- [https://realnewbie.com/coding/python/tenacity-powerful-python-retry-library/](https://realnewbie.com/coding/python/tenacity-powerful-python-retry-library/)
