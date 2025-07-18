# Webstudio 项目设置指南

本指南将帮助你设置 Webstudio 开发环境并成功运行项目。

## 🚀 环境要求

### 必需软件

- **Node.js 20** - 项目要求的版本
- **pnpm ^9.14.0** - 包管理器
- **PostgreSQL 15+** - 数据库
- **Git** - 版本控制

### 可选软件

- **Docker** - 用于运行数据库容器
- **VS Code** - 推荐的开发环境（支持 Dev Container）

## 📦 安装步骤

### 1. 克隆项目

```bash
git clone https://github.com/webstudio-is/webstudio.git
cd webstudio
```

### 2. 安装 Node.js 和 pnpm

```bash
# 使用 nvm 安装 Node.js 20
nvm install 20
nvm use 20

# 安装 pnpm
npm install -g pnpm@9.14.4
```

### 3. 安装项目依赖

```bash
pnpm install
```

## 🗄️ 数据库设置

### 方式一：使用 Dev Container（推荐）

1. 在 VS Code 中打开项目
2. 按 `Cmd+Shift+P`（Mac）或 `Ctrl+Shift+P`（Windows/Linux）
3. 搜索并选择 "Dev Containers: Reopen in Container"
4. 等待容器构建完成（自动包含 PostgreSQL）

### 方式二：使用 Docker

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
```

```bash
# 启动 PostgREST 容器（如果需要使用 PostgREST API）
# 临时使用官方源拉取镜像
docker pull docker.io/postgrest/postgrest:v12.2.0

# ⚠️ 重要：PGRST_DB_ANON_ROLE 必须与数据库中实际存在的角色匹配
# 首先检查数据库中存在哪些角色：
# docker exec -it <postgres-container-name> psql -U <username> -d webstudio -c "\du"

# 方式一：使用 username 角色（推荐，适用于大多数情况）
docker run --name postgrest \
  -e PGRST_DB_URI=postgresql://username:password@host.docker.internal:5432/webstudio \
  -e PGRST_DB_SCHEMAS=public \
  -e PGRST_DB_ANON_ROLE=username \
  -e PGRST_JWT_SECRET=b2320afc800ab8e63adabe83297c6474570cb41cb00c15b43747a0acc4cb06ef \
  -p 3300:3000 \
  -d docker.io/postgrest/postgrest:v12.2.0

# 方式二：如果数据库中有 postgres 角色，可以使用：
# docker run --name postgrest \
#   -e PGRST_DB_URI=postgresql://username:password@host.docker.internal:5432/webstudio \
#   -e PGRST_DB_SCHEMAS=public \
#   -e PGRST_DB_ANON_ROLE=postgres \
#   -e PGRST_JWT_SECRET=b2320afc800ab8e63adabe83297c6474570cb41cb00c15b43747a0acc4cb06ef \
#   -p 3300:3000 \
#   -d docker.io/postgrest/postgrest:v12.2.0

# 验证 PostgREST 是否正常启动
docker logs postgrest

# 测试 PostgREST API 是否工作
curl http://localhost:3300/
```

### 方式三：本地安装 PostgreSQL

```bash
# macOS (使用 Homebrew)
brew install postgresql@15
brew services start postgresql@15

# 创建数据库
createdb webstudio
```

## ⚙️ 环境变量配置

### 1. 复制环境变量文件

```bash
cd apps/builder
# 可选：创建开发环境配置
cp .env .env.development
```

### 2. 修改数据库连接

确保 `apps/builder/.env` 文件中的数据库连接正确：

```bash
# 数据库连接（根据你的设置调整）
DATABASE_URL=postgresql://postgres:pass@localhost:5432/webstudio?pgbouncer=true
DIRECT_URL=postgresql://postgres:pass@localhost:5432/webstudio

# 认证密钥（必需）
AUTH_SECRET="b2320afc800ab8e63adabe83297c6474570cb41cb00c15b43747a0acc4cb06ef"

# 开发模式登录
DEV_LOGIN=true

# 功能开关
FEATURES=*
USER_PLAN=pro
```

### 3. 生成认证密钥（可选）

如果需要生成新的认证密钥：

```bash
# Linux/macOS
openssl rand -hex 32

# 或访问 https://generate-secret.now.sh/64
```

## 🔄 数据库初始化

### 1. 运行数据库迁移

```bash
# 从项目根目录运行
pnpm --filter=./packages/prisma-client migrations migrate --dev --cwd ../../apps/builder
```

### 2. 验证迁移状态

```bash
# 检查迁移状态
pnpm --filter=./packages/prisma-client migrations status --dev --cwd ../../apps/builder
```

成功后应该看到所有迁移都标记为 `(applied)`。

## 🚀 启动项目

### 1. 启动开发服务器

```bash
# 从项目根目录运行
pnpm dev
```

### 2. 访问应用

项目启动后，你可以通过以下地址访问：

- **本地访问**: https://vite.wstd.dev:5173/
- **网络访问**: https://wstd.dev:5173/

## 🛠️ 开发工具

### 启动 Storybook

```bash
# 在新的终端窗口中运行
pnpm storybook:dev
```

访问 http://localhost:6006 查看组件库。

### 其他有用命令

```bash
# 类型检查
pnpm checks

# 代码格式化
pnpm format

# 运行测试
pnpm test

# 构建项目
pnpm build
```

## ❗ 常见问题

### 1. 数据库连接失败

**问题**: `DIRECT_URL is not set` 错误

**解决方案**:

- 确保 PostgreSQL 正在运行
- 检查 `apps/builder/.env` 中的数据库连接字符串
- 使用 `--dev` 参数运行 migrations 命令

### 2. 端口冲突

**问题**: 端口 5173 被占用

**解决方案**:

- 停止占用端口的进程
- 或在 `vite.config.ts` 中修改端口配置

### 3. Node.js 版本警告

**问题**: `Unsupported engine: wanted: {"node":"20"}`

**解决方案**:

- 使用 `nvm use 20` 切换到 Node.js 20
- 或忽略警告（通常不影响功能）

### 4. 权限问题

**问题**: 无法访问某些功能

**解决方案**:

- 确保 `AUTH_SECRET` 已正确设置
- 检查 `DEV_LOGIN=true` 是否已启用

## 🔧 PostgREST 故障排除

### PostgREST 角色错误

**问题**: `role "anon" does not exist` 或 `role "postgres" does not exist`

**原因**: PostgREST 配置的 `PGRST_DB_ANON_ROLE` 与数据库中实际存在的角色不匹配

**解决步骤**:

1. **检查数据库中存在的角色**:

   ```bash
   docker exec -it <postgres-container-name> psql -U <username> -d webstudio -c "\du"
   ```

2. **根据实际角色调整 PostgREST 配置**:

   - 如果看到 `username` 角色，使用 `PGRST_DB_ANON_ROLE=username`
   - 如果看到 `postgres` 角色，使用 `PGRST_DB_ANON_ROLE=postgres`

3. **重新启动 PostgREST 容器**:
   ```bash
   docker stop postgrest && docker rm postgrest
   # 然后使用正确的角色重新运行容器
   ```

### 开发登录失败

**问题**: 开发模式登录时出现数据库连接错误

**解决步骤**:

1. **确保所有服务都在运行**:

   ```bash
   docker ps  # 检查 PostgreSQL 和 PostgREST 容器状态
   ```

2. **检查环境变量配置**:

   - `DATABASE_URL` 和 `DIRECT_URL` 中的用户名密码要正确
   - `POSTGREST_URL` 要指向正确的端口（通常是 3300）

3. **验证服务连接**:

   ```bash
   # 测试数据库连接
   docker exec -it <postgres-container-name> psql -U username -d webstudio -c "SELECT 1;"

   # 测试 PostgREST API
   curl http://localhost:3300/
   ```

4. **检查 PostgREST 日志**:
   ```bash
   docker logs postgrest
   ```

## 🎯 下一步

设置完成后，你可以：

1. **探索主应用** - 访问构建器界面
2. **查看组件库** - 启动 Storybook
3. **阅读代码** - 从 `apps/builder/app` 开始
4. **查看文档** - 阅读其他文档文件

## 📚 相关文档

- [数据库配置详细说明](./database-setup.md)
- [Storybook 使用指南](./storybook-guide.md)
- [故障排除指南](./troubleshooting.md) - 如果遇到问题请查看此文档
