# Webstudio 开发文档

这个目录包含 Webstudio 项目的开发相关文档。

## 文档列表

- [项目设置指南](./setup-guide.md) - 如何设置和运行 Webstudio 开发环境
- [数据库配置](./database-setup.md) - 数据库初始化和配置说明
- [Storybook 使用指南](./storybook-guide.md) - 组件开发和展示工具使用说明
- [故障排除指南](./troubleshooting.md) - 常见问题解决方案

## 快速开始

如果你是第一次设置项目，请按以下顺序阅读文档：

1. [项目设置指南](./setup-guide.md)
2. [数据库配置](./database-setup.md)
3. [Storybook 使用指南](./storybook-guide.md)

如果遇到问题，请查看：4. [故障排除指南](./troubleshooting.md)

## 项目概述

Webstudio 是一个开源的可视化开发平台，专为开发者、设计师和跨职能团队设计。它允许用户拥有数据、组件和基础设施的完全控制权。

### 技术栈

- **React 18** (canary版本) - 前端框架
- **Remix** - 全栈React框架
- **TypeScript** - 类型安全
- **PostgreSQL** - 数据库
- **Prisma** - ORM
- **pnpm** - 包管理器
- **Vite** - 构建工具
- **Storybook** - 组件开发工具

### 项目结构

```
webstudio/
├── apps/
│   └── builder/          # 主要的构建器UI应用
├── packages/             # 30+个功能包
│   ├── react-sdk/        # JavaScript/TypeScript API
│   ├── css-engine/       # CSS渲染引擎
│   ├── design-system/    # 设计系统组件
│   ├── prisma-client/    # 数据库客户端
│   └── ...
├── fixtures/             # 示例项目和模板
└── docs/                 # 项目文档
```
