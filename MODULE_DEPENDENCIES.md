# OPTEE-QEMU 模块依赖关系与数据流

## 概述

本文档详细说明 optee-qemu 仓库中各模块之间的依赖关系、数据流向和通信协议。

---

## 系统分层架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层 (Application Layer)             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐         ┌──────────────────┐              │
│  │ optee_examples  │◄────────┤  optee_test      │ (测试)       │
│  │ (示例应用)      │         │  (测试套件)      │              │
│  └────────┬────────┘         └──────────────────┘              │
│           │                                                     │
│           ▼                                                     │
├─────────────────────────────────────────────────────────────────┤
│                    REE 侧 - 正常世界 (Normal World)           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Linux Kernel (linux/)                                    │  │
│  │  - OP-TEE 驱动                                            │  │
│  │  - 设备树                                                  │  │
│  │  - 网络、文件系统                                         │  │
│  └────────┬────────────────────────────────────────────────┘  │
│           │                                                     │
│           │ SMC 调用 (Secure Monitor Call)                    │
│           │ TEE Ioctl 和内存映射                             │
│           ▼                                                     │
├─────────────────────────────────────────────────────────────────┤
│               安全监视模式 (Secure Monitor Mode - EL3)         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  trusted-firmware-a (BL31)                                │  │
│  │  - 世界切换                                               │  │
│  │  - SMC 处理                                               │  │
│  │  - 异常处理                                               │  │
│  └────────┬────────────────────────────────────────────────┘  │
│           │                                                     │
│           ▼                                                     │
├─────────────────────────────────────────────────────────────────┤
│                    TEE 侧 - 安全世界 (Secure World)           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  optee_os Core Kernel                                     │  │
│  │  - TEE 操作系统                                           │  │
│  │  - 内存管理                                               │  │
│  │  - TEE 系统调用                                           │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │  Trusted Applications / Secure Partitions                 │  │
│  │  - optee_ftpm (TPM TA)                                    │  │
│  │  - optee_examples (示例 TA)                               │  │
│  │  - 自定义 TA                                              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│             启动和初始化层 (Boot & Initialization)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ u-boot   │  │ TF-A     │  │ optee_os │  │ Linux    │        │
│  │ (BL2)    │  │ (BL31)   │  │ (BL32)   │  │ (BL33)   │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│              支持库和基础设施 (Support & Infrastructure)       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  mbedtls, optee_client, buildroot, qemu, 等                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 模块依赖图

### 完整依赖图

```
                    optee_examples
                    optee_test
                          │
                          ▼
                    optee_client
                   /             \
                  ▼               ▼
             libteec         tee-supplicant
                  │               │
                  └───────┬───────┘
                          │
                ┌─────────▼──────────┐
                │  OP-TEE Driver    │
                │  (Linux Kernel)   │
                └────────┬──────────┘
                         │
                    ┌────▼────┐
                    │   SMC   │ ◄─────────── TF-A (BL31)
                    └────┬────┘
                         │
        ┌────────────────▼────────────────┐
        │   optee_os Core              │
        │  ├─ crypto (mbedtls)         │
        │  ├─ drivers                  │
        │  ├─ mm (内存管理)            │
        │  └─ kernel (内核)            │
        └────┬───────────────────────┬─┘
             │                       │
        ┌────▼─────┐      ┌──────────▼──┐
        │ optee_ftpm│      │ optee_rust  │
        │ (TPM TA)  │      │ (Rust 支持) │
        └───────────┘      └─────────────┘
             │
         ┌───▼─────┐
         │ ms-tpm- │
         │ 20-ref  │
         └─────────┘
```

### 按依赖级别分类

**第 0 级（独立，无依赖）**：
- `toolchains/` - 工具链
- `mbedtls/` - 加密库
- `ms-tpm-20-ref/` - TPM 参考实现

**第 1 级（依赖第 0 级）**：
- `qemu/` - 依赖 toolchains
- `u-boot/` - 依赖 toolchains, 可能依赖 mbedtls
- `linux/` - 依赖 toolchains
- `buildroot/` - 依赖 toolchains

**第 2 级（依赖第 0-1 级）**：
- `trusted-firmware-a/` - 依赖 toolchains, 可能依赖 u-boot 产物
- `optee_os/` - 依赖 toolchains, mbedtls

**第 3 级（依赖第 0-2 级）**：
- `optee_client/` - 依赖 optee_os 产物, linux
- `optee_ftpm/` - 依赖 optee_os, ms-tpm-20-ref

**第 4 级（依赖第 0-3 级）**：
- `optee_examples/` - 依赖 optee_os, optee_client
- `optee_test/` - 依赖 optee_os, optee_client
- `optee_rust/` - 依赖 optee_os, optee_client

**第 5 级（集成）**：
- `build/` - 协调所有模块的构建

---

## 关键通信协议

### 1. REE ↔ TEE 通信

#### A. 基于 SMC 的通信

**调用链**:
```
用户应用 (libteec)
    ↓
Linux OP-TEE 驱动
    ↓
SMC 指令 (ARM SMC 指令集)
    ↓
TF-A 安全监视模式 (BL31)
    ↓
optee_os (TEE OS)
```

**数据结构**：
- `struct tee_ioctl_invoke_arg` - 调用参数
- `struct tee_ioctl_param` - 参数传递
- `struct tee_ioctl_session_arg` - 会话参数

#### B. 共享内存

```
┌──────────────┐
│ Linux (REE)  │
│ 应用程序     │
│ malloc()     │ ◄──────┐
└──────┬───────┘        │
       │                │
   teec_allocate_shared │ 注册内存
   _memory()            │
       │                │
       ▼                │
┌──────────────┐        │
│ libteec      │        │
│ 库函数       │        │
└──────┬───────┘        │
       │                │
   ioctl(TEEC_IOC_   ───┤
   SHM_REGISTER)    │
       │                │
       ▼                │
┌──────────────┐        │
│ Linux Driver │        │
│ 建立映射     │        │
└──────┬───────┘        │
       │                │
       ▼                │
┌──────────────────────┐│
│ optee_os             ││
│ 共享内存记录         │◄──┘
└──────────────────────┘
```

#### C. 会话管理

```
序列 1: 打开会话
┌────────────────┐
│ TEEC_Init      │ 初始化 TEE 连接
└────────┬───────┘
         │
┌────────▼───────┐
│ TEEC_OpenSess  │ 打开会话
│ ion()          │
└────────┬───────┘
         │ SMC 调用
         ▼
┌──────────────────────────┐
│ optee_os                 │
│ 创建会话、加载 TA        │
└──────────────────────────┘
         │
         ▼ 返回会话句柄
┌────────────────────────┐
│ 应用程序获得句柄       │
│ 可以继续调用命令       │
└────────────────────────┘

序列 2: 调用命令
┌────────────────┐
│ TEEC_Invoke    │ 调用 TA 中的命令
│ Command()      │
└────────┬───────┘
         │ SMC 调用
         │ 参数、共享内存地址
         ▼
┌──────────────────────────┐
│ optee_os                 │
│ 查找 TA, 执行命令        │
└──────────────────────────┘
         │
         ▼ 返回结果
┌────────────────────────┐
│ 应用程序获得结果       │
└────────────────────────┘
```

---

### 2. 启动序列和初始化流

```
硬件上电
  │
  ▼
┌─────────────────────┐
│ ROM Code            │ (硬件固定)
│ 执行 SPL            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ u-boot SPL (BL1)    │ (出自 u-boot/)
│ ├─ 初始化 DDR      │
│ ├─ 初始化控制器    │
│ └─ 加载 BL2        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ u-boot 主程序 (BL2) │ (出自 u-boot/)
│ ├─ 完整系统初始化  │
│ ├─ 加载设备树      │
│ ├─ 加载 Linux 内核 │
│ ├─ 加载 BL31      │
│ └─ 加载 OP-TEE    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ TF-A BL31           │ (出自 trusted-firmware-a/)
│ ├─ 初始化异常处理  │
│ ├─ 初始化 PSCI    │
│ ├─ 初始化电源管理  │
│ └─ 注册 SMC 处理器 │
└──────────┬──────────┘
           │
    ┌──────┴──────┐
    │ 分支到两个世界
    ▼              ▼
┌──────────┐   ┌──────────────────┐
│ Normal   │   │ Secure           │
│ World    │   │ World            │
└──────┬───┘   └────────┬─────────┘
       │                │
       ▼                ▼
┌──────────────┐  ┌─────────────────┐
│ Linux 内核   │  │ optee_os        │
│ (BL33)       │  │ ├─ 初始化内核  │
│ (出自linux/) │  │ ├─ 初始化 TA  │
│              │  │ └─ 等待 SMC   │
└──────┬───────┘  └────────────────┘
       │
       ▼
┌───────────────────────────┐
│ Linux 用户态              │
│ ├─ init 进程              │
│ ├─ tee-supplicant (daemon)│
│ ├─ 应用程序              │
│ └─ OP-TEE 驱动加载        │
└───────────────────────────┘
```

---

### 3. 内存映射关系

```
ARM 处理器的内存空间
┌──────────────────────────────────┐
│  正常世界 (Normal World)         │ ◄─ REE 地址空间
│  - Linux 内核                    │
│  - 用户应用                      │
│  - 共享内存                      │ ◄─ 由 optee_client 管理
└──────────┬───────────────────────┘
           │
      世界切换 (World Switch)
      由 TF-A 管理
           │
┌──────────▼───────────────────────┐
│  安全世界 (Secure World)         │ ◄─ TEE 地址空间
│  - optee_os 内核                 │
│  - optee_os 堆栈                 │
│  - TA 共享库                     │
│  - optee_ftpm 内存               │
│  - 安全存储                      │
└──────────────────────────────────┘

MMU (内存管理单元) 为每个世界维护独立的页表
```

---

### 4. 系统调用流

#### tee-supplicant 的作用

```
optee_os 内核
    │
    ├─ 需要文件 I/O（安全存储）
    ├─ 需要随机数
    ├─ 需要获取时间
    └─ 需要其他 REE 服务
    │
    ▼ (SYS_* 系统调用)
    │
┌────────────────────────┐
│ 上下文切换到 REE       │
└────────────┬───────────┘
             │
             ▼
┌────────────────────────────┐
│ tee-supplicant 守护进程    │
│ (出自 optee_client/)      │
│                           │
│ 处理系统调用              │
│ ├─ 文件操作               │
│ ├─ 内存操作               │
│ ├─ 设备操作               │
│ └─ 其他服务               │
└────────────┬──────────────┘
             │
             ▼ (返回结果)
┌────────────────────────┐
│ 切换回 TEE             │
│ (SMC 调用完成)         │
└────────────────────────┘
```

---

## 数据流示例

### 示例 1：执行一个简单的加密操作

```
流程：应用程序 → TA → 加密库 → 返回结果

1. 应用程序准备
   ├─ 初始化 TEE 连接（optee_client）
   ├─ 打开 AES TA 会话
   └─ 准备输入数据和缓冲区

2. 调用 TEEC_InvokeCommand()
   ├─ optee_client 库
   ├─ Linux OP-TEE 驱动
   └─ SMC 调用到安全监视

3. TF-A 处理 SMC
   ├─ 验证参数
   ├─ 查找 TA
   └─ 跳转到 TA

4. TA 处行（optee_examples/aes）
   ├─ 调用 libmbedcrypto 函数
   ├─ 执行 AES 加密（来自 optee_os/crypto）
   └─ 结果存储到共享内存

5. 返回
   ├─ TA 返回结果代码
   ├─ TF-A 返回到 REE
   ├─ Linux 驱动处理返回
   └─ optee_client 返回给应用

6. 应用程序获得结果
   ├─ 读取共享内存中的加密数据
   └─ 继续应用逻辑
```

### 示例 2：安全存储读写

```
流程：应用 → TA → tee-supplicant → 文件系统 → tee-supplicant → TA → 应用

1. 应用调用 TA 的存储接口
   │
   ▼
2. TA 使用 tee_fs_* API
   │
   ▼
3. optee_os 检测到存储操作
   │
   ├─ 如果是 REE FS，触发系统调用
   │
   ▼
4. 切换到 REE 执行 SYS_*
   │
   ▼
5. tee-supplicant 处理
   ├─ 打开/读取/写入文件系统
   ├─ 通过 /dev/teeX 与驱动通信
   └─ 返回结果
   │
   ▼
6. 返回到 TA
   ├─ TA 继续执行
   └─ 返回结果到应用
```

---

## 构建依赖关系

### 编译顺序

```
1. toolchains/     # 准备工具链
   └─ 输出：交叉编译器, 库, 头文件

2. mbedtls/        # 编译加密库
   └─ 输出：libmbedcrypto.*

3. optee_os/       # 编译 TEE OS
   ├─ 输入：mbedtls 库
   └─ 输出：tee.bin, TA 开发工具, 头文件

4. 并行编译：
   ├─ u-boot/     # 编译启动加载器
   ├─ linux/      # 编译 Linux 内核
   └─ qemu/       # 编译 QEMU（如果使用）

5. trusted-firmware-a/  # 编译 TF-A BL31
   ├─ 输入：u-boot 产物
   └─ 输出：bl31.bin

6. optee_client/   # 编译客户端库
   ├─ 输入：optee_os 头文件
   └─ 输出：libteec.so, tee-supplicant

7. optee_examples/ 和 optee_test/  # 编译示例和测试
   ├─ 输入：optee_os, optee_client
   └─ 输出：TA 和应用程序

8. buildroot/      # 生成根文件系统
   ├─ 输入：所有应用程序
   └─ 输出：rootfs.cpio.gz

9. 集成和打包      # build/ 进行最终整合
   └─ 输出：完整的系统镜像
```

---

## 文件依赖详表

### optee_os 依赖

```
optee_os/
 ├─ core/crypto/     需要: mbedtls headers
 ├─ core/drivers/    需要: 设备树, HAL
 ├─ core/kernel/     需要: architecture headers
 └─ core/tee/        需要: optee_os 内部接口
```

### optee_client 依赖

```
optee_client/
 ├─ libteec/        需要: optee_os/out/export-user_ta/
 ├─ tee-supplicant/ 需要: optee_os/out/export-user_ta/
 └─ scripts/        需要: Linux kernel headers
```

### Linux 内核依赖

```
linux/drivers/optee/  需要: optee_os headers (TEE 接口定义)
```

### optee_examples 依赖

```
optee_examples/hello_world/
 ├─ ta/             需要: optee_os/out/export-ta_arm64/
 └─ host/           需要: optee_client/out/libteec.so
```

---

## 关键接口定义

### 1. TEE Client API (出自 optee_client)

```c
// 初始化
TEEC_Result TEEC_InitializeContext(...)
TEEC_Result TEEC_FinalizeContext(...)

// 会话管理
TEEC_Result TEEC_OpenSession(...)
TEEC_Result TEEC_CloseSession(...)

// 命令调用
TEEC_result TEEC_InvokeCommand(...)

// 内存管理
TEEC_result TEEC_AllocateSharedMemory(...)
TEEC_result TEEC_RegisterSharedMemory(...)
TEEC_result TEEC_ReleaseSharedMemory(...)

// 操作参数
TEEC_Operation 结构体定义
TEEC_Param 参数数组
```

### 2. TEE 系统调用接口 (出自 optee_os)

```c
// TA 可以调用的接口
TEE_Result TEE_OpenPersistentObject(...)
TEE_Result TEE_ReadObjectData(...)
TEE_Result TEE_WriteObjectData(...)

// 加密操作
TEE_Result TEE_CryptoAlgorithmGetDescription(...)
TEE_Result TEE_DigestDoFinal(...)

// 随机数
TEE_Result TEE_GenerateRandom(...)

// 属性获取
TEE_Result TEE_GetSystemTime(...)
```

---

## 通信协议栈

```
应用层
  ↓
libteec (TEE Client API)
  ↓
Linux OP-TEE 驱动
  ↓
Device Interface (ioctl, mmap)
  ↓
SMC/ERET (ARM 指令集)
  ↓
TF-A SMC Handler
  ↓
optee_os Entry Point
  ↓
TA 执行 / 系统调用处理
  ↓
tee-supplicant (如需 REE 服务)
  ↓
Linux 系统调用 (open, read, write 等)
```

---

## 循环依赖检查

```
✓ 无循环依赖检测到

build/ ──┐
         ├─→ u-boot/ ──┐
         │              ├─→ trusted-firmware-a/
         ├─→ optee_os/ ─┤
         │              ├─→ optee_client/ ──┬─→ optee_examples/
         ├─→ linux/ ────┘                   └─→ optee_test/
         │
         └─→ buildroot/
```

---

## 版本兼容性

| 组件 | 版本 | 兼容性 |
|-----|------|------|
| OP-TEE OS | v3.x 以上 | 兼容 optee_client v3.x |
| optee_client | v3.x 以上 | 兼容 OP-TEE OS v3.x |
| TF-A | v2.x 以上 | 支持 OP-TEE 集成 |
| Linux Kernel | 4.x+ | 兼容 OP-TEE 驱动 |
| mbedtls | 2.x 以上 | OP-TEE 支持 |
| Buildroot | 2018.x+ | 支持 ARM 交叉编译 |

---

## 扩展点

### 添加新的 TA 不会影响

- optee_os 核心
- optee_client
- Linux 驱动
- 其他 TA

### 添加新驱动需要

- 更新 optee_os 子目录 Makefile
- 可能需要更新 Linux 驱动
- 可能需要设备树修改

### 支持新平台需要

- 在 build/ 中添加 platform.mk
- 在 trusted-firmware-a/plat/ 中添加代码
- 在 u-boot/ 中添加支持
- 在 optee_os/core/arch/ 中添加必要代码

---

## 常见问题

**Q: 为什么要同时编译 optee_os 和 optee_client？**

A: optee_client 需要 optee_os 提供的头文件和接口定义。TA 运行在 optee_os 中，而应用程序使用 optee_client 与 TA 通信。

**Q: tee-supplicant 是必须的吗？**

A: 如果 TA 不需要访问文件系统、生成随机数或获取时间，可以不需要。但大多数现实应用都需要。

**Q: 能否单独编译某个模块？**

A: 可以，但需要确保依赖项已经编译。例如，单独编译 optee_examples 需要 optee_os 和 optee_client 的输出。

**Q: 修改 optee_os 后需要重新编译什么？**

A: 需要重新编译 optee_client 和所有 TA（optee_examples, optee_test 等）。

---

## 相关文档

- REPOSITORY_ARCHITECTURE.md - 完整的仓库架构说明
- DIRECTORY_GUIDE.md - 各目录的详细说明

