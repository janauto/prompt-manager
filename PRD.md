# Prompt Manager — 完整产品需求文档 (PRD)

> 专为 AI 辅助编程设计的 Prompt 模板管理工具，支持变量填充 + 截屏 OCR 自动预填。

---

## 一、产品定位

**一句话描述：** 全局可用的 Prompt 模板管理器，支持变量填充 + 截屏 OCR 自动预填，让开发者在任何 AI 编码场景下秒级调用高质量 Prompt。

**目标用户：** 个人开发者，日常使用 VS Code + 多种 AI IDE（Codex、ChatGPT、Claude 网页端）

**产品形态：** Raycast 插件（macOS）

---

## 二、技术栈

- **平台：** Raycast（macOS 效率工具）
- **语言：** TypeScript + React
- **SDK：** `@raycast/api` (最新版)
- **工具库：** `@raycast/utils`
- **数据存储：** 本地 JSON 文件（存储在 `environment.supportPath` 下）
- **Node.js：** 使用 nvm 切换到 Node v20 LTS（v20.19.6 已安装）
- **图标：** 需要在 `assets/` 目录下提供 `extension-icon.png`（128x128 PNG）

**重要：** 开发时必须使用 Node v20（`nvm use 20`），不要使用 Node v25，否则 npm 会导致 node_modules 损坏。

---

## 三、package.json 配置格式

Raycast 扩展的 `package.json` 必须包含以下结构：

```json
{
  "name": "prompt-manager",
  "version": "1.0.0",
  "description": "专为 AI 辅助编程设计的 Prompt 模板管理工具",
  "author": "linxiansheng",
  "license": "MIT",
  "icon": "extension-icon.png",
  "scripts": {
    "build": "ray build",
    "dev": "ray develop",
    "lint": "ray lint",
    "fix-lint": "ray lint --fix",
    "publish": "npx @raycast/api@latest publish"
  },
  "dependencies": {
    "@raycast/api": "^1.104.16",
    "@raycast/utils": "^2.2.4",
    "uuid": "^14.0.0"
  },
  "devDependencies": {
    "@types/node": "^20.17.0",
    "@types/react": "^18.3.0",
    "@types/uuid": "^10.0.0",
    "typescript": "^5.6.0"
  },
  "raycast": {
    "commands": [
      {
        "name": "search-prompts",
        "title": "Search Prompts",
        "subtitle": "Prompt Manager",
        "description": "搜索并使用 Prompt 模板",
        "mode": "view",
        "icon": "extension-icon.png"
      },
      {
        "name": "manage-prompts",
        "title": "Manage Prompts",
        "subtitle": "Prompt Manager",
        "description": "管理 Prompt 模板（增删改）",
        "mode": "view",
        "icon": "extension-icon.png"
      }
    ]
  }
}
```

**注意：**
- `commands` 必须放在 `raycast` 字段下（不是顶层）
- 每个 command 的 `icon` 字段是可选的，但顶层 `icon` 字段是必须的
- `mode` 可以是 `"view"` 或 `"no-view"`

---

## 四、tsconfig.json 配置

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "include": ["src/**/*", "raycast-env.d.ts"],
  "compilerOptions": {
    "allowJs": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "lib": ["ES2022"],
    "module": "ES2022",
    "moduleResolution": "bundler",
    "noEmit": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "strict": true,
    "target": "ES2022",
    "types": ["node"]
  }
}
```

---

## 五、功能需求

### P0 — MVP 核心功能

#### 1. Prompt 模板搜索与使用（search-prompts 命令）

- 列出所有 Prompt 模板，支持按名称/标签搜索
- 支持按分类筛选（下拉菜单）
- 收藏的 Prompt 置顶显示
- 点击 Prompt 后：
  - 如果没有变量：直接复制模板到剪贴板
  - 如果有变量：跳转到变量填充表单，填完后复制组装好的文本到剪贴板
- 支持快捷键：
  - `Cmd+F` 切换收藏
  - `Cmd+Shift+C` 直接复制原始模板

#### 2. Prompt 模板管理（manage-prompts 命令）

- 列出所有模板，支持搜索
- `Cmd+N` 新建模板
- 点击模板进入编辑
- `Cmd+Backspace` 删除模板（带确认弹窗）
- 编辑表单包含：名称、描述、分类、标签（逗号分隔）、模板内容

#### 3. 变量填充表单

- 模板中使用 `{{变量名}}` 格式定义变量
- 调用时自动渲染表单，变量类型支持：
  - `text`：单行文本输入（Form.TextField）
  - `select`：下拉选择（Form.Dropdown）
  - `code`：多行代码输入（Form.TextArea）
- 变量可设默认值和 placeholder
- 表单底部显示组装后文本预览
- 提交后复制最终文本到剪贴板

#### 4. 内置默认 Prompt 模板

提供以下预置模板：

| ID | 名称 | 分类 | 变量 |
|----|------|------|------|
| code-review-general | 通用代码审查 | Code Review | 语言(select), 代码内容(code) |
| debug-error | 错误排查 | Debug | 错误信息(code), 语言(select), 相关代码(code), 已尝试方法(text) |
| refactor-component | 重构组件 | Refactor | 语言(select), 设计原则(select), 代码内容(code) |
| test-unit | 生成单元测试 | Test | 语言(select), 测试框架(select), 覆盖率目标(select), 代码内容(code) |
| explain-code | 解释代码 | Explain | 语言(select), 代码内容(code) |

---

## 六、数据结构

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
  hotkey?: string;               // 可选快捷键
  favorite: boolean;             // 是否收藏
  createdAt: string;             // ISO 时间戳
  updatedAt: string;             // ISO 时间戳
}
```

### PromptVariable

```typescript
interface PromptVariable {
  name: string;           // 变量名（与模板中 {{变量名}} 对应）
  label: string;          // 表单显示标签
  type: "text" | "select" | "code";  // 变量类型
  options?: string[];     // select 类型的选项列表
  default?: string;       // 默认值
  placeholder?: string;   // 输入框占位文字
}
```

### Category

```typescript
interface Category {
  id: string;       // 分类 ID
  name: string;     // 显示名称
  icon: string;     // emoji 图标
  parentId?: string; // 父分类 ID（支持多级目录）
}
```

---

## 七、项目目录结构

```
prompt-manager/
├── PRD.md                      # 本文档
├── package.json                # Raycast 扩展配置
├── tsconfig.json               # TypeScript 配置
├── raycast-env.d.ts            # Raycast 类型声明
├── assets/
│   └── extension-icon.png      # 扩展图标（128x128 PNG）
├── src/
│   ├── search-prompts.tsx      # 搜索命令入口
│   ├── manage-prompts.tsx      # 管理命令入口
│   ├── components/
│   │   ├── PromptForm.tsx      # 变量填充表单
│   │   └── utils.ts            # 工具函数（copyToClipboard 等）
│   ├── lib/
│   │   ├── storage.ts          # 数据读写（JSON 文件）
│   │   └── template.ts         # 模板解析与变量替换
│   ├── data/
│   │   └── default-prompts.ts  # 内置默认 Prompt 模板
│   └── types/
│       └── index.ts            # TypeScript 类型定义
└── README.md
```

---

## 八、关键实现要点

### 1. 数据存储

- 使用 `environment.supportPath` 获取 Raycast 的数据目录
- 首次运行时，如果数据文件不存在，自动写入默认 Prompt 模板
- 读写操作使用 `fs.readFileSync` / `fs.writeFileSync`

### 2. 模板解析

- 使用正则 `/\{\{([^}]+)\}\}/g` 提取模板中的变量名
- `renderTemplate(template, values)` 函数将变量值替换回模板中
- `getDefaultValues(variables)` 函数为变量生成默认值

### 3. Raycast API 使用

- `List` + `List.Item` 显示 Prompt 列表
- `List.Dropdown` 实现分类筛选
- `List.Section` 分组显示（收藏 / 全部）
- `ActionPanel` + `Action` 定义操作菜单
- `Form` + `Form.TextField` / `Form.Dropdown` / `Form.TextArea` 实现表单
- `useNavigation` 的 `push` / `pop` 实现页面跳转
- `Clipboard.copy` 复制到剪贴板
- `showToast` 显示操作反馈
- `confirmAlert` 删除确认弹窗

### 4. 图标

- 需要在 `assets/` 目录下提供 `extension-icon.png`
- 尺寸：128x128 像素，PNG 格式
- 可以用 SVG 转 PNG：`sips -s format png icon.svg --out extension-icon.png`

---

## 九、开发步骤建议

1. **初始化项目：** 用 `create-raycast-extension` CLI 或手动创建，确保 Node v20
2. **安装依赖：** `npm install @raycast/api @raycast/utils uuid && npm install -D typescript @types/react @types/node @types/uuid`
3. **创建类型定义：** `src/types/index.ts`
4. **创建数据层：** `src/lib/storage.ts` + `src/lib/template.ts`
5. **创建默认模板：** `src/data/default-prompts.ts`
6. **创建搜索命令：** `src/search-prompts.tsx`
7. **创建管理命令：** `src/manage-prompts.tsx`
8. **创建表单组件：** `src/components/PromptForm.tsx`
9. **创建图标：** `assets/extension-icon.png`
10. **构建测试：** `npm run dev` 启动开发模式，`npm run build` 构建

---

## 十、已知问题

之前的开发过程中遇到了以下问题，需要注意：

1. **Node v25 + npm 11 会导致 node_modules 损坏** —— 必须使用 Node v20
2. **`npx ray build` 报 `TypeError: Cannot read properties of undefined (reading 'map')`** —— 可能与 package.json 格式或依赖版本有关，建议用 `create-raycast-extension` 工具从模板创建项目，然后替换 src 代码
3. **`npx ray lint` 报 icon 相关错误** —— 确保 `assets/extension-icon.png` 存在且是有效 PNG

---

## 十一、后续功能（P1）

- 截屏 OCR 自动预填（全局快捷键 `Cmd+Shift` 触发）
- 上下文自动抓取（抓取剪贴板代码填入变量）
- Prompt 历史版本管理
- JSON 格式导入/导出
- 最近使用记录
