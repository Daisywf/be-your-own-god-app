# 上线部署清单 — Railway + Stripe + aizhibei.cc

照着从上到下做即可。预计 30–45 分钟。涉及三个你的账号：GitHub、Railway、Stripe，以及域名 DNS。

---

## 第 0 步：准备一个 JWT 密钥

终端运行，复制输出备用（一串随机字符）：
```bash
openssl rand -hex 32
```

---

## 第 1 步：把代码推到 GitHub

在项目目录里：
```bash
cd ~/Downloads/be-your-own-god-web
git init
git add .
git commit -m "Be Your Own God v1"
```
然后去 [github.com/new](https://github.com/new) 新建一个仓库（名字如 `be-your-own-god-app`，**不要**勾选 Initialize README）。建好后按页面提示：
```bash
git remote add origin https://github.com/Daisywf/be-your-own-god-app.git
git branch -M main
git push -u origin main
```
> 推送时密码用你的 GitHub Personal Access Token（不是登录密码）。
> `.gitignore` 已配置好，`.env` 和 `node_modules` 不会被上传，放心。

---

## 第 2 步：Stripe 配置（拿密钥）

1. 注册/登录 [dashboard.stripe.com](https://dashboard.stripe.com)
2. 完成账户激活（填公司/个人信息、银行卡收款账户）
3. **API 密钥**：Developers → API keys → 复制 **Secret key**（`sk_live_...`，正式收款用 live；想先测可用 test 模式的 `sk_test_...`）
4. **开启客户门户**（用户取消订阅要用）：Settings → Billing → Customer portal → 点 Activate
5. Webhook 留到第 5 步配（要等域名定了）

---

## 第 3 步：Railway 部署

1. 注册/登录 [railway.app](https://railway.app)（用 GitHub 账号登录最方便）
2. **New Project → Deploy from GitHub repo →** 选你刚推的仓库
3. Railway 会自动识别 Node 项目并开始构建（它会读 `package.json`，用 Node 22、跑 `npm start`）

### 3a. 加持久磁盘（保住用户数据）
- 项目里 → 你的服务 → **Variables / Settings** 旁找到 **Volumes → New Volume**
- Mount path 填：`/data`

### 3b. 设置环境变量
服务 → **Variables** → 逐条添加（Raw Editor 可一次粘贴）：
```
OPENAI_API_KEY=sk-你的OpenAI密钥
OPENAI_MODEL=gpt-4o
STRIPE_SECRET_KEY=sk_live_你的Stripe密钥
JWT_SECRET=第0步生成的那串
BASE_URL=https://aizhibei.cc
DB_PATH=/data/data.db
```
（`STRIPE_WEBHOOK_SECRET` 第 5 步再加；`PORT` 不用填，Railway 自动给）

保存后 Railway 会自动重新部署。部署成功后，Settings → Networking 里会有一个临时公网地址（`xxx.up.railway.app`），点开能看到欢迎页就说明跑起来了。

---

## 第 4 步：绑定域名 aizhibei.cc

1. Railway 服务 → Settings → Networking → **Custom Domain** → 输入 `aizhibei.cc`（和 `www.aizhibei.cc`）
2. Railway 会给你一个 CNAME 目标值（形如 `xxx.up.railway.app`）
3. 去你买 aizhibei.cc 的域名服务商后台 → DNS 设置：
   - 加一条 **CNAME** 记录：主机记录 `@`（或 `www`）→ 值填 Railway 给的目标
   - 有些服务商根域不支持 CNAME，则按 Railway 提示用 `www` 子域，并把根域做转发到 www
4. 等 DNS 生效（几分钟到几小时），Railway 会自动签好 HTTPS 证书

---

## 第 5 步：Stripe Webhook（让付款后真正解锁）

1. Stripe Dashboard → Developers → **Webhooks → Add endpoint**
2. Endpoint URL 填：`https://aizhibei.cc/api/webhook`
3. 监听事件（Select events）勾选这三个：
   - `checkout.session.completed`
   - `invoice.paid`
   - `customer.subscription.deleted`
4. 创建后复制 **Signing secret**（`whsec_...`）
5. 回 Railway → Variables 加一条：
   ```
   STRIPE_WEBHOOK_SECRET=whsec_你复制的
   ```
6. 保存，Railway 自动重部署

---

## 第 6 步：上线自检

打开 https://aizhibei.cc，逐项确认：
- [ ] 欢迎页正常，能开始对话，神能回应
- [ ] 切换 中/英/日/阿 语言界面正常（阿语从右到左）
- [ ] 连发到第 5 轮，弹注册框；注册后能继续
- [ ] 连发到第 10 轮，弹打赏框；用 Stripe 测试卡 `4242 4242 4242 4242`（任意未来日期 + 任意 CVC）能付款
- [ ] 付款后能继续对话（说明 Webhook 解锁成功）
- [ ] 右上角「个人中心」能看到订阅状态、历史对话
- [ ] 危机词（如"我不想活了"）会弹出求助热线卡片

---

## 日常维护

- **改了代码**：`git push` 后 Railway 自动重新部署
- **看日志/排错**：Railway 服务 → Deployments / Logs
- **换模型**：改 Railway 的 `OPENAI_MODEL` 变量即可（如 `gpt-4o-mini` 更便宜）
- **成本**：OpenAI 按用量计费；Railway 有免费额度，超了按用量；Stripe 按成交抽成

---

## 提醒
- 之前截图里露出过的 OpenAI key，上线前务必去后台删掉、重新生成一个干净的填到 Railway。
- 先用 Stripe **test 模式**（`sk_test_` + 测试卡）把整条付费流程跑通，确认无误再换成 live 正式收款。
