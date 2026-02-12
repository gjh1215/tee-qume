# OPTEE-QEMU 仓库架构与模块详解

## 概述

optee-qemu 是一个完整的 OP-TEE（Open Portable Trusted Execution Environment）开发和验证环境，集成了安全执行环境（TEE）的各个核心组件。本文档详细说明仓库中各个模块的功能、目录结构和职责。

---

## 目录结构速览

```
optee-qemu/
├── build/                 # 构建系统和平台配置
├── buildroot/             # 根文件系统构建工具
├── hafnium/               # 安全分区管理器（SPM）
├── linux/                 # Linux 内核
├── linux-arm-ffa-user/    # ARM FFA 用户态模块
├── mbedtls/               # 加密库
├── ms-tpm-20-ref/         # 微软 TPM 2.0 参考实现
├── optee_client/          # OP-TEE 客户端库
├── optee_examples/        # OP-TEE 示例应用
├── optee_ftpm/            # 固件 TPM 实现
├── optee_os/              # OP-TEE 操作系统核心
├── optee_rust/            # Rust 支持库
├── optee_test/            # OP-TEE 测试套件
├── SCP-firmware/          # 系统控制处理器固件
├── qemu/                  # QEMU 模拟器
├── toolchains/            # 编译工具链
├── trusted-firmware-a/    # ARM 信任固件
├── trusted-services/      # 可信服务框架
├── u-boot/                # U-Boot 启动加载程序
├── xen/                   # Xen 虚拟化管理程序
├── out/                   # 输出目录
├── out-br/                # BuildRoot 输出目录
└── readme/                # 文档目录
```

---

## 核心模块详解

### 1. **optee_os/** - OP-TEE 操作系统核心
**职责**：TEE 的核心实现，运行在安全世界

**关键子目录**：
- `core/` - 核心代码实现
  - `arch/` - 架构相关代码（ARM 支持等）
  - `crypto/` - 加密算法实现
  - `drivers/` - 驱动程序
  - `kernel/` - 内核代码
  - `lib/` - 通用库函数
  - `mm/` - 内存管理
  - `tee/` - TEE 特定功能
  - `pta/` - 伪 TA（PTA）实现

- `ldelf/` - 动态链接和加载器

- `ta/` - 可信应用（TA）模板

- `lib/` - 库文件目录

- `mk/` - 构建脚本

- `scripts/` - 辅助脚本

**主要功能**：
- TEE 核心操作系统实现
- 安全内存管理和隔离
- TEE 系统调用接口
- 可信应用加载和管理

---

### 2. **optee_client/** - OP-TEE 客户端库
**职责**：REE（Rich Execution Environment，正常世界）的 OP-TEE 接口库

**关键子目录**：
- `libteec/` - TEE 客户端库
- `libckteec/` - CK 接口实现
- `libseteec/` - SETEEC 实现
- `libteeacl/` - TEE 访问控制库
- `tee-supplicant/` - TEE 信号处理守护进程
- `scripts/` - 安装和配置脚本
- `ci/` - 持续集成配置

**主要功能**：
- 正常世界应用程序与 TEE 的通信接口
- TEE 会话和命令管理
- 共享内存管理
- TEE 信号处理和支持功能

---

### 3. **build/** - 构建系统
**职责**：整个项目的构建配置和编排

**关键文件和子目录**：
- `Makefile` - 主构建文件
- `common.mk` - 通用配置
- `toolchain.mk` - 工具链配置
- `*.mk` - 各个平台的构建配置（qemu.mk, hikey.mk 等）
- `qemu_v8/`, `fvp/`, `hikey960/` 等 - 平台特定的构建配置
- `br-ext/` - BuildRoot 扩展
- `kconfigs/` - Kconfig 配置文件

**支持的平台**：
- QEMU（qemu.mk, qemu_v8.mk）
- HiKey, HiKey960（ARM 开发板）
- Juno（ARM 开发板）
- FVP（ARM 固定虚拟平台）
- NanoPc-T6
- RaspberryPi 3（rpi3.mk）
- STM32MP1
- SynQuacer
- iMX 系列（imx.mk）
- Versal 系列（Xilinx FPGA）
- ZynqMP（Xilinx）

**主要功能**：
- 统一的构建入口点
- 多平台支持
- 依赖管理
- 输出文件组织

---

### 4. **hafnium/** - 安全分区管理器（SPM）
**职责**：实现 ARM Firmware Framework for M-profile 规范，管理 TEE 中的多个安全分区

**关键子目录**：
- `src/` - 源代码
- `inc/` - 头文件
- `build/` - 构建输出
- `tools/` - 工具集
- `test/` - 测试代码
- `docs/` - 文档

**主要功能**：
- 实现 ARM Firmware Framework (FF-M) 规范
- 安全分区隔离和管理
- 多个 TA 的并发执行
- 分区间通信（IPC）
- 中断处理
- 内存隔离和保护
- 与 REE 的通信

---

### 5. **trusted-firmware-a/** - ARM 信任固件 (TF-A)
**职责**：安全监视模式（Secure Monitor）和启动代码实现

**关键子目录**：
- `bl1/`, `bl2/`, `bl31/` 等 - 不同启动阶段的代码
- `drivers/` - 驱动程序
- `lib/` - 库函数
- `services/` - 服务实现
- `plat/` - 平台特定代码
- `tools/` - 工具

**主要功能**：
- TEE 和 REE 之间的安全切换
- 安全监视模式实现
- 启动过程管理
- 电源管理
- PSCI（电源状态坐标接口）实现

---

### 6. **linux/** - Linux 内核
**职责**：REE 侧的 Linux 操作系统内核

**目录结构**：
- 标准 Linux 内核结构
- `arch/` - 架构特定代码
- `drivers/` - 设备驱动
- `fs/` - 文件系统
- `kernel/` - 内核核心

**主要功能**：
- REE 操作系统
- 设备驱动支持
- OP-TEE 驱动整合
- 应用程序执行环境

---

### 7. **qemu/** - QEMU 模拟器
**职责**：虚拟化环境

**主要功能**：
- ARM 架构模拟（如 ARMv8 等）
- 完整系统仿真
- 多核处理器支持
- 设备模拟
- GDB 调试支持

---

### 8. **u-boot/** - U-Boot 启动加载程序
**职责**：固件级别的启动代码

**关键目录**：
- `arch/` - 架构支持
- `board/` - 开发板支持
- `drivers/` - 驱动程序
- `cmd/` - 命令实现

**主要功能**：
- 硬件初始化
- 驱动加载
- Linux 内核加载和启动
- 命令行界面

---

### 9. **buildroot/** - 根文件系统构建工具
**职责**：构建最小 Linux 根文件系统

**关键目录**：
- `package/` - 软件包配置
- `board/` - 开发板支持
- `arch/` - 架构配置
- `toolchain/` - 工具链支持
- `configs/` - 默认配置

**主要功能**：
- 交叉编译环境设置
- 根文件系统生成
- 软件包管理
- 启动脚本配置

---

### 10. **optee_test/** - OP-TEE 测试套件
**职责**：OP-TEE 功能验证和测试

**关键子目录**：
- `ta/` - 可信应用测试
- `host/` - 主机测试代码
- `cert/` - 认证相关测试
- `scripts/` - 测试脚本

**主要功能**：
- 功能测试
- 安全性验证
- 性能测试
- 回归测试

---

### 11. **optee_examples/** - OP-TEE 示例应用
**职责**：演示 OP-TEE 功能的示例代码

**示例包括**：
- `hello_world/` - 基础 TA 示例
- `aes/` - AES 加密示例
- `ecdsa/` - ECDSA 签名示例
- `sha/` - SHA 哈希示例
- `secure_storage/` - 安全存储示例
- `random/` - 随机数生成示例
- `ecdh/` - ECDH 密钥交换示例
- `hotp/` - HOTP 一次性密码示例
- `sign_verify/` - 签名验证示例

**主要功能**：
- 加密操作演示
- TA 开发参考
- API 使用示例

---

### 12. **optee_client/ - tee-supplicant**
**职责**：TEE 信号处理守护进程

**功能**：
- 处理 TEE 系统调用
- 访问文件系统代理
- 内存管理代理
- 随机数生成服务

---

### 13. **optee_ftpm/** - 固件 TPM 实现
**职责**：提供 TPM 2.0 功能的可信应用

**关键文件**：
- `fTPM.c` - TPM 实现
- `platform/` - 平台特定代码
- `reference/` - MS 参考实现代码
- `include/` - 头文件

**主要功能**：
- 固件 TPM 2.0 实现
- 基于 MS 参考实现
- 测量启动（Measured Boot）支持
- 密钥存储和管理

---

### 14. **optee_rust/** - Rust 支持库
**职责**：为 OP-TEE 提供 Rust 语言支持

**关键子目录**：
- `optee-teec/` - Rust TEE 客户端库
- `optee-utee/` - Rust TEE 用户态库
- `cargo-optee/` - Cargo 工具支持
- `crates/` - Rust crate 集合
- `examples/` - Rust 示例代码

**主要功能**：
- Rust 编程语言支持
- 安全的内存管理
- FFI 绑定
- Rust TA 开发支持

---

### 15. **mbedtls/** - Mbed TLS 加密库
**职责**：提供加密和 SSL/TLS 功能

**关键目录**：
- `library/` - 库实现
- `include/` - 公共 API
- `programs/` - 工具程序
- `tests/` - 测试用例

**主要功能**：
- AES, RSA, ECDSA 等加密算法
- SSL/TLS 实现
- 哈希函数
- 随机数生成

---

### 16. **ms-tpm-20-ref/** - 微软 TPM 2.0 参考实现
**职责**：提供 TPM 2.0 参考代码

**主要功能**：
- TPM 2.0 核心实现
- 命令处理
- 固件存储
- optee_ftpm 的源基础

---

### 17. **SCP-firmware/** - 系统控制处理器固件
**职责**：管理系统级功能

**关键目录**：
- `module/` - 功能模块
- `framework/` - 框架代码
- `product/` - 产品特定代码
- `arch/` - 架构支持

**主要功能**：
- 电源管理
- 性能监控
- 传感器管理
- 系统级服务

---

### 18. **xen/** - Xen 虚拟化管理程序
**职责**：虚拟化层（可选）

**主要功能**：
- 虚拟机监视程序
- 多个 VM 管理
- 系统资源隔离
- 虚拟设备模拟

---

### 19. **linux-arm-ffa-user/** - ARM FFA 用户态模块
**职责**：在 Linux 用户态实现 ARM FFA 接口

**关键文件**：
- `arm_ffa_user.c` - 用户态 FFA 驱动
- `arm_ffa_user.h` - FFA 接口定义
- `load_module.sh` - 模块加载脚本

**主要功能**：
- 用户态 FFA 驱动
- 应用程序与 SPM 通信
- 分区间通信支持

---

### 20. **trusted-services/** - 可信服务框架
**职责**：提供高级可信服务框架

**关键子目录**：
- `components/` - 可复用组件
- `deployments/` - 部署配置
- `protocols/` - 协议实现
- `platform/` - 平台适配
- `tools/` - 开发工具

**主要功能**：
- 服务化架构
- 模块化设计
- 可复用组件库
- 多种部署模式

---

### 21. **toolchains/** - 编译工具链
**职责**：存储预构建的交叉编译工具

**内容**：
- 交叉编译器（GCC, LLVM 等）
- 链接器和二进制工具
- 库和头文件

**主要功能**：
- 提供统一的编译环境
- 支持多个目标架构

---

### 22. **out/** 和 **out-br/** - 输出目录
**职责**：存储构建输出

- `out/` - OP-TEE 和其他组件的输出
- `out-br/` - BuildRoot 输出

**包含内容**：
- 编译后的二进制文件
- 库文件
- 根文件系统镜像
- 内核镜像

---

### 23. **readme/** - 文档目录
**职责**：项目文档和参考资料

**可能包含**：
- README 文件
- 架构文档
- 用户指南
- API 参考

---

## 系统架构关键概念

### TEE 和 REE 的分离

```
┌─────────────────────────────────────┐
│      Rich Execution Environment     │
│  (REE - 正常世界 / Normal World)    │
│                                     │
│  ┌────────────────────────────────┐ │
│  │      Linux Kernel              │ │
│  │  (Build with OP-TEE driver)    │ │
│  ├────────────────────────────────┤ │
│  │   User Space Applications       │ │
│  │  (optee_client, optee_examples) │ │
│  └────────────────────────────────┘ │
└─────────────────────────────────────┘
           ↑        ↓
    Communication via OP-TEE Driver
           ↑        ↓
┌─────────────────────────────────────┐
│   Trusted Execution Environment     │
│   (TEE - 安全世界 / Secure World)   │
│                                     │
│  ┌────────────────────────────────┐ │
│  │   OP-TEE OS (optee_os)         │ │
│  │   ├─ Core Kernel               │ │
│  │   ├─ Memory Management         │ │
│  │   └─ TEE Syscalls              │ │
│  ├────────────────────────────────┤ │
│  │  Trusted Applications (TA)      │ │
│  │  ├─ optee_ftpm                 │ │
│  │  ├─ optee_examples TAs         │ │
│  │  └─ Custom TAs                 │ │
│  ├────────────────────────────────┤ │
│  │  Security Services             │ │
│  │  ├─ Encryption/Decryption      │ │
│  │  ├─ Secure Storage             │ │
│  │  └─ Random Generation          │ │
│  └────────────────────────────────┘ │
└─────────────────────────────────────┘
           ↑        ↓
    Secure Monitor Mode (TF-A)
           ↑        ↓
┌─────────────────────────────────────┐
│   Secure Monitor / Bootloader       │
│                                     │
│  ├─ trusted-firmware-a (BL31)       │ │
│  └─ u-boot (BL2, SPL)              │ │
└─────────────────────────────────────┘
           ↑        ↓
┌─────────────────────────────────────┐
│      Hardware (ARM Processor)       │
│                                     │
│  ├─ Secure World Exception Level   │ │
│  ├─ Normal World Exception Level    │ │
│  └─ Virtualization Extensions       │ │
└─────────────────────────────────────┘
```

---

## 启动序列

1. **硬件启动** → 执行 ROM 代码
2. **SPL/BL1** (u-boot) → 初始化硬件
3. **BL2** (u-boot) → 加载 BL31 和内核
4. **BL31** (TF-A) → 安全监视模式初始化
5. **OP-TEE Kernel** (optee_os) → TEE 内核启动
6. **Linux Kernel** → REE 启动
7. **应用运行** → User space 应用和 TA 执行

---

## 依赖关系

```
应用层
  ├─ optee_examples (使用 optee_client 和 TA)
  ├─ optee_test (测试 optee_os)
  └─ 用户应用 (使用 optee_client)
        ↓
中间层
  ├─ optee_client (客户端库和 tee-supplicant)
  └─ linux (REE 操作系统)
        ↓
TEE 层
  ├─ optee_os (OP-TEE 操作系统核心)
  ├─ optee_ftpm (TPM 实现)
  ├─ hafnium (SPM - 可选)
  └─ 各种 TA
        ↓
固件/启动层
  ├─ trusted-firmware-a (安全监视)
  ├─ u-boot (启动加载)
  └─ SCP-firmware (系统控制)
        ↓
支持库
  ├─ mbedtls (加密库)
  ├─ ms-tpm-20-ref (TPM 参考)
  └─ optee_rust (Rust 支持)
```

---

## 构建流程

1. **工具链准备** (toolchains/)
2. **BuildRoot 配置** (buildroot/)
3. **U-Boot 编译** (u-boot/)
4. **Linux Kernel 编译** (linux/)
5. **OP-TEE 编译** (optee_os/)
6. **客户端库编译** (optee_client/)
7. **示例和测试编译** (optee_examples/, optee_test/)
8. **根文件系统生成** (buildroot/)
9. **整合和打包** (build/)

---

## 关键功能模块

### 安全性相关
- **mbedtls** - 加密算法
- **optee_ftpm** - TPM 实现
- **optee_os/crypto** - TEE 加密操作

### 存储相关
- **optee_os/core/tee** - 安全存储管理
- **optee_client/libteec** - 客户端存储访问

### 通信相关
- **optee_client/libteec** - TEE-REE 通信
- **linux-arm-ffa-user** - FFA 通信
- **hafnium** - 分区间通信

### 启动和初始化
- **u-boot** - 一级启动加载
- **trusted-firmware-a** - 安全监视和二级启动
- **optee_os** - TEE 初始化

---

## 平台支持矩阵

| 平台 | 配置文件 | 支持情况 |
|------|---------|---------|
| QEMU ARMv8 | qemu_v8.mk | ✓ 完整支持 |
| QEMU 32-bit | qemu.mk | ✓ 支持 |
| HiKey (ARM) | hikey.mk | ✓ 支持 |
| HiKey960 | hikey960.mk | ✓ 支持 |
| ARM Juno | juno.mk | ✓ 支持 |
| ARM FVP | fvp.mk | ✓ 支持 |
| RaspberryPi 3 | rpi3.mk | ✓ 部分支持 |
| STM32MP1 | stm32mp1.mk | ✓ 支持 |
| iMX | imx.mk | ✓ 支持 |
| Xilinx Versal | versal.mk | ✓ 支持 |
| Xilinx ZynqMP | zynqmp.mk | ✓ 支持 |

---

## 常见开发任务

### 编写新的 Trusted Application (TA)
1. 在 `optee_examples/` 中创建新目录
2. 使用 `optee_os/ta/` 中的模板
3. 实现 TA 接口
4. 编译和测试

### 添加新驱动
1. 在 `optee_os/core/drivers/` 中添加驱动代码
2. 更新相关的 sub.mk 文件
3. 测试驱动功能

### 支持新平台
1. 在 `build/` 中添加 `platform.mk` 配置
2. 在 `trusted-firmware-a/plat/` 中添加平台代码
3. 在 `u-boot/` 中添加平台支持
4. 在 `optee_os/core/arch/` 中添加必要的架构代码

### 调试和测试
1. 使用 QEMU 进行仿真
2. 运行 `optee_test/` 中的测试套件
3. 使用 GDB 进行调试

---

## 关键文件参考

| 文件 | 位置 | 用途 |
|------|------|------|
| Makefile | build/ | 主构建文件 |
| qemu.mk | build/ | QEMU 平台配置 |
| common.mk | build/ | 通用配置 |
| sub.mk | optee_os/core/ | OP-TEE 子目录配置 |
| README.md | 各模块/ | 模块说明 |
| Makefile | 各模块/ | 模块构建配置 |

---

## 更多资源

- **官方文档**: http://optee.readthedocs.io
- **GitHub 仓库**: 
  - OP-TEE: https://github.com/OP-TEE
  - Hafnium: https://github.com/TF-Hafnium
  - TF-A: https://github.com/TrustedFirmware-A
  - Xen: https://github.com/xen-project

---

## 版本信息

- **OP-TEE 仓库架构文档**
- **生成日期**: 2026年2月12日
- **适用范围**: optee-qemu 完整开发环境

---

## 许可证

本文档涉及的所有模块遵循其各自的开源许可证（主要为 BSD-3-Clause 和 GPL）。详见各模块的 LICENSE 文件。

