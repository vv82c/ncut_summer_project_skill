---
name: ncpp-summer-project
description: 北方工业大学小学期C/C++课设代码生成助手 — 贴入项目书全文，解析需求，分步生成完整管理系统代码框架
---

# NCUT 小学期 C/C++ 课设助手

接收使用者贴入的项目书全文 → 解析需求 → 先输出框架确认 → 再生成完整代码。

---

## ⚠️ 硬性规则（每次生成代码前重读）

以下规则**贯穿全流程、所有代码文件**，不得违反：

**排版**
- 一行只写一条语句；禁止 `a=1,b=2;` 逗号拼接
- `if`/`for`/`while`/`do`/`switch`/`case`/`default` **独占一行**
- 所有控制结构**必须 `{ }`**，即使只有一行

**注释**
- **仅 `//`**，禁用 `/* */`
- `//`后不留空格：`//这是注释` ✓  `// 这是注释` ✗
- 每个函数前标注 `//功能：` `//参数：` `//返回值：`

**链表（硬性）**
- **禁用**所有标准库容器（含 STL `std::list`）
- 手动实现，推荐双向链表+头尾哨兵（见参考项目）
- 至少 3 个功能模块使用链表

**函数**
- 除 main 外 ≥10 个函数；每个 ≤80 行
- 动宾命名：`addXxx()` `modifyXxx()` `queryXxx()` `generateReport()`
- 严格限制全局变量

**其他**
- 常量全大写：`MAX_RECORD_NUM`；变量完整单词：`doctorCount`
- 数据文件默认**二进制方式**读写：`fwrite`/`fread`（C）或 `fstream::write`/`read`（C++）
- 结构体字符串字段默认 `char[XX]` 而非 `string`
- 采用多文件架构（structs.h / linked_list.h / 各模块独立 .cpp）
- 输入必须限制缓冲区长度：`scanf("%19s", buf)` 或 `cin >> setw(20) >> buf`
- 源文件统一 UTF-8 无 BOM；编译加 `-finput-charset=UTF-8 -fexec-charset=UTF-8`；运行时 `SetConsoleOutputCP(65001)`（C++）或 `system("chcp 65001")`（C）。**不要用 GBK**——VS Code 终端 / Windows Terminal / Win10+ cmd 默认 UTF-8，GBK 必定乱码。

---

## 阶段一：解析 + 框架输出

### Step 0 — 询问语言风格

收到项目书后**首先询问**：

> 请确认代码风格：
> - **A) C 风格** — `.c`，`printf`/`scanf`，`char[]`+`strcpy`/`strcmp`，`FILE*`+`fread`/`fwrite`，`malloc`/`free`
> - **B) C++ 风格** — `.cpp`，`cout`/`cin`，`string`，`fstream`，`new`/`delete`，`R"(...)"`菜单

| 维度 | C 风格 | C++ 风格 |
|------|--------|---------|
| 扩展名/头文件 | `.c` `stdio.h` `stdlib.h` `string.h` | `.cpp` `iostream` `iomanip` `fstream` |
| 输入 | `scanf("%19s", buf)` | `cin >> var` |
| 输出对齐 | `printf("%-10s", buf)` | `cout << setw(10) << var` |
| 字符串 | `char[50]` + `strcpy`/`strcmp` | `char[50]` 或 `string` |
| 文件读写 | `FILE*` + `fread`/`fwrite`（二进制） | `fstream` + `read()`/`write()`（二进制） |
| 内存 | `malloc`/`free` | `new`/`delete` |
| 链表 | `struct ListNode { void* data; struct ListNode* next; }` | 双向链表模板+哨兵（见参考项目 linked_list.h） |
| 编码 | `system("chcp 65001")` + 编译 `-fexec-charset=UTF-8` | `SetConsoleOutputCP(65001)` + 编译 `-fexec-charset=UTF-8` |
| 菜单 | `printf` 逐行 | `cout << R"(...)"` |

### Step 1 — 需求解析

按**三层数据模型**提取实体。大多数项目书符合此模型，若只有两层或一层则**自动降级**（跳过不存在的层）：

- **第1层 基础信息**：「基本信息」「基础信息」→ 编号主键，静态属性
- **第2层 明细/库存/排班**：「入库」「进货」「登记」「排班」→ 含数量/状态，引用第1层编号
- **第3层 交易/流水**：「销售」「挂号」「租用」「退车」→ 含金额/时间，只追加不修改

**属性类型推断**：

| 关键词 | 类型 | 约束 |
|--------|------|------|
| "编号""单号""流水号" | `char[20]` | 主键唯一 |
| "名称""姓名""品牌" | `char[50]` | 非空 |
| "价格""金额""押金""租金" | `float` | >0 |
| "数量""库存""人数""床位""号源" | `int` | >=0，联动增减 |
| "日期""时间" | `char[11]` | YYYY-MM-DD |
| "状态""是否XX" | `int` | 0/1 |
| "备注" | `char[200]` | 可空 |

输出**项目框架摘要**（确认后再生成代码）：

```
╔══════════════════════════════════════════╗
║          【项目框架分析】                 ║
║   项目：XXX管理系统  |  语言：C/C++      ║
╚══════════════════════════════════════════╝

▌数据文件（二进制存储）：
  pwd.dat                    — 密码
  xxx_base.dat               — 第1层基础信息
  xxx_detail.dat             — 第2层明细/库存
  xxx_deal.dat               — 第3层交易流水

▌结构体（共 N 个）：
  struct XxxBase { ... };
  struct XxxDetail { ... };
  struct XxxDeal { ... };

▌函数清单（共 M 个）：
  ⬜ [5分]  setPassword() / changePassword() / verifyPassword()
  ⬜ [10分] addXxx()                     — 录入基础信息
  ⬜ [10分] processXxx()                 — 核心业务（★链表）
  ⬜ [10分] modifyXxx() / deleteXxx()    — 修改与删除（★链表）
  ⬜ [10分] displayAll()                 — 显示全部（★链表）
  ⬜ [5分]  queryXxx()                   — 查询
  ⬜ [5分]  statisticsXxx()             — 统计
  ⬜ [5分]  generateReport()             — 汇总报表
  ⬜ [5分]  exitSystem()                 — 退出
  ⬜ [10分] [扩展功能—待勾选]
  —————————————————————————————————————
  合计：100分  |  ★ = 链表模块（≥3个）

▌核心校验链：
  输入编号 → 查base → 查detail → 写入 → 联动更新
```

### Step 2 — 扩展功能选择

从项目书「扩展功能」章节**原样提取**，不预设。让使用者勾选到 ≥10 分。

### Step 3 — 确认环节

汇总语言风格、项目名称、链表模块、扩展功能，让使用者最终确认。

---

## 阶段二：完整代码生成

### 生成顺序

1. **structs.h** — 业务结构体（不含 ListNode，ListNode 仅在 linked_list.h）
2. **linked_list.h** — 链表模板（含 ListNode + 操作，写法见下方参考项目）
3. **password.h** — 密码管理完整实现
4. **file_io.h** — 文件工具模板（二进制读写封装）
5. **utils.h** — 工具函数（日期校验、输入安全、ID 生成等）
6. **main.cpp** — 主函数 + 菜单循环（写法见下方可迁移模式）
7. **[模块].cpp** — 按函数清单逐个实现（校验链写法见下方可迁移模式）

### 从参考项目提取的可迁移模式

以下模式来自真实完成的 DIY 攒机项目，**适用于任何管理系统**，生成代码时直接套用：

**① 菜单循环模式**
```
main():
  设置编码(SetConsoleOutputCP(65001) 或 chcp 65001)
  → 登录校验(最多3次)
  → do-while循环:
      清屏 → 打印 R"(...)" 主菜单 → 读入选择
      → switch(choice): case 1..N 调用对应模块函数
      → default: 提示无效
  → while(choice != 0)
```
每个模块内部如果有子功能，同样用「清屏→子菜单→switch」嵌套。

**② 校验链模式（所有业务函数通用骨架）**
```
void processXxx():
  1.加载链表(loadFileToLinkedList 全量读入 .dat)
  2.提示输入关键字段(编号/日期等，均做非空+格式校验)
  3.查第1层：编号在 base 链表中是否存在？不存在→提示+return
  4.查第2层：是否重复/状态不符/数量不足？→提示+return
  5.组装记录，链表 push_back + 文件追加(appendToFile)
  6.联动更新：修改关联链表中的数量/状态字段
  7.回写关联文件(overwriteFileFromList)
  8.写日志(writeLog) + 成功提示
```

**③ 链表调用方式**
```cpp
LinkedList<PartBase> stockList;                          //声明
loadFileToLinkedList(FILE, stockList, parsePartBase);    //加载
stockList.push_back(newRecord);                          //添加
PartBase* found = stockList.find_if(predicate);          //查找
stockList.for_each(printFunc);                           //遍历显示
stockList.remove_if(predicate);                          //按条件删
overwriteFileFromList(FILE, stockList, serializeXxx);    //回写
```

**④ 文件 IO 封装方式**
- `loadFileToLinkedList(filename, list, parser)` — ifstream 逐行读 → parser 转结构体 → push_back
- `saveLinkedListToFile(filename, list, serializer)` — ofstream(trunc) 覆写全量
- `appendToFile(filename, item, serializer)` — ofstream(app) 追加单条
- `overwriteFileFromList(filename, list, serializer)` — saveLinkedListToFile 的别名

**⑤ 函数命名与注释密度**
- 模块文件头：`//XXX管理模块(使用动态链表)` + `//功能简述`
- 序列化函数：`parseXxx(line)` / `serializeXxx(item)` — 一一对应结构体
- 业务函数：`addXxx()` `modifyXxx()` `deleteXxx()` `queryXxx()` — 动宾结构
- 每个函数前 `//功能：` `//参数：` `//返回值：` 三行注释
- 校验/联动/回写步骤均加行内注释

---

## 评分自检清单（全部代码输出后）

```
═══════════════════════════════════════════
            【评分项自检】
═══════════════════════════════════════════
[✓] [5分]  密码管理
[✓] [10分] 录入基本信息
[✓] [10分] 核心业务处理
[✓] [10分] 修改与删除
[✓] [10分] 显示全部信息
[✓] [5分]  查询
[✓] [5分]  统计
[✓] [5分]  汇总报表
[✓] [5分]  退出系统
[✓] [5分]  代码规范
[✓] [10分] 动态链表（用于：[模块列表]）
[✓] [10分] 扩展功能：[具体条目]
───────────────────────────────────────────
预计得分：100分
═══════════════════════════════════════════
```

---

## 📌 参考项目解剖（DIY攒机 · C++风格）

> 来源：真实完成项目 | 用于提取**可迁移模式**，不是复制业务逻辑

### 提取什么 / 忽略什么 / 冲突处理

| | 做法 |
|---|---|
| ✅ 提取 | 菜单循环结构、校验链写法、文件读写封装、链表调用方式、函数命名与注释密度、设计决策思路 |
| ❌ 忽略 | 具体的业务逻辑（配件/配置单/销售——攒机特有）、具体的字段名和结构体定义 |
| ⚠️ 冲突 | 存储格式→以 Skill 二进制为准；字段类型→以 Skill `char[]` 为准；文件拆分→以 Skill 多文件为准；**菜单/校验链/链表写法→以本参考为准** |

### 骨架 — 文件树 + 数据流

```
DIY_PC_System/
├── main.cpp              # SetConsoleOutputCP(65001) → 登录(3次) → do-while菜单 → switch(0~10)
├── common.h              # ★单头文件：所有struct/常量/序列化声明/函数声明
├── linked_list.h         # 双向链表模板(头尾哨兵head_/tail_, 仅此一处定义)
├── file_io.h             # 文件I/O模板：load/save/append/overwrite
├── password_manager.cpp  # XOR加密(0x5A) + 首次初始化 + 修改 + 登录校验
├── parts_manager.cpp     # 配件增删改(链表) + parsePartBase/serializePartBase
├── purchase_manager.cpp  # 进货：校验编号→校验流水号→写detail→联动stock+1
├── config_sale.cpp       # 配置单销售：逐配件校验状态→自动计价→联动状态
├── display_query.cpp     # 显示全部 + 多条件查询(表格setw对齐)
├── statistics_report.cpp # 统计+汇总报表(月度进销/门店销售)
├── extension.cpp         # 10个扩展功能(实时时间/价格历史/自动ID/日志/校验)
├── utils.cpp             # 工具：R"(...)"菜单/日期校验(含闰年)/安全输入/ID生成
├── data/*.dat            # 文本管道格式(参考项目用文本，生成新项目用二进制)
└── log/log.txt           # 操作日志(追加写入)
```

**数据流**：启动→全量加载.dat到链表→所有CRUD操作内存链表→每次修改后回写文件

### 关键肉 — 两类代表性代码

**链表模板（双向+哨兵，比单向链表更专业）**：
```cpp
template<typename T> class LinkedList {
    ListNode<T>* head_;  //头哨兵(不存数据)
    ListNode<T>* tail_;  //尾哨兵
    int count_;
public:
    LinkedList() { head_=new ListNode<T>(); tail_=new ListNode<T>();
                   head_->next=tail_; tail_->prev=head_; count_=0; }
    void push_back(const T& item) { /*插在tail_之前，无需判空*/ }
    T* find_if(bool(*pred)(const T&)) { /*从head_->next遍历到tail_*/ }
    void for_each(void(*visit)(const T&)) { /*同上*/ }
    bool remove_if(bool(*pred)(const T&)) { /*哨兵使边界统一*/ }
};
```

**校验链（进货入库，完整5步）**：
```cpp
void addPurchase() {
    //1.加载链表
    LinkedList<PartBase> stockList; loadFileToLinkedList(FILE,stockList,parsePartBase);
    LinkedList<PartDetail> detailList; loadFileToLinkedList(FILE,detailList,parsePartDetail);
    //2.输入+校验(非空/格式/正数)
    //3.查base—编号存在？
    if(!stockList.find_if(matchId)) { cout<<"编号不存在！"; return; }
    //4.查detail—流水号重复？
    if(detailList.find_if(matchSerial)) { cout<<"流水号已存在！"; return; }
    //5.组装→链表push_back→文件追加
    detailList.push_back(record); appendToFile(FILE,record,serializeXxx);
    //6.联动更新stockList中库存+quantity→回写stock文件
    //7.writeLog("进货入库："+id); 成功提示
}
```

### 灵魂 — 设计决策（知道"为什么"才能适配新项目）

| # | 决策 | 为什么 | 新项目建议 |
|---|------|--------|-----------|
| 1 | 文本管道格式 | 可调试——记事本直接打开 .dat 查看数据 | **本 Skill 用二进制**（fread/fwrite 更快，结构体直接读写更简洁） |
| 2 | PartDetail 冗余 PartBase 字段(name/brand/price) | 进货时复制快照——后续 PartBase 修改不影响历史记录 | 所有"流水/明细"都应冗余关联实体的关键字段 |
| 3 | 双向链表+头尾哨兵 | 哨兵消除空链表判断，push_back 永远"插在 tail_之前" | 推荐沿用 |
| 4 | 单头文件 common.h | 无循环依赖，include 只需一行 | **本 Skill 用多文件**（模块独立，教师更认可） |
| 5 | 全量加载+全量回写 | 不做增量 fseek——简单，不会出现"文件与内存不同步"的 bug | 推荐沿用 |
| 6 | 序列化与结构体一一对应 | 新增字段只需改 parse/serialize 两个函数 | 推荐沿用 |

### 可直接迁移的通用零件

以下三个文件**完全不依赖业务**，生成任何项目时直接复用结构：
- `linked_list.h` — 双向链表模板（哨兵设计）
- `file_io.h` — 加载/保存/追加/覆写四合一模板
- `utils.cpp` 中的 `validateDate()`/`readLine()`/`waitForKey()`/`clearScreen()`

---

## 注意事项

- Step 0 不可跳过；Step 2 必须从项目书原文提取扩展功能
- 每个代码文件输出时默念一次⚠️硬性规则
- ListNode 仅在 linked_list.h 定义，不得重复
- 输入必须限制缓冲区长度
- 若项目书不符合三层模型，自动降级
- **生成代码时优先参照「可迁移模式」中的菜单/校验链/链表写法，而非凭空创作**

---

## 🔧 编译与部署（代码生成后必做）

### 必须生成 `build.bat`

每完成一个项目，**自动在项目根目录生成 `build.bat`**，内容模板：

```bat
@echo off
cd /d "%~dp0"
if exist PROGRAM.exe del /q PROGRAM.exe
echo 正在编译(静态链接 + UTF-8)...
g++ -std=c++11 -O2 -static -finput-charset=UTF-8 -fexec-charset=UTF-8 -o PROGRAM.exe [所有.cpp空格分隔] -lstdc++
if %ERRORLEVEL% EQU 0 (echo [成功] 编译完成！) else (echo [失败] 编译出错！)
pause
```

### 必须生成 `.vscode/tasks.json`

VS Code 默认只编译当前活动文件→链接报错。生成此文件后 `Ctrl+Shift+B` 才能正确编译：

```json
{
    "version": "2.0.0",
    "tasks": [{
        "label": "build project",
        "type": "shell",
        "command": "g++",
        "args": ["-std=c++11","-O2","-static","-finput-charset=UTF-8","-fexec-charset=UTF-8","-o","PROGRAM.exe","[所有.cpp]","-lstdc++"],
        "options": {"cwd": "${workspaceFolder}"},
        "group": {"kind":"build","isDefault":true},
        "problemMatcher": ["$gcc"]
    }]
}
```

### 必须生成 `.vscode/settings.json`

```json
{ "files.encoding": "utf8", "files.autoGuessEncoding": false }
```

### 编译参数说明

| 参数 | 作用 | 为什么 |
|------|------|--------|
| `-static` | 静态链接 | 避免依赖 MSYS2 DLL；**大幅降低 Windows Defender 误报** |
| `-O2` | 优化 | 替代 `-g`（调试符号也会触发杀软） |
| `-finput-charset=UTF-8` | 源文件编码 | 源码用 UTF-8 保存 |
| `-fexec-charset=UTF-8` | 输出编码 | exe 字符串存为 UTF-8，与 `SetConsoleOutputCP(65001)` 匹配 |

### 常见问题速查

| 问题 | 原因 | 解决 |
|------|------|------|
| Ctrl+Shift+B 只编译当前文件，链接报错 | VS Code 默认 task 只编译活动文件 | 用 `.vscode/tasks.json` 覆盖 |
| exe 被 Windows Defender 秒删 | 动态链接+调试符号触发启发式扫描 | `-static -O2`，去掉 `-g` |
| 中文乱码 | 源文件/编译/控制台编码不一致 | UTF-8 全链路（见硬性规则） |
| IDE 报 `reference to 'byte' is ambiguous` | `using namespace std;` 与 `<windows.h>` 的 byte 冲突 | 在 `#include <windows.h>` 前加 `#define _HAS_STD_BYTE 0` |
| 多 .cpp 中 static 变量冲突（单翻译单元编译时） | 不同文件用了同名 static 变量 | 给每个文件的 static 变量加文件前缀如 `g_aptRoomNoBuf` |