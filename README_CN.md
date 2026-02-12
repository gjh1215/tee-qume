# OPTEE-QEMU 技术文档索引与快速开始

## 📚 文档导航

本目录包含 optee-qemu 仓库的完整技术文档。请根据需要选择合适的文档阅读。

---

## 📖 文档列表

### 1. **REPOSITORY_ARCHITECTURE.md** - 仓库架构全景
**适合场景**：想要理解整个 optee-qemu 项目的架构和各个模块的总体功能

**核心内容**：
- ✓ 完整的目录树结构
- ✓ 23 个主要模块的功能说明
- ✓ TEE/REE 分离架构
- ✓ 启动序列
- ✓ 依赖关系概览
- ✓ 常见开发任务指南

**推荐阅读时间**：30-45 分钟

---

### 2. **DIRECTORY_GUIDE.md** - 目录详细指南
**适合场景**：想要了解某个特定目录的用途、子目录结构和工作流

**核心内容**：
- ✓ 按层次分类的 23 个目录详解
- ✓ 每个目录的关键文件说明
- ✓ 输入输出文件说明
- ✓ 工作流示例
- ✓ 修改和扩展指南
- ✓ 快速参考表

**推荐阅读时间**：20-30 分钟（或按需查阅）

---

### 3. **MODULE_DEPENDENCIES.md** - 模块依赖关系
**适合场景**：想要理解模块间的依赖关系、通信协议和数据流

**核心内容**：
- ✓ 分层架构图
- ✓ 模块依赖图
- ✓ 四种关键通信协议详解
- ✓ 启动序列流程图
- ✓ 内存映射关系
- ✓ 系统调用流程
- ✓ 详细数据流示例
- ✓ 构建依赖顺序
- ✓ 文件依赖详表

**推荐阅读时间**：45-60 分钟

---

## 🚀 快速开始

### 场景 1：我是 OP-TEE 新手

**推荐阅读顺序**：
1. 本文档 (5 分钟)
2. REPOSITORY_ARCHITECTURE.md → 系统架构部分 (10 分钟)
3. DIRECTORY_GUIDE.md → 核心模块部分 (15 分钟)
4. 官方文档：http://optee.readthedocs.io

**快速命令**：
```bash
# 编译完整系统
cd /home/gjh/optee-qemu
make -f build/Makefile PLATFORM=qemu_v8 -j$(nproc)

# 运行 QEMU
qemu-system-aarch64 -nographic \
  -bios out-br/qemu_v8/firmware.elf \
  -initrd out-br/qemu_v8/root.cpio.gz \
  -m 1G -smp 2
```

---

### 场景 2：我需要开发一个新的 TA

**推荐阅读顺序**：
1. DIRECTORY_GUIDE.md → `optee_examples/` 部分
2. DIRECTORY_GUIDE.md → `optee_os/` 部分
3. DIRECTORY_GUIDE.md → `optee_client/` 部分
4. MODULE_DEPENDENCIES.md → 数据流示例 1

**快速命令**：
```bash
# 创建新 TA
mkdir /home/gjh/optee-qemu/optee_examples/my_crypto
cd my_crypto
mkdir ta host

# 参考现有示例
cp ../aes/ta/Makefile ta/
cp ../aes/ta/user_ta_header_defines.h ta/

# 编译
cd ..
make TA_DEV_KIT_DIR=/home/gjh/optee-qemu/optee_os/out/.../export-ta_arm64
```

---

### 场景 3：我需要修改或调试 OP-TEE 核心

**推荐阅读顺序**：
1. REPOSITORY_ARCHITECTURE.md → optee_os 部分
2. DIRECTORY_GUIDE.md → `optee_os/` 部分
3. MODULE_DEPENDENCIES.md → 启动序列
4. MODULE_DEPENDENCIES.md → 数据流示例

**快速命令**：
```bash
# 编译 OP-TEE（调试版本）
cd /home/gjh/optee-qemu/optee_os
make CFG_DEBUG_LEVEL=3 PLATFORM=qemu_v8

# 编译后重新集成
cd /home/gjh/optee-qemu
make -f build/Makefile PLATFORM=qemu_v8

# 在 QEMU 中调试
# 终端 1: 运行 QEMU
qemu-system-aarch64 -nographic -gdb tcp::1234,server,nowait \
  ...

# 终端 2: GDB 调试
aarch64-linux-gnu-gdb
(gdb) target remote localhost:1234
(gdb) file optee_os/out/*/core/tee.elf
```

---

### 场景 4：我需要支持新的硬件平台

**推荐阅读顺序**：
1. REPOSITORY_ARCHITECTURE.md → 平台支持矩阵
2. DIRECTORY_GUIDE.md → `build/` 部分
3. DIRECTORY_GUIDE.md → `u-boot/`, `trusted-firmware-a/` 部分

**核心步骤**：
1. 在 `build/` 中创建 `platform.mk`
2. 在 `trusted-firmware-a/plat/` 中添加平台特定代码
3. 在 `u-boot/` 中添加平台支持
4. 在 `optee_os/core/arch/` 中添加必要代码

---

### 场景 5：我需要理解系统的启动流程

**推荐阅读顺序**：
1. REPOSITORY_ARCHITECTURE.md → 启动序列
2. MODULE_DEPENDENCIES.md → 启动序列和初始化流
3. DIRECTORY_GUIDE.md → 第一层和第二层模块

**关键流程**：
```
ROM Code
   ↓
u-boot SPL (BL1)
   ↓
u-boot (BL2)
   ↓
TF-A (BL31) - 安全监视
   ↓
├─ OP-TEE (BL32) - 安全世界
└─ Linux (BL33) - 正常世界
```

---

## 📊 模块功能速查表

| 模块 | 职责 | 依赖 | 输出 |
|------|------|------|------|
| **optee_os** | TEE 操作系统核心 | mbedtls | tee.bin, headers |
| **optee_client** | REE 侧客户端库 | optee_os headers | libteec.so |
| **trusted-firmware-a** | 安全监视模式 | u-boot | bl31.bin |
| **u-boot** | 启动加载程序 | toolchain | u-boot.bin |
| **linux** | REE 操作系统 | toolchain | vmlinux |
| **optee_examples** | 示例应用和 TA | optee_os, optee_client | TA 和应用 |
| **optee_test** | 测试套件 | optee_os, optee_client | 测试二进制 |
| **mbedtls** | 加密库 | toolchain | libmbedcrypto |
| **buildroot** | 根文件系统 | 所有应用 | rootfs.cpio.gz |
| **qemu** | ARM 模拟器 | toolchain | qemu-system-aarch64 |
| **optee_ftpm** | TPM 2.0 实现 | optee_os, ms-tpm-20-ref | TPM TA |
| **hafnium** | SPM（可选） | toolchain | hafnium 镜像 |

---

## 🔍 按任务查找文档

### 我想要...

#### 构建系统
- [ ] 编译完整项目 → DIRECTORY_GUIDE.md → `build/` 
- [ ] 添加新平台支持 → REPOSITORY_ARCHITECTURE.md → 常见开发任务
- [ ] 修改编译配置 → DIRECTORY_GUIDE.md → 修改和扩展指南

#### 应用开发
- [ ] 开发新 TA → DIRECTORY_GUIDE.md → `optee_examples/`
- [ ] 开发客户端应用 → DIRECTORY_GUIDE.md → `optee_client/`
- [ ] 使用加密功能 → REPOSITORY_ARCHITECTURE.md → 关键功能模块
- [ ] 使用存储功能 → DIRECTORY_GUIDE.md → 场景 3

#### 系统理解
- [ ] 理解整体架构 → REPOSITORY_ARCHITECTURE.md
- [ ] 理解启动流程 → MODULE_DEPENDENCIES.md → 启动序列
- [ ] 理解 REE/TEE 通信 → MODULE_DEPENDENCIES.md → 关键通信协议
- [ ] 理解内存管理 → MODULE_DEPENDENCIES.md → 内存映射关系

#### 调试和测试
- [ ] 调试代码 → DIRECTORY_GUIDE.md → 工作流示例 → 场景 3
- [ ] 运行测试 → DIRECTORY_GUIDE.md → `optee_test/`
- [ ] 性能优化 → DIRECTORY_GUIDE.md → 快速参考

---

## 📁 文档文件位置

所有文档都位于 `/home/gjh/optee-qemu/readme/` 目录中：

```
readme/
├── REPOSITORY_ARCHITECTURE.md    # 这是你现在在读的文件列表
├── DIRECTORY_GUIDE.md            # 详细的目录指南
├── MODULE_DEPENDENCIES.md        # 模块依赖和数据流
└── README.txt                    # 此文件
```

---

## 🎯 关键概念速览

### TEE (Trusted Execution Environment)
- 安全的执行环境
- 在 ARM 处理器的安全世界运行
- 由 OP-TEE OS 管理
- 运行可信应用 (TA)

### REE (Rich Execution Environment)
- 正常的应用环境
- 在 ARM 处理器的正常世界运行
- 运行 Linux 和用户应用
- 通过 OP-TEE 驱动与 TEE 通信

### TA (Trusted Application)
- 在 TEE 中运行的应用
- 由 optee_os 管理
- 例如：optee_ftpm, optee_examples

### SMC (Secure Monitor Call)
- 从正常世界到安全世界的调用
- 由 ARM 的 SMC 指令触发
- 由 TF-A 处理
- 实现世界切换

### 会话 (Session)
- 应用与 TA 之间的连接
- 通过 TEEC_OpenSession() 建立
- 支持多个并发命令

---

## 🔗 相关资源

### 官方资源
- **OP-TEE 官方文档**: http://optee.readthedocs.io
- **OP-TEE GitHub**: https://github.com/OP-TEE
- **TF-A 官方文档**: https://trustedfirmware-a.readthedocs.io
- **Hafnium**: https://hafnium.readthedocs.io

### 标准和规范
- **ARM TrustZone**: ARM 官方文档
- **SMC Calling Convention**: ARM SMCCC 规范
- **ARM Firmware Framework**: ARM FF-M 规范
- **TPM 2.0 规范**: TCG

### 工具和资源
- **QEMU 文档**: https://www.qemu.org/documentation/
- **Buildroot**: https://buildroot.org/
- **U-Boot**: https://www.denx.de/wiki/U-Boot

---

## 💡 学习建议

### 初级阶段（1-2 周）
1. 阅读本文档和 REPOSITORY_ARCHITECTURE.md
2. 编译项目并在 QEMU 中运行
3. 运行 optee_examples 和 optee_test
4. 理解基本的 REE/TEE 通信

### 中级阶段（2-4 周）
1. 阅读 DIRECTORY_GUIDE.md 和 MODULE_DEPENDENCIES.md
2. 开发自己的 TA
3. 调试和优化代码
4. 理解系统启动过程

### 高级阶段（1-3 月）
1. 修改 OP-TEE 核心代码
2. 支持新的硬件平台
3. 优化性能和安全性
4. 贡献代码到官方仓库

---

## ❓ 常见问题速答

**Q: 我应该从哪里开始？**
A: 先读这个文档的快速开始部分，然后选择适合你的场景。

**Q: 每个文档多长？**
A: 
- 本文档：10-15 分钟
- REPOSITORY_ARCHITECTURE.md：30-45 分钟
- DIRECTORY_GUIDE.md：20-30 分钟（或按需查阅）
- MODULE_DEPENDENCIES.md：45-60 分钟

**Q: 我可以离线阅读这些文档吗？**
A: 是的，所有文档都是 Markdown 格式，可以用任何文本编辑器打开。

**Q: 文档是否会定期更新？**
A: 应该在项目有重大变化时更新。

---

## 📝 文档版本信息

- **生成日期**: 2026年2月12日
- **基础版本**: OP-TEE QEMU v2.14.0（TF-A）
- **覆盖范围**: 
  - ✓ build/ 系统
  - ✓ 23 个主要模块
  - ✓ 启动和初始化
  - ✓ 通信协议
  - ✓ 开发工作流
  
---

## 📞 获得帮助

如果文档中有不清楚的地方：

1. **查阅官方文档**: http://optee.readthedocs.io
2. **查看代码**: 大多数答案都在源代码中
3. **查看示例**: optee_examples/ 中有实际代码
4. **查看测试**: optee_test/ 中有详细用法
5. **社区讨论**: OP-TEE 邮件列表或 GitHub Issues

---

## 📄 文档许可

这些文档遵循与 optee-qemu 仓库相同的许可证。

---

**祝你学习愉快！** 🎓

