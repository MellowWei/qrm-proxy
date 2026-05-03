# QRM Proxy · Vercel 部署指南

## 一次性部署 · 5分钟完成

### 第一步：注册Vercel
访问 https://vercel.com → 用GitHub账号登录（免费）

### 第二步：拿Anthropic API Key
访问 https://console.anthropic.com → API Keys → Create Key
复制保存这个key（只显示一次）

### 第三步：本地准备
把这个 `vercel-proxy` 文件夹放到任何位置，比如桌面。
文件结构应该是：
```
vercel-proxy/
├── api/
│   └── analyze.js
├── vercel.json
└── package.json
```

### 第四步：部署（两种方法选一）

#### 方法A · 用Vercel CLI（推荐）
```bash
npm install -g vercel
cd vercel-proxy
vercel
```
跟着提示走：
- Set up and deploy? → **Y**
- Which scope? → 选你自己
- Link to existing project? → **N**
- Project name? → `qrm-proxy`（或任何名字）
- In which directory is your code located? → `./`
- Override settings? → **N**

部署完成后，你会拿到一个URL，比如：
`https://qrm-proxy-xxx.vercel.app`

#### 方法B · 用Vercel网页（不用命令行）
1. 把 `vercel-proxy` 文件夹推到一个GitHub仓库
2. Vercel Dashboard → Add New Project → 导入那个仓库
3. 直接点 Deploy

### 第五步：设置环境变量
部署成功后：
1. 进入Vercel Dashboard → 你的项目 → Settings → Environment Variables
2. 添加：
   - **Name**: `ANTHROPIC_API_KEY`
   - **Value**: 第二步拿到的API key
   - **Environments**: 全选（Production / Preview / Development）
3. 保存
4. 回到 Deployments 标签 → 最新的部署右上角 ··· → Redeploy

### 第六步：测试
浏览器访问：
`https://你的项目名.vercel.app/api/analyze`

应该看到 `Method not allowed`——这是对的。说明proxy活着。

### 第七步：修改前端
打开 `main.js`，找到这一行：
```javascript
const resp = await fetch('https://api.anthropic.com/v1/messages', {
```

改成：
```javascript
const resp = await fetch('https://你的项目名.vercel.app/api/analyze', {
```

完成。

---

## 安全说明

- API key只存在Vercel服务器的环境变量里
- 前端永远看不到key，不会被盗用
- Vercel免费层每月10万次调用，对你的网站完全够用
- 默认CORS允许所有来源（`*`）——如果想锁死只允许你自己的网站调用，把 `analyze.js` 里的 `'Access-Control-Allow-Origin': '*'` 改成 `'Access-Control-Allow-Origin': 'https://mellowwei.github.io'`

## 故障排查

- **依然 Failed to fetch**: 检查URL是否正确，是否包含 `/api/analyze`
- **500 error · API key not configured**: Vercel环境变量没设好或没redeploy
- **CORS错误**: 改 `analyze.js` 中的 Access-Control-Allow-Origin
- **API成本**: 每次分析约0.01美元，自己用基本免费

---

魏珏然 · 2026 · BCI-HRP
