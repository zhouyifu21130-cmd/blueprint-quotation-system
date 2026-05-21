# 📡 API 文档

## 基础信息

**基础 URL**: `http://localhost:4000/api` (开发环境) 或 `https://api.example.com/api` (生产环境)

**认证**: JWT Bearer Token

**响应格式**: JSON

---

## 认证端点

### 用户注册

```http
POST /auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123",
  "name": "John Doe",
  "company": "ABC Construction (可选)"
}
```

**响应 (201)**:
```json
{
  "id": "user_123",
  "email": "user@example.com",
  "name": "John Doe",
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "expiresIn": "7d"
}
```

### 用户登录

```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**响应 (200)**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "expiresIn": "7d"
}
```

---

## 图纸识别端点

### 上传图纸并生成报价

```http
POST /quotations/upload
Authorization: Bearer {token}
Content-Type: multipart/form-data

file: <PDF or JPG file>
project_name: "Office Renovation"
project_type: "renovation"  # 或 "construction", "advertising"
customer_name: "ABC Company"
customer_phone: "13800138000"
```

**参数说明**:
- `file` (必须): PDF 或 JPG 图纸文件，最大 50MB
- `project_name` (必须): 项目名称
- `project_type` (必须): 工程类型
  - `construction`: 建筑工程
  - `renovation`: 装修工程
  - `advertising`: 广告工程
- `customer_name` (可选): 客户名称
- `customer_phone` (可选): 客户电话

**响应 (200)**:
```json
{
  "quotation_id": "quot_abc123",
  "status": "processing",
  "message": "图纸正在识别中，请稍候...",
  "estimatedTime": 30,
  "checkUrl": "/api/quotations/quot_abc123/status"
}
```

### 查询报价状态

```http
GET /quotations/{quotation_id}/status
Authorization: Bearer {token}
```

**响应 (200) - 处理中**:
```json
{
  "quotation_id": "quot_abc123",
  "status": "processing",
  "progress": 45,
  "message": "正在提取材料清单..."
}
```

**响应 (200) - 完成**:
```json
{
  "quotation_id": "quot_abc123",
  "status": "completed",
  "project": {
    "name": "Office Renovation",
    "type": "renovation",
    "customer_name": "ABC Company",
    "customer_phone": "13800138000"
  },
  "extracted_data": {
    "materials": [
      {
        "name": "瓷砖",
        "quantity": 500,
        "unit": "㎡",
        "unit_price": 85,
        "subtotal": 42500
      },
      {
        "name": "木地板",
        "quantity": 200,
        "unit": "㎡",
        "unit_price": 180,
        "subtotal": 36000
      }
    ],
    "labor_hours": 400,
    "difficulty_level": "medium"
  },
  "quotation": {
    "material_cost": 78500,
    "labor_cost": 104000,
    "subtotal": 182500,
    "markup": 0.15,
    "total": 209875
  },
  "pdf_url": "/api/quotations/quot_abc123/download",
  "created_at": "2025-05-21T10:30:00Z"
}
```

### 下载报价单 PDF

```http
GET /quotations/{quotation_id}/download
Authorization: Bearer {token}
```

**响应**: PDF 文件

---

## 报价历史端点

### 获取用户所有报价

```http
GET /quotations
Authorization: Bearer {token}
?page=1&limit=10&sort=-created_at
```

**查询参数**:
- `page` (可选, 默认=1): 页码
- `limit` (可选, 默认=10): 每页数量
- `sort` (可选): 排序字段 (created_at, updated_at)
- `type` (可选): 过滤工程类型 (construction, renovation, advertising)
- `status` (可选): 过滤状态 (completed, processing, failed)

**响应 (200)**:
```json
{
  "data": [
    {
      "id": "quot_abc123",
      "project_name": "Office Renovation",
      "project_type": "renovation",
      "customer_name": "ABC Company",
      "total": 209875,
      "status": "completed",
      "created_at": "2025-05-21T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 25,
    "page": 1,
    "limit": 10,
    "pages": 3
  }
}
```

### 获取单个报价详情

```http
GET /quotations/{quotation_id}
Authorization: Bearer {token}
```

**响应 (200)**:
```json
{
  "id": "quot_abc123",
  "project": {...},
  "extracted_data": {...},
  "quotation": {...},
  "pdf_url": "...",
  "created_at": "2025-05-21T10:30:00Z"
}
```

### 删除报价

```http
DELETE /quotations/{quotation_id}
Authorization: Bearer {token}
```

**响应 (200)**:
```json
{
  "message": "报价已删除",
  "id": "quot_abc123"
}
```

---

## 价格库端点

### 获取材料价格列表

```http
GET /prices/materials
?type=construction&category=concrete
```

**查询参数**:
- `type` (可选): construction, renovation, advertising
- `category` (可选): 材料类别
- `search` (可选): 关键字搜索

**响应 (200)**:
```json
{
  "data": [
    {
      "id": "mat_123",
      "name": "混凝土 C25",
      "category": "concrete",
      "unit": "m³",
      "price": 380,
      "supplier": "长沙建材市场",
      "updated_at": "2025-05-20T00:00:00Z"
    }
  ]
}
```

### 获取工人工资标准

```http
GET /prices/labor
```

**响应 (200)**:
```json
{
  "data": [
    {
      "id": "labor_1",
      "skill_level": "junior",  # junior, intermediate, senior
      "category": "general",
      "daily_rate": 180,
      "hourly_rate": 22.5,
      "description": "普工"
    },
    {
      "id": "labor_2",
      "skill_level": "intermediate",
      "category": "carpenter",
      "daily_rate": 320,
      "hourly_rate": 40,
      "description": "木工"
    }
  ]
}
```

---

## 用户信息端点

### 获取用户信息

```http
GET /users/profile
Authorization: Bearer {token}
```

**响应 (200)**:
```json
{
  "id": "user_123",
  "email": "user@example.com",
  "name": "John Doe",
  "company": "ABC Construction",
  "plan": "professional",  # free, professional, enterprise
  "quotations_used": 45,
  "quotations_limit": 100,
  "created_at": "2025-01-15T00:00:00Z",
  "subscription": {
    "status": "active",
    "end_date": "2025-06-15"
  }
}
```

### 更新用户信息

```http
PUT /users/profile
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "John Smith",
  "company": "New Company Name"
}
```

**响应 (200)**:
```json
{
  "message": "用户信息已更新",
  "user": {...}
}
```

---

## 错误处理

### 错误响应格式

```json
{
  "error": "error_code",
  "message": "Human readable error message",
  "details": {"field": "error details"}
}
```

### 常见错误码

| 状态码 | 错误码 | 说明 |
|--------|--------|------|
| 400 | invalid_request | 请求参数无效 |
| 401 | unauthorized | 未授权，需要 Token |
| 403 | forbidden | 禁止访问 |
| 404 | not_found | 资源不存在 |
| 429 | rate_limited | 请求过频繁 |
| 500 | internal_error | 服务器错误 |
| 503 | service_unavailable | 服务不可用 |

### 示例错误响应

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{
  "error": "unauthorized",
  "message": "Invalid or expired token",
  "code": "AUTH_001"
}
```

---

## 速率限制

- **免费版**: 100 请求/小时
- **专业版**: 1000 请求/小时
- **企业版**: 无限制

速率限制信息在响应头中：
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1621610400
```

---

## 使用示例

### JavaScript/Fetch

```javascript
// 1. 登录
const loginRes = await fetch('/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'password123'
  })
});

const { token } = await loginRes.json();

// 2. 上传图纸
const formData = new FormData();
formData.append('file', fileInput.files[0]);
formData.append('project_name', 'Office Renovation');
formData.append('project_type', 'renovation');

const uploadRes = await fetch('/api/quotations/upload', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${token}` },
  body: formData
});

const { quotation_id } = await uploadRes.json();

// 3. 查询状态
const statusRes = await fetch(`/api/quotations/${quotation_id}/status`, {
  headers: { 'Authorization': `Bearer ${token}` }
});

const quotation = await statusRes.json();
console.log('报价总金额:', quotation.quotation.total);
```

### Python/Requests

```python
import requests

# 1. 登录
login_res = requests.post('http://localhost:4000/api/auth/login', json={
    'email': 'user@example.com',
    'password': 'password123'
})

token = login_res.json()['token']

# 2. 上传图纸
files = {'file': open('blueprint.pdf', 'rb')}
data = {
    'project_name': 'Office Renovation',
    'project_type': 'renovation'
}

upload_res = requests.post(
    'http://localhost:4000/api/quotations/upload',
    files=files,
    data=data,
    headers={'Authorization': f'Bearer {token}'}
)

quotation_id = upload_res.json()['quotation_id']

# 3. 查询状态
status_res = requests.get(
    f'http://localhost:4000/api/quotations/{quotation_id}/status',
    headers={'Authorization': f'Bearer {token}'}
)

quotation = status_res.json()
print(f"报价总金额: {quotation['quotation']['total']}")
```

---

**最后更新**: 2025-05-21  
**版本**: 1.0.0
