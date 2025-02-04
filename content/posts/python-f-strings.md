---
title: "Understanding f-strings in Python"
date: 2025-02-04T01:00:00
tags:
  - python
categories: 
 - development
 
draft: false
url: "python-f-strings"
summary: "Understanding f-strings in Python"
description: "Understanding f-strings in Python"
---

# Understanding f-strings in Python

The `f` in front of the quotes is for **f-strings**, which stands for **formatted strings** in Python. They allow you to embed expressions (like variables or calculations) directly inside a string by enclosing them in curly braces `{}`.

## Why Use f-strings?  
1. **Readable and concise**: Easier to read and write compared to older formatting methods.  
2. **Efficient**: Faster and more efficient than older techniques like `format()` or `%` formatting.  
3. **Flexible**: Can embed any valid Python expression.  

---

## Example: Building a URL with f-strings  

```python
url = f"https://{fortigate_ip}/api/v2/monitor/system/interface/arp"
```

The value of `fortigate_ip` (which is `"192.168.1.99"`) gets inserted into the string, making `url` equal to:  
```
"https://192.168.1.99/api/v2/monitor/system/interface/arp"
```

---

## Another Example: Embedding Variables in a String  

```python
name = "FortiGate"
version = "7.2.1"

print(f"My firewall is {name} running version {version}.")
```

### Output:  
```
My firewall is FortiGate running version 7.2.1.
```

---

## Without f-strings  

Before f-strings, you would have to use the `format()` method:  

```python
print("My firewall is {} running version {}.".format(name, version))
```

As you can see, f-strings are a **cleaner**, **faster**, and more **modern** approach to string formatting in Python.  

---

f-strings were introduced in Python 3.6 and have since become the preferred way to format strings in modern Python code. If you're writing automation scripts or working with APIs, f-strings can help make your code more readable and efficient.
