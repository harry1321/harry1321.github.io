---
sys:
  pageId: "29c1f336-f350-80ef-951c-f9cebfbd3c6e"
  createdTime: "2025-10-30T07:09:00.000Z"
  lastEditedTime: "2025-11-10T09:50:00.000Z"
  propFilepath: "/posts/f-string/index.md"
title: "Python f-string"
date: "2025-11-10T00:00:00Z"
description: "掌握 Python 的 f-string 基礎，學會在程式碼中優雅地處理字串格式化。從基本語法到進階技巧，學習如何編寫更簡潔、更易於維護的程式碼。"
tags:
  - "🏷️Python"
categories:
  - "⌨️軟體開發"
author: "Harry Yang"
draft: false
url: "/posts/f-string"
section: "posts"
---

# 快速有效的格式化字串


## 基本


Python f-string 提供簡單且更易於閱讀的格式化方法，來設定想要表達的字串型式，可以看到下面的是示範。大括號表示的是佔位符號 (Replacement Field)，透過定義大括號的變數，可以直接在字面表達中嵌入程式中的任何變數，並透過修飾子 (modifier) 來 formatting 變數達到想要的輸出格式。自 Python 3.8 起，在 `{ }` 中直接加入 `=` 運算符，能將變數名稱與變數值顯示在一起，方便偵錯。


```python
name = "Joe"
age = 25
wallet = 123.45
lunch = 9.99
dinner = 19.99

# Using f-string, by add the letter 'f' or 'F' before your string
print(f"My name is {name} and I'm {age} years old")
# Formatting strings
print(f"I've ${wallet:.2f} in my pocket.")
# Calculations right in the string!
print(f"Today I've spent ${lunch + dinner:.2f} on my meals.")  
# Useful for debugging
print(f"{lunch = }, {dinner = }")
""" Output:
My name is Joe and I'm 25 years old
I've $123.45 in my pocket.
Today I've spent $29.98 on my meals.
lunch = 9.99, dinner = 19.99
"""
```


## 更靈活的用法


在上面的範例就可以看到 f-string 的 placeholder 不僅能接受一般的變數，也能直接寫入運算式讓程式直接計算結果。所以 f-string 也能直接取 list 或 dictionary 中的元素，同樣能使用 function 來計算回傳值也是可以的。

- list 和 dictionary 取值回傳

```python
fruits = ["apple", "banana", "orange"]
user_info = {
    "name": "Alice",
    "age": 28,
    "city": "New York"
}
print(f"User: {user_info['name']} from {user_info['city']}")
print(f"First fruit: {fruits[0]}")
print(f"All fruits: {', '.join(fruits)}")
""" Output:
User: Alice from New York
First fruit: apple
All fruits: apple, banana, orange
"""
```

- 使用 function 進行運算，包含使用條件表達式來限制輸出的內容。

```python
numbers = [1, 2, 3, 4, 5]
text = "python"
print(f"List length: {len(numbers)}") # List length: 5
print(f"Maximum value: {max(numbers)}") # Maximum value: 5
print(f"Uppercase text: {text.upper()}") # Uppercase text: PYTHON
```


```python
# Simple conditional formatting
score = 85
print(f"Result: {'Pass' if score >= 70 else 'Fail'}")
# Multiple conditions
value = 42
print(f"Status: {'High' if value > 75 else 'Medium' if value > 25 else 'Low'}")

""" Output:
Result: Pass
Status: Medium
"""
```


# 修飾字串輸出格式


了解 f-string 的使用方式後，接下來要探討如何調整輸出字串的外觀格式，包含數字格式和文字排版。良好的文字格式和排版，是為了確保輸出的文字能夠有效、清楚且用更容易閱讀的方式傳遞和紀錄，是輸出專業資料的實用技巧。


要達到控制字串格式，需要在 f-string 保留的輸出變數會函數之後加上冒號，後面跟著想要表達的格式(format specifiers)。首先將從數字的格式化開始，介紹如何編寫各式各樣的格式規範。


## 數字格式


根據不同需求，對數字的格式要求也不相同，在金融領域可能需要用到千位分隔符來表示數字，在科學領域需要用到科學符號。數字格式化的需求可能包含這幾種，小數點後的位數、千位分隔符號、數字排版、數字補零(zero padding)、百分比等，以下就用表格來呈現如何使用這些格式內容。


|                | Example Output | Replacement Field | Fill | Width | Grouping | Precision | Type |
| -------------- | -------------- | ----------------- | ---- | ----- | -------- | --------- | ---- |
| 小數點後的位數+千位分隔符號 | 4,125.60       | {float:.2f}       |      |       | ,        | .2        | f    |
| 小數點後的位數+數字補零   | 04125.60       | {float:08.2f}     | 0    | 8     |          | .2        | f    |
| 保留兩位數的科學表示     | 4.1e+03        | {float:.2g}       |      |       |          | .2        | g    |
| 保留八位數的科學表示     | 4125.6         | {float:.8g}       |      |       |          | .8        | g    |
| 百分比            | 37%            | {percent:.0%}     |      |       |          | .0        | %    |
| 數字補零           | 0010           | {integer:04d}     | 0    | 4     |          |           | d    |
| 數字排版           |     10         | {integer: 4d}     |      | 4     |          |           | d    |


## 時間類型(datetime)的格式規範


當想要輸出時間類型(datetime)的字串內容，除了使用 strftime 的方法直接進行轉換之外，f-string也直接支援能套用[相同的格式規範](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes)來進行格式化。


```python
import datetime
somedate = datetime.date(1971, 5, 26)
print(f"It was {somedate.strftime('%B %d, %Y')}.")
print(f"It was {somedate:%B %d, %Y}.")

""" Output
It was May 26, 1971.
It was May 26, 1971.
"""
```


## 客製化字串輸出格式


除了上面介紹的常用輸出規範之外，在 Python 中能夠自訂一個實體的 f-string 格式規範。這時候需要在類別中定義 `__format__` 方法，並設定規範關鍵字及要如何輸出這個規範格式。這個方法會根據括號內冒號後的格式規範，來控制類別實例在 f-string 中的表示方式。


```python
class Text:
    def __init__(self, text: str) -> None:
        self.text = text
    def __format__(self, format_spec: str) -> str:
        match format_spec:
            case 'upper':
                return self.text.upper()
            case 'lower':
                return self.text.lower()
            case 'length':
                return str(len(self.text))
            case _:
                raise ValueError(f'Format specifier "{format_spec}" does not exist')

mytext = Text("LovePython")
print(f"{mytext:upper}")  # LOVEPYTHON
print(f"{mytext:lower}")  # lovepython
print(f"{mytext:length}") # 10
print(f"{mytext:bit}")    # ValueError: Format specifier "bit" does not exist.
```


## 字串對齊和填充


文字內容有了清楚簡潔的排版更能輔助閱讀，讓注意力更專注在內容上。要達到簡易的排版，不需要手動排版補充空格符號，使用`<N`、 `>N`、`^N` 這三個符號分別能調整文字為靠左、靠右和置中，並設定`N` 來保留所需要的字元數。透過以下簡單範例就能理解：


```python
# Basic text alignment
name = "Python"
width = 20
print(f"Left aligned:   |{name:<20}|")
print(f"Right aligned:  |{name:>20}|")
print(f"Center aligned: |{name:^20}|")
""" Output:
Left aligned:   |Python              |
Right aligned:  |              Python|
Center aligned: |       Python       |
"""
```


## 建立多行字串


了解如何要求數字和字串的格式後，就能活用這些規則生產一篇簡潔且工整的報告內容，並將其應用在呈現資料統計、程式內容偵錯或程式運作日誌等各式各樣的日常報表。處理較大的文字區塊或複雜的字串格式時，持續使用單行 f-string 會使得程式碼雜亂且難以閱讀。因此 f-string 支援文字區塊的使用方法，讓撰寫多行文字變得輕鬆簡易。使用方法非常簡單，只需要將字串包圍在兩個連續的三個雙引號(單引號)就能創建 f-string 文字區塊。以下就以一個範例來示範如何用 f-string 產出一段簡潔且工整的文字內容：


```python
# Basic multiline f-string
name = "Alice"
role = "Data Scientist"
experience = 5
profile = f"""
Team Member Profile:
------------------
Name: {name}
Role: {role}
Experience: {experience} years
"""

""" Output
Team Member Profile:
------------------
Name: Alice
Role: Data Scientist
Experience: 5 years
"""
```


然而在一份報告中可能會包含不同的內容區塊，比如一份會計報告包含資產、負債和股東權益。為了讓文字內容達到模組化的管理，也能將各區塊的文字內容分割，在最後輸出時再合併成完整的報告，如此一來更能提升整體程式碼的可讀性。


```python
# Building a report with multiple f-strings
title = "Sales Analysis"
period = "Q3 2024"
sales = 150000
growth = 27.5
header = f"""
{title}
{'=' * len(title)}
Period: {period}
"""
metrics = f"""
Performance Metrics:
- Total Sales: ${sales:,}
- Growth Rate: {growth}%
"""
notes = f"""
Additional Notes:
- All figures are preliminary
- Data updated as of {period}
"""
# Combining all sections
full_report = f"{header}\n{metrics}\n{notes}"
print(full_report)

""" Output
Sales Analysis
=============
Period: Q3 2024
Performance Metrics:
- Total Sales: $150,000
- Growth Rate: 27.5%
Additional Notes:
- All figures are preliminary
- Data updated as of Q3 2024
"""
```


# 實際應用


了解完 f-string 的使用方法，接著用幾個適用情境來示範 f-string 如何實際應用。


## SQL


```python
table_name = "users"
columns = ["name", "age", "city"]
condition = "age > 30"
sql_query = f"""
SELECT {', '.join(columns)}
FROM {table_name}
WHERE {condition}
ORDER BY registration_date DESC;
"""
```


## 統計資料


```python
# Statistical data presentation
dataset = {
    'mean': 67.89123,
    'median': 65.5,
    'std_dev': 12.34567,
    'min_value': 42.5,
    'max_value': 95.75
}
stats_summary = f"""
Statistical Analysis
-------------------
Mean:        {dataset['mean']:.2f}
Median:      {dataset['median']:.1f}
Std Dev:     {dataset['std_dev']:.3f}
Range:       {dataset['min_value']:.1f} - {dataset['max_value']:.1f}
"""
print(stats_summary)
```


## Logging與除錯


```python
# Enhanced debugging with f-strings
def analyze_data(data_list, threshold):
    # Debug information about input parameters
    print(f"[DEBUG] Processing {len(data_list)} items with threshold {threshold}")

    filtered_data = [x for x in data_list if x > threshold]

    # Debug information about processing results
    print(f"[DEBUG] Found {len(filtered_data)} items above threshold")
    print(f"[DEBUG] Filtered ratio: {len(filtered_data)/len(data_list):.1%}")

    return filtered_data
# Example usage
data = [1, 5, 3, 7, 2, 9, 4, 6]
result = analyze_data(data, 5)
```

---

# 附錄

**參考來源**

- [https://www.datacamp.com/tutorial/python-f-string](https://www.datacamp.com/tutorial/python-f-string)

- [https://www.pythonmorsels.com/string-formatting/](https://www.pythonmorsels.com/string-formatting/)

- [https://youtu.be/9saytqA0J9A?t=980&si=ZAkzSmXc-lvqCNhO](https://youtu.be/9saytqA0J9A?t=980&si=ZAkzSmXc-lvqCNhO)

- [https://fstring.help/cheat/](https://fstring.help/cheat/)

**延伸篇章**
- https://www.cnblogs.com/pythonista/p/18852424
