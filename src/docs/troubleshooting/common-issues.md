# 常见问题解决方案

本文档收集了 VoiceHub 使用过程中的常见问题及其解决方案。

## 安装和配置问题

### 1. Prisma 客户端初始化错误

**问题描述**：
```
PrismaClientInitializationError: Prisma has detected that this project was built on Vercel...
```

**解决方案**：

1. **重新生成 Prisma 客户端**：
```bash
npx prisma generate
```

2. **清除缓存并重新安装**：
```bash
rm -rf node_modules
rm package-lock.json
npm install
npx prisma generate
```

3. **检查环境变量**：
确保 `.env` 文件中的 `DATABASE_URL` 配置正确：
```env
DATABASE_URL="postgresql://username:password@localhost:5432/voicehub"
```

### 2. 数据库连接失败

**问题描述**：
```
Error: P1001: Can't reach database server at localhost:5432
```

**解决方案**：

1. **检查 PostgreSQL 服务状态**：
```bash
# Windows
net start postgresql-x64-13

# Linux/macOS
sudo systemctl start postgresql
```

2. **验证数据库连接**：
```bash
psql -h localhost -p 5432 -U username -d voicehub
```

3. **检查防火墙设置**：
确保端口 5432 未被防火墙阻止

4. **重置数据库连接**：
```bash
npx prisma db push
npx prisma migrate reset
```

### 3. 环境变量未加载

**问题描述**：
应用无法读取环境变量，功能异常

**解决方案**：

1. **检查 .env 文件位置**：
确保 `.env` 文件在项目根目录

2. **验证环境变量格式**：
```env
# 正确格式
DATABASE_URL="postgresql://user:pass@localhost:5432/db"
JWT_SECRET="your-secret-key"
NEXTAUTH_SECRET="your-nextauth-secret"

# 错误格式（不要有空格）
DATABASE_URL = "postgresql://..."
```

3. **重启开发服务器**：
```bash
npm run dev
```

### 4. 依赖安装失败

**问题描述**：
```
npm ERR! peer dep missing
```

**解决方案**：

1. **清除缓存**：
```bash
npm cache clean --force
```

2. **删除 node_modules 重新安装**：
```bash
rm -rf node_modules package-lock.json
npm install
```

3. **使用 yarn 替代 npm**：
```bash
yarn install
```

4. **检查 Node.js 版本**：
确保使用 Node.js 18+ 版本

## 运行时问题

### 1. 页面加载缓慢

**问题描述**：
页面加载时间过长，用户体验差

**解决方案**：

1. **检查数据库查询**：
优化慢查询，添加必要的索引：
```sql
-- 为常用查询字段添加索引
CREATE INDEX idx_songs_status ON "Song"(status);
CREATE INDEX idx_songs_created_at ON "Song"("createdAt");
CREATE INDEX idx_users_username ON "User"(username);
```

2. **启用数据缓存**：
```typescript
// 使用 useLazyFetch 进行数据缓存
const { data: songs } = await useLazyFetch('/api/songs', {
  key: 'songs-list',
  server: false
})
```

3. **优化图片加载**：
```vue
<template>
  <NuxtImg
    src="/images/logo.png"
    loading="lazy"
    width="200"
    height="100"
  />
</template>
```

4. **检查网络连接**：
确保服务器网络连接稳定

### 2. 音乐播放问题

**问题描述**：
音乐无法播放或播放中断

**解决方案**：

1. **检查音乐链接有效性**：
```typescript
const validateMusicUrl = async (url: string) => {
  try {
    const response = await fetch(url, { method: 'HEAD' })
    return response.ok
  } catch {
    return false
  }
}
```

2. **处理跨域问题**：
在 `nuxt.config.ts` 中配置 CORS：
```typescript
export default defineNuxtConfig({
  nitro: {
    cors: {
      origin: true,
      credentials: true
    }
  }
})
```

3. **检查浏览器兼容性**：
确保使用支持 HTML5 Audio 的现代浏览器

4. **音频格式支持**：
使用广泛支持的音频格式（MP3、AAC）

### 3. 用户认证问题

**问题描述**：
用户无法登录或频繁掉线

**解决方案**：

1. **检查 JWT 配置**：
```typescript
// server/api/auth/login.post.ts
const token = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET!,
  { expiresIn: '7d' } // 设置合适的过期时间
)
```

2. **验证密码加密**：
```typescript
// 确保密码正确加密和验证
const isValid = await bcrypt.compare(password, user.password)
```

3. **检查 Cookie 设置**：
```typescript
setCookie(event, 'auth-token', token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'lax',
  maxAge: 60 * 60 * 24 * 7 // 7 天
})
```

4. **清除浏览器缓存**：
清除浏览器的 Cookie 和本地存储

### 4. 数据同步问题

**问题描述**：
前端显示的数据与数据库不一致

**解决方案**：

1. **刷新数据缓存**：
```typescript
// 强制刷新数据
await refreshCookie('auth-token')
await $fetch('/api/songs', { 
  method: 'GET',
  headers: { 'cache-control': 'no-cache' }
})
```

2. **检查数据更新逻辑**：
确保数据修改后正确更新缓存：
```typescript
const updateSong = async (id: string, data: UpdateSongData) => {
  await $fetch(`/api/songs/${id}`, {
    method: 'PUT',
    body: data
  })
  
  // 刷新相关数据
  await refreshData()
}
```

3. **使用实时更新**：
考虑使用 WebSocket 或 Server-Sent Events 进行实时数据同步

## 权限和安全问题

### 1. 权限验证失败

**问题描述**：
用户无法访问应有权限的功能

**解决方案**：

1. **检查用户角色**：
```sql
-- 查询用户角色和权限
SELECT u.username, u.role, up.permission
FROM "User" u
LEFT JOIN "UserPermission" up ON u.id = up."userId"
WHERE u.id = 'user-id';
```

2. **验证权限检查逻辑**：
```typescript
// composables/useAuth.ts
export const hasPermission = (permission: string): boolean => {
  const user = useAuthUser()
  if (!user.value) return false
  
  return user.value.permissions.includes(permission) ||
         user.value.role === 'SUPER_ADMIN'
}
```

3. **重新分配权限**：
通过管理后台重新分配用户权限

### 2. 安全漏洞

**问题描述**：
发现潜在的安全风险

**解决方案**：

1. **输入验证**：
```typescript
// 验证用户输入
import { z } from 'zod'

const songSchema = z.object({
  title: z.string().min(1).max(100),
  artist: z.string().min(1).max(100),
  url: z.string().url().optional()
})

export default defineEventHandler(async (event) => {
  const body = await readBody(event)
  const validatedData = songSchema.parse(body)
  // 处理验证后的数据
})
```

2. **SQL 注入防护**：
使用 Prisma ORM 自动防护 SQL 注入

3. **XSS 防护**：
```vue
<template>
  <!-- 使用 v-text 而不是 v-html -->
  <div v-text="userInput"></div>
  
  <!-- 如果必须使用 HTML，进行清理 -->
  <div v-html="sanitizeHtml(userInput)"></div>
</template>
```

4. **CSRF 防护**：
```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  security: {
    headers: {
      crossOriginEmbedderPolicy: 'require-corp',
      crossOriginOpenerPolicy: 'same-origin',
      crossOriginResourcePolicy: 'same-origin'
    }
  }
})
```

## 性能问题

### 1. 内存泄漏

**问题描述**：
应用运行一段时间后内存使用过高

**解决方案**：

1. **检查事件监听器**：
```typescript
// 确保清理事件监听器
onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
```

2. **清理定时器**：
```typescript
const timer = setInterval(() => {
  // 定时任务
}, 1000)

onUnmounted(() => {
  clearInterval(timer)
})
```

3. **优化数据库连接**：
```typescript
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
  binaryTargets = ["native"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  // 配置连接池
  relationMode = "prisma"
}
```

### 2. 数据库性能问题

**问题描述**：
数据库查询缓慢，影响应用性能

**解决方案**：

1. **添加数据库索引**：
```sql
-- 为常用查询添加索引
CREATE INDEX CONCURRENTLY idx_songs_status_created 
ON "Song"(status, "createdAt" DESC);

CREATE INDEX CONCURRENTLY idx_users_role 
ON "User"(role);

CREATE INDEX CONCURRENTLY idx_schedules_date 
ON "Schedule"(date);
```

2. **优化查询语句**：
```typescript
// 避免 N+1 查询问题
const songsWithDetails = await prisma.song.findMany({
  include: {
    submitter: {
      select: {
        id: true,
        username: true
      }
    },
    votes: {
      select: {
        id: true,
        userId: true
      }
    }
  }
})
```

3. **使用分页查询**：
```typescript
const getSongs = async (page: number, pageSize: number = 20) => {
  const skip = (page - 1) * pageSize
  
  const [songs, total] = await Promise.all([
    prisma.song.findMany({
      skip,
      take: pageSize,
      orderBy: { createdAt: 'desc' }
    }),
    prisma.song.count()
  ])
  
  return { songs, total, pages: Math.ceil(total / pageSize) }
}
```

4. **启用查询缓存**：
```typescript
// 使用 Redis 缓存查询结果
const getCachedSongs = async () => {
  const cached = await redis.get('songs:list')
  if (cached) {
    return JSON.parse(cached)
  }
  
  const songs = await prisma.song.findMany()
  await redis.setex('songs:list', 300, JSON.stringify(songs)) // 5分钟缓存
  
  return songs
}
```

## 部署问题

### 1. Vercel 部署失败

**问题描述**：
在 Vercel 上部署时构建失败

**解决方案**：

1. **检查构建命令**：
```json
// package.json
{
  "scripts": {
    "build": "nuxt build",
    "vercel-build": "prisma generate && nuxt build"
  }
}
```

2. **配置环境变量**：
在 Vercel 控制台中设置所有必要的环境变量

3. **检查依赖版本**：
确保所有依赖版本兼容：
```json
{
  "engines": {
    "node": ">=18.0.0"
  }
}
```

4. **配置 Vercel 设置**：
```json
// vercel.json
{
  "functions": {
    "server/api/**/*.ts": {
      "maxDuration": 30
    }
  }
}
```

### 2. 数据库迁移问题

**问题描述**：
生产环境数据库迁移失败

**解决方案**：

1. **备份数据库**：
```bash
pg_dump -h hostname -U username -d database_name > backup.sql
```

2. **检查迁移文件**：
```bash
npx prisma migrate status
npx prisma migrate resolve --applied "migration_name"
```

3. **手动执行迁移**：
```bash
npx prisma migrate deploy
```

4. **回滚迁移**：
```bash
# 恢复备份
psql -h hostname -U username -d database_name < backup.sql
```

## 调试技巧

### 1. 启用调试模式

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  devtools: { enabled: true },
  debug: process.env.NODE_ENV === 'development',
  logging: {
    level: 'verbose'
  }
})
```

### 2. 查看错误日志

```bash
# 查看 Nuxt 开发服务器日志
npm run dev -- --verbose

# 查看生产环境日志
pm2 logs voicehub
```

### 3. 数据库调试

```typescript
// 启用 Prisma 查询日志
const prisma = new PrismaClient({
  log: ['query', 'info', 'warn', 'error'],
})
```

### 4. 网络请求调试

```typescript
// 添加请求拦截器
$fetch.create({
  onRequest({ request, options }) {
    console.log('Request:', request, options)
  },
  onResponse({ response }) {
    console.log('Response:', response.status, response.body)
  },
  onResponseError({ response }) {
    console.error('Response Error:', response.status, response.body)
  }
})
```

## 获取帮助

如果以上解决方案无法解决您的问题，可以通过以下方式获取帮助：

1. **查看详细文档**：
   - [开发指南](../development/overview)
   - [API 文档](../development/api-development)

2. **检查 GitHub Issues**：
   - 搜索已知问题
   - 提交新的 Issue

3. **社区支持**：
   - 参与讨论
   - 寻求社区帮助

4. **联系维护者**：
   - 通过邮件联系项目维护者
   - 提供详细的错误信息和复现步骤

记住，在寻求帮助时，请提供：
- 详细的错误信息
- 复现步骤
- 环境信息（操作系统、Node.js 版本等）
- 相关的配置文件内容