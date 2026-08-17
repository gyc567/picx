# PicX 部署指南

PicX 是一款基于 GitHub API 开发的图床工具，采用 **Vue 3 + Vite + Element Plus** 技术栈构建，支持 PWA，可一键部署到 GitHub Pages 并对外提供服务。

---

## 方式一：通过 GitHub Actions 自动部署（推荐）

### 1. Fork 项目

访问 [PicX 仓库](https://github.com/XPoet/picx)，点击右上角 **Fork** 按钮，将项目复制到你的 GitHub 账号下。

### 2. 配置 GitHub Secrets

部署到 GitHub Pages 需要配置以下密钥，进入仓库 **Settings → Secrets and variables → Actions**，点击 **New repository secret**：

| Secret 名称 | 说明 | 示例 |
|---|---|---|
| `GH_PAGES_DEPLOY` | GitHub Personal Access Token | `ghp_xxxxxxxxxxxx` |
| `MY_USER_NAME` | GitHub 用户名 | `your-username` |
| `MY_USER_EMAIL` | GitHub 邮箱 | `your@email.com` |

#### 生成 Personal Access Token

1. 进入 GitHub **Settings → Developer settings → Personal access tokens → Fine-grained tokens**
2. 点击 **Generate new token**
3. 设置 Token 名称、过期时间
4. 在 **Permissions** 中勾选：
   - `Contents`: Read and write
   - `Pages`: Read and write
5. 点击 **Generate token**，复制生成的 Token

### 3. 启用 GitHub Pages

1. 进入仓库 **Settings → Pages**
2. **Source** 选择 **GitHub Actions**
3. 保存设置

### 4. 触发部署

将代码推送到 `master` 分支即可触发 GitHub Actions 自动构建部署：

```bash
git clone https://github.com/YOUR_USERNAME/picx.git
cd picx
git push origin master
```

部署完成后访问 `https://YOUR_USERNAME.github.io/picx` 或你配置的自定义域名。

---

## 方式二：手动部署

### 1. 克隆项目

```bash
git clone https://github.com/XPoet/picx.git
cd picx
```

### 2. 安装依赖

项目使用 pnpm 作为包管理器：

```bash
npm install -g pnpm
pnpm install
```

### 3. 配置环境变量

复制环境变量模板并修改：

```bash
cp .env.production .env.local
```

编辑 `.env.local`，修改以下配置：

```env
# 是否启用 PWA
VITE_USE_PWA = true

# PicX GitHub APP Client ID（使用官方或自建）
VITE_CLIENT_ID = Iv1.274fe6f96551b91f

# 重定向 URI（修改为你部署后的域名）
VITE_REDIRECT_URI = https://your-domain.com

# GitHub OAuth 授权地址
VITE_AUTHORIZE_URI = https://github.com/login/oauth/authorize

# PicX GitHub APP 安装地址
VITE_INSTALL_URL = https://github.com/apps/picx-app/installations/select_target
VITE_INSTALL_URL_USER = https://github.com/apps/picx-app/installations/new/permissions?target_id=
```

### 4. 构建项目

```bash
pnpm build
```

构建产物输出到 `dist/` 目录。

### 5. 部署到 GitHub Pages

#### 方法 A：使用 gh-pages 包

```bash
npm install -g gh-pages
pnpm build
gh-pages -d dist -m "Update PicX"
```

#### 方法 B：手动上传

将 `dist/` 目录内容推送到 `gh-pages` 分支：

```bash
cd dist
git init
git remote add origin https://github.com/YOUR_USERNAME/picx.git
git add .
git commit -m "Update PicX"
git push -f origin HEAD:gh-pages
```

---

## 方式三：部署到 Vercel

### 1. 连接仓库

1. 注册/登录 [Vercel](https://vercel.com)
2. 点击 **Add New → Project**
3. 导入你的 PicX 仓库

### 2. 配置构建

- **Framework Preset**: Vite
- **Build Command**: `pnpm build`
- **Output Directory**: `dist`

### 3. 设置环境变量

在 Vercel 项目 settings 中添加环境变量：

```
VITE_USE_PWA = true
VITE_CLIENT_ID = Iv1.274fe6f96551b91f
VITE_REDIRECT_URI = https://your-vercel-project.vercel.app
VITE_AUTHORIZE_URI = https://github.com/login/oauth/authorize
VITE_INSTALL_URL = https://github.com/apps/picx-app/installations/select_target
VITE_INSTALL_URL_USER = https://github.com/apps/picx-app/installations/new/permissions?target_id=
```

### 4. 部署

点击 **Deploy**，Vercel 会自动构建并部署。

---

## 方式四：部署到 Netlify

### 1. 连接仓库

1. 注册/登录 [Netlify](https://netlify.com)
2. 点击 **Add new site → Import an existing project**
3. 选择 GitHub 仓库

### 2. 配置构建

- **Build command**: `pnpm build`
- **Publish directory**: `dist`

### 3. 设置环境变量

在 Netlify 项目 settings 中添加相同的环境变量（去掉 `VITE_` 前缀，Netlify 会自动处理）。

### 4. 部署

点击 **Deploy site`，Netlify 会自动构建并部署。

---

## 方式五：部署到自定义服务器

### 1. 构建项目

```bash
pnpm build
```

### 2. 上传 dist 目录

将 `dist/` 目录内容上传到服务器的 Web 目录（如 `/var/www/picx`）。

### 3. 配置 Nginx

```nginx
server {
    listen 80;
    server_name your-domain.com;

    root /var/www/picx;
    index index.html;

    # Vue History 路由支持
    location / {
        try_files $uri $uri/ /index.html;
    }

    # PWA 缓存配置
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

### 4. 配置 HTTPS

使用 Let's Encrypt 免费证书：

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

### 5. 重载 Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 自定义 GitHub App（可选）

PicX 默认使用官方 GitHub App，如果你需要使用自己的 GitHub App：

### 1. 创建 GitHub App

1. 进入 GitHub **Settings → Developer settings → GitHub Apps**
2. 点击 **New GitHub App**
3. 配置：
   - **Homepage URL**: 你的部署地址
   - **Callback URL**: `https://your-domain.com`
   - **Webhook**: 取消勾选（PicX 不需要 Webhook）
   - **Permissions**: `Contents: Read and write`, `Metadata: Read-only`

4. 创建后，复制 **Client ID**
5. 生成 **Client secret**

### 2. 修改环境变量

```env
VITE_CLIENT_ID = your-client-id
VITE_REDIRECT_URI = https://your-domain.com
```

---

## 常见问题

### Q: 部署后图片上传失败？

检查是否正确配置了 GitHub OAuth 授权，参考 [PicX 官方文档 - 授权登录](https://picx-docs.xpoet.cn/usage-guide/config.html#github-oauth-%E6%8E%88%E6%9D%83%E7%99%BB%E5%BD%95)。

### Q: PWA 无法离线访问？

确保 `VITE_USE_PWA = true`，并且部署时使用了 HTTPS（Service Worker 只在安全上下文中生效）。

### Q: GitHub Pages 加载空白？

检查 `vite.config.ts` 中的 `base` 配置，确保与仓库名称一致。如果部署到 `https://username.github.io/picx`，`base` 应为 `/picx/`。

### Q: 如何绑定自定义域名？

1. 在仓库根目录添加 `CNAME` 文件，内容为你的域名
2. 在 DNS 服务商处添加 CNAME 记录指向 `YOUR_USERNAME.github.io`
3. 在 GitHub Pages 设置中启用自定义域名并启用 HTTPS

---

## 项目结构

```
picx/
├── public/              # 静态资源
├── src/                 # 源代码
│   ├── assets/          # 资源文件
│   ├── components/      # 组件
│   ├── views/           # 页面
│   ├── stores/          # 状态管理
│   ├── router/          # 路由
│   ├── plugins/         # 插件配置
│   ├── locales/         # 国际化
│   └── utils/           # 工具函数
├── docs/                # 文档
├── index.html           # 入口 HTML
├── vite.config.ts       # Vite 配置
├── .env.production      # 生产环境变量
└── package.json
```

---

## 快速参考

| 命令 | 说明 |
|---|---|
| `pnpm install` | 安装依赖 |
| `pnpm dev` | 开发模式 |
| `pnpm build` | 构建生产版本 |
| `pnpm serve` | 预览构建结果 |
| `pnpm lint` | ESLint 检查 |
| `pnpm format` | Prettier 格式化 |

---

> 更多信息请参考 [PicX 官方文档](https://picx-docs.xpoet.cn)
