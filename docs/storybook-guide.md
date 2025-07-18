# Storybook 使用指南

本文档详细介绍如何在 Webstudio 项目中使用 Storybook 进行组件开发和展示。

## 📚 什么是 Storybook？

**Storybook** 是一个用于开发和测试 UI 组件的开源工具，可以把它理解为"组件展示厅"。

### 主要功能

- **组件隔离开发** - 单独开发和测试每个组件，不依赖整个应用
- **可视化文档** - 自动生成组件文档和使用示例
- **交互式测试** - 实时调整组件属性，查看不同状态
- **设计系统管理** - 统一管理所有 UI 组件
- **视觉回归测试** - 自动检测 UI 变化

## 🚀 启动 Storybook

### 基本启动

```bash
# 从项目根目录启动 Storybook 开发服务器
pnpm storybook:dev

# 构建 Storybook 静态文件
pnpm storybook:build
```

启动后访问 **http://localhost:6006** 查看组件库。

### 配置文件

Storybook 的主要配置文件位于 `.storybook/main.ts`：

```typescript
export default {
  stories: [
    {
      directory: "../apps/builder",
      titlePrefix: "Builder",
    },
    {
      directory: "../packages/design-system",
      titlePrefix: "Design System",
    },
    // ... 更多组件包
  ],
  framework: {
    name: "@storybook/react-vite",
    options: {},
  },
  addons: [
    "@storybook/addon-controls",
    "@storybook/addon-actions",
    "@storybook/addon-backgrounds",
  ],
};
```

## 📦 Webstudio 中的组件分类

### 1. Builder 组件

**路径**: `apps/builder/app/**/*.stories.tsx`

包含构建器界面的核心组件：

- **登录页面** (`auth/login.stories.tsx`)
- **代码编辑器** (`builder/shared/code-editor.stories.tsx`)
- **表达式编辑器** (`builder/shared/expression-editor.stories.tsx`)
- **CSS 编辑器** (`builder/shared/css-editor/css-editor.stories.tsx`)
- **文本编辑器** (`canvas/features/text-editor/text-editor.stories.tsx`)
- **断点选择器** (`builder/features/breakpoints/breakpoints-selector.stories.tsx`)
- **阻塞警告** (`builder/features/blocking-alerts/alert.stories.tsx`)

### 2. Design System 组件

**路径**: `packages/design-system/src/**/*.stories.tsx`

基础设计系统组件（按钮、输入框、对话框等）。

### 3. CSS Engine 组件

**路径**: `packages/css-engine/**/*.stories.tsx`

CSS 渲染和处理相关组件。

### 4. Icons 组件

**路径**: `packages/icons/**/*.stories.tsx`

图标库组件。

### 5. SDK Components

包含多个 SDK 组件包：

- **React Components** (`packages/sdk-components-react/`)
- **Radix Components** (`packages/sdk-components-react-radix/`)
- **Animation Components** (`packages/sdk-components-animation/`)

## 📝 编写 Stories

### 基本 Story 结构

```typescript
// login.stories.tsx
import type { JSX } from "react";
import type { StoryFn } from "@storybook/react";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import { Login } from "./login";

export default {
  component: Login,
};

const createRouter = (element: JSX.Element) =>
  createBrowserRouter([
    {
      path: "*",
      element,
      loader: () => null,
    },
  ]);

export const Basic: StoryFn<typeof Login> = () => {
  const router = createRouter(
    <Login isGoogleEnabled={false} isSecretLoginEnabled />
  );
  return <RouterProvider router={router} />;
};
```

### Story 命名规范

- **文件名**: `组件名.stories.tsx`
- **导出名**: 描述性的名称（如 `Basic`, `WithError`, `Loading`）
- **组件分组**: 通过目录结构和 `titlePrefix` 组织

### 常用 Story 模式

#### 1. 基础展示

```typescript
export const Default = () => <Button>Click me</Button>;
```

#### 2. 多种状态

```typescript
export const Primary = () => <Button variant="primary">Primary</Button>;
export const Secondary = () => <Button variant="secondary">Secondary</Button>;
export const Disabled = () => <Button disabled>Disabled</Button>;
```

#### 3. 交互式控件

```typescript
export const Interactive = {
  args: {
    label: "Button",
    primary: false,
    size: "medium",
  },
  argTypes: {
    size: {
      control: { type: "select" },
      options: ["small", "medium", "large"],
    },
  },
};
```

## 🎨 Storybook 插件功能

### 1. Controls 插件

- 实时调整组件属性
- 支持文本、数字、布尔值、选择器等控件
- 自动从 TypeScript 类型生成控件

### 2. Actions 插件

- 记录组件事件（点击、输入等）
- 在 Actions 面板中查看事件日志
- 调试组件交互行为

### 3. Backgrounds 插件

- 切换不同背景色
- 测试组件在不同背景下的表现
- 支持自定义背景色

## 🔧 开发工作流

### 1. 组件开发流程

```bash
# 1. 启动 Storybook
pnpm storybook:dev

# 2. 创建组件文件
# components/MyComponent.tsx

# 3. 创建 Story 文件
# components/MyComponent.stories.tsx

# 4. 在 Storybook 中预览和调试

# 5. 完善组件和 Stories
```

### 2. 最佳实践

#### Story 组织

- 每个组件至少有一个基础 Story
- 为不同状态创建独立 Stories
- 使用描述性的 Story 名称
- 按功能模块组织 Stories

#### 组件文档

```typescript
export default {
  title: "Design System/Button",
  component: Button,
  parameters: {
    docs: {
      description: {
        component: "这是一个通用按钮组件，支持多种样式和状态。",
      },
    },
  },
  argTypes: {
    variant: {
      description: "按钮样式变体",
      control: "select",
      options: ["primary", "secondary", "danger"],
    },
  },
};
```

## 🧪 测试和调试

### 1. 视觉测试

Webstudio 使用 **Lost Pixel** 进行视觉回归测试：

```bash
# 运行视觉测试
VISUAL_TESTING=true pnpm storybook:build
```

### 2. 交互测试

```typescript
// 在 Story 中添加交互测试
export const WithInteraction = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole("button");

    await userEvent.click(button);
    await expect(canvas.getByText("Clicked!")).toBeInTheDocument();
  },
};
```

### 3. 可访问性测试

```typescript
// 使用 a11y 插件检查可访问性
export default {
  parameters: {
    a11y: {
      config: {
        rules: [
          {
            id: "color-contrast",
            enabled: true,
          },
        ],
      },
    },
  },
};
```

## 🎯 实际使用场景

### 1. 设计系统维护

- 统一查看所有组件
- 确保设计一致性
- 快速发现样式问题

### 2. 组件开发

- 独立开发新组件
- 测试不同属性组合
- 验证边界情况

### 3. 文档生成

- 自动生成组件文档
- 展示使用示例
- 提供交互式演示

### 4. 团队协作

- 设计师查看组件实现
- 产品经理了解功能状态
- 开发者共享组件库

## 🚨 常见问题

### 1. Stories 不显示

**原因**: 文件路径或命名不符合配置

**解决方案**:

```bash
# 检查文件是否匹配 stories 配置
# 确保文件名包含 .stories.tsx
# 检查目录路径是否正确
```

### 2. 组件样式异常

**原因**: 样式依赖或主题配置缺失

**解决方案**:

```typescript
// 在 .storybook/preview.tsx 中添加全局样式
import '../path/to/global.css';

export const decorators = [
  (Story) => (
    <ThemeProvider>
      <Story />
    </ThemeProvider>
  ),
];
```

### 3. 路由相关组件报错

**原因**: 组件依赖路由上下文

**解决方案**:

```typescript
// 使用路由装饰器
const withRouter = (Story) => (
  <MemoryRouter>
    <Story />
  </MemoryRouter>
);

export const decorators = [withRouter];
```

## 📚 相关资源

- [Storybook 官方文档](https://storybook.js.org/docs)
- [React Storybook 教程](https://storybook.js.org/tutorials/intro-to-storybook/react/en/get-started/)
- [项目设置指南](./setup-guide.md)
- [数据库配置](./database-setup.md)

## 🎉 总结

Storybook 是 Webstudio 项目中重要的开发工具，它帮助我们：

- **提高开发效率** - 独立开发和测试组件
- **保证质量** - 视觉测试和交互测试
- **改善协作** - 统一的组件展示平台
- **维护文档** - 自动生成的交互式文档

通过合理使用 Storybook，可以显著提升组件开发的质量和效率。
