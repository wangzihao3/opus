# Project Context

## Purpose

Opus 是一个完全开放、免版税的音频编解码器，专为互联网交互式语音和音频传输而设计。该项目的主要目标包括：

- **实时通信**: 支持 VoIP、视频会议、游戏内聊天等应用场景
- **广泛适应性**: 从低比特率窄带语音到高质量立体声音乐的全方位支持
- **网络鲁棒性**: 在有损网络环境下保持音频质量
- **深度学习增强**: Opus 1.5 引入了基于深度学习的冗余编码（DRED）和 LPCNet 技术，用于丢包隐藏和带宽扩展

## Tech Stack

### 核心技术
- **C 语言**: 主要实现语言（支持 C89/C99 标准）
- **深度学习**: Python/Keras/TensorFlow 用于模型训练
- **SIMD 优化**: SSE2, SSSE3, AVX, AVX2/FMA, NEON, ARM Dot Product

### 构建系统
- **Autoconf/Automake/Libtool**: 主要构建系统
- **CMake**: Windows 和替代构建系统
- **Meson**: 另一个替代构建系统

### 编解码器组件
- **CELT**: 低延迟音频编解码器（用于低延迟场景）
- **SILK**: 语音编解码器（用于高质量语音）
- **LPCNet**: 基于深度学习的语音合成和声码器
- **DRED**: 深度冗余编码（用于丢包恢复）

### 相关标准
- IETF RFC 6716: Opus 编解码器规范
- IETF RFC 7587: Opus over RTP
- IETF RFC 7845: Ogg 封装的 Opus

## Project Conventions

### Code Style

#### 许可证头文件
所有源文件必须包含以下 BSD 3-Clause 许可证声明：

```c
/* Copyright (c) YEAR Xiph.Org Foundation, Skype Limited
   Written by AUTHORS */
/*
   Redistribution and use in source and binary forms, with or without
   modification, are permitted provided that the following conditions
   are met:

   - Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.

   - Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.

   THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
   ``AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
   LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
   A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER
   OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
   EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
   PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
   PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
   LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
   NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
   SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
*/
```

#### 命名约定
- 函数: `snake_case`（例如: `opus_encode`, `celt_decode`）
- 类型定义: `PascalCase`（例如: `OpusEncoder`, `CELTEncoder`）
- 宏定义: `UPPER_CASE`（例如: `MAX_ENCODER_BUFFER`, `FIXED_POINT`）
- 结构体成员: `snake_case`

#### 文件组织
- `silk/`: SILK 编解码器实现
- `celt/`: CELT 编解码器实现
- `src/`: 顶层 Opus API 实现
- `dnn/`: 深度学习相关代码（LPCNet, DRED）
- `include/`: 公共头文件
- `tests/`: 测试代码

#### C 语言标准
- 默认支持浮点运算
- 可通过 `--enable-fixed-point` 或定义 `FIXED_POINT` 宏切换到定点运算
- 定点版本主要用于嵌入式环境
- 兼容 C89 和 C99 编译器

### Architecture Patterns

#### 编解码器架构
- **混合设计**: SILK（语音）和 CELT（音频）的混合架构
- **模式切换**: 根据应用类型（VoIP、音频、低延迟）自动切换
- **带宽可变**: NB (4kHz), MB (6kHz), WB (8kHz), SWB (12kHz), FB (20kHz)

#### 内存管理
- 使用 `VAR_ARRAYS`（C99 变长数组）或栈分配
- 避免动态内存分配以确保实时性能
- 使用 RESTRICT 关键字优化指针别名

#### 平台优化
- 运行时 CPU 特性检测
- 编译时优化选项（`-march=native`, `-mfpu=neon` 等）
- 分离的平台特定实现（x86/, arm/ 目录）

### Testing Strategy

#### 自动化测试
```bash
make check  # 运行所有单元测试和系统测试
```

#### 测试类型
- **单元测试**: 各个模块的独立测试（`celt/tests/`, `silk/tests/`）
- **系统测试**: 完整编解码器测试
- **测试向量**: 标准测试向量（从外部下载）

#### 测试向量测试
```bash
curl -OL https://opus-codec.org/docs/opus_testvectors-rfc8251.tar.gz
tar -zxf opus_testvectors-rfc8251.tar.gz
./tests/run_vectors.sh ./ opus_newvectors 48000
```

#### DRED/LPCNet 测试
- 使用 demo 程序进行编解码测试
- PLC（丢包隐藏）测试需要错误模式文件

### Git Workflow

#### 分支策略
- `main`: 主分支，稳定版本
- 功能分支: 用于新功能开发

#### 提交约定
- 提交信息应清晰描述更改内容
- 参见最近的提交历史：
  - "Remove useless extern "C" declarations"
  - "List LACE/NoLACE/BWE arrays"
  - "Fix extern declarations to make C++ happy"

#### 版本管理
- 使用 `update_version` 脚本自动同步版本号
- 遵循语义化版本控制

## Domain Context

### 音频编解码基础
- **采样率**: 支持 8kHz - 48kHz
- **帧大小**: 2.5ms, 5ms, 10ms, 20ms, 40ms, 60ms
- **比特率**: 6 kb/s - 510 kb/s（可变比特率）
- **声道**: 单声道和立体声

### 深度学习组件
- **LPCNet**: 结合线性预测的 WaveRNN 变体，用于语音合成
- **DRED**: 基于变分自编码器的冗余编码，用于丢包恢复
- **FARGAN/OSCE**: 带宽扩展和特征提取

### 应用模式
- **voip**: 语音通话优化
- **audio**: 音乐和通用音频
- **restricted-lowdelay**: 超低延迟模式（< 5ms）

## Important Constraints

### 技术约束
- **实时性能**: 必须满足实时处理要求
- **延迟敏感**: 某些模式下延迟必须 < 5ms
- **兼容性**: 必须保持与旧版本 Opus 的向后兼容性
- **跨平台**: 支持 Linux, Windows, macOS, 嵌入式系统

### 性能约束
- **CPU 使用**: 应在普通 CPU 上运行（约 3 GFLOP）
- **内存使用**: 避免动态内存分配
- **SIMD 依赖**: 推荐但不强制要求 SIMD 支持

### 法律约束
- **许可证**: BSD 3-Clause 许可证
- **专利**: 免版税专利和版权许可（参见 COPYING 文件）
- **IETF 标准**: 遵循 RFC 6716 规范

### 编译器行为假设
代码依赖于二补码架构的常见实现定义行为：
- 负数的右移符合二补码算术规则
- 有符号整数转换的模 2^N 约简
- 负数整数除法向零截断
- 64 位整数类型支持

## External Dependencies

### 构建依赖
- **autoconf/automake/libtool**: 主要构建系统
- **gcc/clang**: C 编译器
- **make**: 构建工具

### 可选依赖
- **NE10**: ARM NEON 优化库（用于 FFT 和 MDCT）
- **TensorFlow/Keras**: 深度学习模型训练

### 深度学习模型
- **LPCNet 模型**: 通过 `download_model.sh` 下载
- **DRED 模型**: RdOVAE 编码器/解码器权重
- **训练数据**: 从 OpenSLR 等来源获取

### 外部项目
- **opus-tools**: Ogg 封装的 Opus 文件工具
- https://gitlab.xiph.org/xiph/opus-tools.git

### 标准组织
- **IETF**: 互联网工程任务组（RFC 标准）
- **Xiph.Org**: 基金会支持的编解码器项目
