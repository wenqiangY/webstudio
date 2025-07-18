# Webstudio 故障排除指南

本文档总结了 Webstudio 开发环境中常见问题的解决方案。

## 🔧 PostgREST 相关问题

### 1. 角色不存在错误

**错误信息**:

```
{
  code: '22023',
  details: null,
  hint: null,
  message: 'role "anon" does not exist'
}
```

或

```
{
  code: '22023',
  details: null,
  hint: null,
  message: 'role "postgres" does not exist'
}
```

**原因**: PostgREST 配置的 `PGRST_DB_ANON_ROLE` 与数据库中实际存在的角色不匹配。

**解决步骤**:

1. **检查数据库中存在的角色**:

   ```bash
   # 查看所有数据库角色
   docker exec -it <postgres-container-name> psql -U <username> -d webstudio -c "\du"
   ```

2. **确认 PostgreSQL 容器配置**:

   ```bash
   # 检查容器环境变量
   docker inspect <postgres-container-name> | grep -A 10 -B 10 "Env"
   ```

3. **根据实际角色调整 PostgREST 配置**:

   停止现有容器：

   ```bash
   docker stop postgrest && docker rm postgrest
   ```

   使用正确的角色重新启动：

   ```bash
   # 如果数据库中有 username 角色
   docker run --name postgrest \
     -e PGRST_DB_URI=postgresql://username:password@host.docker.internal:5432/webstudio \
     -e PGRST_DB_SCHEMAS=public \
     -e PGRST_DB_ANON_ROLE=username \
     -e PGRST_JWT_SECRET=b2320afc800ab8e63adabe83297c6474570cb41cb00c15b43747a0acc4cb06ef \
     -p 3300:3000 \
     -d docker.io/postgrest/postgrest:v12.2.0
   ```

4. **验证 PostgREST 启动**:

   ```bash
   # 查看启动日志
   docker logs postgrest

   # 测试 API 连接
   curl http://localhost:3300/
   ```

### 2. 数据库连接问题

**错误信息**: 连接超时或连接被拒绝

**解决步骤**:

1. **检查 PostgreSQL 容器状态**:

   ```bash
   docker ps -a | grep postgres
   ```

2. **确认端口映射**:

   ```bash
   # PostgreSQL 应该映射到 5432 端口
   # PostgREST 应该映射到 3300 端口
   docker port <container-name>
   ```

3. **检查网络连接**:
   ```bash
   # 测试数据库连接
   docker exec -it <postgres-container-name> psql -U username -d webstudio -c "SELECT 1;"
   ```

## 🔐 认证相关问题

### 开发登录失败

**问题**: 使用开发密钥登录时失败

**解决步骤**:

1. **确认环境变量设置**:

   ```bash
   # 检查 apps/builder/.env 文件
   grep -E "(DEV_LOGIN|AUTH_SECRET)" apps/builder/.env
   ```

2. **验证密钥格式**:

   - 基本格式: `AUTH_SECRET`
   - 带邮箱格式: `AUTH_SECRET:email@example.com`

3. **检查服务状态**:

   ```bash
   # 确保所有必需服务都在运行
   docker ps
   # 应该看到 PostgreSQL 和 PostgREST 容器
   ```

4. **查看应用日志**:
   ```bash
   # 在项目根目录运行开发服务器并查看日志
   pnpm dev
   ```

## 🗄️ 数据库相关问题

### 数据库不存在

**错误信息**: `database "webstudio" does not exist`

**解决方案**:

```bash
# 创建数据库
docker exec -it <postgres-container-name> psql -U username -d postgres -c "CREATE DATABASE webstudio;"
```

### 迁移失败

**错误信息**: 数据库迁移相关错误

**解决方案**:

```bash
# 运行数据库迁移
cd packages/prisma-client
pnpm migrations --dev --cwd ../../apps/builder
```

## 🔍 调试技巧

### 1. 检查容器状态

```bash
# 查看所有容器
docker ps -a

# 查看特定容器日志
docker logs <container-name>

# 进入容器调试
docker exec -it <container-name> bash
```

### 2. 验证服务连接

```bash
# 测试 PostgreSQL
docker exec -it <postgres-container-name> psql -U username -d webstudio -c "SELECT version();"

# 测试 PostgREST
curl -s http://localhost:3300/ | jq .

# 测试应用服务器
curl -s http://localhost:5173/
```

### 3. 环境变量检查

```bash
# 查看环境变量
cat apps/builder/.env

# 检查特定变量
grep "DATABASE_URL" apps/builder/.env
grep "POSTGREST_URL" apps/builder/.env
```

## 📋 常用命令速查

```bash
# 重启所有服务
docker restart <postgres-container> postgrest

# 清理并重新开始
docker stop postgrest && docker rm postgrest
docker stop <postgres-container> && docker rm <postgres-container>

# 查看端口占用
lsof -i :5432  # PostgreSQL
lsof -i :3300  # PostgREST
lsof -i :5173  # Vite 开发服务器

# 重新构建项目
pnpm install
pnpm build
```

## 🆘 获取帮助

如果以上解决方案都无法解决您的问题，请：

1. 查看 [项目 Issues](https://github.com/webstudio-is/webstudio/issues)
2. 在 [GitHub Discussions](https://github.com/webstudio-is/webstudio-community/discussions) 提问
3. 提供详细的错误信息和环境配置

## 📚 相关文档

- [项目设置指南](./setup-guide.md)
- [数据库配置指南](./database-setup.md)
- [Storybook 使用指南](./storybook-guide.md)
