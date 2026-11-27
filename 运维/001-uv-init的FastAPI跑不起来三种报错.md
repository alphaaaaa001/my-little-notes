# `uv init` 出来的 FastAPI 跑不起来 → 先认报错是哪一种

**症状**

三种报错，三种病，别混着治：

```
# ① 命令压根不存在
fastapi : 无法将“fastapi”项识别为 cmdlet、函数...
CommandNotFoundException

# ② 能跑，但只打印 Hello
uv run fastapi-day1 dev _init_.py
Hello from fastapi-day1!

# ③ 起来了，但找不到 app
Could not find FastAPI app in module, try using --app
```

**为什么**

三件事被同一个名字混在了一起：

1. **框架的 CLI** 在 `fastapi[standard]` 里——只装 `fastapi` 没有这个命令。
2. **`uv run <项目名>`** 跑的是 `pyproject.toml` 里 `[project.scripts]` 注册的那个入口（`uv init` 自动把项目名注册成命令，指向 `pkg:main`）。它只会执行你的 `main()`，不是框架。
3. **`Could not find FastAPI app`** ＝ 指定文件里没有名叫 `app` 的变量。新版 `uv init` 用 **src 布局**，代码很容易还留在 `src/<pkg>/__init__.py`，而 `main.py` 是空的。

**解法**

```powershell
uv add "fastapi[standard]"        # ① 装 CLI
# 把 FastAPI(...) 和路由写进 src/<pkg>/main.py（别写在 __init__.py）
uv run fastapi dev src/<pkg>/main.py
```

起来之后 CLI 会打一行，这就是它的"地址"：

```
Using import string: fastapi_day1.main:app
                     包      模块 变量
```

**记住**

> 报错说"命令不存在"，先问：**这个命令是谁装的？**
> 报错说"找不到 app"，先问：**我让它读的那个文件，真的是我写代码那个文件吗？**

---

*2026-11-27 · 第一次自己跑 FastAPI Day 1*
