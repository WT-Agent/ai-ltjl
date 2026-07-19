<div align="center">

# [网腾无限AI - 聊天记录与关系分析]

**[一个支持情感雷达探测打卡与五种特色聊天流派的 AI 聊天对话诊断与亲密关系分析工具，具备深色玻璃拟态自适应交互与微信端 H5 体验]**

[Vue 3] · [TypeScript] · [Vite] · [Vanilla CSS] · [开源协议 MIT]

[![GitHub stars](https://img.shields.io/github/stars/WT-Agent/ai-ltjl?style=social)](https://github.com/WT-Agent/ai-ltjl)
[![GitHub license](https://img.shields.io/github/license/WT-Agent/ai-ltjl)](https://github.com/WT-Agent/ai-ltjl/blob/main/LICENSE)

[在线演示](#在线演示) · [快速启动](#快速启动) · [参与贡献](#参与贡献) · [支持一下](#支持一下)

</div>

## 关于我们

团队成员均来自 C9 等顶尖学府，在字节、腾讯、阿里的工程师组成，全职创业研发开源 AI 应用产品，让所有人感受 AI 的魅力。

本项目旨在为个人提供客观、严谨且富有趣味性的对话沟通诊断与亲密关系评估服务。用户只需输入双方的角色关系（如：相亲对象，男生A与女生B）与一整段真实的聊天对话历史记录，AI 即可根据交往心理学自动输出潜台词精细破译、话权平衡硬核诊断、推进关系破局指南及每日雷达探测打卡金句。页面内置了支持物理发音的互动“情感雷达探测”印章，供用户进行趣味消遣。

**我们不搞概念，不卖课，只写能跑起来的代码。**

欢迎 Star、Fork、提 Issue，一起让这个项目变得更好用。

核心特性：
- **极简自适应交互**：提供毛玻璃质感的深色玻璃拟态自适应 Web 界面，高度适配移动端 H5 微信浏览器与 PC 体验。
- **情意指数探测雷达印章 (Radar Stamp)**：基于前端 Web Audio API 动态合成清脆的雷达探测或声呐碰击声效，点击印章即可累积雷达扫描次数并伴随渐隐上升动画。
- **五大沟通分析流派**：
  - **情感亲密度评估**：侧重分析字里行间的温差，评估当前双向亲密关联深度。
  - **潜台词深度剖析**：主攻对短促、冷淡敷衍或撤回等模糊行为背后的深层心理意图进行精细翻译。
  - **关系话术诊断书**：指出沟通中低情商、过度索取、道德绑架或自我感动的致命聊天失误。
  - **破局行动指南针**：为目前陷入冷场、尴尬、僵化或聊不下去的局势，制定可立刻操作的话题重置与线下邀约推进指南。
  - **舔狗红线预警机**：对极度卑微、单方面输出、高黏度高频倒贴式投资进行红线预警。
- **AI 情感科学度看板**：自动提取 AI 回复中的共识数据，以表格/单轨进度条在前端直观展示双向亲密度、话权平衡度、潜台词暴露度、关系红线风险及未来推进潜力。
- **演示案例与分享卡片**：内置 30 条不同聊天情境的聊天记录分析与情感诊断精彩演示样例，并支持一键卡片化截图分享。
- **一键零成本部署**：纯前端静态网页结构，支持零成本部署于 Vercel、GitHub Pages 或 CDN/OSS 静态托管服务。
- **安全开发代理**：本地开发支持使用个人 API 密钥发起代理请求，密钥由 Vite 服务器中转，无需担心前端泄露。
- **裂变解锁与留存**：内置微信朋友圈扫码分享拦截与额度重置机制，提升流量转化与留存。

## 快速启动

### 1. 克隆项目
```bash
git clone https://github.com/WT-Agent/ai-ltjl.git
cd ai-ltjl
```

### 2. 安装依赖
项目强制使用 pnpm 作为包管理器：
```bash
pnpm install
```

### 3. 配置本地开发环境变量
复制并修改环境变量配置文件：
```bash
cp .env.example .env
```
根据微应用的功能类型，在 `.env` 中配置您的开发者密钥：
- `DEEPSEEK_API_KEY`: 您的 DeepSeek 开发者 API 密钥（用于文本生成任务）
- `DASHSCOPE_API_KEY`: 您的通义千问/通义万相开发者 API 密钥（用于多模态与生图任务）

### 4. 启动本地开发服务
```bash
pnpm dev
```
启动成功后在浏览器访问控制台输出的地址即可。

### 5. 生产构建打包
```bash
pnpm build
```
打包后生成的 `dist` 目录即为纯静态网页资源，可直接上传部署。

## 脚手架集成说明

本模板由私有总控仓库 `ai.wuxian.xyz` 中的 `@wuxian/cli` 脚手架统一管理，支持以下批量运维操作：

### 初始化或更新单个子项目

```bash
node bin/cli.js ai-ltjl
```

脚手架将自动：
1. 读取子仓库的 `README.md` 首行作为 Prompt 主题。
2. 注入 Vue 3 静态页面结构及标准配置文件。
3. 保留原有的 `.git` 配置与 `README.md`，不覆盖个性化内容。

### 批量同步所有子项目

```bash
node bin/cli.js all
```

将模板的最新变更（如 SSO 逻辑、额度控制）一键同步至全部 31 个子项目。

### Agent 配置维护接口

```bash
# 读取子项目配置
node bin/cli.js get ai-ltjl

# 写入/更新配置（支持热更新 prompt、model、title、temperature 等）
node bin/cli.js set ai-ltjl prompt "你是一个结合人际社会交往心理学家、聊天潜台词破译专家、亲密关系评估顾问以及幽默“舔狗/红线预警机”的智能聊天记录关系分析师..."
node bin/cli.js set ai-ltjl model deepseek-chat
```

## 联系方式

- GitHub Issues: [提交反馈](https://github.com/WT-Agent/ai-ltjl/issues)
- 邮箱: us@wuxian.xyz

## 打赏支持

如果本项目对您有帮助，欢迎请作者喝杯咖啡。您的支持是持续维护与更新的动力。

<div align="center">

**微信支付** | **支付宝**
:---:|:---:
<img src="./asset/tenpay.png" width="200" alt="微信支付"> | <img src="./asset/alipay.png" width="200" alt="支付宝">

</div>

## 版权与许可

本项目基于 MIT License 开源协议。

Copyright (c) 2026. All rights reserved.
