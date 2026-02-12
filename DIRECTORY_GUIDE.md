# OPTEE-QEMU 目录用途详细指南

## 快速导航

本文档按字母顺序和逻辑层次详细说明 optee-qemu 仓库中每个主要目录的用途。

---

## 按层次分类的目录

### 第一层：启动和固件层

#### 1. **build/** - 编译构建系统
**完整路径**: `/optee-qemu/build/`

**用途**:
- 整个项目的统一构建入口
- 跨平台编译配置
- 依赖管理和编排

**关键文件**:
- `Makefile` - 主构建文件入口
- `common.mk` - 所有平台通用配置
- `toolchain.mk` - 工具链配置
- `*.mk` - 各平台特定配置（qemu.mk, hikey.mk 等）
- `trusted-services.mk` - 可信服务配置
- `rust.exp` - Rust 支持配置

**子目录说明**:

| 子目录 | 用途 |
|-------|------|
| `br-ext/` | BuildRoot 扩展配置 |
| `qemu_v8/` | QEMU ARMv8 平台支持 |
| `fvp/` | ARM FVP 平台配置 |
| `hikey960/` | HiKey960 开发板配置 |
| `rpi3/` | 树莓派3 配置 |
| `stm32mp/` | STM32MP1 配置 |
| `ti/` | TI 芯片配置 |
| `imx/` | i.MX 系列芯片配置 |
| `zynqmp/` | Xilinx ZynqMP 配置 |
| `versal/` | Xilinx Versal 配置 |
| `kconfigs/` | Kconfig 配置文件 |
| `nanopc-t6/` | NanoPc-T6 配置 |

**工作流程**:
```
$ cd /optee-qemu
$ make -f build/Makefile PLATFORM=qemu_v8
```

**典型修改**:
- 添加新平台支持
- 调整编译标志
- 配置工具链参数

---

#### 2. **u-boot/** - 启动加载程序
**完整路径**: `/optee-qemu/u-boot/`

**用途**:
- 设备启动阶段一和二的代码
- 硬件初始化
- 内核加载

**关键目录**:

| 目录 | 功能 |
|-----|------|
| `arch/` | 架构特定代码（ARM, ARM64 等）|
| `board/` | 开发板特定代码 |
| `drivers/` | 各类硬件驱动（网络、存储、USB 等）|
| `common/` | 通用代码 |
| `cmd/` | U-Boot 命令实现 |
| `lib/` | 库函数 |
| `tools/` | 编译工具 |
| `include/` | 头文件 |

**输出文件**:
- `u-boot.bin` - 二进制启动程序
- `u-boot` - ELF 格式可执行文件
- System.map - 符号表

**核心功能**:
- 初始化 CPU, MMU, 缓存
- 初始化内存
- 加载并运行 Linux 内核
- 提供命令行交互接口

---

#### 3. **trusted-firmware-a/** - ARM 信任固件
**完整路径**: `/optee-qemu/trusted-firmware-a/`

**用途**:
- 安全监视模式（EL3）代码
- TEE 和 REE 之间的安全切换
- 电源管理
- 启动流程二级加载

**关键文件和目录**:

| 文件/目录 | 用途 |
|----------|------|
| `bl1/` | 一级启动加载程序代码 |
| `bl2/` | 二级启动加载程序代码 |
| `bl31/` | 运行时固件（Secure Monitor）|
| `bl32/` | 安全操作系统（OP-TEE）|
| `plat/` | 平台特定代码 |
| `drivers/` | 驱动程序 |
| `lib/` | 库函数 |
| `services/` | PSCI 等服务 |

**重要概念**:
- **BL1** - ROM 加载程序
- **BL2** - 平台初始化
- **BL31** - 监视模式固件
- **BL32** - OP-TEE OS

**主要功能**:
- SMCCC (SMC Calling Convention) 实现
- 世界切换（World Switch）
- 异常处理
- 电源管理（PSCI）

---

#### 4. **SCP-firmware/** - 系统控制处理器固件
**完整路径**: `/optee-qemu/SCP-firmware/`

**用途**:
- 系统级管理（可选）
- 电源管理
- 传感器管理
- 性能扩展

**关键目录**:

| 目录 | 用途 |
|-----|------|
| `module/` | 功能模块 |
| `framework/` | 框架代码 |
| `product/` | 产品配置 |
| `arch/` | 架构支持 |

---

### 第二层：操作系统层

#### 5. **optee_os/** - OP-TEE 操作系统核心
**完整路径**: `/optee-qemu/optee_os/`

**用途**:
- TEE 侧的操作系统内核
- 可信应用的执行环境
- 安全资源管理

**核心子目录**:

```
optee_os/
├── core/                    # 核心实现
│   ├── arch/               # 架构相关代码
│   │   ├── arm/           # 32位 ARM
│   │   ├── arm64/         # 64位 ARM
│   │   └── riscv/         # RISC-V 支持
│   ├── crypto/            # 加密算法
│   │   ├── aes/          # AES 实现
│   │   ├── rsa/          # RSA 实现
│   │   └── ...
│   ├── drivers/           # 设备驱动
│   ├── kernel/            # 内核核心
│   ├── lib/               # 库函数
│   ├── mm/                # 内存管理
│   ├── tee/               # TEE 特定功能
│   └── pta/               # 伪可信应用
├── ldelf/                  # 动态链接和加载
├── ta/                     # TA 应用模板
├── lib/                    # 公共库
├── mk/                     # 构建脚本
├── scripts/                # 工具脚本
└── keys/                   # 签名密钥
```

**详细功能模块**:

| 模块 | 功能描述 |
|-----|---------|
| `core/arch/` | CPU 架构初始化、异常处理、TLB 管理 |
| `core/crypto/` | AES, RSA, SHA, ECDSA 等算法 |
| `core/drivers/` | 设备驱动（GIC, UART, 存储等）|
| `core/kernel/` | 线程调度、中断处理、系统调用 |
| `core/mm/` | 虚拟地址管理、页表、堆栈管理 |
| `core/tee/` | 安全存储、会话管理、TA 管理 |
| `core/pta/` | 系统伪 TA（系统调用实现）|
| `ldelf/` | ELF 加载和动态链接 |
| `ta/` | TA 开发模板和基础代码 |

**重要特性**:
- 微内核架构
- 支持多个 TA 并发
- 内存隔离
- 安全中断处理
- TEE 系统调用接口

**构建输出**:
- `tee.bin` - OP-TEE 二进制
- `tee-init_load_addr=*` - 带加载地址的版本
- 各种库和测试二进制文件

---

#### 6. **linux/** - Linux 内核
**完整路径**: `/optee-qemu/linux/`

**用途**:
- REE（正常世界）操作系统内核
- 应用程序执行环境
- OP-TEE 驱动集成

**标准目录结构**:

| 目录 | 用途 |
|-----|------|
| `arch/` | 架构特定代码 |
| `drivers/` | 设备驱动 |
| `fs/` | 文件系统 |
| `kernel/` | 内核核心 |
| `mm/` | 内存管理 |
| `net/` | 网络协议栈 |
| `security/` | 安全相关 |
| `include/` | 头文件 |

**OP-TEE 相关**:
- OP-TEE 驱动位于 `drivers/optee/`
- 支持 TEE-REE 通信
- 提供 `/dev/teeX` 设备接口

**构建输出**:
- `arch/arm/boot/Image` 或 `vmlinux` - 内核镜像
- `System.map` - 符号表
- `.config` - 内核配置

---

#### 7. **xen/** - Xen 虚拟化管理程序
**完整路径**: `/optee-qemu/xen/`

**用途**:
- 虚拟机监视程序（可选）
- 多 VM 管理
- 系统资源隔离

**关键目录**:
- `xen/` - 主要代码
- `tools/` - 工具集
- `docs/` - 文档

---

### 第三层：应用和库层

#### 8. **optee_client/** - OP-TEE 客户端库
**完整路径**: `/optee-qemu/optee_client/`

**用途**:
- REE 侧与 TEE 通信的库
- 会话和命令管理
- 共享内存处理

**关键子目录**:

```
optee_client/
├── libteec/              # TEE 客户端库
│   ├── include/         # 公共 API 头文件
│   └── src/             # 实现代码
├── libckteec/           # CK（Cryptoki）接口
├── libseteec/           # SETEEC 接口
├── libteeacl/           # 访问控制库
├── tee-supplicant/      # TEE 支持守护进程
│   └── src/
├── scripts/             # 安装脚本
└── ci/                  # CI 配置
```

**核心 API 和工具**:

| 组件 | 功能 |
|-----|------|
| `libteec` | 标准 TEE 客户端库 |
| `tee-supplicant` | 系统调用代理守护进程 |
| `libckteec` | PKCS#11 兼容接口 |
| `libseteec` | SE（安全元素）接口 |

**关键函数**:
- `TEEC_InitializeContext()` - 初始化 TEE 连接
- `TEEC_OpenSession()` - 打开 TA 会话
- `TEEC_InvokeCommand()` - 调用 TA 命令
- `TEEC_CloseSession()` - 关闭会话

**输出文件**:
- `libteec.so` - 共享库
- `tee-supplicant` - 守护进程可执行文件

---

#### 9. **optee_examples/** - OP-TEE 示例应用
**完整路径**: `/optee-qemu/optee_examples/`

**用途**:
- OP-TEE 功能演示
- 开发者参考代码
- API 使用示例

**示例项目**:

| 示例 | 说明 |
|-----|------|
| `hello_world/` | 基础 TA 和客户端 |
| `aes/` | AES 加密/解密演示 |
| `ecdsa/` | ECDSA 数字签名 |
| `sha/` | SHA 哈希计算 |
| `acipher/` | 非对称密码（RSA）|
| `ecdh/` | ECDH 密钥交换 |
| `secure_storage/` | 安全存储访问 |
| `random/` | 随机数生成 |
| `hotp/` | 一次性密码 |
| `sign_verify/` | 签名和验证 |
| `plugins/` | TA 插件示例 |

**目录结构**:
```
example_name/
├── ta/                  # TA 源代码
│   └── Makefile
├── host/                # 主机应用代码
│   └── Makefile
└── Makefile             # 顶级 Makefile
```

**编译方式**:
```bash
cd optee_examples/hello_world
make TA_DEV_KIT_DIR=../optee_os/out/.../export-ta_arm64
```

---

#### 10. **optee_test/** - OP-TEE 测试套件
**完整路径**: `/optee-qemu/optee_test/`

**用途**:
- 系统功能验证
- 回归测试
- 认证测试

**关键子目录**:

| 目录 | 用途 |
|-----|------|
| `ta/` | 测试 TA 实现 |
| `host/` | 主机测试程序 |
| `cert/` | 认证相关测试 |
| `scripts/` | 测试脚本 |

**测试类型**:
- 单元测试
- 功能测试
- 性能测试
- 安全测试

---

#### 11. **optee_ftpm/** - 固件 TPM 实现
**完整路径**: `/optee-qemu/optee_ftpm/`

**用途**:
- 在 OP-TEE 中实现 TPM 2.0
- 提供 TPM 功能
- 支持测量启动

**关键文件**:

| 文件/目录 | 用途 |
|----------|------|
| `fTPM.c` | 主实现文件 |
| `platform/` | 平台适配代码 |
| `reference/` | MS TPM 参考实现 |
| `include/` | 头文件 |
| `tee/` | TEE 集成代码 |

**功能**:
- TPM 命令处理
- 密钥和证书存储
- PCR（Platform Configuration Register）管理
- 事件日志记录

---

#### 12. **optee_rust/** - Rust 语言支持
**完整路径**: `/optee-qemu/optee_rust/`

**用途**:
- 为 OP-TEE 提供 Rust 语言支持
- 内存安全的 TA 开发
- Rust 工具链集成

**关键子目录**:

| 目录 | 用途 |
|-----|------|
| `optee-teec/` | Rust TEE 客户端库 |
| `optee-utee/` | Rust TEE 用户态库 |
| `cargo-optee/` | Cargo 工具扩展 |
| `crates/` | Rust crate 集合 |
| `examples/` | Rust 示例代码 |
| `projects/` | 完整项目示例 |

**主要功能**:
- Rust FFI 绑定
- 安全的内存管理
- Cargo 构建支持
- TA 模板

---

#### 13. **buildroot/** - 根文件系统构建
**完整路径**: `/optee-qemu/buildroot/`

**用途**:
- 构建最小 Linux 根文件系统
- 软件包管理
- 交叉编译环境

**关键目录**:

| 目录 | 用途 |
|-----|------|
| `package/` | 软件包定义 |
| `board/` | 开发板支持 |
| `arch/` | 架构配置 |
| `toolchain/` | 工具链支持 |
| `system/` | 系统配置 |
| `configs/` | 默认配置 |
| `fs/` | 文件系统构建 |
| `boot/` | 启动文件 |

**重要文件**:
- `Config.in` - 配置菜单
- `.config` - 构建配置
- `Makefile` - 主 Makefile

**构建输出**:
- 根文件系统镜像（.cpio.gz 或 .ext4）
- 文件系统树（`output/target/`）

---

### 第四层：支持库和工具

#### 14. **mbedtls/** - Mbed TLS 加密库
**完整路径**: `/optee-qemu/mbedtls/`

**用途**:
- 提供加密和 SSL/TLS 功能
- OP-TEE 密码学基础
- 证书处理

**关键目录**:

| 目录 | 用途 |
|-----|------|
| `library/` | 库实现源代码 |
| `include/` | 公共 API 头文件 |
| `programs/` | 工具程序 |
| `tests/` | 测试用例 |
| `scripts/` | 辅助脚本 |
| `configs/` | 配置文件 |

**主要模块**:
- AES, DES 等对称加密
- RSA, ECDSA 等非对称加密
- SHA, MD5 等哈希算法
- SSL/TLS 协议
- X.509 证书处理

---

#### 15. **ms-tpm-20-ref/** - 微软 TPM 参考实现
**完整路径**: `/optee-qemu/ms-tpm-20-ref/`

**用途**:
- TPM 2.0 核心实现代码
- optee_ftpm 的基础

**内容**:
- TPM 命令处理
- 持久化存储
- 密钥和对象管理

---

#### 16. **qemu/** - QEMU 模拟器
**完整路径**: `/optee-qemu/qemu/`

**用途**:
- 完整系统仿真
- ARM 架构模拟
- 开发和测试环境

**关键特性**:
- ARMv8 仿真
- 多核支持
- 虚拟设备
- GDB 调试集成
- TCG/KVM 加速

**构建输出**:
- `qemu-system-aarch64` - 64位 ARM 仿真器
- `qemu-system-arm` - 32位 ARM 仿真器

---

#### 17. **linux-arm-ffa-user/** - ARM FFA 用户态模块
**完整路径**: `/optee-qemu/linux-arm-ffa-user/`

**用途**:
- 在 Linux 用户态实现 ARM FFA 接口
- 应用程序与 SPM 通信

**关键文件**:

| 文件 | 用途 |
|-----|------|
| `arm_ffa_user.c` | FFA 驱动实现 |
| `arm_ffa_user.h` | FFA 接口定义 |
| `load_module.sh` | 模块加载脚本 |

**功能**:
- FFA 调用包装
- 分区通信
- 内存共享管理

---

#### 18. **trusted-services/** - 可信服务框架
**完整路径**: `/optee-qemu/trusted-services/`

**用途**:
- 高级可信服务框架
- 模块化服务实现
- 多种部署模式

**关键目录**:

| 目录 | 用途 |
|-----|------|
| `components/` | 可复用组件库 |
| `deployments/` | 部署配置 |
| `protocols/` | 协议实现 |
| `platform/` | 平台适配 |
| `tools/` | 开发工具 |
| `external/` | 外部依赖 |
| `docs/` | 文档 |

**可用服务**:
- 密码学服务
- 存储服务
- 计数器服务
- 随机数服务

---

#### 19. **toolchains/** - 编译工具链
**完整路径**: `/optee-qemu/toolchains/`

**用途**:
- 存储预构建的交叉编译工具
- 统一的编译环境

**包含的工具**:
- GCC/Clang 交叉编译器
- 链接器和 binutils
- C 库（glibc/musl）
- 调试工具（GDB）

---

### 第五层：输出和文档

#### 20. **out/** - OP-TEE 构建输出
**完整路径**: `/optee-qemu/out/`

**用途**:
- 存储 OP-TEE 构建结果

**典型结构**:
```
out/
└── arm64-plat-qemu_v8
    ├── core/
    ├── ta/
    ├── export-ta_arm64/
    ├── export-user_ta/
    └── ...
```

**输出文件**:
- `tee.bin` - TEE 镜像
- `ta/` - 可信应用
- `export-*/` - 导出的头文件和库

---

#### 21. **out-br/** - BuildRoot 构建输出
**完整路径**: `/optee-qemu/out-br/`

**用途**:
- 存储 BuildRoot 构建结果

**典型结构**:
```
out-br/
└── qemu_v8
    ├── target/
    ├── images/
    ├── build/
    └── ...
```

**关键文件**:
- `images/rootfs.cpio.gz` - 根文件系统
- `images/Image` - Linux 内核
- `target/` - 安装树

---

#### 22. **readme/** - 文档目录
**完整路径**: `/optee-qemu/readme/`

**用途**:
- 存储项目文档
- 技术参考
- 用户指南

**包含文件**:
- README 文件
- 架构文档
- API 参考
- 平台指南

---

#### 23. **.repo/** - Repo 元数据
**完整路径**: `/optee-qemu/.repo/`

**用途**:
- Repo 工具元数据和管理信息
- Git 仓库引用

**不应修改**：此目录由 Repo 工具管理

---

## 工作流示例

### 场景 1：编译一个完整的 QEMU 系统

```bash
cd /optee-qemu
make -f build/Makefile PLATFORM=qemu_v8 -j$(nproc)
```

**涉及的目录**:
1. `build/` - 构建配置
2. `u-boot/` - 编译启动加载程序
3. `trusted-firmware-a/` - 编译 TF-A
4. `optee_os/` - 编译 OP-TEE
5. `linux/` - 编译 Linux 内核
6. `buildroot/` - 构建根文件系统
7. `out/`, `out-br/` - 输出结果

---

### 场景 2：开发新的 TA

```bash
# 1. 创建 TA 项目
mkdir optee_examples/my_app/{ta,host}

# 2. 实现 TA（使用 optee_os/ta 的模板）
# 3. 实现客户端（使用 optee_client/libteec）
# 4. 使用 optee_examples 中的 Makefile 模板

# 5. 编译
cd optee_examples/my_app
make TA_DEV_KIT_DIR=../optee_os/out/.../export-ta_arm64

# 6. 测试
# 使用 optee_test 中的测试框架
```

**涉及的目录**:
- `optee_examples/` - 项目位置
- `optee_os/` - TA 开发工具
- `optee_client/` - 客户端库
- `out/` - 构建输出

---

### 场景 3：调试和测试

```bash
# 1. 编译调试版本
make -f build/Makefile PLATFORM=qemu_v8 DEBUG=1

# 2. 运行 QEMU
qemu-system-aarch64 -nographic -smp 2 \
  -m 1G -bios out/br/qemu_v8/firmware.elf \
  -initrd out-br/qemu_v8/root.cpio.gz

# 3. 在 QEMU 中运行测试
optee_test_app

# 4. 使用 GDB 调试
# 另一终端：
arm-linux-gnueabihf-gdb
(gdb) target remote :1234
(gdb) break main
(gdb) continue
```

**涉及的目录**:
- `build/` - 构建配置
- `qemu/` - 模拟器
- `optee_test/` - 测试
- `out/`, `out-br/` - 输出

---

## 修改和扩展指南

### 添加新驱动

1. 在 `optee_os/core/drivers/` 中创建驱动代码
2. 更新 `optee_os/core/drivers/sub.mk`
3. 在相应的 conf 文件中启用驱动

### 添加新平台支持

1. 在 `build/` 中创建 `platform.mk`
2. 在 `trusted-firmware-a/plat/` 中添加代码
3. 在 `u-boot/` 中添加平台代码
4. 在 `buildroot/board/` 中添加根文件系统配置

### 集成第三方库

1. 下载源代码到对应的子目录
2. 创建或更新 Makefile 配置
3. 在 `build/` 中添加构建规则

---

## 常见问题和故障排除

### Q: 如何只编译 OP-TEE 而不编译其他组件？

A: 
```bash
cd optee_os
make -f core/core.mk \
  PLATFORM=qemu_v8 \
  ARCH=arm64 \
  CROSS_COMPILE=aarch64-linux-gnu-
```

### Q: 如何调试 TA？

A:
1. 在 `optee_os/` 中启用 DEBUG
2. 在 TA 代码中设置断点
3. 使用 GDB 附加到 QEMU 进程

### Q: 如何添加自己的配置选项？

A:
1. 在 `build/` 中的 Makefile 中添加
2. 通过环境变量或命令行参数传递
3. 在平台特定的 mk 文件中使用

---

## 快速参考

| 任务 | 主要目录 |
|-----|---------|
| 修改启动流程 | `u-boot/`, `trusted-firmware-a/` |
| 添加 TA | `optee_examples/`, `optee_os/ta/` |
| 修改 TEE OS | `optee_os/core/` |
| 调试应用 | `optee_client/`, `optee_test/` |
| 配置平台 | `build/` |
| 修改文件系统 | `buildroot/` |
| 编写驱动 | `optee_os/core/drivers/`, `linux/drivers/` |
| 性能优化 | `optee_os/`, `linux/` |

---

## 相关文档

- REPOSITORY_ARCHITECTURE.md - 仓库整体架构概述
- 各模块的 README.md 文件
- 官方文档: http://optee.readthedocs.io

