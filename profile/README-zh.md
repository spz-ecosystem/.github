# SPZ 生态系统

> 围绕 Khronos SPZ 3D Gaussian Splatting 压缩规范的开源工具链与合规验证体系

## 核心项目

### spz2glb（v2.0.1 稳定版）

**职责边界**：只做两件事——**SPZ→GLB 格式封装**与 **GLB 分发交付链路**

- **核心定位**：无损打包（SPZ 压缩流原封不动存入 GLB）
- **仓库地址**：https://github.com/spz-ecosystem/spz2glb
- **在线演示**：[GitHub Pages](https://spz-ecosystem.github.io/spz2glb/)
- **关键特性**：
  - WASM 内存与 API 能力（预分配、显式释放、统计与双档配置）
  - 双端协同：浏览器侧轻量预览/快速校验；本地 CLI 重任务转换/批处理/深度验证
  - 三层验证：结构验证 / 无损验证 / 解码一致性验证

### spz_gatekeeper（v2.0.1 稳定版）

**职责边界**：仅做 L2 的 SPZ 合法性检查器，审计 header、flags 和 TLV trailer 扩展，同时不破坏原始 SPZ 的基础解码行为

- **核心定位**：SPZ 扩展合规审查与兼容性验证
- **仓库地址**：https://github.com/spz-ecosystem/spz_gatekeeper
- **在线演示**：
  - GitHub Pages: https://spz-ecosystem.github.io/spz_gatekeeper/
  - CloudBase (国内加速): https://openclaw-spz-3gt7x2sya7c10ef2-1355411679.tcloudbaseapp.com/
- **关键特性**：
  - 校验 SPZ header：magic、version、点数、SH degree、flags、reserved
  - 校验 base payload 之后的 TLV trailer 布局
  - 校验已知厂商扩展（如 Adobe Safe Orbit Camera `0xADBE0002`）
  - 对更高版本 SPZ 发出 warning，继续 best-effort 校验

---

## 二维码原生分发体系（实验性）

> **实验性声明**：这是一套实验性的 3D 资产分发体系，旨在为社区面临的**文件体积庞大难以传输**、**存储成本高昂**、**标准碎片化导致兼容性混乱**、**缺乏轻量化即时分发手段**等问题，提供一种以二维码为统一载体的实验性解法。

以二维码为统一载体的 SPZ/GLB 全链路分发与验证体系：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       二维码原生分发体系架构                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│    数据准入层            分发授权层              消费转换层                │
│    ┌──────────┐       ┌──────────┐        ┌──────────┐                 │
│    │  SPZ     │       │  门卫    │        │  spz2glb │                 │
│    │  数据集  │──────→│  授权    │───────→│  网页端  │                 │
│    └──────────┘       └──────────┘        └──────────┘                 │
│                             │                      │                   │
│                             ↓                      ↓                   │
│                        生成带授权                扫码即转换              │
│                        信息的二维码              WASM 无损打包           │
│                             │                      │                   │
│                             └──────────────────────┘                   │
│                                           │                            │
│                                           ↓                            │
│                                    GLB 下载交付                        │
│                                    （或二维码转发）                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 体系流程

| 阶段 | 职责 | 载体 | 实现状态 |
|------|------|------|----------|
| **准入校验** | 门卫对 SPZ 进行 L2 合法性检查 | SPZ 文件 | ✅ `spz_gatekeeper check-spz` |
| **授权分发** | 合法 SPZ/GLB 生成可分发二维码 | 二维码 | 🚧 门卫内置生成（规划中） |
| **扫码校验** | 门卫校验二维码中的授权信息 | 二维码 | 🚧 `spz_gatekeeper verify-qr`（规划中） |
| **扫码转换** | spz2glb 网页端读取二维码并转换 | 二维码→SPZ→GLB | 🚧 v2.1 版本支持 |
| **二维码转发** | GLB 也可通过二维码二次分发 | 二维码 | 🚧 实验性支持 |
| **扫码即渲染** | 浏览器直接渲染（远期探索） | 二维码→渲染 | 🔬 WebGL/Three.js（实验性） |

### 核心设计

- **扫码即校验**：门卫在二维码中嵌入 SPZ 文件哈希与合规签名，扫码时先校验后转换，确保数据来源可信
- **扫码即转换**：spz2glb 网页端识别二维码，自动拉取 SPZ 并执行 WASM 无损打包为 GLB
- **二维码转发**：生成的 GLB 可再次封装为二维码，实现资产的链式轻量分发
- **数据集准入基准**：所有进入生态的 SPZ 数据必须通过门卫校验并生成授权二维码

### 解决的社区痛点

| 痛点 | 传统方案 | 二维码原生体系 |
|------|----------|----------------|
| **文件体积庞大，传输困难** | 依赖云盘/邮件传输大文件 | 二维码轻量分享，扫码即下载转换 |
| **存储成本高昂** | 需长期托管大文件 | 二维码指向对象存储，按需拉取 |
| **标准碎片化，兼容性混乱** | 多种格式扩展各自为政 | 门卫统一校验，合规数据才生成二维码 |
| **缺乏即时分发手段** | 安装专用软件或插件 | 微信/小程序扫码即用，无需安装 |
| **移动端体验差** | 大文件下载崩溃、渲染卡顿 | WASM 预检内存预算，超限引导至 CLI |

### 技术约束

- **HTTPS 直链**：二维码指向的 SPZ/GLB 文件必须配置 CORS
- **移动端内存预算**：32MB 档位，超限文件引导至 CLI 处理
- **状态**：实验性开发中，v2.1 版本实现扫码即转换，远期探索扫码即渲染

---

## 快速导航

| 项目 | 文档 | 演示 |
|------|------|------|
| spz2glb | [Wiki](https://github.com/spz-ecosystem/spz2glb/wiki) | [GitHub Pages](https://spz-ecosystem.github.io/spz2glb/) |
| spz_gatekeeper | [README](https://github.com/spz-ecosystem/spz_gatekeeper/blob/main/README-zh.md) | [GitHub Pages](https://spz-ecosystem.github.io/spz_gatekeeper/) / [CloudBase](https://openclaw-spz-3gt7x2sya7c10ef2-1355411679.tcloudbaseapp.com/) |

---

## 关联项目

- [spz_gatekeeper](https://github.com/spz-ecosystem/spz_gatekeeper) -  SPZ 合法性检查器
- [spz2glb](https://github.com/spz-ecosystem/spz2glb) - SPZ→GLB 格式封装与分发工具
- [spz-entropy](https://github.com/spz-ecosystem/spz-entropy) - SPZ 熵编码优化（开发中）
- [KHR_gaussian_splatting](https://github.com/KhronosGroup/glTF/pull/2490) - Khronos 官方 3D Gaussian Splatting glTF 扩展
- [KHR_gaussian_splatting_compression_spz_2](https://github.com/KhronosGroup/glTF/pull/2531) - Khronos SPZ_2 压缩扩展提案（当前草案）

---

## 关于 SPZ 生态系统

SPZ 生态系统致力于开发和维护 Khronos SPZ 3D Gaussian Splatting 压缩规范的开源工具。我们的目标是让高质量的 3D 内容压缩在整个 3D 图形和 AIGC 行业中变得易于访问、高效且兼容。

**设计原则**：
- **单一职责**：每个工具只做明确边界内的事
- **可验证**：所有产出必须通过标准化验证
- **双端协同**：浏览器侧轻量交互，CLI 侧重任务处理
- **二维码原生**：以二维码为统一分发入口，实现准入-授权-消费全链路闭环，为 3D 资产分发困境提供实验性解法

---

## 社区指南

我们致力于营造一个开放、包容和协作的社区。所有参与者都应遵守我们的[行为准则](https://github.com/spz-ecosystem/.github/blob/main/CODE_OF_CONDUCT.md)。

## 如何贡献

我们欢迎大家的贡献！无论您是开发者、研究人员还是行业合作伙伴，都有多种方式参与其中：

1. **代码贡献**：向我们的核心仓库提交拉取请求
2. **问题报告**：通过报告错误或请求功能来帮助我们改进
3. **文档**：改进我们的指南、教程和 API 参考
4. **社区支持**：在讨论中回答问题并帮助新贡献者

如需详细说明，请阅读我们的[贡献指南](https://github.com/spz-ecosystem/.github/blob/main/CONTRIBUTING.md)。

---

## 联系与合作

- **组织邮箱**：175851233+gugu23456789@users.noreply.github.com
- **Khronos 规范 PR**：[KhronosGroup/glTF#2534](https://github.com/KhronosGroup/glTF/pull/2534)
- **维护者**：[gugu23456789](https://github.com/gugu23456789)

我们积极寻求与行业合作伙伴合作，尤其是从事 3D AIGC、数字孪生和实时渲染领域的合作伙伴。如果您有兴趣成为核心维护者或参与项目，请联系我们！

---

## 开源协议

该组织由 SPZ 生态系统社区维护，并采用 [MIT 许可证](https://github.com/spz-ecosystem/.github/blob/main/LICENSE) 授权。

