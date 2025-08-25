# 配置说明

本文档详细介绍了 VoiceHub 的各种配置选项和设置。

## 环境变量配置

### 必需环境变量

| 变量名            | 必填 | 说明                   | 示例值                                                   |
|----------------|----|----------------------|-------------------------------------------------------|
| `DATABASE_URL` | ✅  | PostgreSQL 数据库连接字符串  | `postgresql://user:pass@host:5432/db?sslmode=require` |
| `JWT_SECRET`   | ✅  | JWT 令牌签名密钥，建议至少32个字符 | `your-very-secure-jwt-secret-key-32-chars`            |

### 可选环境变量

| 变量名            | 必填 | 说明               | 默认值           |
|----------------|----|------------------|---------------|
| `NODE_ENV`     | ❌  | 运行环境             | `development` |

## 数据库配置

### PostgreSQL 设置

VoiceHub 使用 PostgreSQL 作为主数据库。推荐配置：

```sql
-- 创建数据库
CREATE DATABASE voicehub;

-- 创建用户（可选）
CREATE USER voicehub_user WITH PASSWORD 'your_password';

-- 授予权限
GRANT ALL PRIVILEGES ON DATABASE voicehub TO voicehub_user;
```

### 连接字符串格式

```
postgresql://[用户名]:[密码]@[主机]:[端口]/[数据库名]?[参数]
```

常用参数：
- `sslmode=require` - 强制使用 SSL 连接
- `sslmode=disable` - 禁用 SSL（仅开发环境）
- `schema=public` - 指定数据库模式

### 云数据库配置

#### Vercel Postgres
```env
DATABASE_URL="postgres://default:password@host-pooler.region.postgres.vercel-storage.com:5432/verceldb?sslmode=require"
```

#### Supabase
```env
DATABASE_URL="postgresql://postgres:password@db.project.supabase.co:5432/postgres?sslmode=require"
```

#### Railway
```env
DATABASE_URL="postgresql://postgres:password@containers-us-west-1.railway.app:5432/railway?sslmode=require"
```

## 站点配置

VoiceHub 支持通过管理后台动态配置站点设置。

### 基本信息

- **站点标题**：显示在浏览器标题栏和导航栏
- **站点描述**：SEO 描述和系统介绍
- **站点Logo**：支持上传自定义 Logo

## 权限系统配置

### 角色定义

VoiceHub 内置三种用户角色：

1. **超级管理员** (`SUPER_ADMIN`)
   - 拥有所有系统权限
   - 可以管理其他管理员
   - 可以修改系统配置

2. **管理员** (`ADMIN`)
   - 日常管理权限
   - 排期管理、歌曲管理
   - 用户管理（除管理员外）

3. **普通用户** (`USER`)
   - 基本点歌、投票权限
   - 查看排期和通知

### 权限配置

权限配置文件位于 `server/utils/permissions.ts`：

```typescript
export const PERMISSIONS = {
  // 用户管理
  MANAGE_USERS: 'manage_users',
  MANAGE_ADMINS: 'manage_admins',
  
  // 歌曲管理
  MANAGE_SONGS: 'manage_songs',
  MANAGE_SCHEDULE: 'manage_schedule',
  
  // 系统管理
  MANAGE_SYSTEM: 'manage_system',
  MANAGE_BACKUP: 'manage_backup',
  
  // 通知管理
  MANAGE_NOTIFICATIONS: 'manage_notifications',
};
```

## 音乐服务配置

VoiceHub 使用第三方音乐 API 提供搜索和播放功能。

:::warning 免责声明
VoiceHub 不提供音乐内容，所有音乐内容均来自第三方平台。请确保遵守相关平台的服务条款和版权规定。
:::

### 支持的音乐平台

- **网易云音乐**：搜索和播放链接获取
- **QQ音乐**：搜索和播放链接获取

### API 配置

音乐 API 配置通过代理服务实现，无需额外配置。

## 下一步

配置完成后，请继续阅读：

- [首次运行](./first-run) - 初始设置和配置
- [管理员指南](../admin-guide/overview) - 了解管理功能
- [故障排除](../troubleshooting/common-issues) - 解决常见问题