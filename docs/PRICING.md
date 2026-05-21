# 💳 商用化和定价配置

本指南说明如何为您的应用配置商用模式。

## 定价方案设计

### 方案对比

| 功能 | 免费版 | 专业版 | 企业版 |
|-----|-------|--------|--------|
| 月报价次数 | 5 | 100 | 无限 |
| 项目数量 | 5 | 无限 | 无限 |
| 报价单自定义 | ❌ | ✅ | ✅ |
| 数据分析 | ❌ | ✅ | ✅ |
| API 接口 | ❌ | ❌ | ✅ |
| 优先支持 | ❌ | ✅ | ✅ |
| 价格 | 免费 | ¥99/月 | 联系销售 |

## 集成支付系统

### 1. 使用 Stripe（国际）

```bash
# 安装 Stripe SDK
npm install stripe
```

**后端配置**:

```typescript
// backend/src/services/stripe.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-04-10',
});

export async function createCheckoutSession(userId: string, plan: 'professional' | 'enterprise') {
  const prices = {
    professional: 'price_professional_monthly',
    enterprise: 'price_enterprise_monthly',
  };

  const session = await stripe.checkout.sessions.create({
    payment_method_types: ['card'],
    line_items: [
      {
        price: prices[plan],
        quantity: 1,
      },
    ],
    mode: 'subscription',
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/billing/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/billing/cancel`,
    client_reference_id: userId,
  });

  return session;
}

export async function handleWebhook(event: Stripe.Event) {
  switch (event.type) {
    case 'customer.subscription.updated':
      // 更新用户订阅状态
      break;
    case 'customer.subscription.deleted':
      // 处理订阅取消
      break;
  }
}
```

**前端集成**:

```typescript
// frontend/components/PricingCard.tsx
import { loadStripe } from '@stripe/js';

export function PricingCard({ plan }: { plan: 'professional' }) {
  const handleSubscribe = async () => {
    const response = await fetch('/api/billing/create-checkout-session', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ plan }),
    });

    const { sessionId } = await response.json();
    const stripe = await loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLIC_KEY!);
    
    await stripe?.redirectToCheckout({ sessionId });
  };

  return (
    <div>
      <h3>专业版</h3>
      <p>¥99/月</p>
      <button onClick={handleSubscribe}>立即订阅</button>
    </div>
  );
}
```

### 2. 使用支付宝（推荐国内）

```bash
npm install alipay-sdk
```

**后端配置**:

```typescript
// backend/src/services/alipay.ts
import AlipaySdk from 'alipay-sdk';

const alipaySdk = new AlipaySdk({
  appId: process.env.ALIPAY_APP_ID,
  privateKey: process.env.ALIPAY_PRIVATE_KEY,
  alipayPublicKey: process.env.ALIPAY_PUBLIC_KEY,
});

export async function createPaymentOrder(userId: string, plan: string) {
  const prices = {
    professional: 9900, // 99元，单位：分
    enterprise: 29900,
  };

  const result = await alipaySdk.exec('alipay.trade.app.pay', {
    bizContent: JSON.stringify({
      out_trade_no: `${userId}_${Date.now()}`,
      total_amount: prices[plan as keyof typeof prices] / 100,
      subject: `Blueprint Quotation System - ${plan}`,
      product_code: 'QUICK_MSECURITY_PAY',
    }),
  });

  return result;
}
```

## 用户限额控制

```typescript
// backend/src/middleware/rateLimiter.ts
import rateLimit from 'express-rate-limit';

const planLimits = {
  free: 5,      // 每月 5 次
  professional: 100,
  enterprise: 99999,
};

export async function checkQuotationLimit(userId: string) {
  const user = await User.findById(userId);
  const plan = user.subscription?.plan || 'free';
  const limit = planLimits[plan as keyof typeof planLimits];

  const monthStart = new Date();
  monthStart.setDate(1);
  monthStart.setHours(0, 0, 0, 0);

  const count = await Quotation.countDocuments({
    user_id: userId,
    created_at: { $gte: monthStart },
  });

  if (count >= limit) {
    throw new Error(`Monthly limit of ${limit} quotations reached`);
  }

  return true;
}
```

## 数据库扩展

```sql
-- 添加订阅表
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  plan VARCHAR(50) NOT NULL,  -- free, professional, enterprise
  status VARCHAR(50) NOT NULL DEFAULT 'active',  -- active, canceled, expired
  stripe_subscription_id VARCHAR(255),
  alipay_trade_id VARCHAR(255),
  start_date TIMESTAMP NOT NULL DEFAULT NOW(),
  end_date TIMESTAMP,
  auto_renewal BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- 添加配额表
CREATE TABLE quotas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  quotations_used INT DEFAULT 0,
  quotations_limit INT NOT NULL,
  month_start DATE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- 添加发票表
CREATE TABLE invoices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  subscription_id UUID REFERENCES subscriptions(id),
  amount DECIMAL(10, 2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'CNY',
  status VARCHAR(50) NOT NULL DEFAULT 'pending',  -- pending, paid, cancelled
  invoice_number VARCHAR(50) UNIQUE,
  payment_date TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

## 账单管理页面

```typescript
// frontend/app/billing/page.tsx
export default function BillingPage() {
  const [plan, setPlan] = useState('free');
  const [invoices, setInvoices] = useState([]);

  useEffect(() => {
    // 获取用户订阅状态
    fetch('/api/billing/subscription').then(r => r.json()).then(data => {
      setPlan(data.plan);
    });

    // 获取发票列表
    fetch('/api/billing/invoices').then(r => r.json()).then(data => {
      setInvoices(data);
    });
  }, []);

  return (
    <div>
      <h1>账单中心</h1>
      
      <div className="current-plan">
        <p>当前计划: <strong>{plan}</strong></p>
        <button onClick={() => navigate('/billing/upgrade')}>升级</button>
      </div>

      <div className="invoices">
        <h2>发票记录</h2>
        {invoices.map(invoice => (
          <div key={invoice.id}>
            <p>{invoice.invoice_number} - ¥{invoice.amount}</p>
            <p>状态: {invoice.status}</p>
            <a href={`/api/billing/invoices/${invoice.id}/pdf`}>下载</a>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 合同和 T&C

创建文件 `docs/TERMS_OF_SERVICE.md` 和 `docs/PRIVACY_POLICY.md`

## 营销和分析

```typescript
// 追踪转换
import { analytics } from '@/lib/analytics';

export function trackConversion(plan: string) {
  analytics.track('plan_purchased', {
    plan,
    timestamp: new Date(),
  });
}

// 计算 MRR（月度经常性收入）
export async function calculateMRR() {
  const activeSubscriptions = await Subscription.find({
    status: 'active',
  }).populate('user');

  const mrr = activeSubscriptions.reduce((total, sub) => {
    const prices = { professional: 99, enterprise: 299 };
    return total + prices[sub.plan as keyof typeof prices];
  }, 0);

  return mrr;
}
```

## 营销复制

### 销售主张

**标题**: "5 分钟内从图纸到报价 - AI 驱动的报价系统"

**副标题**: "建筑、装修和广告工程的精准报价。无需手动计算。"

**关键优点**:
1. 🤖 AI 自动识别图纸内容
2. ⚡ 5 分钟内生成报价
3. 📊 基于长沙实时市场价
4. 💰 3 倍提高报价效率
5. 📄 专业报价单自动生成

---

**更新**: 2025-05-21
