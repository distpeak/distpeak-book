# 开发指南概览

欢迎参与 VoiceHub 项目开发！本指南将帮助您了解项目架构、开发流程和最佳实践。

## 项目架构

### 技术栈

VoiceHub 采用现代化的全栈技术架构：

**前端技术**：
- **Nuxt 3** - Vue.js 全栈框架
- **Vue 3** - 渐进式 JavaScript 框架
- **TypeScript** - 类型安全的 JavaScript
- **Tailwind CSS** - 实用优先的 CSS 框架
- **Headless UI** - 无样式的可访问组件

**后端技术**：
- **Nuxt 3 Server API** - 服务端 API 路由
- **Prisma** - 现代化数据库 ORM
- **PostgreSQL** - 关系型数据库
- **bcrypt** - 密码加密
- **jsonwebtoken** - JWT 身份验证

**开发工具**：
- **Vite** - 快速构建工具
- **ESLint** - 代码质量检查
- **Prettier** - 代码格式化
- **TypeScript** - 类型检查

### 项目结构

```
VoiceHub/
├── components/          # Vue 组件
│   ├── ui/             # 基础 UI 组件
│   ├── forms/          # 表单组件
│   ├── layout/         # 布局组件
│   └── features/       # 功能组件
├── pages/              # 页面路由
│   ├── index.vue       # 首页
│   ├── login.vue       # 登录页
│   ├── dashboard.vue   # 管理后台
│   └── ...
├── server/             # 服务端代码
│   └── api/            # API 路由
│       ├── auth/       # 身份验证
│       ├── users/      # 用户管理
│       ├── songs/      # 歌曲管理
│       └── ...
├── prisma/             # 数据库相关
│   ├── schema.prisma   # 数据库模式
│   └── migrations/     # 数据库迁移
├── composables/        # 组合式 API
├── middleware/         # 中间件
├── plugins/            # 插件
├── utils/              # 工具函数
├── types/              # TypeScript 类型定义
└── public/             # 静态资源
```

## 开发环境设置

### 环境要求

- **Node.js** >= 18.0.0
- **npm** >= 8.0.0 或 **yarn** >= 1.22.0
- **PostgreSQL** >= 13.0
- **Git** >= 2.0.0

### 快速开始

1. **克隆项目**：
```bash
git clone https://github.com/laoshuikaixue/VoiceHub.git
cd VoiceHub
```

2. **安装依赖**：
```bash
npm install
```

3. **配置环境变量**：
```bash
cp .env.example .env
# 编辑 .env 文件，配置数据库连接等
```

4. **初始化数据库**：
```bash
npx prisma migrate dev
npx prisma db seed
```

5. **启动开发服务器**：
```bash
npm run dev
```

## 核心概念

### 组合式 API (Composables)

VoiceHub 大量使用 Vue 3 的组合式 API 来组织业务逻辑：

**用户管理**：
- `useAuth()` - 身份验证和用户状态
- `useUser()` - 用户信息管理
- `usePermissions()` - 权限检查

**歌曲管理**：
- `useSongs()` - 歌曲数据管理
- `usePlayer()` - 音乐播放器控制
- `useSchedule()` - 排期管理

**数据获取**：
- `useFetch()` - Nuxt 内置数据获取
- `useAsyncData()` - 异步数据处理
- `useLazyFetch()` - 懒加载数据获取

### 状态管理

使用 Nuxt 3 内置的状态管理：

```typescript
// composables/useAuth.ts
export const useAuth = () => {
  const user = useState<User | null>('auth.user', () => null)
  
  const login = async (credentials: LoginCredentials) => {
    // 登录逻辑
  }
  
  const logout = async () => {
    // 登出逻辑
  }
  
  return {
    user: readonly(user),
    login,
    logout
  }
}
```

### 数据库模型

使用 Prisma 定义数据模型：

```prisma
// prisma/schema.prisma
model User {
  id        String   @id @default(cuid())
  username  String   @unique
  email     String   @unique
  password  String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  songs     Song[]
  votes     Vote[]
}

model Song {
  id          String     @id @default(cuid())
  title       String
  artist      String
  url         String?
  status      SongStatus @default(PENDING)
  submitterId String
  
  submitter   User       @relation(fields: [submitterId], references: [id])
  votes       Vote[]
  schedules   Schedule[]
}
```

## 开发流程

### 分支管理

采用 Git Flow 工作流：

- **main** - 生产环境分支
- **develop** - 开发环境分支
- **feature/** - 功能开发分支
- **hotfix/** - 紧急修复分支

### 代码规范

**TypeScript 规范**：
```typescript
// 使用接口定义类型
interface User {
  id: string
  username: string
  email: string
  role: Role
}

// 使用泛型提高代码复用性
interface ApiResponse<T> {
  data: T
  message: string
  success: boolean
}

// 使用枚举定义常量
enum SongStatus {
  PENDING = 'PENDING',
  APPROVED = 'APPROVED',
  REJECTED = 'REJECTED',
  PLAYED = 'PLAYED'
}
```

**Vue 组件规范**：
```vue
<template>
  <div class="component-wrapper">
    <!-- 模板内容 -->
  </div>
</template>

<script setup lang="ts">
// 导入
import { ref, computed } from 'vue'

// 类型定义
interface Props {
  title: string
  items: Item[]
}

// Props 和 Emits
const props = defineProps<Props>()
const emit = defineEmits<{
  select: [item: Item]
}>()

// 响应式数据
const selectedItem = ref<Item | null>(null)

// 计算属性
const filteredItems = computed(() => {
  // 计算逻辑
})

// 方法
const handleSelect = (item: Item) => {
  selectedItem.value = item
  emit('select', item)
}
</script>

<style scoped>
.component-wrapper {
  /* 样式 */
}
</style>
```

### API 开发

**服务端 API 路由**：
```typescript
// server/api/songs/index.get.ts
export default defineEventHandler(async (event) => {
  try {
    // 身份验证
    const user = await requireAuth(event)
    
    // 查询参数
    const query = getQuery(event)
    
    // 业务逻辑
    const songs = await prisma.song.findMany({
      where: {
        status: query.status as SongStatus
      },
      include: {
        submitter: true,
        votes: true
      }
    })
    
    return {
      success: true,
      data: songs
    }
  } catch (error) {
    throw createError({
      statusCode: 500,
      statusMessage: 'Internal Server Error'
    })
  }
})
```

**客户端 API 调用**：
```typescript
// composables/useSongs.ts
export const useSongs = () => {
  const songs = ref<Song[]>([])
  
  const fetchSongs = async (status?: SongStatus) => {
    const { data } = await $fetch('/api/songs', {
      query: { status }
    })
    songs.value = data
  }
  
  const createSong = async (songData: CreateSongData) => {
    const { data } = await $fetch('/api/songs', {
      method: 'POST',
      body: songData
    })
    return data
  }
  
  return {
    songs: readonly(songs),
    fetchSongs,
    createSong
  }
}
```

## 功能开发指南

### 添加新功能

1. **创建数据模型**：
   - 在 `prisma/schema.prisma` 中定义数据模型
   - 运行 `npx prisma migrate dev` 创建迁移

2. **创建 API 路由**：
   - 在 `server/api/` 下创建相应的路由文件
   - 实现 CRUD 操作

3. **创建组合式 API**：
   - 在 `composables/` 下创建业务逻辑
   - 封装数据获取和状态管理

4. **创建 Vue 组件**：
   - 在 `components/` 下创建 UI 组件
   - 使用 TypeScript 和 Tailwind CSS

5. **创建页面**：
   - 在 `pages/` 下创建页面文件
   - 配置路由和权限

### 数据库操作

**查询数据**：
```typescript
// 基础查询
const users = await prisma.user.findMany()

// 条件查询
const activeUsers = await prisma.user.findMany({
  where: {
    status: 'ACTIVE'
  }
})

// 关联查询
const userWithSongs = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    songs: true,
    votes: true
  }
})

// 分页查询
const songs = await prisma.song.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
  orderBy: {
    createdAt: 'desc'
  }
})
```

**数据修改**：
```typescript
// 创建数据
const newUser = await prisma.user.create({
  data: {
    username: 'newuser',
    email: 'user@example.com',
    password: hashedPassword
  }
})

// 更新数据
const updatedUser = await prisma.user.update({
  where: { id: userId },
  data: {
    email: 'newemail@example.com'
  }
})

// 删除数据
await prisma.user.delete({
  where: { id: userId }
})

// 批量操作
await prisma.song.updateMany({
  where: {
    status: 'PENDING'
  },
  data: {
    status: 'APPROVED'
  }
})
```

### 权限控制

**中间件权限检查**：
```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to) => {
  const { user } = useAuth()
  
  if (!user.value) {
    return navigateTo('/login')
  }
})

// middleware/admin.ts
export default defineNuxtRouteMiddleware((to) => {
  const { user, hasPermission } = useAuth()
  
  if (!user.value || !hasPermission('MANAGE_USERS')) {
    throw createError({
      statusCode: 403,
      statusMessage: 'Forbidden'
    })
  }
})
```

**组件级权限控制**：
```vue
<template>
  <div>
    <button 
      v-if="hasPermission('MANAGE_SONGS')"
      @click="deleteSong"
    >
      删除歌曲
    </button>
  </div>
</template>

<script setup lang="ts">
const { hasPermission } = useAuth()

const deleteSong = () => {
  // 删除逻辑
}
</script>
```

## 测试

### 单元测试

使用 Vitest 进行单元测试：

```typescript
// tests/composables/useAuth.test.ts
import { describe, it, expect } from 'vitest'
import { useAuth } from '~/composables/useAuth'

describe('useAuth', () => {
  it('should login successfully with valid credentials', async () => {
    const { login } = useAuth()
    
    const result = await login({
      username: 'testuser',
      password: 'password123'
    })
    
    expect(result.success).toBe(true)
  })
})
```

### 集成测试

测试 API 端点：

```typescript
// tests/api/songs.test.ts
import { describe, it, expect } from 'vitest'
import { $fetch } from '@nuxt/test-utils'

describe('/api/songs', () => {
  it('should return songs list', async () => {
    const response = await $fetch('/api/songs')
    
    expect(response.success).toBe(true)
    expect(Array.isArray(response.data)).toBe(true)
  })
})
```

## 部署

### 开发环境

```bash
# 启动开发服务器
npm run dev

# 类型检查
npm run type-check

# 代码检查
npm run lint

# 格式化代码
npm run format
```

### 生产环境

```bash
# 构建项目
npm run build

# 预览构建结果
npm run preview

# 启动生产服务器
npm start
```

### Docker 部署

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

## 最佳实践

### 代码组织

1. **单一职责**：每个函数和组件只负责一个功能
2. **可复用性**：提取公共逻辑到组合式 API
3. **类型安全**：充分利用 TypeScript 的类型系统
4. **错误处理**：统一的错误处理机制

### 性能优化

1. **懒加载**：使用 `useLazyFetch` 进行数据懒加载
2. **缓存策略**：合理使用缓存减少重复请求
3. **代码分割**：按需加载组件和页面
4. **图片优化**：使用 Nuxt Image 优化图片

### 安全考虑

1. **输入验证**：验证所有用户输入
2. **权限检查**：在服务端进行权限验证
3. **密码安全**：使用强密码策略和加密存储
4. **CSRF 防护**：防止跨站请求伪造攻击

## 贡献指南

### 提交代码

1. **Fork 项目**到您的 GitHub 账户
2. **创建功能分支**：`git checkout -b feature/new-feature`
3. **提交更改**：`git commit -m "Add new feature"`
4. **推送分支**：`git push origin feature/new-feature`
5. **创建 Pull Request**

### 代码审查

- 确保代码符合项目规范
- 添加必要的测试
- 更新相关文档
- 通过所有 CI 检查

## 下一步

深入了解具体开发主题：

- [API 开发](./api-development) - 详细的 API 开发指南
- [组件开发](./component-development) - Vue 组件开发最佳实践
- [数据库设计](./database-design) - 数据库模型设计指南
- [测试指南](./testing) - 完整的测试策略
- [部署指南](./deployment) - 生产环境部署

如果遇到问题，请查看：

- [故障排除](../troubleshooting/development-issues) - 开发常见问题
- [FAQ](../troubleshooting/faq) - 常见问题解答