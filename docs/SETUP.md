# 🚀 快速开始指南

## 系统要求

- Node.js >= 18.0
- npm >= 9.0 或 yarn >= 3.0
- PostgreSQL >= 14.0
- Docker & Docker Compose（可选，推荐）

## 方案 A：本地开发（推荐新手）

### 1. 克隆仓库

```bash
git clone https://github.com/zhouyifu21130-cmd/blueprint-quotation-system.git
cd blueprint-quotation-system
```

### 2. 获取 Claude API Key

1. 访问 [Anthropic Console](https://console.anthropic.com/)
2. 登录或注册账户
3. 创建新的 API Key
4. 复制 Key（形如：`sk-ant-...`）

### 3. 创建环境变量文件

```bash
cp .env.example .env.local
```

编辑 `.env.local`，填入您的信息：

```env
CLAUDE_API_KEY=sk-ant-your_actual_key_here
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/blueprint_quotation
```

### 4. 安装依赖

```bash
# 安装根目录依赖
npm install

# 安装前端依赖
cd frontend && npm install && cd ..

# 安装后端依赖
cd backend && npm install && cd ..

# 安装数据库工具依赖
cd database && npm install && cd ..
```

### 5. 启动 PostgreSQL

**Windows (通过 WSL2):**
```bash
wsl
sudo service postgresql start
```

**macOS (通过 Homebrew):**
```bash
brew services start postgresql
```

**Linux:**
```bash
sudo service postgresql start
```

### 6. 初始化数据库

```bash
# 创建数据库
createdb -U postgres blueprint_quotation

# 导入 schema
psql -U postgres blueprint_quotation < database/schema.sql

# 导入价格库数据
node database/scripts/seed-prices.js
```

### 7. 启动开发服务器

```bash
# 从根目录运行
npm run dev

# 或分别运行
npm run dev:frontend  # 前端：http://localhost:3000
npm run dev:backend   # 后端：http://localhost:4000
```

✅ 完成！访问 http://localhost:3000

---

## 方案 B：Docker 快速启动（推荐）

### 1. 准备

```bash
# 克隆仓库
git clone https://github.com/zhouyifu21130-cmd/blueprint-quotation-system.git
cd blueprint-quotation-system

# 复制环境文件
cp .env.example .env.local

# 编辑 .env.local，至少填入 CLAUDE_API_KEY
```

### 2. 启动 Docker

```bash
# 构建并启动所有服务
docker-compose up --build

# 或后台运行
docker-compose up -d --build
```

### 3. 初始化数据库（第一次）

```bash
# 在另一个终端
docker-compose exec backend npm run db:seed
```

✅ 完成！访问：
- 前端：http://localhost:3000
- 后端 API：http://localhost:4000
- 数据库：localhost:5432

### 常用 Docker 命令

```bash
# 停止服务
docker-compose down

# 查看日志
docker-compose logs -f

# 只查看后端日志
docker-compose logs -f backend

# 重启某个服务
docker-compose restart backend

# 删除所有数据（谨慎）
docker-compose down -v
```

---

## 方案 C：云端部署（生产环境）

详见 [部署指南](./DEPLOYMENT.md)

---

## ✅ 验证安装

### 1. 检查前端

打开 http://localhost:3000，应该看到登录页面

### 2. 检查后端 API

```bash
curl http://localhost:4000/api/health

# 应该返回：
# {"status":"ok","version":"1.0.0"}
```

### 3. 检查数据库

```bash
psql -U postgres blueprint_quotation

# 连接成功后，查看表
\dt

# 应该看到 materials, prices, projects 等表
```

---

## 🐛 常见问题

### Q: 启动时报 "Cannot find module"

**A:** 重新安装依赖
```bash
rm -rf node_modules package-lock.json
npm install
```

### Q: PostgreSQL 连接失败

**A:** 检查数据库配置
```bash
# 测试连接
psql -U postgres -h localhost -d blueprint_quotation

# 或检查环境变量
echo $DATABASE_URL
```

### Q: Claude API Key 无效

**A:** 
1. 访问 https://console.anthropic.com/account/billing/overview
2. 确认账户有余额或订阅
3. 重新生成 API Key
4. 更新 .env.local

### Q: 端口已被占用

**A:** 修改环境变量或关闭占用端口的程序
```bash
# Linux/Mac 查找占用端口的进程
lsof -i :3000
lsof -i :4000

# 或在 docker-compose.yml 中修改映射端口
# ports:
#   - "3001:3000"  # 改为 3001
```

### Q: Docker 无法连接到数据库

**A:** 
```bash
# 检查 Docker 网络
docker-compose down -v
docker-compose up --build

# 强制重建
docker-compose build --no-cache
```

---

## 📚 下一步

- [API 文档](./API.md) - 了解 API 端点
- [部署指南](./DEPLOYMENT.md) - 部署到生产环境
- [商用化配置](./PRICING.md) - 设置付费模式
- [价格库管理](./PRICE-DATABASE.md) - 更新价格数据

---

**需要帮助？** 提交 Issue 或联系支持：support@example.com
