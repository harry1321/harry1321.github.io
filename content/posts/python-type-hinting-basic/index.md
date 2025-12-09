---
sys:
  pageId: "2c31f336-f350-80e1-b13f-c4d286e8647a"
  createdTime: "2025-12-08T10:10:00.000Z"
  lastEditedTime: "2025-12-08T11:21:00.000Z"
  propFilepath: "posts/python-type-hinting-basic/index.md"
title: "Unlock Python Type Hinting’s Superpowers: 5 Techniques You Need to Know"
date: "2025-12-08T00:00:00Z"
description: "介紹 Python 型別提示 (Type Hint) 的基本用法，將介紹 TypeAlias、TypedDict、TypeVar 等技術及使用原則。Type Hint 搭配靜態型別檢查器，能大幅提高程式碼可讀性、減少執行前錯誤，並強化 IDE 工具的功能。"
tags:
  - "🏷️Python"
categories:
  - "⌨️軟體開發"
author: "Harry Yang"
draft: false
url: "posts/python-type-hinting-basic"
section: "posts"
---

{{< alert "circle-info" >}}
[Type Hints cheat sheet is here](https://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html)
{{< /alert >}}

---

# Type Hinting 不只是裝飾品

許多 Python 開發者沒有習慣將型別提示 (type hint) 視為必要的程式碼，包含作者在之前開發時也是如此。因為在編寫 Python 程式時不一定要預設變數類型，缺少變數類型在實質上對程式執行沒有影響。直譯式的 Python 也正因擁有這樣的靈活性，而廣受歡迎，但也帶來了開發上的挑戰和問題。在開發過程中，缺乏類型訊息可能導致程式碼不易理解、維護成本提高、提升重構時的風險。


當型別提示與靜態型別檢查器 (static type checker) 結合使用時，它們就從單純的基礎註釋，轉變成一個強大的維護開發工具。不但能為提升程式碼可讀性，也增強 IDE 功能，在執行腳本前就捕獲到可能的錯誤。型別提示真正價值在於能夠：

- **提高可讀性**：開發時可以預期知道函數或變數的資料型別，不需要額外翻閱文件或原始碼。
- **減少錯誤**：型別提示可以搭配靜態型別檢查工具，在程式執行前找出潛在的錯誤或問題。
- **工具整合**：現代 IDE 可以利用型別提示，提供更智慧的自動補全和建議。

本文將揭示五種實用且具影響力的技術，這些技術遠超 `name: str` 這樣基礎的註釋。


## 基本標註


最基本的型別提示使用是，為變數指定一個類型。例如，有一個整數變數，可以這樣標註


```python
age: int = 38

# type checker will report an error
age: int = "38"

```

- List標註為`list[ElementType]`，不管裡面的數值個數。
- Tuple標註為`tuple[Type1, Type2, ...]`，通常會標出數目與個別型別。
- Dict標註為`dict[KeyType, ValueType]`，key與value的型別會個別標註。
- Set標註為`set[ElementType]`，不管裡面的數值個數。

但這種顯而易見的變數，實際上不太需要添加提示，因為查看程式碼就能清楚地看到變數類型。接下來將會使用這個使用者函數來為範例說明，當變數變得更複雜，如何為函數參數添加類型提示，讓類型提示真正開始發揮作用。這個函數要求輸入使用者鍵入名字、姓氏或者年齡，然後將會回傳一個 `dict`。現在將型別提示加入函數的定義中，來幫助理解。


```python
name = "Corey"
age = 38

def create_user(
        first_name: str,
        last_name: str,
        age=None,
    )->dict:

    email = f"(first_name.lower())_{last_name.lower()}@example.com"

    return {
        "first_name": first_name,
        "last_name": last_name,
        "email": email,
        "age": age,
    }

user1 = create_user("Corey", "Schafer", age=38)
user2 = create_user("John", "Doe")

print(user1)
print(user2)
```


## Optional types and the None type


當變數可以接受多種類型或甚至是 `None` 時，就需要使用到聯集與 `None` 的標註語法。在Python 3.10以上可以使用聯集符號 `|` 來實現。在這個範例中，年齡不一定需要在使用時注入，可以是 `None` 或 `int` 。實現的語法如下：


```python
def create_user(
        first_name: str,
        last_name: str,
        age: int | None = None,
    ) -> dict:
    ...

# before python 3.10
from typing import Optional

def create_user(
        first_name: str,
        last_name: str,
        age: Optional[int] = None,
    ) -> dict:
    ...
```


## TypeAlias


當程式碼中的變數變多變雜，如在範例中函數回傳值的字典，目前的寫法是 `dict[str, str | int | None]`。這樣寫不但難以理解實際的變數涵義，更增加維護的難度。此時可以賦予變數一個型別別名 `TypeAlias`，來增加程式碼的可讀性，方便開發人員記憶和管理。


```python
def create_user(
    ...
    ) -> dict[str, str | int | tuple[int, int, int] | None]:
    ...

# define what age type is and what a User class is
# give TypeAlias for self-define type
# after 3.12 you can explicitly point out a TypeAlias
# with keyword type at the front
# the old usage is: User = dict[str, str | int | None]
type User = dict[str, str | int | None]

def create_user(
        first_name: str,
        last_name: str,
        age: int | None = None,
    ) -> User:
...
```


## NewType


現在根據開發需求，將在 `user` 中新增一個顏色喜好的屬性，且只接受 RGB 顏色碼。不過同樣是使用 3 個整數組成的色碼，可能是 RGB 顏色或 HSL 值。雖然定義了 `TypeAlias` 可以提高可讀性，但如果兩個不同的概念，卻共享相同的結構，則沒辦法防止混淆的使用。


```python
...

RGB: tuple[int, int, int]
HSL: tuple[int, int, int]

...

def create_user(
        first_name: str,
        last_name: str,
        age: int | None = None,
        fav_color: RGB | None = None,
    ) -> User:

...

# mypy won't catch this logical error!
user1 = create_user("Corey", "Schafer",
                    age=38,
                    fav_color=(109, 123, 134)
                    ) # using RGB color
user2 = create_user ("John", "Doe",
                    fav_color=(206, 10, 48)
                    ) # using HSL color
```


透過 `NewType` 能建立一個新型別並賦予名稱，即使底層結構相同，類型檢查器也會將其視為不同的類型。如此一來就可以解決這種問題，在使用上會強制規範使用型別名稱，來方便區分用途，獨立了底層結構相同的型別。在編寫程式時，更能夠理解資料背後的意圖，而不僅僅是其結構，從而避免一些簡單的別名會遺漏的邏輯錯誤。


```python
from typing import NewType

# A TypeAlias helps with readability
# NewType prevents from miss used
RGB = NewType("RGB", tuple[int, int, int])
HSL = NewType("HSL", tuple[int, int, int])
...

# Mypy sees no error here
user2 = create_user ("John", "Doe",
                    fav_color=(206, 10, 48)
                    ) # wrong use of HSL color
user2 = create_user ("John", "Doe",
                    fav_color=RGB(110, 124, 135)
                    ) # correcting the mistake
```


# 字典類型的標註手法 TypedDict


當變數的型態變得複雜，例如在範例中的函數回傳一個字典，其可接受的類別有三種 `str, int, None`。因為型別檢查器不會檢查每個 key 對應的 value 型別，導致若在預期是整數的年齡欄位，輸入字串類型的年齡 ，卻不會出現錯誤的問題。


這時需要明確指定每一個變數對應的型別，其中解決這個問題的一個方法，是透過 `TypedDict` 來定義每個 key 對應的 value 類型，來限制字典的 key-value pair 的變數。


```python
from typing import NewType, TypedDict

RGB = NewType("RGB", tuple[int, int, int])

class User(TypedDict):
    first_name: str,
    last_name: str,
    age: int | None,
    fav_color: RGB | None

def create_user(
    ...
    ) -> User:
...
```


另一個方法是使用既有的類別來幫助型別檢查，使用 `@dataclass` 來定義資料類別，是 Python 的一個裝飾器，用於自動生成在主要用於儲存資料的類別中常用的特殊方法。


```python
from dataclass import dataclass
from typing import NewType

RGB = NewType("RGB", tuple[int, int, int])

@dataclass
class User:
    first_name: str,
    last_name: str,
    age: int | None = None,
    fav_color: RGB | None = None

def create_user(
    ...
    ) -> User:
    ...
    return User(
        first_name=first_name,
        last_name=last_name,
        email=email,
        age=age,
        fav_color=fav_color
    )
...
```

{{<bookmark
    name="Python Data Classes: A Comprehensive Tutorial"
    link="https://www.datacamp.com/tutorial/python-data-classes"
    description="A beginner-friendly tutorial on Python data classes and how to use them in practice">}}


{{< alert "lightbulb" >}}
- `TypedDict` 通常用於處理已經存在且具有明確定義的字典資料結構，且不需要其他的功能性函數。  
- `dataclass` 適用於創在一個嶄新的資料結構，其靈活性提供額外的函數使用功能。
{{< /alert >}}

---

# 用 TypeVar 取代 Any 保持 IDE 靈活性


## 泛型 (Generic)


泛型是一種設計方法，允許程式工程師在創建函式或類別時，不先指定具體的變數型別，在實例化時，再透過參數指定型別。這讓得程式碼更具彈性，且可重複利用，同時保證型別安全。因此在設計泛型類別的物件時，當變數可以是任意型別時，可以使用 `Any` 來標記這這類變數。但實際上，除了在被使用在args與kwargs上，應該盡量避免使用 `Any`。因為這將會失去型別標註的大部分功能，像是 IDE 將無法識別創建實例後的變數推斷，失去編寫程式碼的便利性。


## TypeVar：創建泛型變量


在 Python Typing 中，`TypeVar` (泛型變量) 允許在聲明類型時不指定具體的類型，保留空位，在實例化後以用具體的型別來決定，解決這個問題的痛點。


在 Python 3.12 前使用，需要從 `typing` 引入，並在使用前先聲明一個泛型變量。


在 Python 3.12 後，函數中定義泛型變量的語法被簡化了，將不再需要外部導入或定義 `TypeVar`。泛型變量直接在函數名後，使用中括號進行宣告即可。


以是將針對泛型更改後的範例使用說明

- 目前完成的功能包含，建立 `user` 實例並放入 `list`
- 要新建一個函數，其功能是從一個 `list` 隨機抽一位使用者和使用者的email
- 這個函數能接受存放 `str` 和 `user` 的 `list`

```python
import random

from dataclass import dataclass
from typing import NewType

...

def random_choice(items: list[User]) -> User:
    return random.choice(items)

user1 = create_user("Corey", "Schafer", age=38, fav_color=RGB(206, 10, 48))
user2 = create_user("John", "Doe",fav_color=RGB(110, 124, 135))

users = [user1, user2]
rando_user = random_choice (users)
print(rando_user)

emails = [user.email for user in users]
rando_email = random_choice(emails)
print(rando_email)
```


根據這個需求，可以指定輸入參數的型別為 `list[str] | list[User]` ，並輸出 `str | User` ;但如果使用 `TypeVar` 將不必再擔心輸入的參數。當傳入一組 email 時，IDE 將會自動識別函數的輸出會是字串，方便後續開發可以直接使用字串相關的函數，保持 Type Hinting 在 IDE 中帶來的優勢。


```python
from typing import TypeVar

# syntax for Python 3.11 and earlier
T = TypeVar("T")
def random_choice_old(items: list[T]) -> T:
    # ... returns a random item

# preferred syntax for Python 3.12+
def random_choice[T](items: list[T]) -> T:
    # ... returns a random item

```


# 第三方套件的 Type Hints 安裝


在實務開發中，像是使用到 `requests` 或 `pandas` 等的第三方函式庫，因為它們可能不包含類型提示。當你匯入一個沒有類型資訊的函式庫時，mypy 通常會發出警告，並將該套建中的所有物件都視為 `Any` 類型，從而有效地停用了所有型別檢查功能。


解決方案是安裝套件各自的 "stub packages"，這是由社群維護的套件包含套件的型別提示資訊。它們的名稱通常遵循 `types-<套件名稱>` 的模式。


例如，如果你正在使用 `requests`，`mypy` 會可能報錯說找不到類型資訊。要解決這個問題，只需安裝對應的 stub packages。


```bash
pip install types-requests
uv add --dev types-requests

```


# A New Way of Thinking About Type Hinting


Python Type Hint 遠不止於簡單的註解，它們能提升程式碼品質，提早發現 bug 並讓程式碼更易於理解的強大工具。掌握 `TypeAlias`、`TypedDict` 和 `TypeVar` 等技術，將提升在程式設計中專業度。


在未來的開發中，這些原則可能會幫助撰寫 Type Hint:

- 輸入要盡可能通用(general)，輸出應盡量具體(specific)
- 良好的標註應該提供足夠的訊息，保持明確性與簡潔性的平衡，避免過於冗長或複雜
- 謹慎使用 `Any`，在某些情況下是有用的，但過度使用會削弱類型標註的優點
- 適當的使用動態型別，在與靜態 Type Hint 間取得一個適當的平衡點
- 結合文檔來提高可讀性和維護性的技巧，以提供更全面的理解

---


# 參考來源


[Python Type Hint 型別提示教學 - 基礎篇](https://zsl0621.cc/python/type-hint#typeddict)


[用Python Typing提升程式碼的可維護性: 從基本標註到泛型標註](https://medium.com/ai-blog-tw/python-typing-guide-1d44f40790a)


[Python Type Hint 型別提示教學 - 泛型](https://zsl0621.cc/python/type-hint-generic#%E5%AF%A6%E6%88%B0%E6%B3%9B%E5%9E%8B%E6%8A%BD%E8%B1%A1%E6%96%B9%E6%B3%95-abstractmethod)


[Python dataclass 教學：輕鬆定義資料類別](https://haosquare.com/python-dataclass/)


[Python: Type Aliases vs New Types](https://gist.github.com/simonwhitaker/3a24d669b26b4f373690c6eb113c3ae7)


[Corey Schafer—Python Tutorial: Type Hints - From Basic Annotations to Advanced Generics](https://www.youtube.com/watch?v=RwH2UzC2rIo)
