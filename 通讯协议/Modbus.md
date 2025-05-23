### 背景与定义

**Modbus 协议** 是一种基于主从结构（Master-Slave）的通信协议，由美国 Modicon 公司于 1979 年为其 PLC 系统开发。该协议设计简单、开放、易于实现，成为工业设备通信的事实标准。

它定义了 **主设备（Master）** 和 **从设备（Slave）** 之间 **数据交换的规则和格式**，实现不同厂商设备之间的互通。涉及了应用层和数据链路层

主设备 **不能接收广播的响应**；每次只与一个从设备通信。只有主设备发起通信（主动），从设备才能响应（被动）

### Modbus 协议的三种模式

| 协议类型     | 物理层传输        | 编码格式     | 应用场景            |
| ------------ | ----------------- | ------------ | ------------------- |
| Modbus RTU   | 串口 (RS-232/485) | 二进制 + CRC | 工业控制现场最常用  |
| Modbus ASCII | 串口              | ASCII + LRC  | 早期系统或调试场合  |
| Modbus TCP   | 以太网            | TCP/IP       | 上位机通信/远程监控 |

在嵌入式系统中最常用的是 **Modbus RTU** 和 **Modbus TCP**。

### 功能码（Function Codes）

功能码定义了主机请求的操作类型。常见如下：

| 功能码 | 功能               | 操作对象             |
| ------ | ------------------ | -------------------- |
| 0x01   | 读线圈（开关输出） | 位                   |
| 0x02   | 读离散输入         | 位                   |
| 0x03   | 读保持寄存器       | 16位寄存器（可读写） |
| 0x04   | 读输入寄存器       | 16位寄存器（只读）   |
| 0x05   | 写单个线圈         | 位                   |
| 0x06   | 写单个保持寄存器   | 寄存器               |
| 0x0F   | 写多个线圈         | 位                   |
| 0x10   | 写多个保持寄存器   | 寄存器               |

### 地址模型

Modbus 协议将设备内部寄存器划分为四类，每类的起始地址逻辑编号不同：

| 寄存器类型         | 起始编号 | 功能码         | 说明                 |
| ------------------ | -------- | -------------- | -------------------- |
| 线圈（Coils）      | 00001    | 0x01           | 读写，1bit 开关输出  |
| 离散输入（Inputs） | 10001    | 0x02           | 只读，1bit 输入      |
| 输入寄存器（IR）   | 30001    | 0x04           | 只读，16bit 模拟量   |
| 保持寄存器（HR）   | 40001    | 0x03/0x06/0x10 | 可读写，16bit 模拟量 |

Modbus 寄存器是通信协议的“抽象接口”，而物理寄存器是 MCU 操作硬件的“控制接口”。

###  定义 Modbus 寄存器表（作为数据映射）

你需要定义如下几个数组，对应 Modbus 四类寄存器：

```c
uint16_t holding_registers[100];   // 保持寄存器 40001~40100
uint16_t input_registers[100];     // 输入寄存器 30001~30100
uint8_t  coils[100];               // 线圈 00001~00100（1位）
uint8_t  discrete_inputs[100];     // 离散输入 10001~10100（1位）
```

这些数组是**Modbus 主从通信的数据载体**。STM32 在通信过程中对这些数据进行读写操作。

### Modbus RTU 模式

**帧格式：[ 地址(1B) ] [ 功能码(1B) ] [ 数据(nB) ] [ CRC校验(2B) ]**

| 字段     | 长度   | 描述                |
| -------- | ------ | ------------------- |
| 从站地址 | 1 字节 | 目标设备地址        |
| 功能码   | 1 字节 | 操作类型（读/写等） |
| 数据     | N 字节 | 操作相关参数        |
| CRC 校验 | 2 字节 | 确保帧完整性        |

**例子（读取寄存器）：**

```c
01 03 00 10 00 02 CRC16
```

- 01：从机地址
- 03：功能码（读保持寄存器）
- 00 10：起始地址
- 00 02：读取数量
- CRC16：校验码

### Modbus ASCII 模式

- 所有内容均使用 ASCII 表示，例如一个字节 `0x3A` 会发送为两个字符：`'3'` 和 `'A'`
- **以冒号 `:` 开始，回车换行 `\r\n` 结束**，易于串口调试观察
- 传输效率较低，但抗干扰性和可读性高

**帧格式：[起始符(0x3A)] [ 地址(2字符) ] [ 功能码(2字符) ] [ 数据(2n字符) ] [ LRC(2字符) ] CR LF**

**例子（读取寄存器）：**

```c
:010300000002FB\r\n
```

- `:`：起始符
- `01`：从机地址
- `03`：功能码（读保持寄存器）
- `0000`：起始寄存器地址
- `0002`：读取两个寄存器
- `FB`：LRC 校验
- `\r\n`：结束符

### Modbus TCP 模式

- 基于标准 **TCP/IP 协议栈**
- 不再使用 CRC/LRC，因 TCP 已内建校验机制
- 支持多个客户端连接（如多个上位机）
- 不需要考虑串口波特率、校验位等

**报文结构：**

```
MBAP Header (7 bytes) + PDU (功能码 + 数据)
```

**MBAP Header（Modbus 应用协议头）：**

| 字段名         | 长度 | 含义                            |
| -------------- | ---- | ------------------------------- |
| Transaction ID | 2B   | 事务编号，主站生成用于匹配响应  |
| Protocol ID    | 2B   | 协议标识，Modbus 固定为 0x0000  |
| Length         | 2B   | 后续字段长度（单位：字节）      |
| Unit ID        | 1B   | 从站地址（通常为 0xFF 或 0x01） |

**PDU（协议数据单元）：**

与 RTU/ASCII 相同，即：**功能码 + 数据**

**例子（读取寄存器）：**

| 字节序列      | 含义                    |
| ------------- | ----------------------- |
| `00 01`       | 事务 ID                 |
| `00 00`       | 协议 ID（Modbus）       |
| `00 06`       | 长度                    |
| `01`          | 单元 ID（从站地址）     |
| `03`          | 功能码                  |
| `00 00 00 02` | 起始地址 0x0000，读两个 |

## Modbus RTU vs ASCII vs TCP：概述对比

| 特性       | Modbus RTU             | Modbus ASCII                 | Modbus TCP                   |
| ---------- | ---------------------- | ---------------------------- | ---------------------------- |
| 传输介质   | 串口（RS-232/RS-485）  | 串口（RS-232/RS-485）        | 网络（TCP/IP）               |
| 编码方式   | 二进制（紧凑）         | ASCII 码（字符形式）         | 基于 TCP/IP 的二进制封包     |
| 通信效率   | ⏫ 高                   | ⏬ 低                         | ⏫ 非常高                     |
| 报文起止符 | 无（通过时间间隔判断） | 有明确起止符（`:` 到`\r\n`） | 使用 TCP 封包管理            |
| 校验方式   | CRC-16                 | LRC                          | TCP 本身保证可靠性，无 CRC   |
| 地址使用   | 设备地址（1~247）      | 同 RTU                       | 单播多用于 Unit ID，非必须   |
| 典型场景   | 工业设备、PLC          | 老旧设备，兼容性好           | 嵌入式联网设备、网关、PC系统 |
