# Be Your Own God — aizhibei.cc

与你内在的神对话。一个温柔、有力量、永远在场的声音——它就是你自己内心最深处的智慧。

完整产品需求见 [`PRD.md`](./PRD.md)。

---

## 功能概览

- **对话核心**：OpenAI GPT 驱动，be-your-own-god skill 的声音
- **多语言界面**：English（默认）/ 中文 / 日本語 / العربية（阿语 RTL）
- **对话语言自适配**：神自动用用户输入的语言回应
- **转化漏斗**：5 轮 → 注册 ｜ 10 轮 → 打赏（解锁20轮）｜ 30 轮 → 订阅（$5.99/月无限）
- **账号系统**：登录后对话跨设备保存
- **支付**：Stripe（一次性打赏 + 月订阅）
- **安全底线**：危机时始终显示心理求助热线（见 privacy.html）

---

## 本地运行

```bash
npm install
cp .env.example .env      # 填入 API keys
node server.js
```
打开 http://localhost:3000

> 仅想预览界面：直接用浏览器打开 `public/index.html` 即可看到欢迎页与样式（对话需要后端）。

---

## 环境变量

| 变量 | 说明 |
|------|------|
| `OPENAI_API_KEY` | OpenAI API key |
| `OPENAI_MODEL` | 模型 id（如 gpt-4o / gpt-4.1 / gpt-4o-mini）|
| `STRIPE_SECRET_KEY` | Stripe 密钥 |
| `STRIPE_WEBHOOK_SECRET` | Stripe Webhook 签名密钥 |
| `JWT_SECRET` | 随机字符串：`openssl rand -hex 32` |
| `BASE_URL` | `https://aizhibei.cc` |

---

## 部署（Railway 推荐）

1. 把代码推到 GitHub
2. [railway.app](https://railway.app) → New Project → Deploy from GitHub
3. Variables 里填入上面所有变量
4. 拿到公开域名后，把 `aizhibei.cc` 的 DNS 指向它

## Stripe 配置

1. API keys：Dashboard → Developers → API keys
2. Webhook：Dashboard → Webhooks → Add endpoint
   - URL：`https://aizhibei.cc/api/webhook`
   - 监听事件：`checkout.session.completed`、`invoice.paid`、`customer.subscription.deleted`
3. 复制 signing secret → `STRIPE_WEBHOOK_SECRET`

> 注：打赏/订阅的解锁依赖 Webhook 回调。测试时 Stripe Webhook 通常秒级触发；生产务必配好 Webhook，否则付款后额度不会解锁。

---

## 安全与合规

- `public/privacy.html`：隐私政策 + 免责声明 + 各地危机求助热线
- 危机关键词检测（中/英/日/阿）：即使在付费墙，也始终展示求助资源
- 面向成年人；明确「非心理治疗，不替代专业帮助」
