# OPTEE-QEMU 技术文档汇总

## 📚 文档总览

此目录包含 **optee-qemu** 仓库的完整技术文档，共 **4 个主要文档**，约 **2,500+ 行**，详细说明了整个项目的架构、模块、依赖关系和工作流。

---

## 📖 文档列表

### 1️⃣ **README_CN.md** (快速导航和索引)
- **大小**: 9.6 KB | **行数**: 367 行
- **用途**: 快速查找和导航其他文档
- **包含内容**:
  - 4 个场景的快速开始指南
  - 模块功能速查表
  - 按任务查找文档
  - 常见问题解答
  - 学习路径建议

**何时阅读**: ⏱️ 5-15 分钟（第一篇推荐阅读）

---

### 2️⃣ **REPOSITORY_ARCHITECTURE.md** (仓库架构全景)
- **大小**: 18 KB | **行数**: 667 行
- **用途**: 理解整个项目的总体架构和各模块职责
- **包含内容**:
  - 完整的目录结构树
  - 23 个核心模块的详细说明
  - TEE/REE 分离架构图
  - 启动序列
  - 依赖关系概览
  - 平台支持矩阵
  - 常见开发任务指南

**何时阅读**: ⏱️ 30-45 分钟（初学者必读）

**适合场景**:
- 第一次接触 OP-TEE QEMU
- 需要了解项目整体结构
- 想要快速上手开发

---

### 3️⃣ **DIRECTORY_GUIDE.md** (目录详细指南)
- **大小**: 18 KB | **行数**: 815 行  
- **用途**: 深入了解每个目录的具体用途和结构
- **包含内容**:
  - 5 层次的 23 个目录逐一详解
  - 每个目录的关键文件说明
  - 输入输出文件说明
  - 子目录功能表
  - 3 个详细工作流示例
  - 修改和扩展指南
  - 快速参考表

**何时阅读**: ⏱️ 20-30 分钟（或按需查阅）

**适合场景**:
- 需要在某个特定目录工作
- 想要了解目录间的关系
- 需要找到某个特定功能的代码位置

---

### 4️⃣ **MODULE_DEPENDENCIES.md** (模块依赖和数据流)
- **大小**: 26 KB | **行数**: 696 行
- **用途**: 理解模块间的依赖关系和通信方式
- **包含内容**:
  - 分层架构完整图解
  - 详细的模块依赖图
  - 4 种关键通信协议
  - 启动序列流程图
  - 内存映射关系
  - 系统调用流程详解
  - 2 个详细的数据流示例
  - 构建依赖顺序和文件依赖详表
  - 关键接口定义
  - 版本兼容性表

**何时阅读**: ⏱️ 45-60 分钟

**适合场景**:
- 需要理解模块间如何协作
- 调试跨模块问题
- 优化编译流程
- 理解系统启动和通信

---

## 🎯 快速导航

### 按角色查找文档

#### 👨‍💻 项目经理
- 📄 README_CN.md → 快速概览
- 📄 REPOSITORY_ARCHITECTURE.md → 系统架构部分

#### 🎓 新手开发者
1. 📄 README_CN.md → 5-10 分钟
2. 📄 REPOSITORY_ARCHITECTURE.md → 30-45 分钟
3. 📄 DIRECTORY_GUIDE.md → 20-30 分钟
4. 📄 MODULE_DEPENDENCIES.md → 45-60 分钟（可选）

#### 🔧 核心开发者
1. 📄 REPOSITORY_ARCHITECTURE.md → 快速复习
2. 📄 DIRECTORY_GUIDE.md → 查看相关目录
3. 📄 MODULE_DEPENDENCIES.md → 深入理解
4. 代码 + 官方文档

#### 🏗️ 系统架构师
- 📄 MODULE_DEPENDENCIES.md → 完整系统图
- 📄 REPOSITORY_ARCHITECTURE.md → 架构概览
- 📄 DIRECTORY_GUIDE.md → 实现细节

#### 🧪 测试/QA
- 📄 DIRECTORY_GUIDE.md → optee_test 部分
- 📄 README_CN.md → 快速开始

---

### 按任务查找文档

| 任务 | 推荐文档 | 部分 |
|------|---------|------|
| 编译第一个项目 | README_CN.md | 快速开始 → 场景 1 |
| 开发新 TA | README_CN.md | 快速开始 → 场景 2 |
| 修改 OP-TEE 核心 | README_CN.md | 快速开始 → 场景 3 |
| 支持新平台 | README_CN.md | 快速开始 → 场景 4 |
| 理解启动流程 | README_CN.md | 快速开始 → 场景 5 |
| 查找某个文件 | DIRECTORY_GUIDE.md | 相应目录部分 |
| 理解模块如何协作 | MODULE_DEPENDENCIES.md | 关键通信协议 |
| 查看构建顺序 | MODULE_DEPENDENCIES.md | 构建依赖关系 |
| 理解整体架构 | REPOSITORY_ARCHITECTURE.md | 系统架构 |

---

## 📊 文档统计

```
总计文档数：4 个
总计行数：2,545 行
总计大小：~72 KB

分布：
- 架构和概览: 1,363 行 (REPOSITORY_ARCHITECTURE + MODULE_DEPENDENCIES)
- 目录指南: 815 行 (DIRECTORY_GUIDE)
- 快速开始: 367 行 (README_CN)

覆盖范围：
✓ 23 个主要模块
✓ 5 层系统架构
✓ 4 种通信协议
✓ 完整启动流程
✓ 详细工作流示例
✓ 常见问题解答
```

---

## 🎓 学习路径

### 路径 1：完全新手（推荐 2-3 小时）

```
第 1 天：
1. 阅读 README_CN.md (10 分钟)
2. 阅读 REPOSITORY_ARCHITECTURE.md (40 分钟)
3. 快速浏览 DIRECTORY_GUIDE.md 的目录列表 (10 分钟)

第 2 天：
4. 编译项目 (参考 README_CN.md 的快速命令)
5. 运行 QEMU 和示例
6. 重新阅读 REPOSITORY_ARCHITECTURE.md 的相关部分
7. 查阅 DIRECTORY_GUIDE.md 找具体代码位置

第 3 天：
8. 阅读 MODULE_DEPENDENCIES.md 的"关键通信协议"部分
9. 尝试修改示例代码
10. 调试并观察系统行为
```

### 路径 2：有基础开发者（推荐 1-2 小时）

```
1. 快速阅读 README_CN.md (5 分钟)
2. 查看 REPOSITORY_ARCHITECTURE.md 相关部分 (20 分钟)
3. 深入 DIRECTORY_GUIDE.md 查看特定模块 (30 分钟)
4. 根据需要查阅 MODULE_DEPENDENCIES.md (15-30 分钟)
5. 开始编写代码
```

### 路径 3：系统设计者（推荐 2-3 小时）

```
1. 阅读 REPOSITORY_ARCHITECTURE.md (40 分钟)
2. 阅读 MODULE_DEPENDENCIES.md 全部 (60 分钟)
3. 阅读 DIRECTORY_GUIDE.md 的工作流部分 (30 分钟)
4. 查看源代码验证理解
```

---

## 🔍 核心概念速览

所有文档都包含以下核心概念的详细说明：

### 系统架构
- **TEE (Trusted Execution Environment)** - 安全执行环境
- **REE (Rich Execution Environment)** - 正常执行环境
- **世界切换** - REE ↔ TEE 转换

### 关键组件
- **OP-TEE OS** - TEE 侧操作系统
- **OP-TEE Client** - REE 侧通信库
- **TF-A** - 安全监视模式
- **Linux Kernel** - REE 操作系统

### 通信方式
- **SMC** - Secure Monitor Call
- **TEE Ioctl** - 设备驱动接口
- **共享内存** - REE/TEE 共享数据

---

## ✨ 文档特色

### ✓ 完整性
- 覆盖所有 23 个核心模块
- 详解 5 层系统架构
- 说明所有关键流程

### ✓ 实用性
- 包含具体的代码位置
- 提供快速命令参考
- 包含实际工作流示例

### ✓ 易读性
- 清晰的目录结构
- 大量的图表和表格
- 按场景分类指导

### ✓ 可查询性
- 完整的索引和导航
- 快速参考表
- 按任务查找

---

## 📚 与官方文档的关系

| 官方资源 | 本文档的补充 |
|---------|-----------|
| OP-TEE 官方文档 | 提供仓库视图，官方提供功能视图 |
| TF-A 官方文档 | 补充 ARM 侧启动流程详解 |
| 源代码 | 提供导航和理解上下文 |
| GitHub Wiki | 更新和详细的系统设计图 |

**建议**: 本文档作为**入门和导航工具**，配合**官方文档和源代码**深入学习。

---

## 💡 使用建议

### 第一次使用
1. 从 README_CN.md 开始
2. 选择适合你的场景
3. 按推荐顺序阅读其他文档

### 查找特定信息
1. 在 README_CN.md 中查找相关关键字
2. 如果找不到，查阅相应的详细文档
3. 使用编辑器的查找功能 (Ctrl+F)

### 深度学习
1. 完整阅读所有 4 个文档
2. 对照源代码理解
3. 实际编写代码验证理解

---

## 🎯 文档目标用户

- ✓ OP-TEE 初学者
- ✓ 需要快速上手的开发者
- ✓ 项目经理和架构师
- ✓ 需要理解代码库结构的人
- ✓ 新入职团队成员
- ✓ 贡献开源代码的开发者

---

## 📋 文档清单

```
readme/
├── README_CN.md                    # 📍 从这里开始
│   ├── 快速导航
│   ├── 5 个快速开始场景
│   ├── 模块速查表
│   ├── 按任务查找
│   └── 常见问题
│
├── REPOSITORY_ARCHITECTURE.md      # 📍 系统全景
│   ├── 目录结构树
│   ├── 23 个模块详解
│   ├── TEE/REE 架构
│   ├── 启动序列
│   ├── 依赖关系
│   └── 平台支持矩阵
│
├── DIRECTORY_GUIDE.md              # 📍 目录深入
│   ├── 5 层次 23 个目录
│   ├── 每个目录详解
│   ├── 子目录功能表
│   ├── 工作流示例
│   └── 修改指南
│
└── MODULE_DEPENDENCIES.md          # 📍 依赖和通信
    ├── 分层架构图
    ├── 依赖关系图
    ├── 通信协议详解
    ├── 启动序列流程
    ├── 数据流示例
    ├── 构建依赖顺序
    └── 接口定义
```

---

## 🚀 立即开始

### 第一步：选择你的角色

- 👶 **完全新手**: 阅读 README_CN.md 的 "快速开始 → 场景 1"
- 💻 **有经验的开发者**: 阅读 README_CN.md 的 "按任务查找文档"
- 🏗️ **系统架构师**: 直接读 MODULE_DEPENDENCIES.md

### 第二步：编译运行

```bash
cd /home/gjh/optee-qemu
make -f build/Makefile PLATFORM=qemu_v8 -j$(nproc)
```

### 第三步：运行 QEMU

```bash
qemu-system-aarch64 -nographic \
  -bios out-br/qemu_v8/firmware.elf \
  -initrd out-br/qemu_v8/root.cpio.gz \
  -m 1G -smp 2
```

### 第四步：继续学习

参考相应的文档完成你的任务！

---

## 📞 需要帮助？

1. **查阅本文档** - 可能已有答案
2. **查阅官方文档** - http://optee.readthedocs.io
3. **查看源代码** - 最可靠的参考
4. **查看示例** - optee_examples/ 和 optee_test/
5. **社区讨论** - OP-TEE 邮件列表和 GitHub

---

## 📝 文档管理

- **生成日期**: 2026年2月12日
- **基础版本**: OP-TEE v3.x, TF-A v2.14.0
- **格式**: Markdown
- **编码**: UTF-8
- **许可**: 与 optee-qemu 仓库相同

---

**祝你学习愉快！开始探索 OP-TEE QEMU 的世界吧！** 🎉

