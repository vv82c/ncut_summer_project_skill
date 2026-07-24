# 🎓 NCUT 小学期 C/C++ 课设助手

> 北方工业大学 · 人工智能与计算机学院 · 计算机科学与技术系  
> 贴入项目书 → 分步确认 → 一键生成完整管理系统代码



## 📖 简介

一个 AI Skill / Prompt，专为北方工业大学"学术工程实践 I/II"课程设计而生。

只需**贴入项目书全文**，AI 会自动：

1. 🔍 解析需求，提取实体和数据模型
2. 🏗️ 输出框架分析，让你确认
3. ✅ 交互式勾选扩展功能
4. 🚀 一键生成完整的 C/C++ 管理系统代码
5. 📦 自动配置 VS Code 编译环境（`tasks.json` / `build.bat`）

**10 分钟完成一整个课设项目的代码框架！**

---

## 🛠️ 使用方式

### 你需要的

- 一个支持 **System Prompt / Custom Instruction** 的 AI 编程工具（推荐 [Reasonix](https://reasonix.com)、Cursor、Claude Code 等）
- **g++**（MinGW-w64 或 MSYS2，编译用）
- **VS Code**（可选）

### 安装

将 `SKILL.md` 的内容作为 **System Prompt / Project Rules / Custom Instruction** 注入到你的 AI 工具中。

不同工具的操作方式：

| 工具 | 操作 |
|------|------|
| **Reasonix** | `Reasonix → 设置 → Skills → 导入` 或直接对话中 `/install-skill` |
| **Cursor** | `Settings → Rules → Project Rules` 粘贴 SKILL.md 内容 |
| **Claude Code** | 项目根目录创建 `CLAUDE.md`，粘贴 SKILL.md 内容 |
| **其他** | 对话开始时粘贴 SKILL.md 全文作为 system prompt |

### 启动

在对话中贴入项目书全文即可。AI 会引导你完成：

```
Step 0 → 选择 C 风格 或 C++ 风格
Step 1 → 查看框架分析（结构体/函数清单/数据文件）
Step 2 → 勾选扩展功能（≥10 分）
Step 3 → 最终确认 → 开始生成代码
```

确认后 AI 会逐文件生成完整源码，并自动生成 `build.bat` 和 `.vscode/` 配置。

### 编译运行

```bash
# 双击项目目录中的 build.bat
# 或命令行：
g++ -std=c++11 -O2 -static -finput-charset=UTF-8 -fexec-charset=UTF-8 -o program.exe *.cpp -lstdc++
```

---

## ✨ 特性

| 特性 | 说明 |
|------|------|
| 🔄 **双语言** | 支持 C（`.c`/`printf`/`FILE*`）和 C++（`.cpp`/`cout`/`fstream`） |
| 🔗 **手动链表** | 双向链表 + 头尾哨兵，≥3 个模块使用，禁用 STL |
| 📐 **三层模型** | 自动识别「基础→明细→交易」三层数据模型 |
| 💯 **评分导向** | 按 NCUT 评分标准生成函数清单，目标 100 分 |
| 🔤 **UTF-8 全链路** | 源文件 → 编译 → 控制台统一 UTF-8，告别中文乱码 |
| 🛡️ **防误报** | 编译参数 `-static -O2`，避免 Windows Defender 误删 exe |
| ⚙️ **VS Code 集成** | 自动生成 `.vscode/tasks.json`、`settings.json`、`build.bat` |
| 📝 **代码规范** | 动宾命名、每函数 ≤80 行、`//` 注释、独立模块架构 |

---

## 📁 生成的代码结构

```
YourProject/
├── structs.h              # 业务结构体定义
├── linked_list.h          # 双向链表模板（头尾哨兵）
├── password.h / .cpp      # XOR 加密密码管理
├── file_io.h              # 二进制文件读写模板
├── utils.h / .cpp         # 工具函数集（日期校验、进度条、日志等）
├── main.cpp               # 主菜单循环 + 登录
├── *_manager.cpp          # 各业务模块（增删改查）
├── display_query.cpp      # 显示全部 + 多条件查询
├── statistics_report.cpp  # 统计 + 学年报表
├── extension.cpp          # 扩展功能模块
├── .vscode/
│   ├── tasks.json         # VS Code 编译任务
│   └── settings.json      # 文件编码配置
├── build.bat              # 一键编译脚本
├── data/                  # 二进制数据文件 (.dat)
└── log/                   # 操作日志
```

---

## 🎯 适用项目

基于三层数据模型，适用于绝大多数管理系统课设：

| 项目类型 | 第1层基础 | 第2层明细 | 第3层交易 |
|----------|-----------|-----------|-----------|
| 🏠 公寓管理系统 | 宿舍/学生 | 住宿登记 | — |
| 🏥 门诊管理系统 | 医生/科室 | 排班/号源 | 挂号 |
| 📚 图书管理系统 | 图书/读者 | 库存 | 借阅 |
| 🚗 租车管理系统 | 车辆/客户 | 车辆状态 | 租车/还车 |
| 🛒 商城管理系统 | 商品/会员 | 库存 | 订单 |

> 只有两层或一层的项目会自动降级。

---

## 📋 硬性编码规则

| 规则 | 说明 |
|------|------|
| 注释 | 仅 `//`，禁用 `/* */`，`//` 后不留空格 |
| 链表 | 手动实现双向链表 + 哨兵，禁用 STL |
| 函数 | ≥10 个，每个 ≤80 行，动宾命名（`addXxx`/`modifyXxx`/`queryXxx`） |
| 文件 | 多文件架构，二进制 `fread`/`fwrite` |
| 字符串 | 结构体用 `char[XX]`，不用 `string` |
| 编码 | UTF-8 全链路（源文件 → 编译 → 控制台） |
| 输入 | 限制缓冲区：`scanf("%19s", buf)` 或 `cin >> setw(20) >> buf` |

---

## 🔧 编译参数

```bash
g++ -std=c++11 -O2 -static \
    -finput-charset=UTF-8 \
    -fexec-charset=UTF-8 \
    -o program.exe \
    *.cpp \
    -lstdc++
```

| 参数 | 作用 |
|------|------|
| `-static` | 静态链接，避免依赖 MSYS2 DLL，降低杀软误报 |
| `-O2` | 优化编译，不包含调试符号 |
| `-finput-charset=UTF-8` | 源文件 UTF-8 编码 |
| `-fexec-charset=UTF-8` | 输出字符串 UTF-8 编码 |

---

## ⚠️ 常见问题

<details>
<summary><b>Q: 中文显示乱码？</b></summary>

检查三点：① 源文件保存 UTF-8 ② 编译加 `-fexec-charset=UTF-8` ③ 代码调用 `SetConsoleOutputCP(65001)`。本 Skill 已自动处理。
</details>

<details>
<summary><b>Q: exe 被 Windows Defender 秒删？</b></summary>

用 `-static -O2`（不用 `-g`）。如仍被拦截，将项目目录加入 Defender 排除列表。
</details>

<details>
<summary><b>Q: VS Code 中 Ctrl+Shift+B 只编译当前文件？</b></summary>

Skill 已自动生成 `.vscode/tasks.json`，重新 `Ctrl+Shift+B` 选择 `build project` 即可。
</details>

<details>
<summary><b>Q: IDE 报 `byte` 歧义错误？</b></summary>

`using namespace std;` 与 `<windows.h>` 冲突。在 `<windows.h>` 之前加 `#define _HAS_STD_BYTE 0`（Skill 已处理）。
</details>

---

## 📄 License

MIT © 2025