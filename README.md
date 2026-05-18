# Prompt Manager

⚡ 专为 AI 辅助编程设计的 Prompt 模板管理桌面应用，支持变量填充，让开发者在任何 AI 编码场景下秒级调用高质量 Prompt。

## 功能特性

- **🔍 模板搜索** - 快速搜索和筛选 Prompt 模板，收藏置顶显示
- **⚙️ 模板管理** - 创建、编辑、删除自定义 Prompt 模板
- **📝 变量填充** - 支持 `{{变量名}}` 格式，自动渲染填充表单
- **📋 一键复制** - 填充完成后自动复制到剪贴板
- **🏷️ 分类标签** - 按分类组织模板，支持标签筛选
- **⭐ 收藏功能** - 常用模板收藏置顶

## 预置模板

| 名称 | 分类 | 用途 |
|------|------|------|
| 通用代码审查 | Code Review | 审查代码质量 |
| 错误排查 | Debug | 排查和修复 bug |
| 重构组件 | Refactor | 优化代码结构 |
| 生成单元测试 | Test | 编写测试用例 |
| 解释代码 | Explain | 理解代码逻辑 |

## 技术栈

- **前端**: React 19 + TypeScript + Vite
- **桌面框架**: Tauri 2 (Rust)
- **状态管理**: Zustand
- **样式**: 原生 CSS

## 开发

### 环境要求

- Node.js 18+
- Rust (通过 [rustup](https://rustup.rs/) 安装)
- macOS 10.15+ / Windows 10+ / Linux

### 安装依赖

```bash
cd prompt-app
npm install
```

### 开发模式

```bash
npm run tauri:dev
```

### 构建生产版本

```bash
npm run tauri:build
```

构建产物位于 `src-tauri/target/release/bundle/` 目录下。

## 项目结构

```
prompt-app/
├── src/
│   ├── components/     # React 组件
│   ├── data/          # 默认模板数据
│   ├── store/         # Zustand 状态管理
│   ├── types/         # TypeScript 类型定义
│   └── utils/         # 工具函数
├── src-tauri/         # Tauri 后端 (Rust)
│   ├── src/
│   └── tauri.conf.json
└── package.json
```

## 数据结构

### PromptTemplate

```typescript
interface PromptTemplate {
  id: string;                    // UUID
  name: string;                  // 显示名称
  description: string;           // 简短描述
  category: string;              // 分类 ID
  tags: string[];                // 标签列表
  template: string;              // 模板内容（含 {{变量}} 占位符）
  variables: PromptVariable[];   // 变量定义
  favorite: boolean;             // 是否收藏
  createdAt: string;             // 创建时间
  updatedAt: string;             // 更新时间
}
```

### PromptVariable

```typescript
interface PromptVariable {
  name: string;                              // 变量名
  label: string;                             // 表单显示标签
  type: "text" | "select" | "code";          // 变量类型
  options?: string[];                        // select 类型的选项
  default?: string;                          // 默认值
  placeholder?: string;                      // 输入框占位文字
}
```

## 快捷键

- `⌘N` - 新建模板（管理页面）
- `⌘F` - 切换收藏

## License

MIT
