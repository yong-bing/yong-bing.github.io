---
title: "TOML：Python 开发者的配置文件新选择"
date: 2025-1-15 16:00:00 +0800
categories: [Python Features]
tags: [Python, toml]
---

一些语言用于编写和运行程序，比如我们熟知的 Python Java 等等；还有些语言只用来传递信息，例如 JSON、YAML，和这次要介绍的 toml。


TOML (Tom's Obvious, Minimal Language) 是一种配置文件格式，由 GitHub 联合创始人 Tom Preston-Werner 于 2013 年创建。
它以类似 Python 的语法和完美的类型对应能力，正在成为 Python 开发中配置文件的优秀选择。

## TOML 基础语法

### 键值对
最基本的数据结构是键值对：

```toml
name = "TOML Example"
enabled = true
birthday = 1990-01-01T12:00:00Z
hobbies = ["reading", "coding"]
```
在 Python 中，这些数据可以被解析为字典：

```python
{'name': 'TOML Example', 'enabled': True, 'birthday': datetime.datetime(1990, 1, 1, 12, 0, tzinfo=<toml.tz.TomlTz object at 0x000001C01BCB11B0>), 'hobbies': ['reading', 'coding']}
```

### 表（Tables）
使用方括号定义表，类似于python中的字典：

```toml
[database]
host = "localhost"
port = 5432
username = "admin"
```

或者像字典一样使用花括号：

```toml
database = {host = "localhost", port = 5432, username = "admin"}
```

Pyhton 中解析后就是嵌套字典：

```python
{'database': {'host': 'localhost', 'port': 5432, 'username': 'admin'}}
```

### 嵌套表

使用 Python 中类似的 `.` 语法表示嵌套表：

```toml
[server.http]
port = 80
timeout = 30

[server.https]
port = 443
cert = "cert.pem"
```

解析后的数据结构：

```python
{'server': {'http': {'port': 80, 'timeout': 30}, 'https': {'port': 443, 'cert': 'cert.pem'}}}
```

## 在 Python 中使用 TOML

### 安装和导入
从 Python 3.11 开始，标准库中已包含 `tomllib` 模块。对于更早版本，我们需要安装 `tomli`：

```bash
# Python < 3.11
pip install tomli

# Python >= 3.11 无需安装
```

```python
# Python >= 3.11
import tomllib

# Python < 3.11
# import tomli as tomllib
```

### 基本读取操作
```python
# 读取 TOML 文件
with open("config.toml", "rb") as f:
    config = tomllib.load(f)

# 解析 TOML 字符串
toml_str = """
[server]
host = "localhost"
port = 8000
"""
config = tomllib.loads(toml_str)
```

## 实际应用示例

### 1. 项目配置管理
```python
# config.toml
[app]
name = "MyApp"
version = "1.0.0"
debug = true

[database]
host = "localhost"
port = 5432
username = "admin"
password = "secret"
```

```python
from dataclasses import dataclass

@dataclass
class DatabaseConfig:
    host: str
    port: int
    username: str
    password: str

@dataclass
class AppConfig:
    name: str
    version: str
    debug: bool
    database: DatabaseConfig

def load_config(path: str) -> AppConfig:
    with open(path, "rb") as f:
        config_dict = tomllib.load(f)

    return AppConfig(
        name=config_dict["app"]["name"],
        version=config_dict["app"]["version"],
        debug=config_dict["app"]["debug"],
        database=DatabaseConfig(**config_dict["database"])
    )
```

### 2. 配置验证
```python
from typing import TypedDict

class DatabaseConfig(TypedDict):
    host: str
    port: int
    username: str
    password: str

def validate_config(config_dict: dict) -> bool:
    try:
        db_config = DatabaseConfig(**config_dict["database"])

        if not (1024 <= db_config["port"] <= 65535):
            raise ValueError("Invalid port number")

        return True
    except (KeyError, TypeError, ValueError) as e:
        print(f"Configuration error: {e}")
        return False
```

### 3. 环境变量支持
```python
import os

def load_config_with_env():
    with open("config.toml", "rb") as f:
        config = tomllib.load(f)

    if "DATABASE_URL" in os.environ:
        config["database"]["url"] = os.environ["DATABASE_URL"]

    return config
```

### 4. 配置合并
```python
def deep_merge(base: dict, override: dict) -> dict:
    """深度合并两个配置字典"""
    result = base.copy()
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result

# 合并默认配置和本地配置
with open("default_config.toml", "rb") as f:
    default_config = tomllib.load(f)

with open("local_config.toml", "rb") as f:
    local_config = tomllib.load(f)

final_config = deep_merge(default_config, local_config)
```

## TOML 的优势

1. 相比 JSON 更适合人类阅读和编写
2. 相比 YAML 的规范更简单，不依赖空格缩进
3. 相比 INI 文件格式更强大，支持更复杂的数据结构
4. 原生支持日期时间格式
5. 在 Python 生态系统中得到广泛支持

## 最佳实践

1. 使用类型注解增加代码可维护性
2. 实现配置验证确保数据有效性
3. 考虑使用环境变量覆盖机制
4. 为重要配置添加文档注释
5. 使用数据类（dataclass）或类型字典（TypedDict）增强类型安全
6. 保持配置文件结构清晰，适当使用注释
7. 合理使用多行字符串处理长文本

## 注意事项

1. 键名不能重复
2. 表名必须是唯一的
3. 数组中的元素类型要保持一致
4. 读取文件时必须使用二进制模式（"rb"）
5. Python 3.11 之前需要安装额外的库

## Outro

TOML 凭借其简单明了的语法和强大的表达能力，结合 Python 的类型系统和生态支持，为开发者提供了一个理想的配置管理解决方案。无论是小型脚本还是大型项目，TOML 都能满足不同场景的配置需求，帮助开发者构建更加可维护和可靠的应用程序。
