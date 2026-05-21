# 🚀 部署指南

本指南涵盖了三种主流部署方案：Vercel + Railway（推荐）、Docker 自主部署、以及国内云服务。

---

## 方案 A：Vercel + Railway + Supabase（⭐ 推荐）

这是最简单、最经济的生产部署方案。

### 优点
- ✅ 前端自动部署（Vercel）
- ✅ 后端自动扩展（Railway）
- ✅ 数据库托管（Supabase）
- ✅ 全球 CDN
- ✅ 免费配额足以应对初期需求

### 成本估算
| 服务 | 免费额度 | 超出后 |
|-----|--------|-------|
| Vercel | 无限制 | $20/月 |
| Railway | $5/月 | $0.10/小时 |
| Supabase | 500MB | $25/月 |
| Claude API | 按使用 | $0.001-0.003/次 |
| **总计** | **~$5/月** | **$50-100/月** |

### 步骤 1：准备 GitHub 仓库

```bash
# 确保所有代码都已提交
git add .
git commit -m "Ready for deployment"
git push origin main
```

### 步骤 2：创建 Supabase 数据库

1. 访问 [Supabase](https://supabase.com/)
2. 点击 "New Project"
3. 选择地区（推荐新加坡以获得最低延迟）
4. 设置数据库密码
5. 等待创建完成
6. 在 Settings → Database → Connection String 获取 URL

```bash
# 格式为
psql "postgresql://postgres:[PASSWORD]@[HOST]:[PORT]/postgres"
```

### 步骤 3：初始化 Supabase 数据库

```bash
# 连接到 Supabase
psql "postgresql://postgres:[PASSWORD]@[HOST]:[PORT]/postgres"

# 然后执行
\i database/schema.sql
\i database/seeds/prices.sql
```

或使用 Supabase SQL Editor：
1. 打开 Supabase Dashboard
2. 点击 "SQL Editor"
3. 新建 Query
4. 粘贴 `database/schema.sql` 内容
5. 执行
6. 重复粘贴 `database/seeds/prices.sql`

### 步骤 4：部署前端到 Vercel

1. 访问 [Vercel](https://vercel.com/)
2. 点击 "New Project"
3. 选择 GitHub 仓库
4. 在 "Import Git Repository" 中搜索 `blueprint-quotation-system`
5. 点击 Import
6. 配置环境变量：
   - `NEXT_PUBLIC_API_URL` = `https://your-backend.railway.app`（待部署后更新）
7. 点击 Deploy

✅ Vercel 自动构建并部署，完成后会给您一个 URL（如 `https://blueprint-quotation-system.vercel.app`）

### 步骤 5：部署后端到 Railway

1. 访问 [Railway](https://railway.app/)
2. 点击 "New Project"
3. 选择 "GitHub Repo"
4. 连接您的 GitHub 账户
5. 选择 `blueprint-quotation-system` 仓库
6. Railway 会自动检测项目结构
7. 添加环境变量：
   ```
   DATABASE_URL=postgresql://postgres:[PASSWORD]@[HOST]:[PORT]/postgres
   CLAUDE_API_KEY=sk-ant-your_key_here
   JWT_SECRET=your_random_secret_key
   NODE_ENV=production
   ```
8. 点击 Deploy

✅ Railway 自动构建并部署，完成后会给您一个 URL（如 `https://your-backend-production.up.railway.app`）

### 步骤 6：更新前端环境变量

1. 返回 Vercel Dashboard
2. 打开您的项目
3. 点击 Settings → Environment Variables
4. 更新 `NEXT_PUBLIC_API_URL`：
   ```
   https://your-backend-production.up.railway.app
   ```
5. 点击 Save
6. 等待自动重新部署

✅ 完成！您的应用现在已上线。

### 验证部署

```bash
# 测试前端
curl https://your-app.vercel.app

# 测试后端
curl https://your-backend-production.up.railway.app/api/health

# 应该返回
# {"status":"ok","version":"1.0.0"}
```

---

## 方案 B：Docker + 自主服务器

适合希望完全控制的用户。

### 前置条件
- VPS 或云服务器（1GB RAM 以上）
- SSH 访问权限
- Docker 和 Docker Compose

### 步骤 1：配置服务器

```bash
# SSH 登录到服务器
ssh root@your_server_ip

# 更新系统
apt-get update && apt-get upgrade -y

# 安装 Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# 安装 Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 验证
docker --version
docker-compose --version
```

### 步骤 2：克隆和配置项目

```bash
# 克隆仓库
git clone https://github.com/zhouyifu21130-cmd/blueprint-quotation-system.git
cd blueprint-quotation-system

# 创建生产环境文件
cp .env.example .env.production

# 编辑配置
nano .env.production

# 设置以下值
# CLAUDE_API_KEY=sk-ant-your_key
# JWT_SECRET=your_random_secret_key
# NODE_ENV=production
```

### 步骤 3：使用 Nginx 反向代理

创建 `docker-compose.prod.yml`：

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend

  frontend:
    build:
      context: ./frontend
    environment:
      NEXT_PUBLIC_API_URL=https://api.yourdomain.com

  backend:
    build:
      context: ./backend
    env_file: .env.production
    depends_on:
      - postgres

  postgres:
    image: postgres:16-alpine
    env_file: .env.production
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 步骤 4：配置 SSL 证书（Let's Encrypt）

```bash
# 安装 Certbot
apt-get install certbot python3-certbot-nginx -y

# 生成证书
certbot certonly --standalone -d yourdomain.com -d api.yourdomain.com

# 证书保存在 /etc/letsencrypt/live/
```

### 步骤 5：启动应用

```bash
# 后台启动
docker-compose -f docker-compose.prod.yml up -d

# 查看日志
docker-compose -f docker-compose.prod.yml logs -f

# 初始化数据库
docker-compose -f docker-compose.prod.yml exec backend npm run db:seed
```

✅ 应用已部署。访问 `https://yourdomain.com`

---

## 方案 C：国内云服务（阿里云/腾讯云）

### 阿里云 ECS + RDS

**成本**: ¥50-200/月

```bash
# 创建 ECS 实例（2GB 内存，1Mbps 带宽）
# 选择 Ubuntu 20.04 LTS

# SSH 后安装 Docker（同方案 B）

# 创建 RDS PostgreSQL 实例
# 选择 2GB 内存，100GB 存储

# 获取 RDS 连接地址，更新 .env
```

### 腾讯云轻量应用服务器

**成本**: ¥40-100/月

```bash
# 购买轻量应用服务器
# 选择 Ubuntu 20.04 LTS，2GB 内存

# SSH 后执行同方案 B 步骤
```

---

## 监控和维护

### 设置监控

```bash
# 后端健康检查
curl -I https://api.yourdomain.com/api/health

# 数据库备份
docker-compose exec postgres pg_dump -U postgres blueprint_quotation > backup.sql
```

### 日志管理

```bash
# 查看实时日志
docker-compose logs -f

# 查看特定服务
docker-compose logs -f backend

# 导出日志
docker-compose logs backend > backend.log
```

### 更新应用

```bash
# 拉取最新代码
git pull origin main

# 重新构建和部署
docker-compose up -d --build
```

---

## 性能优化

### 数据库索引

```sql
-- 为常用查询字段创建索引
CREATE INDEX idx_quotations_user_id ON quotations(user_id);
CREATE INDEX idx_quotations_created_at ON quotations(created_at);
CREATE INDEX idx_prices_material_id ON prices(material_id);
```

### CDN 配置

在 `frontend/next.config.js`：

```javascript
module.exports = {
  images: {
    domains: ['cdn.yourdomain.com'],
  },
  // 启用压缩
  compress: true,
  // 启用 SWR 缓存
  swcMinify: true,
};
```

### 缓存策略

```javascript
// 后端缓存
app.use(compression());
app.use(express.static('public', {
  maxAge: '1d',
  etag: false
}));
```

---

## 备份和恢复

### 自动备份脚本

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# 备份数据库
docker-compose exec -T postgres pg_dump -U postgres blueprint_quotation > "$BACKUP_DIR/db_$DATE.sql"

# 备份文件
tar -czf "$BACKUP_DIR/files_$DATE.tar.gz" /data/uploads

# 删除 7 天前的备份
find $BACKUP_DIR -type f -mtime +7 -delete

echo "Backup completed: $DATE"
```

添加到 crontab：
```bash
crontab -e
# 每天凌晨 2 点执行
0 2 * * * /path/to/backup.sh
```

---

## 常见问题

**Q: 部署后数据库连接失败**

A: 检查 DATABASE_URL 格式和防火墙规则

**Q: Claude API 请求超时**

A: 增加超时时间，或使用异步队列

**Q: 存储空间不足**

A: 定期清理旧文件，或升级存储

---

**需要帮助？** 联系：support@example.com
