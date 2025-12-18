---
title: '汇编上机考试指北'
publishDate: '2025-12-18'
updatedDate: '2025-12-18'
description: '由于汇编的特性，上机考看这一篇就够了'
tags:
  - 汇编
  - learning
heroImage: { src: './thumbnail.jpg', color: '#4a4fceff' }
language: '中文'
draft: false
---

笔者用六个小时速通了汇编上级考试要点，第二天速通上机题，这里是我和AI一起整理的上级级别的汇编笔记和要点，上机题会这些足够了，汇编考的一般不会太难，都是很简单很基础的题目，一方面现在很多高校计算机专业不再学汇编，另一方面汇编代码无论编程还是调试都很折磨，相关的资料又比较少，上机题考太难有点说不过去。

首先简单介绍一下一个完整的汇编代码的基本结构：

一个完整的汇编代码一般由三段组成：数据段、堆栈段和代码段

而一个`SEGMENT` + `ENDS`就是一个段

```
;数据段
DATA SEGMENT
  INPUT_BUF   DB 100          ; 最大允许输入 100 个字符
              DB ?            ; DOS 会在这里填入实际长度
              DB 100 DUP(?)   ; 存放字符串的空间
DATA ENDS

;堆栈段
STACK SEGMENT
    db 100 dup(?)
STACK ENDS

;代码段
CODE  SEGMENT

;在主程序开始前需要一个ASSUME字段,ES是可选
    ASSUME CS:CODE,DS:DATA,SS:STACK,ES:DATA


;主程序开始
START:
    ;要先对DS进行初始化，如果用到ES，还需要初始化ES
    MOV AX, DATA
    MOV DS, AX

    ;结束
    MOV AH, 4Ch
    INT 21h
CODE ENDS
END  START
;程序结束

```

其中ASSUME字段需要定义这几个寄存器：

DS: 数据段寄存器

SS: 堆栈段寄存器

CS: 代码段寄存器

ES: 附加段寄存器（可选）

下面是一些概念笔记：

一个主label： `START:` + `END START`

一个段：`SEGMENT` + `ENDS`

一个函数：`CALL` + `PRCO` + `RET` + `ENDP`

DB: a byte --12H

DW: a word --1234H

留空：？

首先我建议你先一字一句地过一遍下面的内容，然后找几道例题尝试读懂代码，并且我可以负责任地告诉你**最有价值地内容在本文最后**

---

# 基础概念

### **第一章：寄存器 (Registers)**

核心概念：寄存器是CPU内部的“口袋”或“草稿纸”，是CPU能直接、高速访问的存储单元。**你的一切操作，几乎都必须通过寄存器来中转。** 把它们想象成你的双手，内存（RAM）是你的大仓库，你必须先把东西从仓库拿到手上，才能进行加工。

8086 CPU的寄存器主要分为四类：

**1. 通用寄存器 (General Purpose Registers)**

它们都是16位的，但可以拆成两个8位来用。

- **AX (Accumulator Register - 累加器)**
  - **16位**：`AX`
  - **8位拆分**：`AH` (高8位), `AL` (低8位)
  - **主要用途**：算术运算的默认“主角”。乘法、除法等很多指令默认就使用`AX`或`AL`来存放操作数或结果。它是最重要的寄存器，没有之一。

- **BX (Base Register - 基址寄存器)**
  - **16位**：`BX`
  - **8位拆分**：`BH` (高8位), `BL` (低8位)
  - **主要用途**：**内存寻址的“指针”**。当你需要访问内存中某个地址的数据时，经常会把这个地址先放到`BX`里。语法是 `[BX]`。

- **CX (Count Register - 计数器)**
  - **16位**：`CX`
  - **8位拆分**：`CH` (高8位), `CL` (低8位)
  - **主要用途**：**天生的循环计数器**。`LOOP`指令会自动使用`CX`的值作为循环次数。做循环时，首选`CX`。

- **DX (Data Register - 数据寄存器)**
  - **16位**：`DX`
  - **8位拆分**：`DH` (高8位), `DL` (低8位)
  - **主要用途**：**算术运算的“辅助”和I/O的“信使”**。
    - 在32位乘除法中，`DX`与`AX`组合成`DX:AX`存放结果或被除数。
    - 在与外部设备（如屏幕、键盘）交互时，`DX`常用来存放地址或端口号。**打印字符串时，字符串地址必须在`DX`里。**

**2. 指针和变址寄存器 (Pointer & Index Registers)**

它们主要用于内存寻址和堆栈操作，一般不拆分使用。

- **SP (Stack Pointer - 堆栈指针)**：永远指向**栈顶**。`PUSH`和`POP`指令会自动修改它的值。你一般不需要手动修改它。
- **BP (Base Pointer - 基址指针)**：通常用于在堆栈中寻址，寻找传递给子程序的参数。初学阶段可以暂时把它看作一个备用的`BX`。
- **SI (Source Index - 源变址寄存器)**：在字符串操作中，通常指向**源字符串**。
- **DI (Destination Index - 目的变址寄存器)**：在字符串操作中，通常指向**目的字符串**。

**3. 段寄存器 (Segment Registers)**

**核心概念**：8086的内存地址是“段地址:偏移地址”的形式。段寄存器就负责存放“段地址”。

- **CS (Code Segment - 代码段)**：存放当前正在执行的指令的段地址。你不能直接修改它。
- **DS (Data Segment - 数据段)**：存放程序中定义的数据的段地址。**这是最重要的段寄存器！程序一开始必须手动初始化它，否则你无法访问任何变量。**
- **SS (Stack Segment - 堆栈段)**：存放堆栈的段地址。
- **ES (Extra Segment - 附加段)**：一个备用的数据段寄存器，在字符串操作中常作为目的段。

**4. 标志寄存器 (Flags Register)**

它不是一个单一的寄存器，而是一组记录状态的“开关”（标志位）。你不会直接修改它，但算术和逻辑指令会**自动**改变它的状态。**条件跳转指令会根据它的状态来决定是否跳转。**

**考试必记的几个标志位：**

- **ZF (Zero Flag - 零标志位)**：如果一次运算的**结果为0**，则`ZF=1`；否则`ZF=0`。
  - `JE` (Jump if Equal) 和 `JZ` (Jump if Zero) 指令就是看它。
- **CF (Carry Flag - 进位标志位)**：
  - 在**无符号数**加法中，如果最高位产生**进位**，`CF=1`。
  - 在**无符号数**减法中，如果需要**借位**，`CF=1`。
  - `JA` (Jump if Above) 和 `JB` (Jump if Below) 看它。
- **SF (Sign Flag - 符号标志位)**：如果一次运算的**结果为负数**（最高位为1），则`SF=1`。
  - `JG` (Jump if Greater) 和 `JL` (Jump if Less) 会参考它。
- **OF (Overflow Flag - 溢出标志位)**：在**带符号数**运算中，如果结果**超出了范围**（例如两个正数相加得到一个负数），则`OF=1`。

---

### **第二章：常用指令与注意事项**

**1. 数据传送指令 `MOV`**

- **格式**: `MOV 目的操作数, 源操作数`
- **功能**: 把“源”的值复制给“目的”。
- **注意事项**:
  1.  **`MOV [内存地址1], [内存地址2]` 是绝对非法的！** 不能直接从内存复制到内存。必须通过寄存器中转。
      - 错误: `MOV VAR1, VAR2` (如果两者都是变量)
      - 正确: `MOV AX, VAR2` -> `MOV VAR1, AX`
  2.  **`MOV` 指令不影响任何标志位。**
  3.  操作数的**大小必须匹配**。不能 `MOV AX, BL` (16位 vs 8位)。

**2. 算术运算指令**

- **`ADD` / `SUB` (加 / 减)**
  - 格式: `ADD 目的, 源` (目的 = 目的 + 源)
  - 注意: 都会影响 `ZF`, `CF`, `SF`, `OF` 标志位。
- **`INC` / `DEC` (加一 / 减一)**
  - 格式: `INC 寄存器/内存`
  - 注意: **不影响`CF`进位标志位！** 这是一个常考的细节。
- **`MUL` / `IMUL` (无符号乘法 / 带符号乘法)**
  - 格式: `MUL 源` (8位或16位)
  - **规则 (必记)**:
    - `MUL BL` (8位): `AX = AL * BL`
    - `MUL BX` (16位): `DX:AX = AX * BX` (结果是32位，高16位在DX)
  - `IMUL` 规则相同，但处理的是带符号数。
- **`DIV` / `IDIV` (无符号除法 / 带符号除法)**
  - 格式: `DIV 源` (8位或16位)
  - **规则 (必记)**:
    - `DIV BL` (8位): `AX / BL` -> `AL`=商, `AH`=余数
    - `DIV BX` (16位): `DX:AX / BX` -> `AX`=商, `DX`=余数
  - **注意**: 做16位除法前，如果被除数只有16位在`AX`里，**必须**先用 `CWD` 指令将`AX`的符号位扩展到`DX`，否则结果会出错！

**3. 逻辑与移位指令**

- **`AND`, `OR`, `XOR` (与, 或, 异或)**: 主要用于位操作，比如屏蔽某些位。
  - `AND AX, 0FH` 会只保留`AX`的低4位。
  - `XOR AX, AX` 是最高效的清零指令。
- **`SHL` / `SHR` (逻辑左移 / 右移)**: 用于快速乘以2或除以2。移出的位用0补充。

**4. 比较与跳转指令 (程序流程控制的核心！)**

- **`CMP` (Compare - 比较)**
  - 格式: `CMP 操作数1, 操作数2`
  - 功能: **执行一个“隐形”的减法 (`操作数1 - 操作数2`)**，但**不保存结果**，只根据结果**设置标志位** (`ZF`, `CF`, `SF`, `OF`)。
  - `CMP` 是所有条件跳转的“前戏”。
- **条件跳转指令 (Jumps)**
  - **基于相等/零**:
    - `JE label` / `JZ label`: 如果相等/结果为零 (ZF=1)，则跳转。
    - `JNE label` / `JNZ label`: 如果不相等/结果非零 (ZF=0)，则跳转。
  - **基于无符号数比较 (大于/小于)**:
    - `JA label` (Jump if Above): 如果大于 (CF=0, ZF=0)。
    - `JB label` (Jump if Below): 如果小于 (CF=1)。
    - `JAE label` (Above or Equal): 如果大于等于 (CF=0)。
    - `JBE label` (Below or Equal): 如果小于等于 (CF=1 or ZF=1)。
  - **基于带符号数比较 (大于/小于)**:
    - `JG label` (Jump if Greater): 如果大于。
    - `JL label` (Jump if Less): 如果小于。
    - `JGE label` (Greater or Equal): 如果大于等于。
    - `JLE label` (Less or Equal): 如果小于等于。
- **`JMP` (Unconditional Jump - 无条件跳转)**: `JMP label`，直接跳转。
- **`LOOP` (循环)**: `LOOP label`
  - 自动执行两步：`DEC CX` -> 如果`CX`不为0，则`JMP label`。

**5. 堆栈指令 (Stack Instructions)**

- **`PUSH 源`**: 将一个16位的值压入堆栈。`SP` 的值会减2。
- **`POP 目的`**: 从堆栈顶部弹出一个16位的值。`SP` 的值会加2。
- **规则**: 堆栈是**后进先出 (LIFO)**。`POP` 的顺序必须与 `PUSH` 的顺序**完全相反**！
  - `PUSH AX` -> `PUSH BX` -> `POP BX` -> `POP AX` (正确)
  - `PUSH AX` -> `PUSH BX` -> `POP AX` -> `POP BX` (错误！)

**6. 子程序指令 (Procedures)**

- **`CALL label`**: 调用子程序。它会做两件事：1. 把下一条指令的地址 `PUSH` 到堆栈（以便`RET`能回来）。2. `JMP` 到 `label`。
- **`RET`**: 从子程序返回。它只做一件事：从堆栈 `POP` 出返回地址，然后 `JMP` 到那里。

---

### **第三章：DOS `INT 21h` 功能调用**

**核心概念**: `INT 21h` 是调用DOS系统功能的“万能钥匙”。具体调用哪个功能，由 `AH` 寄存器的值决定。

**考试必背“功能菜单”:**

- **`AH = 01h` - 读取一个字符 (带回显)**
  - **输入**: 无
  - **输出**: `AL` = 用户输入的字符的ASCII码。
  - **注意**: 程序会在这里暂停，等你按下一个键。

- **`AH = 02h` - 显示一个字符**
  - **输入**: `DL` = 要显示的字符的ASCII码。
  - **注意**: 是`DL`，不是`AL`！

- **`AH = 09h` - 显示一个字符串**
  - **输入**: `DS:DX` 指向字符串的**起始地址**。
  - **注意**: 字符串必须以 **`$`** 符号结尾！`$`本身不会被显示。

- **`AH = 0Ah` - 读取一个字符串**
  - **输入**: `DS:DX` 指向一个**特殊格式的缓冲区**。
  - **缓冲区格式**:
    - 第1字节：你定义的最大能容纳的字符数。
    - 第2字节：留空，DOS会填入**用户实际输入的字符数**。
    - 第3字节及以后：留空，DOS会填入用户输入的字符串。
  - **示例**: `BUFFER DB 20, 0, 20 DUP(0)`

- **`AH = 4Ch` - 终止程序**
  - **输入**: `AL` 可以是返回码 (通常设为`0`)。
  - **注意**: 这是**唯一正确**的程序退出方式！你的程序必须以此结尾。

---

### **第四章：考试策略**

1.  **熟记寄存器**：尤其是`AX, BX, CX, DX`的分工和`AH/AL`的拆分。
2.  **背熟 `INT 21h`**：特别是`01h, 02h, 09h, 0Ah, 4Ch`这五个功能的用法，输入/输出寄存器是什么，有什么特殊要求（比如`$`结尾）。
3.  **理解 `CMP` + `Jcc`**: 这是所有逻辑判断的核心。知道何时用无符号比较（`JA/JB`），何时用带符号比较（`JG/JL`）。
4.  **掌握内存访问**: 理解 `MOV AX, [BX]` 和 `MOV AX, BX` 的天壤之别。记住 `OFFSET` 操作符是用来获取变量地址的。
5.  **程序框架**: 记住基本的程序框架结构：段定义 -> `ASSUME` -> `START:` -> 初始化`DS` -> 主逻辑 -> 调用`4Ch`退出。

---

现在你已经基本了解了汇编的指令，但是想要应付上机考试还不够，你需要切实写几个程序感受一下汇编的独特之处。

请你用~~比较靠谱的~~ai实现下面这个程序：

1. 定义X=1234H,读取用户输入的Y=2345H,将相加的结果Z=3579H打印出来

:::caution[请注意！！！]
写完的代码请你重点关注这几个部分：

- 读取输入的时候关注换行符号（13，10），关注进制转换，关注字符串结尾必须有'$'

- 输出的时候关注进制转换，输出方式

- 在程序运行的过程中关注光标的移动
  :::

接下来让我们仔细研究一下输入和输出：

# 输入和输出

### **一、 字符串 (String) 的输入与输出**

**1. 字符串输入 (键盘 -> 内存)**

**工具**: `INT 21h, AH = 0Ah` (带缓冲的键盘输入)

**原理**: 你在内存里准备一个“篮子”，告诉DOS这个篮子有多大，然后DOS会帮你把用户输入的字符（包括最后的回车）装进去，并告诉你实际装了多少个。

**步骤与代码示例**:

**Step 1: 在 `.DATA` 段定义一个特殊格式的缓冲区**

```assembly
.DATA
    MAX_LEN     EQU 80      ; 定义一个常量, 表示最大长度, 方便修改
    INPUT_BUFFER DB MAX_LEN, 0, MAX_LEN DUP(0)
    ;           |        |  |
    ;           |        |  `-- 预留 MAX_LEN 个字节的空间, 用0填充
    ;           |        `---- 第2字节, 留空, DOS会填入用户实际输入的字符数
    ;           `--------- 第1字节, 告诉DOS这个缓冲区最多能装多少字符
```

**Step 2: 在 `.CODE` 段调用 `0Ah` 功能**

```assembly
.CODE
    ; ... 初始化 DS ...

    ; 准备调用
    LEA DX, INPUT_BUFFER    ; 把缓冲区的地址放入 DX
    MOV AH, 0Ah             ; 功能号
    INT 21h                 ; 执行. 程序会在这里暂停, 等待用户输入并按回车

    ; --- 输入完成后的处理 (非常重要！) ---
    ; 此时, 用户输入的内容已经存放在 INPUT_BUFFER 中了
    ; 假设用户输入了 "HELLO"

    ; 获取实际长度
    MOV CL, INPUT_BUFFER[1] ; CL = 5 (H-E-L-L-O 的长度)
    MOV CH, 0               ; 现在 CX = 5

    ; 获取字符串内容的起始地址
    LEA SI, INPUT_BUFFER[2] ; SI 指向 'H'
```

**注意事项**:

- `0Ah` 功能会自动处理退格键等编辑操作。
- 它会把最后那个**回车符 (0Dh)** 也存入缓冲区，在实际字符串的末尾。如果你要进行精确的字符串比较，记得处理掉这个回车符。

**2. 字符串输出 (内存 -> 屏幕)**

**工具**: `INT 21h, AH = 09h` (显示字符串)

**原理**: 你告诉DOS，你的字符串放在内存的哪个位置，DOS就会从那个位置开始一个一个地显示字符，直到遇到一个特殊的结束标志 `$`。

**步骤与代码示例**:

**Step 1: 在 `.DATA` 段定义一个以 `$` 结尾的字符串**

```assembly
.DATA
    WELCOME_MSG DB 'Hello, World!$' ; <-- 结尾必须是美元符号 '$'
    ANOTHER_MSG DB 'This is a test.', 13, 10, '$' ; 可以包含回车换行
```

**Step 2: 在 `.CODE` 段调用 `09h` 功能**

```assembly
.CODE
    ; ... 初始化 DS ...

    LEA DX, WELCOME_MSG     ; 把要显示的字符串的地址放入 DX
    MOV AH, 09h             ; 功能号
    INT 21h                 ; 执行. 屏幕上会显示 "Hello, World!"
```

**注意事项**:

- `$` 符号是**结束标志**，它本身不会被显示。
- 如果你要显示的字符串是刚刚用 `0Ah` 读进来的，你需要手动在它的末尾（覆盖掉那个回车符的位置）加上 `$`。

---

### **二、 单个字符 (Character) 的输入与输出**

这通常用于菜单选择、简单的交互等。

**1. 单个字符输入**

**工具**: `INT 21h, AH = 01h` (读取字符并回显) 或 `AH = 07h/08h` (读取字符不回显)

**原理**: 程序暂停，等待用户按下一个键，然后把这个键的ASCII码返回。

**代码示例**:

```assembly
.CODE
    MOV AH, 01h     ; 功能号. 01h会把你按的键显示在屏幕上, 07h/08h不会
    INT 21h         ; 执行

    ; --- 输入完成 ---
    ; 用户按下的键的ASCII码现在存储在 AL 寄存器中
    CMP AL, 'y'     ; 比如, 比较用户是否按下了 'y' 键
    JE  DO_SOMETHING
```

**2. 单个字符输出**

**工具**: `INT 21h, AH = 02h` (显示单个字符)

**原理**: 你把要显示的字符的ASCII码放入 `DL` 寄存器，DOS就会把它显示出来。

**代码示例**:

```assembly
.CODE
    MOV DL, 'A'     ; 把字符 'A' 的ASCII码放入 DL
    MOV AH, 02h     ; 功能号
    INT 21h         ; 执行. 屏幕上会显示一个 'A'

    ; 打印一个数字 (需要先转换成ASCII)
    MOV DL, 5       ; 这是一个纯数字
    ADD DL, '0'     ; DL = 5 + 30h = 35h, 也就是字符 '5' 的ASCII码
    MOV AH, 02h
    INT 21h         ; 执行. 屏幕上会显示一个 '5'
```

**注意事项**:

- 记住是 `DL` 寄存器，不是 `AL` 或 `DX`！

---

### **三、 数字 (Number) 的输入与输出**

这是最复杂的部分，因为**DOS不能直接读写数字**，它只认识字符。所以，数字的I/O本质上是**字符串与数字之间的转换**。

**1. 数字输入 (字符串 -> 数字)**

**原理**: 先用 `0Ah` 功能把用户输入的数字当作**字符串**读进来 (例如 "123")，然后你自己写算法，把这个字符串转换成一个二进制数 (存入 `AX`)。

**算法：“循环累乘法”**

1.  初始化结果寄存器 `AX = 0`。
2.  从字符串的第一个字符开始循环。
3.  取出一个字符 (例如 `'1'`)。
4.  将字符 `'1'` 减去 `'0'`，得到纯数字 `1`。
5.  将当前结果 `AX` 乘以 `10`。
6.  将新得到的数字 `1` 加到 `AX` 上。
7.  处理下一个字符，重复3-6步。

**代码框架示例**:

```assembly
; 假设用户输入的 "123" 已经用 0Ah 读入 INPUT_BUFFER
; 并且字符串起始地址在 SI, 长度在 CX

    XOR AX, AX              ; AX = 0 (存放最终结果)
    XOR BX, BX              ; BX 临时存放单个数字
    MOV DI, 10              ; DI = 10 (用于乘法)

CONVERT_LOOP:
    MOV BL, [SI]            ; 取出当前字符, e.g., '1'
    SUB BL, '0'             ; 转换成数字, e.g., 1

    MUL DI                  ; AX = AX * 10
    ADD AX, BX              ; AX = AX + 当前数字

    INC SI
    LOOP CONVERT_LOOP       ; CX 控制循环次数

; 循环结束后, AX 中就存放着转换好的二进制数 123
```

**注意**: 这段代码处理的是**无符号数**。处理带符号数需要额外检查第一个字符是否是 `'-'` 负号。

**2. 数字输出 (数字 -> 字符串)**

**原理**: 这是数字输入的逆过程。你自己写算法，把寄存器中的二进制数 (例如 `AX = 123`)，转换成字符串 (`"123"`)，然后用 `09h` (显示字符串) 或 `02h` (逐个显示字符) 打印出来。

**算法：“除10取余法”**

**代码框架示例**:

```assembly
; 假设要打印的数字在 AX 中 (e.g., 123)

    XOR CX, CX              ; CX 作计数器
    MOV BX, 10              ; 除数是 10

DIVIDE_LOOP:
    XOR DX, DX              ; 每次除法前必须清零 DX
    DIV BX                  ; DX:AX / BX -> AX=商, DX=余数

    ADD DL, '0'             ; 将余数转换为ASCII字符
    PUSH DX                 ; 压入堆栈 (顺序会反转)
    INC CX                  ; 计数器加一

    CMP AX, 0               ; 商是否为0?
    JNE DIVIDE_LOOP         ; 不为0则继续

PRINT_LOOP:
    POP DX                  ; 从堆栈中弹出字符 (现在顺序是正确的)
    MOV AH, 02h             ; 用 02h 打印单个字符
    INT 21h
    LOOP PRINT_LOOP         ; CX 控制循环
```

**注意**: 这段代码处理的是**无符号正数**。处理带符号数需要先检查数字是否为负。如果是，先打印一个 `'-'` 负号，然后对原数取反 (`NEG AX`)，再执行上面的正数打印逻辑。

---

在这个过程中，你还需要注意的是根据不同的需求，考虑将字符串输入到一个数组还是一个缓冲区中。这很大程度上会影响你后面的代码是否复杂。

当涉及到不止一个字符串时，就需要用到串处理的相关内容，其中最重要的就是串比较。

# 串比较

---

### **一、 普通比较 (`CMP`) vs. 串比较 (`CMPSB`/`CMPSW`)**

**1. 普通比较指令：`CMP`**

- **核心功能**: 比较**两个单独的**操作数，可以是**寄存器 vs. 寄存器**、**寄存器 vs. 内存**、**内存 vs. 寄存器**，或**寄存器 vs. 立即数**。
- **操作单位**: 一次比较一个字节 (8位)、一个字 (16位) 或一个双字 (32位，在386+ CPU中)。
- **工作原理**: `CMP A, B` 在CPU内部执行 `A - B` 的减法，但**不保存结果**，只根据结果（正、负、零、是否溢出/借位）来**设置标志位** (`ZF`, `CF`, `SF`, `OF`)。
- **指针管理**: **完全手动**。你需要自己用 `INC` 或 `ADD` 来移动指针，以便比较数组或字符串中的下一个元素。
- **循环**: **完全手动**。你需要自己用 `DEC` 和 `JNZ`/`LOOP` 等指令来构建循环。

**`CMP` 的应用场景**:

- 比较单个变量：`CMP AX, 0`
- 检查循环计数器：`CMP CX, 10`
- 在手动实现的循环中，比较数组或字符串中的单个元素：
  ```assembly
  ; 手动比较两个字符串是否相等
  MOV SI, OFFSET STR1
  MOV DI, OFFSET STR2
  MOV CX, LENGTH

      MANUAL_COMPARE_LOOP:
          MOV AL, [SI]
          CMP AL, [DI]        ; <-- 使用 CMP 比较单个字节
          JNE NOT_EQUAL       ; 如果不相等, 马上跳出

          INC SI
          INC DI
          LOOP MANUAL_COMPARE_LOOP

          ; 如果循环正常结束, 说明字符串相等
      EQUAL:
          ; ...
      NOT_EQUAL:
          ; ...
      ```

  **`CMP` 的优点**: 灵活，可以比较任意来源的操作数。
  **`CMP` 的缺点**: 效率低，代码冗长，需要手动管理所有细节。

---

**2. 串比较指令：`CMPSB` / `CMPSW`**

- **核心功能**: **专门用于**比较**内存中**的两个数据块（字符串或数组）。
- **操作单位**: `CMPSB` 一次比较一个**字节** (Byte)。`CMPSW` 一次比较一个**字** (Word)。
- **工作原理**: `CMPSB` 在CPU内部执行 `DS:[SI] - ES:[DI]` 的减法，同样只设置标志位。
- **指针管理**: **完全自动**！执行一次 `CMPSB` 后，CPU 会**自动**根据**方向标志位 (DF)** 来增加或减少 `SI` 和 `DI`。
  - `CLD` 指令 (Clear Direction Flag) -> `DF=0` -> `SI` 和 `DI` **自动递增** (最常用)。
  - `STD` 指令 (Set Direction Flag) -> `DF=1` -> `SI` 和 `DI` **自动递减**。
- **循环**: 通常与**重复前缀 (`REPxx`)** 配合使用，形成一个硬件级别的高效循环。

**`CMPSB` 的应用场景**:

- 高效地比较两个字符串或数组是否相等。
  ```assembly
  ; 使用串比较指令比较两个字符串是否相等
  CLD ; <<<--- 关键步骤1: 确保方向是递增
  LEA SI, STR1
  LEA DI, STR2
  MOV CX, LENGTH
      REPE CMPSB          ; <<<--- 关键步骤2: 一行代码搞定循环比较

      ; --- 检查结果 ---
      JE  EQUAL           ; 如果 REPE 是因为 CX=0 结束的, 说明完全相等, (JE/JZ)
      JNE NOT_EQUAL       ; 如果 REPE 是因为中途不相等而结束的
      ```
  **串指令的优点**: **速度极快，代码极其简洁**。把手动循环的几行代码压缩成了一行。
  **串指令的缺点**: 不够灵活。它只能比较 `DS:[SI]` 和 `ES:[DI]`，操作数来源是固定的。

---

### **二、 串处理时的注意事项**

**1. 段寄存器 `DS` 和 `ES` (致命陷阱)**

- 串指令的操作数永远是 `DS:[SI]` (源) 和 `ES:[DI]` (目的)。
- **注意**: 即使你的源和目的字符串都在同一个数据段，你也**必须**确保 `DS` 和 `ES` 都正确地指向了这个数据段。
- **最佳实践**: 在程序的开头，总是同时初始化 `DS` 和 `ES`。
  ```assembly
  START:
      MOV AX, DATA
      MOV DS, AX
      MOV ES, AX      ; <<<--- 永远不要忘记这一行！
  ```

**2. 方向标志位 `DF` (健壮性关键)**

- `DF` 决定了 `SI` 和 `DI` 是递增还是递减。
- **问题**: DOS 中断 (`INT 21h`) 或其他你调用的子程序，**可能会改变 `DF` 的状态**，而不会通知你。
- **最佳实践**: 在每次使用串操作指令**之前**，都明确地使用 `CLD` (清方向标志，递增模式) 或 `STD` (置方向标志，递减模式) 来设置你期望的方向。不要假设 `DF` 的状态。

  ```assembly
  ; 错误的假设
  LEA SI, SRC
  LEA DI, DST
  MOV CX, LEN
  REP MOVSB       ; <-- 如果此时 DF=1, 你的复制就会从后往前, 可能导致错误

  ; 正确的、健壮的做法
  CLD             ; <<<--- 明确设置方向
  LEA SI, SRC
  LEA DI, DST
  MOV CX, LEN
  REP MOVSB
  ```

**3. 重复前缀 (`REPxx`) 与循环**

`REPxx` 是串指令的“涡轮增压器”。它们会自动使用 `CX` 作为计数器，在硬件层面实现循环。

- **`REP`**: **无条件重复**。只要 `CX > 0` 就一直执行。通常和 `MOVSB` (移动字符串) 或 `STOSB` (存储字符串) 配合。
  - `REP MOVSB`: 复制 `CX` 个字节。
  - `REP STOSB`: 用 `AL` 中的值填充 `CX` 个字节。
- **`REPE` / `REPZ`**: **相等/为零则重复** (Repeat if Equal / Repeat if Zero)。当 `CX > 0` **并且** `ZF=1` (上一次比较相等) 时才继续。通常和 `CMPSB` 配合。
- **`REPNE` / `REPNZ`**: **不相等/不为零则重复** (Repeat if Not Equal / Repeat if Not Zero)。当 `CX > 0` **并且** `ZF=0` (上一次比较不相等) 时才继续。通常和 `SCASB` (扫描字符串) 配合，用于查找某个特定字符。

**4. 跳转条件：如何判断 `REPE CMPSB` 的结果？**

这是上机考试中逻辑判断的**核心**。`REPE CMPSB` 结束后，有两种可能：

1.  **中途因不匹配而终止**:
    - `CX` 的值 > 0。
    - **`ZF` 标志位 = 0** (因为最后一次 `CMPSB` 的结果是不相等)。
    - `SI` 和 `DI` 指向**第一个不匹配**的字符的**下一个位置**。
    - **如何判断**: 使用 `JNE` (Jump if Not Equal) 或 `JNZ` (Jump if Not Zero)。

2.  **因 `CX` 变成 0 而正常结束**:
    - `CX` 的值 = 0。
    - **`ZF` 标志位 = 1** (因为每一次 `CMPSB` 的结果都是相等)。
    - `SI` 和 `DI` 指向两个字符串的末尾之后。
    - **如何判断**: 使用 `JE` (Jump if Equal) 或 `JZ` (Jump if Zero)。

**经典代码模板**:

```assembly
    CLD
    LEA SI, STR1
    LEA DI, STR2
    MOV CX, LENGTH

    REPE CMPSB

    ; 现在检查结果
    JE  STRINGS_ARE_EQUAL   ; 如果跳转, 说明两个字符串完全一样

    ; 如果程序执行到这里, 说明中途不匹配
    JMP STRINGS_ARE_DIFFERENT

STRINGS_ARE_EQUAL:
    ; ... 处理相等的情况 ...
    JMP NEXT_STEP

STRINGS_ARE_DIFFERENT:
    ; ... 处理不相等的情况 ...

NEXT_STEP:
    ; ...
```

**总结**:

- 用 `CMP`，你就是“工人”，需要自己搬砖、计数、判断。
- 用 `CMPSB` + `REPE`，你就是“老板”，只需要下达一个命令，CPU这个“自动化机器人”就会帮你完成所有工作。
- 使用串指令前，永远记住**“三板斧”**：
  1.  **初始化 `DS` 和 `ES`** (在程序开头)。
  2.  **设置方向 `CLD`/`STD`** (在指令前)。
  3.  **设置计数器 `CX`** (在指令前)。
- `REPE CMPSB` 结束后，用 `JE`/`JNE` 来判断最终结果。

---

到现在你又学习了一些知识，又到了实践的环节，请你自己给自己设计一道题目，并尝试独立完成。如果遇到问题和ai交流。（并不是笔者偷懒不想写了）

最后查漏补缺一下

# 查漏补缺

---

下面是学有余力，可以关注的在考试中**可能出现**的知识点。

#### 1. 更深入的算术运算 (有符号数 vs 无符号数)

在一般的题目中主要用 `ADD`/`SUB` 操作地址，但考试可能专门考数值计算。

- **`MUL` vs `IMUL` (乘法)** / **`DIV` vs `IDIV` (除法)**
  - **区别**: 前者处理**无符号数**，后者处理**带符号数 (Integer)**。
  - **陷阱 (必考！)**: `DIV`/`IDIV` 的被除数是 `DX:AX` (32位)。如果你的被除数只有16位在`AX`里：
    - 做无符号除法 `DIV` 前，必须 `XOR DX, DX` (手动将`DX`清零)。
    - 做带符号除法 `IDIV` 前，必须用 `CWD` 指令 (将`AX`的符号位自动扩展到`DX`)。
    - **忘记这一步，结果几乎肯定是错的！**

- **`ADC` / `SBB` (带进/借位的加减法)**
  - **功能**: `ADC AX, BX` 的意思是 `AX = AX + BX + CF` (CF是进位标志位)。`SBB` 同理。
  - **考点**: 当你需要做**超过16位的加减法**时（比如两个32位数相加），必须使用它们。
    ```assembly
    ; 计算 [DX:AX] + [CX:BX] -> 结果存入 [DX:AX]
    ADD AX, BX  ; 先加低16位, 这可能会产生一个进位(CF=1)
    ADC DX, CX  ; 再加高16位, 同时把刚才的进位也加上
    ```

#### 2. 位运算 (Bitwise Operations)

这是汇编的特色，常用于状态判断、数据加密或硬件控制（很小概率会考）。

- **`AND`, `OR`, `XOR`, `NOT`**
  - `AND`: 主要用于“屏蔽”或“清零”某些位。`AND AL, 0FH` 会只保留`AL`的低4位。
  - `XOR`: `XOR AX, AX` 是最高效的清零方法。也用于翻转某些位。
  - `OR`: 主要用于“设置”某些位。`OR AL, 80H` 会将`AL`的最高位置1。
  - `NOT`: 翻转所有位（0变1，1变0）。

- **`TEST`**
  - **功能**: 和 `AND` 一样执行“与”运算，但**不保存结果**，只设置标志位。
  - **考点**: **不破坏操作数**的情况下检查某些位。
    ```assembly
    TEST AL, 01H  ; 检查 AL 的最低位是否为1 (判断奇偶)
    JNZ IS_ODD    ; 如果结果不为0 (ZF=0), 说明最低位是1, 是奇数
    ```

- **`SHL`/`SHR` (逻辑移位)** & **`SAL`/`SAR` (算术移位)**
  - **`SHL`/`SAL`**: 左移，最低位补0。`SHL AX, 1` 等价于 `AX = AX * 2`。
  - **`SHR`**: 逻辑右移，最高位补0。用于**无符号数**的快速除法。`SHR AX, 1` 等价于 `AX = AX / 2`。
  - **`SAR`**: 算术右移，最高位用**原符号位**填充。用于**带符号数**的快速除法。
  - **考点**: 区分 `SHR` 和 `SAR` 的不同应用场景。

#### 3. 堆栈的进阶用法 (参数传递)

用堆栈来保护寄存器了是必须会的，但考试也可能会考用堆栈来传递参数。

- **原理**: 主程序在 `CALL` 子程序之前，先把参数 `PUSH` 进堆栈。子程序内部通过 `BP` 寄存器来访问这些参数。

  ```assembly
  ; 主程序
  MOV AX, 5
  PUSH AX       ; 压入参数
  CALL MY_SUB
  ADD SP, 2     ; 调用结束后要清理堆栈！

  ; 子程序
  MY_SUB PROC
      PUSH BP       ; 保护旧的BP
      MOV BP, SP    ; 让 BP 指向当前的栈顶

      ; 现在可以通过 BP 来访问参数
      ; [BP]   存的是旧的 BP
      ; [BP+2] 存的是返回地址
      ; [BP+4] 存的是我们压入的参数 5
      MOV AX, [BP+4] ; AX = 5

      POP BP        ; 恢复旧的BP
      RET
  MY_SUB ENDP
  ```

  这个稍微复杂，但如果考到，会是拉开差距的题目。

---

# 最后

最后附一些常用的子程序模板，如果你的老师允许携带电子材料，或者有机会使用U盘配置电脑环境（至少我们是这样），这个必不可少。

```
; ============================================
; 子程序：打印换行
; ============================================
PRINT_NEWLINE PROC
    PUSH AX
    PUSH DX

    MOV DL, 0Dh
    MOV AH, 02h
    INT 21h
    MOV DL, 0Ah
    INT 21h

    POP DX
    POP AX
    RET
PRINT_NEWLINE ENDP
; ========================================================================
; 子程序名称: ATOI (ASCII String to Integer)
; 功能: 将字符串转换为 16 位有符号整数
;
; 输入:
;   注意：获取位置要使用LEA而不是MOV
;   DS:SI = 指向字符串的首地址 (例如: "123", "-45", "12a")
;
; 输出:
;   AX = 转换后的整数结果 (例如: 123, -45)
;   SI = 指向停止转换的那个字符 (例如 'a' 或结束符)
;
; 寄存器影响:
;   AX 被修改用于存放结果，其他寄存器 (BX, CX, DX, DI) 自动保存恢复
; ========================================================================
ATOI PROC NEAR
    PUSH BX
    PUSH CX
    PUSH DX
    PUSH DI             ; 用于保存符号标志 (0=正, 1=负)

    XOR BX, BX          ; BX 用于存放最终数值
    XOR CX, CX          ; CX 用于临时存放当前位数字
    XOR DI, DI          ; DI 清零，默认为正数

    ; --- 1. 处理符号位 ---
    MOV AL, [SI]        ; 读取第一个字符
    CMP AL, '-'         ; 是负号吗？
    JNE CHECK_PLUS      ; 不是负号，检查是不是正号
    INC DI              ; 标记为负数 (DI=1)
    INC SI              ; 指针移向下一位
    JMP PROCESS_DIGIT   ; 开始处理数字

CHECK_PLUS:
    CMP AL, '+'         ; 是正号吗？
    JNE PROCESS_DIGIT   ; 也不是，说明直接是数字，开始处理
    INC SI              ; 是正号，跳过，指针移向下一位

    ; --- 2. 循环处理数字位 ---
PROCESS_DIGIT:
    MOV AL, [SI]        ; 读取当前字符

    ; 校验字符是否在 '0'~'9' 之间
    CMP AL, '0'
    JB FINISH_CONVERT   ; 小于 '0' (非数字)，结束
    CMP AL, '9'
    JA FINISH_CONVERT   ; 大于 '9' (非数字)，结束

    SUB AL, '0'         ; ASCII 转 数值 (例如 '3' -> 3)
    MOV CL, AL          ; 暂存在 CL 中 (因为 AX 要用来做乘法)

    ; 核心公式: Result = Result * 10 + Current_Digit
    MOV AX, BX          ; 把当前的 Result 放入 AX
    MOV DX, 10
    MUL DX              ; AX = AX * 10 (结果在 DX:AX，假设不溢出只看 AX)

    ADD AX, CX          ; AX = AX + 当前位
    MOV BX, AX          ; 结果存回 BX

    INC SI              ; 移动到下一个字符
    JMP PROCESS_DIGIT   ; 继续循环

    ; --- 3. 结束处理与符号调整 ---
FINISH_CONVERT:
    MOV AX, BX          ; 将结果放入 AX 准备返回

    CMP DI, 1           ; 检查刚才是不是负数
    JNE ATOI_EXIT       ; 如果不是，直接退出
    NEG AX              ; 如果是负数，取补码 (变为负值)

ATOI_EXIT:
    POP DI
    POP DX
    POP CX
    POP BX
    RET
ATOI ENDP

; ============================================
; 有符号数打印子程序
; 输入：AX (16位有符号整数)

;如果需要对8位符号数扩展到16符号数，使用 CBW (Convert Byte to Word)。
;作用：它检查 AL 的最高位（符号位）。如果 AL 是负的，它自动把 AH 全填成 1（FF）；如果 AL 是正的，它自动把 AH 清零（00）。
;结果：AX 就变成了数值相等的 16 位数。

; ============================================

PRINT_SIGNED_16 PROC NEAR
    PUSH AX
    PUSH DX

    ; --- 1. 判断正负 ---
    CMP AX, 0           ; 和 0 比较
    JGE P_POSITIVE      ; 如果 >= 0，直接打印

    ; --- 2. 如果是负数 ---
    PUSH AX             ; 保护 AX
    MOV DL, '-'         ; 准备打印负号
    MOV AH, 02h
    INT 21h             ; 输出 '-'
    POP AX              ; 恢复 AX

    NEG AX              ; 【关键】取反，把 -5 变成 5，方便后面除法

P_POSITIVE:
    CALL PRINT_NUM      ; 调用基础的数字打印函数(打印绝对值)

    POP DX
    POP AX
    RET
PRINT_SIGNED_16 ENDP

; ============================================
; PRINT_NUM: 打印 AX 中的非负整数（最多5位或更多，视AX大小）
;将寄存器 AX 中的数值（无符号数），转换成对应的十进制字符（ASCII码），并输出到屏幕上。
; ============================================
PRINT_NUM PROC
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX

    XOR CX, CX        ; 位计数
    MOV BX, 10        ; 除数 10

    ; 处理 AX==0 的特殊情况
    CMP AX, 0
    JNE PN_DIV_LOOP
    ; 如果 AX == 0, 直接打印 '0'
    MOV DL, '0'
    MOV AH, 02h
    INT 21h
    JMP PN_DONE

PN_DIV_LOOP:
    ; 循环：用 DX:AX / BX (16-bit 除法)
    ; 每次将余数(DX)压栈，直到 AX==0（商为0）
DIV_LOOP_PN:
    XOR DX, DX        ; 清除高位，准备 16-bit 除法
    DIV BX            ; AX / BX -> AX=quotient, DX=remainder
    PUSH DX           ; 保存 remainder（0..9）
    INC CX
    CMP AX, 0
    JNE DIV_LOOP_PN

    ; 弹出并打印余数（高位先被压入，故弹出时顺序正确）
PRINT_LOOP_PN:
    POP DX
    ADD DL, '0'
    MOV AH, 02h
    INT 21h
    LOOP PRINT_LOOP_PN

PN_DONE:
    POP DX
    POP CX
    POP BX
    POP AX
    RET
PRINT_NUM ENDP
; ============================================
; 子程序：打印换行
; ============================================
PRINT_NEWLINE PROC
    PUSH AX
    PUSH DX

    MOV DL, 0Dh
    MOV AH, 02h
    INT 21h
    MOV DL, 0Ah
    INT 21h

    POP DX
    POP AX
    RET
PRINT_NEWLINE ENDP
========================================================================
; 子程序名称: ATOI (ASCII String to Integer)
; 功能: 将字符串转换为 16 位有符号整数，这里的ATOI转化为的符号数是补码
;
; 输入:
;   注意：获取位置要使用LEA而不是MOV
;   DS:SI = 指向字符串的首地址 (例如: "123", "-45", "12a")
;
; 输出:
;   AX = 转换后的整数结果 (例如: 123, -45)
;   SI = 指向停止转换的那个字符 (例如 'a' 或结束符)
;
; 寄存器影响:
;   AX 被修改用于存放结果，其他寄存器 (BX, CX, DX, DI) 自动保存恢复
; ========================================================================
ATOI PROC NEAR
    PUSH BX
    PUSH CX
    PUSH DX
    PUSH DI             ; 用于保存符号标志 (0=正, 1=负)

    XOR BX, BX          ; BX 用于存放最终数值
    XOR CX, CX          ; CX 用于临时存放当前位数字
    XOR DI, DI          ; DI 清零，默认为正数

    ; --- 1. 处理符号位 ---
    MOV AL, [SI]        ; 读取第一个字符
    CMP AL, '-'         ; 是负号吗？
    JNE CHECK_PLUS      ; 不是负号，检查是不是正号
    INC DI              ; 标记为负数 (DI=1)
    INC SI              ; 指针移向下一位
    JMP PROCESS_DIGIT   ; 开始处理数字

CHECK_PLUS:
    CMP AL, '+'         ; 是正号吗？
    JNE PROCESS_DIGIT   ; 也不是，说明直接是数字，开始处理
    INC SI              ; 是正号，跳过，指针移向下一位

    ; --- 2. 循环处理数字位 ---
PROCESS_DIGIT:
    MOV AL, [SI]        ; 读取当前字符

    ; 校验字符是否在 '0'~'9' 之间
    CMP AL, '0'
    JB FINISH_CONVERT   ; 小于 '0' (非数字)，结束
    CMP AL, '9'
    JA FINISH_CONVERT   ; 大于 '9' (非数字)，结束

    SUB AL, '0'         ; ASCII 转 数值 (例如 '3' -> 3)
    MOV CL, AL          ; 暂存在 CL 中 (因为 AX 要用来做乘法)

    ; 核心公式: Result = Result * 10 + Current_Digit
    MOV AX, BX          ; 把当前的 Result 放入 AX
    MOV DX, 10
    MUL DX              ; AX = AX * 10 (结果在 DX:AX，假设不溢出只看 AX)

    ADD AX, CX          ; AX = AX + 当前位
    MOV BX, AX          ; 结果存回 BX

    INC SI              ; 移动到下一个字符
    JMP PROCESS_DIGIT   ; 继续循环

    ; --- 3. 结束处理与符号调整 ---
FINISH_CONVERT:
    MOV AX, BX          ; 将结果放入 AX 准备返回

    CMP DI, 1           ; 检查刚才是不是负数
    JNE ATOI_EXIT       ; 如果不是，直接退出
    NEG AX              ; 如果是负数，取补码 (变为负值)

ATOI_EXIT:
    POP DI
    POP DX
    POP CX
    POP BX
    RET
ATOI ENDP
; =====================================
; 标准输入转十进制
; 假设 SI 指向缓冲区首地址
; BX 用于存放最终的十进制数值结果
; ====================================
MOV BX, 0       ; 结果清零
MOV CX, 10      ; 准备乘数 10 (0Ah)

NEXT_DIGIT:
    MOV AL, [SI]    ; 取一个字符
    INC SI          ; 指针后移

    ; --- 1. 校验是否结束 ---
    CMP AL, '0'
    JB FINISH       ; 小于 '0' 不是数字，结束
    CMP AL, '9'
    JA FINISH       ; 大于 '9' 不是数字，结束

    ; --- 2. 核心算法：转换 ---
    SUB AL, 30H     ; 【修正点】减去 30H，而不是 40H
    MOV AH, 0       ; 为了不干扰后面加法，把高位清零，现在 AX 就是那个个位数

    ; --- 3. 核心算法：位权 (Result * 10) ---
    PUSH AX         ; 暂时保存当前的个位数

    MOV AX, BX      ; 把之前的总结果拿来
    MUL CX          ; AX = AX * 10 (注意：结果可能溢出到 DX，这里假设数字不大)
    MOV BX, AX      ; 乘完的结果放回 BX

    POP AX          ; 取回刚才那个个位数

    ; --- 4. 核心算法：累加 (+ Current) ---
    ADD BX, AX      ; result = result * 10 + digit

    JMP NEXT_DIGIT  ; 继续处理下一个字符

FINISH:
    ; 此时 BX 中存储的就是 10 进制数值，可以用于 CMP 比较了

; ============================================
; 有符号数打印子程序
; 输入：AX (16位有符号整数)

;如果需要对8位符号数扩展到16符号数，使用 CBW (Convert Byte to Word)。
;作用：它检查 AL 的最高位（符号位）。如果 AL 是负的，它自动把 AH 全填成 1（FF）；如果 AL 是正的，它自动把 AH 清零（00）。
;结果：AX 就变成了数值相等的 16 位数。

; ============================================

PRINT_SIGNED_16 PROC NEAR
    PUSH AX
    PUSH DX

    ; --- 1. 判断正负 ---
    CMP AX, 0           ; 和 0 比较
    JGE P_POSITIVE      ; 如果 >= 0，直接打印

    ; --- 2. 如果是负数 ---
    PUSH AX             ; 保护 AX
    MOV DL, '-'         ; 准备打印负号
    MOV AH, 02h
    INT 21h             ; 输出 '-'
    POP AX              ; 恢复 AX

    NEG AX              ; 【关键】取反，把 -5 变成 5，方便后面除法

P_POSITIVE:
    CALL PRINT_NUM      ; 调用基础的数字打印函数(打印绝对值)

    POP DX
    POP AX
    RET
PRINT_SIGNED_16 ENDP

; ============================================
; PRINT_NUM: 打印 AX 中的非负整数（最多5位或更多，视AX大小）
;将寄存器 AX 中的数值（无符号数），转换成对应的十进制字符（ASCII码），并输出到屏幕上。
; ============================================
PRINT_NUM PROC
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX

    XOR CX, CX        ; 位计数
    MOV BX, 10        ; 除数 10

    ; 处理 AX==0 的特殊情况
    CMP AX, 0
    JNE PN_DIV_LOOP
    ; 如果 AX == 0, 直接打印 '0'
    MOV DL, '0'
    MOV AH, 02h
    INT 21h
    JMP PN_DONE

PN_DIV_LOOP:
    ; 循环：用 DX:AX / BX (16-bit 除法)
    ; 每次将余数(DX)压栈，直到 AX==0（商为0）
DIV_LOOP_PN:
    XOR DX, DX        ; 清除高位，准备 16-bit 除法
    DIV BX            ; AX / BX -> AX=quotient, DX=remainder
    PUSH DX           ; 保存 remainder（0..9）
    INC CX
    CMP AX, 0
    JNE DIV_LOOP_PN

    ; 弹出并打印余数（高位先被压入，故弹出时顺序正确）
PRINT_LOOP_PN:
    POP DX
    ADD DL, '0'
    MOV AH, 02h
    INT 21h
    LOOP PRINT_LOOP_PN

PN_DONE:
    POP DX
    POP CX
    POP BX
    POP AX
    RET
PRINT_NUM ENDP

=========================================================================
; 子程序: HEXSTR_TO_NUM
; 功能:   将一个十六进制字符串转换为一个16位二进制数
; 输入:   DS:SI - 指向字符串的开头
;         CX    - 字符串的长度
; 输出:   BX    - 转换后的数字
; =========================================================================
HEXSTR_TO_NUM PROC
    PUSH AX
    PUSH CX
    PUSH SI

    XOR BX, BX              ; 结果清零 (BX = 0)
CONVERT_LOOP:
    CMP CX, 0               ; 检查是否所有字符都已处理
    JE CONVERT_DONE         ; 如果是, 则完成

    MOV AL, [SI]            ; 取一个字符

    ; --- 核心转换逻辑: 将 '0'-'9', 'A'-'F', 'a'-'f' 转换为数字 0-15 ---
    CMP AL, '9'
    JBE IS_DIGIT
    CMP AL, 'F'
    JBE IS_UPPER_HEX
    CMP AL, 'f'
    JBE IS_LOWER_HEX
    JMP INVALID_CHAR        ; 如果不是有效十六进制字符, 跳过

IS_DIGIT:
    SUB AL, '0'             ; '0'..'9' -> 0..9
    JMP ACCUMULATE

IS_UPPER_HEX:
    SUB AL, 'A'             ; 'A'..'F' -> 0..5
    ADD AL, 10              ; 0..5 -> 10..15
    JMP ACCUMULATE

IS_LOWER_HEX:
    SUB AL, 'a'             ; 'a'..'f' -> 0..5
    ADD AL, 10              ; 0..5 -> 10..15

ACCUMULATE:
    ; --- 核心累加算法: result = result * 16 + digit ---
    PUSH AX                 ; 保存当前转换出的数字 (0-15)

    MOV AX, BX              ; 把之前的总结果拿来
    MOV DX, 16              ; 乘以 16
    MUL DX                  ; AX = AX * 16
    MOV BX, AX              ; 更新总结果

    POP AX                  ; 取回刚才的数字
    MOV AH, 0               ; 清零高位, 因为 AL 中是 0-15
    ADD BX, AX              ; 加上当前位的值

INVALID_CHAR:
    INC SI
    DEC CX
    JMP CONVERT_LOOP

CONVERT_DONE:
    POP SI
    POP CX
    POP AX
    RET
HEXSTR_TO_NUM ENDP

=========================================================================
; 子程序: PRINT_HEX_NUM (你之前的 BTOH 子程序)
; 功能:   将 AX 中的16位数作为4位十六进制数直接打印到屏幕
; 输入:   AX - 要转换的数字
; 输出:   无 (直接在屏幕显示结果)
; =========================================================================
PRINT_HEX_NUM PROC
    PUSH AX
    PUSH CX
    PUSH DX

    MOV BX, AX              ; 把它移到 BX, 因为 ROL BX, CL 更方便
    MOV CX, 4               ; 16/4, 处理四次
ROTATE:
    MOV CL, 4               ; 每次旋转4位
    ROL BX, CL              ; 循环左移四位
    MOV AL, BL              ; 取低8位
    AND AL, 0Fh             ; 只取低4位
    ADD AL, '0'             ; 转换为数字字符
    CMP AL, '9'
    JBE  PRINT_CHAR
    ADD AL, 7               ; 修正为字母
PRINT_CHAR:
    MOV DL, AL
    MOV AH, 02h
    INT 21H

    DEC CH
    JNZ ROTATE

    POP DX
    POP CX
    POP AX
    RET
PRINT_HEX_NUM ENDP

==========================================================
; 子程序：TO_UPPER
; 功能：将 AL 中的字符转为大写
; 原理：AND AL, 11011111B (0DFH) -> 将第6位强制置0
; 注意：会自动忽略非小写字母
; ==========================================================
TO_UPPER PROC NEAR
    CMP AL, 'a'         ; 小于 'a' 不是小写字母
    JB  UPPER_EXIT
    CMP AL, 'z'         ; 大于 'z' 不是小写字母
    JA  UPPER_EXIT

    ; --- 核心指令 ---
    AND AL, 11011111B   ; 也就是 AND AL, 0DFH
    ; ----------------

UPPER_EXIT:
    RET
TO_UPPER ENDP

; ==========================================================
; 子程序：TO_LOWER
; 功能：将 AL 中的字符转为小写
; 原理：OR AL, 00100000B (20H) -> 将第6位强制置1
; 注意：会自动忽略非大写字母
; ==========================================================
TO_LOWER PROC NEAR
    CMP AL, 'A'         ; 小于 'A' 不是大写字母
    JB  LOWER_EXIT
    CMP AL, 'Z'         ; 大于 'Z' 不是大写字母
    JA  LOWER_EXIT

    ; --- 核心指令 ---
    OR  AL, 00100000B   ; 也就是 OR AL, 20H
    ; ----------------

LOWER_EXIT:
    RET
TO_LOWER ENDP

```

----
<p class="signature">— 屿知</p>