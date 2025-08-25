# 安装指南

本指南将帮助您安装和配置 VoiceHub。我们提供两种安装方式：一键部署和本地安装。

## 🚀 一键部署（推荐）

如果您想快速体验 VoiceHub，推荐使用一键部署到云平台：

### Vercel 部署

[![Deploy to Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/laoshuikaixue/VoiceHub&env=DATABASE_URL,JWT_SECRET&envDescription=需要配置数据库连接和JWT密钥&envLink=https://github.com/laoshuikaixue/VoiceHub#环境变量说明)

### Netlify 部署

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/laoshuikaixue/VoiceHub)

### 部署步骤

1. **点击部署按钮**：选择上方的 Vercel 或 Netlify 部署按钮
2. **连接 GitHub**：授权平台访问您的 GitHub 账户
3. **配置环境变量**：
   - `DATABASE_URL`：PostgreSQL 数据库连接字符串
   - `JWT_SECRET`：JWT 令牌签名密钥（至少32个字符的随机字符串）
4. **等待部署**：平台会自动构建和部署应用
5. **访问应用**：部署完成后，您将获得一个可访问的 URL

### 数据库配置

部署时需要配置 PostgreSQL 数据库，推荐使用以下云数据库服务：

#### Supabase（推荐）
1. 访问 [Supabase](https://supabase.com/) 并创建项目
2. 在项目设置中找到数据库连接字符串
3. 格式：`postgresql://postgres:password@db.project.supabase.co:5432/postgres?sslmode=require`

#### Railway
1. 访问 [Railway](https://railway.app/) 并创建 PostgreSQL 数据库
2. 复制提供的连接字符串

#### Neon
1. 访问 [Neon](https://neon.tech/) 并创建数据库
2. 获取连接字符串

### 部署后设置

1. **访问应用**：使用部署平台提供的 URL 访问您的 VoiceHub 实例
2. **创建管理员账户**：首次访问时会自动创建默认管理员账户
   - 用户名：`admin`
   - 密码：`admin123`
3. **修改默认密码**：登录后点击后台管理按钮即可修改默认密码
4. **配置站点信息**：在管理后台设置站点名称、描述等信息

:::tip 提示
一键部署是最快的体验方式，适合快速测试和小规模使用。如需更多自定义配置，请参考下方的本地安装指南。
:::

---

## 💻 本地安装

如果您需要本地开发或更多自定义配置，可以选择本地安装。

## 系统要求

在开始之前，请确保您的系统满足以下要求：

### 必需组件
- **Node.js** 20.0 或更高版本
- **PostgreSQL** 数据库
- **Git**（用于克隆代码库）

## 安装步骤

### 1. 克隆代码库

```bash
git clone https://github.com/laoshuikaixue/VoiceHub.git
cd VoiceHub
```

### 2. 安装依赖

```bash
npm install
```

### 3. 配置环境变量

复制环境变量示例文件：

```bash
cp .env.example .env
```

编辑 `.env` 文件，设置必要的环境变量：

```env
# 数据库连接字符串
DATABASE_URL="postgresql://username:password@host:port/database?sslmode=require"

# JWT 密钥（请使用强随机字符串）
JWT_SECRET="your-very-secure-jwt-secret-key-with-at-least-32-characters"

# 运行环境（可选）
NODE_ENV="development"
```

:::tip 提示
JWT_SECRET 应该是一个至少 32 个字符的强随机字符串。您可以使用在线工具生成，或者运行以下命令：

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```
:::

### 4. 初始化数据库

生成 Prisma 客户端：

```bash
npx prisma generate
```

运行数据库迁移：

```bash
npx prisma migrate deploy
```

### 5. 创建管理员账户

运行脚本创建初始管理员账户：

```bash
node scripts/create-admin.js
```

按照提示输入管理员信息，或使用默认设置：
- 用户名：`admin`
- 密码：`admin123`

### 6. 验证安装

运行数据库检查工具确保一切正常：

```bash
cd scripts && npm run check-db
```

## 启动应用

### 开发模式

```bash
npm run dev
```

应用将在 `http://localhost:3000` 启动。

### 生产模式

构建应用：

```bash
npm run build
```

启动生产服务器：

```bash
npm run start
```

## 验证安装

1. 打开浏览器访问 `http://localhost:3000`
2. 您应该能看到 VoiceHub 的主页
3. 点击右上角的"登录"按钮
4. 使用创建的管理员账户登录
5. 成功登录后，您应该能看到管理后台

## 常见问题

### Prisma 客户端错误

如果遇到 Prisma 客户端初始化错误：

```bash
npx prisma generate
```

### 数据库连接问题

1. 确保 PostgreSQL 服务正在运行
2. 检查 `DATABASE_URL` 是否正确
3. 确保数据库用户有足够的权限

### 端口占用

如果 3000 端口被占用，您可以指定其他端口：

```bash
PORT=3001 npm run dev
```

## 下一步

安装完成后，请继续阅读：

- [配置说明](./configuration) - 详细的配置选项
- [首次运行](./first-run) - 初始设置和配置
- [用户指南](../user-guide/overview) - 了解如何使用 VoiceHub