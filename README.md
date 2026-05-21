# 🏗️ Blueprint Quotation System - 图纸智能报价系统

一个完整的商用级 AI 驱动的建筑工程、装修、广告工程图纸自动识别和智能报价系统。用户只需上传 PDF 或 JPG 图纸，系统自动识别工程内容并生成精准的预算清单。

## ✨ 功能特性

### 核心功能
- 🤖 **AI 智能识别**：支持 Claude 3.5 Vision 识别 PDF/JPG 图纸
- 📊 **三大工程类型**：建筑工程、装修工程、广告工程
- 💰 **精准报价计算**：材料成本 + 工时费用 + 利润率
- 📈 **动态价格库**：集成长沙地区最新市场价（2024-2025年）
- 📄 **报价单生成**：自动生成 PDF/Excel 报价单
- 👥 **多用户系统**：支持用户注册、登录、报价历史
- 💳 **商用化**：内置付费模式、API 限额控制

## 🛠️ 技术栈

### 前端
- Next.js 14 + React 18
- TypeScript
- Tailwind CSS
- shadcn/ui 组件库

### 后端
- Node.js + Express
- TypeScript
- PostgreSQL
- Prisma ORM

### AI & 服务
- Claude 3.5 Vision API
- PDF.js（PDF 处理）

### 部署
- Vercel（前端）
- Railway / Heroku（后端）
- Supabase（数据库）

## 🚀 快速开始

### 前提条件
- Node.js >= 18.0
- npm 或 yarn
- PostgreSQL 数据库
- Claude API Key (https://console.anthropic.com)

### 本地开发

```bash
# 1. 克隆仓库
git clone https://github.com/zhouyifu21130-cmd/blueprint-quotation-system.git
cd blueprint-quotation-system

# 2. 安装依赖
npm install

# 3. 配置环境变量
cp .env.example .env.local
# 编辑 .env.local，填入 API 密钥

# 4. 初始化数据库
npm run db:setup

# 5. 启动开发服务器
npm run dev

# 访问 http://localhost:3000
```

## 📁 项目结构

```
blueprint-quotation-system/
├── frontend/                  # Next.js 前端应用
│   ├── app/                   # App Router 页面
│   ├── components/            # React 组件
│   ├── lib/                   # 工具函数
│   └── public/                # 静态资源
├── backend/                   # Express 后端服务
│   ├── src/
│   │   ├── api/               # API 路由
│   │   ├── services/          # 业务逻辑
│   │   ├── middleware/        # 中间件
│   │   └── config/            # 配置文件
│   └── package.json
├── database/                  # 数据库配置
│   ├── schema.sql             # 数据表设计
│   ├── migrations/            # 迁移文件
│   └── seeds/                 # 种子数据（价格库）
├── docs/                      # 文档
│   ├── SETUP.md               # 安装指南
│   ├── API.md                 # API 文档
│   ├── DEPLOYMENT.md          # 部署指南
│   └── PRICING.md             # 商用化说明
└── docker-compose.yml         # Docker 配置
```

## 📖 文档

- [快速开始指南](./docs/SETUP.md)
- [API 文档](./docs/API.md)
- [部署指南](./docs/DEPLOYMENT.md)
- [商用化配置](./docs/PRICING.md)
- [价格库管理](./docs/PRICE-DATABASE.md)

## 💡 使用流程

1. **用户上传**：用户登录系统，上传 PDF 或 JPG 图纸
2. **AI 识别**：系统调用 Claude Vision API 识别图纸内容
3. **数据提取**：提取工程类型、材料清单、工程量
4. **价格查询**：从长沙价格库查询实时价格
5. **报价计算**：自动计算材料成本 + 工时费用
6. **报价单生成**：生成专业的 PDF/Excel 报价单
7. **历史管理**：用户可查看所有报价历史

## 💰 商用模式

### 免费版
- 每月 5 次报价生成
- 基础功能
- 标准报价单

### 专业版（$9.99/月）
- 每月 100 次报价生成
- 优先处理队列
- 自定义报价单
- 数据分析

### 企业版（联系销售）
- 无限次报价生成
- API 接口
- 专属支持

## 🔑 环境变量

```env
# 数据库
DATABASE_URL=postgresql://user:password@localhost:5432/blueprint_quotation

# Claude API
CLAUDE_API_KEY=sk-ant-...

# JWT
JWT_SECRET=your-secret-key-here

# 服务 URL
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_APP_URL=http://localhost:3000

# 支付（可选）
STRIPE_PUBLIC_KEY=pk_...
STRIPE_SECRET_KEY=sk_...
```

## 🔍 工程识别类型

### 建筑工程
- 结构识别：混凝土、钢筋、钢结构
- 材料提取：水泥、砂石、模板
- 工时估算：基础施工、主体施工

### 装修工程
- 室内识别：水电、防水、吊顶、地面
- 材料提取：瓷砖、木地板、油漆、门窗
- 工时估算：各工种工日数

### 广告工程
- 广告类型：LED 屏、喷绘、发光字
- 材料识别：屏幕规格、面积、工艺
- 安装工时：高度、复杂度、难度系数

## 📊 价格库来源

- 长沙市工程造价信息网
- 长沙市财政评审中心
- 当地建材市场实时采价
- 2024-2025 年官方标准

价格库每月更新一次。

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

MIT License

## 📞 支持

- 📧 Email: support@example.com
- 💬 Discord: [加入我们的社区]
- 🐛 Bug 报告: [GitHub Issues]

## 🎯 路线图

- [ ] 支持更多城市价格库
- [ ] 集成实时报价 API
- [ ] 移动端 App
- [ ] 高级 AI 训练（特定行业模型）
- [ ] Webhook 集成
- [ ] 多语言支持

---

**开发者**: zhouyifu21130-cmd  
**最后更新**: 2025-05-21  
**版本**: 1.0.0
