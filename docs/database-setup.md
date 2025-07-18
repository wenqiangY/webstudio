# 数据库配置指南

本文档详细说明了 Webstudio 项目的数据库配置和管理。

## 📊 数据库架构概述

Webstudio 使用 **PostgreSQL** 作为主数据库，通过 **Prisma ORM** 进行管理。

### 主要数据表

- **User** - 用户信息和认证
- **Team** - 团队管理
- **Project** - 项目数据
- **Build** - 项目构建版本
- **Asset/File** - 资源文件管理
- **Domain** - 自定义域名配置
- **AuthorizationToken** - 访问令牌
- **Product/TransactionLog** - 付费功能相关

## 🔧 环境变量配置

### 必需的环境变量

在 `apps/builder/.env` 文件中配置：

```bash
# 主数据库连接（用于应用程序）
DATABASE_URL=postgresql://postgres:pass@localhost:5432/webstudio?pgbouncer=true

# 直接数据库连接（用于迁移）
DIRECT_URL=postgresql://postgres:pass@localhost:5432/webstudio

# 认证密钥
AUTH_SECRET="your-secret-key-here"
```

### 连接字符串格式

```
postgresql://[用户名]:[密码]@[主机]:[端口]/[数据库名]?[参数]
```

**参数说明**:

- `pgbouncer=true` - 启用连接池支持
- `schema=public` - 指定数据库模式（可选）

## 🗄️ 数据库设置方式

### 方式一：Dev Container（推荐）

Dev Container 自动配置完整的开发环境：

```bash
# 在 VS Code 中
# 1. 打开命令面板 (Cmd+Shift+P)
# 2. 选择 "Dev Containers: Reopen in Container"
# 3. 等待容器构建完成
```

**优势**:

- 自动配置 PostgreSQL + PostgREST
- 预装所有必需工具
- 统一开发环境
- 无需手动数据库设置

### 方式二：Docker 容器

```bash
# 启动 PostgreSQL 容器
docker run --name webstudio-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=pass \
  -e POSTGRES_DB=webstudio \
  -p 5432:5432 \
  -d postgres:15

# 检查容器状态
docker ps

# 停止容器
docker stop webstudio-postgres

# 重启容器
docker start webstudio-postgres

# 删除容器（注意：会丢失数据）
docker rm webstudio-postgres
```

### 方式三：本地安装

#### macOS (Homebrew)

```bash
# 安装 PostgreSQL
brew install postgresql@15

# 启动服务
brew services start postgresql@15

# 创建数据库
createdb webstudio

# 连接数据库
psql webstudio
```

#### Ubuntu/Debian

```bash
# 安装 PostgreSQL
sudo apt update
sudo apt install postgresql postgresql-contrib

# 启动服务
sudo systemctl start postgresql
sudo systemctl enable postgresql

# 创建用户和数据库
sudo -u postgres createuser --interactive
sudo -u postgres createdb webstudio
```

## 🔄 数据库迁移管理

### Prisma 迁移系统

Webstudio 使用自定义的迁移系统，基于 Prisma 但有额外功能。

### 迁移命令

```bash
# 查看迁移状态
pnpm --filter=./packages/prisma-client migrations status --dev --cwd ../../apps/builder

# 应用所有待处理的迁移
pnpm --filter=./packages/prisma-client migrations migrate --dev --cwd ../../apps/builder

# 查看待处理迁移数量
pnpm --filter=./packages/prisma-client migrations pending-count --dev --cwd ../../apps/builder

# 创建新的 schema 迁移
pnpm --filter=./packages/prisma-client migrations create-schema <迁移名称> --dev --cwd ../../apps/builder

# 创建新的数据迁移
pnpm --filter=./packages/prisma-client migrations create-data <迁移名称> --dev --cwd ../../apps/builder
```

### 迁移文件结构

```
packages/prisma-client/prisma/migrations/
├── 20220601192603_start/
│   ├── migration.sql
│   └── migration.ts (可选)
├── 20220608130924_/
├── ...
└── migration_lock.toml
```

### 迁移状态

- **Applied** - 已成功应用
- **Failed** - 应用失败
- **Pending** - 待处理
- **Rolled Back** - 已回滚

## 🔍 数据库管理工具

### 1. Prisma Studio

```bash
# 启动 Prisma Studio（图形化数据库管理）
cd packages/prisma-client
npx prisma studio
```

访问 http://localhost:5555 查看和编辑数据。

### 2. psql 命令行

```bash
# 连接数据库
psql postgresql://postgres:pass@localhost:5432/webstudio

# 常用命令
\dt          # 列出所有表
\d 表名      # 查看表结构
\q           # 退出
```

### 3. 图形化工具

推荐的 PostgreSQL 图形化管理工具：

- **pgAdmin** - 功能全面的管理工具
- **DBeaver** - 跨平台数据库工具
- **TablePlus** - macOS 上的优秀工具

## 📊 数据库性能优化

### 连接池配置

项目使用 pgBouncer 进行连接池管理：

```bash
# 在 DATABASE_URL 中启用
DATABASE_URL=postgresql://postgres:pass@localhost:5432/webstudio?pgbouncer=true
```

### 索引优化

数据库包含多个性能优化索引：

```sql
-- 项目查询优化
@@index([isDeleted, marketplaceApprovalStatus])

-- 构建查询优化
@@index([projectId, createdAt(sort: Desc)])

-- 域名查询优化
@@index([domainId])
```

## 🚨 故障排除

### 常见错误及解决方案

#### 1. `DIRECT_URL is not set`

**原因**: 环境变量未正确加载

**解决方案**:

```bash
# 确保使用 --dev 参数
pnpm --filter=./packages/prisma-client migrations status --dev --cwd ../../apps/builder

# 检查 .env 文件路径和内容
cat apps/builder/.env
```

#### 2. 连接被拒绝

**原因**: PostgreSQL 未运行或连接配置错误

**解决方案**:

```bash
# 检查 PostgreSQL 状态
docker ps  # 如果使用 Docker
brew services list | grep postgresql  # 如果使用 Homebrew

# 测试连接
psql postgresql://postgres:pass@localhost:5432/webstudio
```

#### 3. 迁移失败

**原因**: 数据库状态不一致或权限问题

**解决方案**:

```bash
# 查看详细错误信息
pnpm --filter=./packages/prisma-client migrations status --dev --cwd ../../apps/builder

# 重置数据库（注意：会丢失所有数据）
dropdb webstudio && createdb webstudio
```

#### 4. 权限错误

**原因**: 数据库用户权限不足

**解决方案**:

```sql
-- 授予必要权限
GRANT ALL PRIVILEGES ON DATABASE webstudio TO postgres;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO postgres;
```

## 🔒 安全注意事项

### 1. 生产环境配置

```bash
# 使用强密码
DATABASE_URL=postgresql://secure_user:complex_password@localhost:5432/webstudio

# 限制网络访问
# 在 postgresql.conf 中设置
listen_addresses = 'localhost'

# 在 pg_hba.conf 中配置访问控制
```

### 2. 备份策略

```bash
# 创建备份
pg_dump webstudio > backup_$(date +%Y%m%d_%H%M%S).sql

# 恢复备份
psql webstudio < backup_20240101_120000.sql
```

## 📈 监控和维护

### 1. 性能监控

```sql
-- 查看慢查询
SELECT query, mean_time, calls
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- 查看表大小
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### 2. 定期维护

```sql
-- 更新统计信息
ANALYZE;

-- 清理死元组
VACUUM;

-- 重建索引
REINDEX DATABASE webstudio;
```

## 📚 相关资源

- [Prisma 官方文档](https://www.prisma.io/docs/)
- [PostgreSQL 官方文档](https://www.postgresql.org/docs/)
- [项目设置指南](./setup-guide.md)
