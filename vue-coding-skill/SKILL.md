---
name: vue-coding-skill
description: 个人 Vue 3 前端项目初始化与编码规范。创建、初始化、修改、重构或扩展 Vue 3 项目时使用，遵循 Vue 3 + JavaScript、Composition API、Hash Router、Pinia、Axios、Element Plus、多环境变量与构建命令等偏好。
---

# Vue 3 个人编码规范

## 1. 核心原则

- 默认使用 Vue 3 + JavaScript。
- 默认使用 Composition API 和 `<script setup>`。
- 新增代码遵循本 Skill。
- 修改已有项目时优先保持现有架构，只修改完成需求所必要的代码。
- 不擅自安装依赖、改变目录结构或进行无关重构。
- 避免过度封装、过度拆分和无意义的架构层。

## 2. 项目初始化

### 创建项目

```bash
npm create vue@latest
```

创建过程中所有可选功能全部选择 `No`，不要主动选择 TypeScript、JSX、Vue Router、Pinia、Vitest、ESLint、Prettier 等功能。

进入项目目录后：

```bash
npm install
```

### 安装 Vue 生态

```bash
npm install vue-router
npm install pinia
```

### 安装项目依赖

```bash
npm install axios
npm install element-plus
```

除非用户明确要求，否则不要额外安装其他依赖。

初始化完成后，按第 4、5、6 节配置环境变量、Git 忽略规则和构建命令。

## 3. 目录结构

新项目根目录使用：

```text
project/
├── .env.dev
├── .env.pre
├── .env.prd
├── .gitignore
├── index.html
├── package.json
├── vite.config.js
├── README.md
└── src/
    ├── assets/
    ├── components/
    ├── router/
    │   └── index.js
    ├── stores/
    ├── utils/
    │   └── requests.js
    ├── views/
    ├── App.vue
    └── main.js
```

`src/` 职责：

- `assets/`：图片、字体、全局样式等静态资源。
- `components/`：可复用公共组件。
- `router/`：Vue Router 配置。
- `stores/`：Pinia Store。
- `utils/`：Axios 实例及通用工具。
- `views/`：页面级组件。

不要为了简单功能创建大量额外目录。

## 4. 环境变量文件

在项目根目录创建以下文件：

```text
.env.dev
.env.pre
.env.prd
```

分别表示：

- `.env.dev`：开发环境
- `.env.pre`：预发布环境
- `.env.prd`：生产环境

Vite 仅会将以 `VITE_` 开头的变量暴露给前端代码。

示例：

```env
VITE_APP_ENV=dev
VITE_API_BASE_URL=http://127.0.0.1:8000
```

各环境示例：

`.env.dev`：

```env
VITE_APP_ENV=dev
VITE_API_BASE_URL=http://127.0.0.1:8000
```

`.env.pre`：

```env
VITE_APP_ENV=pre
VITE_API_BASE_URL=https://pre-api.example.com
```

`.env.prd`：

```env
VITE_APP_ENV=prd
VITE_API_BASE_URL=https://api.example.com
```

规则：

- 前端环境变量统一使用 `VITE_` 前缀。
- API 地址统一使用 `VITE_API_BASE_URL`。
- 不在业务代码中硬编码 `baseURL`。
- 包含密钥、Token 等敏感信息的环境变量文件不得提交到 Git。
- 修改已有项目时，优先沿用现有环境变量命名和文件结构。

## 5. Git 配置

创建 `.gitignore`：

```gitignore
.DS_Store
.idea

node_modules
dist
dist-ssr

*.local

.env.local
.env.*.local
```

修改已有项目时，除非用户明确要求，否则保留现有 `.gitignore`。

`.env.dev`、`.env.pre`、`.env.prd` 可提交非敏感配置；本地覆盖文件使用 `.env.*.local`，且不得提交。

## 6. 构建命令

在 `package.json` 中配置：

```json
{
  "scripts": {
    "dev": "vite --mode dev",
    "build": "vite build --mode prd",
    "build:pre": "vite build --mode pre",
    "preview": "vite preview"
  }
}
```

命令说明：

- `npm run dev`：启动开发服务器，加载 `.env.dev`。
- `npm run build`：生产构建，加载 `.env.prd`，产物输出到 `dist/`。
- `npm run build:pre`：预发布构建，加载 `.env.pre`。
- `npm run preview`：本地预览 `dist/` 构建产物。

规则：

- 开发、预发布、生产环境通过 `--mode` 切换，不在代码中写死环境判断。
- 构建前确认对应环境的 `.env.*` 文件已正确配置。
- 不擅自修改 Vite 默认输出目录，除非用户明确要求。

## 7. App.vue

新项目将默认 `App.vue` 替换为：

```vue
<template>
  <h1>hello,欢迎来到新的Vue项目</h1>
</template>

<script setup>
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

html,
body {
  width: 100vw;
  height: 100vh;
  box-sizing: border-box;
  overflow: hidden;
}

#app {
  width: 100vw;
  height: 100vh;
}
</style>
```

删除 Vue 初始化项目自带的无关 Demo 组件、页面和样式。

## 8. main.js

```js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'

import App from './App.vue'
import router from '@/router/index.js'

const app = createApp(App)
const pinia = createPinia()

app.use(pinia)
app.use(router)
app.use(ElementPlus)

app.mount('#app')
```

Pinia 只能创建一次，Element Plus 和 Router 必须正确注册。

## 9. Router

`src/router/index.js`：

```js
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
  history: createWebHashHistory(),
  routes: [

  ],
})

export default router
```

规则：

- 默认使用 Hash Router。
- 主路由统一维护在 `router/index.js`。
- 页面组件默认懒加载。
- `path` 使用小写或 kebab-case。
- `name` 使用 PascalCase。
- 小型项目不主动拆分 `router/modules`。

推荐：

```js
{
  path: '/evaluation-result',
  name: 'EvaluationResult',
  component: () => import('@/views/EvaluationResult.vue'),
}
```

## 10. Axios / Requests

`src/utils/requests.js`：

```js
import axios from "axios";

export const baseURL = import.meta.env.VITE_API_BASE_URL;

const requests = axios.create({
    baseURL: baseURL,
    timeout: 12000,
    headers: {'X-Custom-Header': 'foobar'}
});

requests.interceptors.response.use(function (response) {
    return response;
}, function (error) {
    return Promise.reject(error);
});

export default requests;
```

规则：

- 所有业务 HTTP 请求统一使用 `requests`。
- 页面和 Store 不重复创建 Axios 实例。
- 业务代码不直接使用 `axios.get()`、`axios.post()`。
- 不混用 Axios、Fetch 和其他请求库。
- `baseURL` 从 `import.meta.env.VITE_API_BASE_URL` 读取，timeout、拦截器统一在 `requests.js` 管理。

使用：

```js
import requests from '@/utils/requests.js'
```

## 11. Vue SFC

所有 `.vue` 文件必须使用：

```vue
<template>

</template>

<script setup>

</script>

<style>

</style>
```

强制：

1. `template` 第一。
2. `script setup` 第二。
3. `style` 第三。
4. 默认 Composition API。
5. 不主动使用 Options API。

## 12. Vue 函数

Vue 文件中自行定义的业务函数只允许箭头函数。

正确：

```js
// 获取任务列表
const getTaskList = async () => {

}

// 查看任务详情
const viewTask = (taskId) => {

}
```

禁止：

```js
function getTaskList() {}
async function getTaskList() {}
```

所有自行定义的业务函数上方必须有简短中文注释说明用途，不要求简单函数使用复杂 JSDoc。

## 13. script setup 内部顺序

推荐顺序：

```js
// 1. import
import { computed, onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'

import useTaskStore from '@/stores/task.js'

// 2. Store
const taskStore = useTaskStore()

// 3. Router
const router = useRouter()

// 4. 响应式变量
const loading = ref(false)
const taskList = ref([])

// 5. computed
const taskCount = computed(() => taskList.value.length)

// 6. 业务函数

// 获取任务列表
const getTaskList = async () => {

}

// 7. 生命周期
onMounted(() => {
  getTaskList()
})
```

同类代码集中，不随意交叉排列。

## 14. Pinia

默认使用 Options Store：

```js
import { defineStore } from 'pinia'

const useXxxStore = defineStore('xxx', {
  state: () => ({

  }),
  actions: {

  }
})

export default useXxxStore
```

Store 命名统一：

```text
useUserStore
useTaskStore
useDatasetStore
useResultStore
```

Store 主要负责跨组件/跨页面共享状态及相关业务操作。单页面临时状态不强制放入 Store。

### 禁止 Store 顶部辅助函数

禁止：

```js
const formatTask = (task) => {
  return task
}

const useTaskStore = defineStore('task', {
})
```

与 Store 状态相关的逻辑放入 `actions`；真正通用的纯工具函数放入 `src/utils/`。

## 15. Components

- 多页面复用组件放 `components/`。
- 页面专属简单内容不强制拆组件。
- 不为了拆组件而拆组件。
- 文件名使用 PascalCase，且具有业务含义。

推荐：

```text
TaskTable.vue
DatasetPreview.vue
MemoryTable.vue
StatusTag.vue
```

避免 `Test.vue`、`Temp.vue`、`New.vue`、`Component1.vue`。

## 16. Views

页面级组件统一放 `views/`：

```text
Home.vue
Login.vue
Dataset.vue
Evaluation.vue
EvaluationResult.vue
```

页面负责页面布局、页面级状态、Store/请求调用、组件组合和页面交互。

## 17. 命名

Vue 文件使用 PascalCase。

JavaScript 变量和函数使用 camelCase：

```js
const taskList = []
const currentPage = 1
const getTaskList = async () => {}
```

Store 使用 `useXxxStore`。

名称必须表达业务含义，避免无意义缩写和 `test1`、`data1`、`temp`、`aaa` 等正式命名。

## 18. Import

Import 位于 `<script setup>` 最上方，推荐顺序：

1. Vue 核心。
2. Vue Router / Pinia / Element Plus 等第三方依赖。
3. Store。
4. Components。
5. Utils。
6. 其他本地模块。

优先使用 `@/`：

```js
import { onMounted, ref } from 'vue'
import { ElMessage } from 'element-plus'

import useTaskStore from '@/stores/task.js'
import TaskTable from '@/components/TaskTable.vue'
import requests from '@/utils/requests.js'
```

避免不必要的多层 `../../../` 相对路径。

## 19. CSS

推荐：

```css
.task-card {
  width: 100%;
  padding: 20px;
  border-radius: 8px;
}
```

规则：

- `{` 前留空格。
- 一个属性一行。
- 冒号后留空格。
- 优先 class。
- 避免大量内联 style。
- 页面私有样式放当前 Vue 文件。
- 公共/全局样式放 `assets/`。
- 不主动引入额外 CSS 框架。
- `scoped` 根据现有项目风格和实际需求决定。

## 20. Element Plus

已有 Element Plus 时优先使用成熟基础组件：

- 按钮：`el-button`
- 表格：`el-table`
- 分页：`el-pagination`
- 弹窗：`el-dialog`
- 表单：`el-form`
- 输入：`el-input`
- 下拉：`el-select`
- 消息：`ElMessage`
- 确认：`ElMessageBox`

不要重复手写 Element Plus 已成熟解决的复杂基础组件，也不要把适合简单 HTML/CSS 的结构强行 Element Plus 化。

## 21. 异步代码

HTTP 请求优先 `async/await`：

```js
// 获取任务列表
const getTaskList = async () => {
  const response = await requests.get('/task_list')
  taskList.value = response.data.data
}
```

- 避免无必要的 Promise 链式嵌套。
- 请求失败不要静默吞异常。
- 合理维护 loading。
- 不创建无实际价值的异步包装函数。

## 22. 错误处理

用户需要感知的请求失败可以使用 Element Plus：

```js
// 获取任务列表
const getTaskList = async () => {
  try {
    const response = await requests.get('/task_list')
    taskList.value = response.data.data
  } catch (error) {
    console.error(error)
    ElMessage.error('获取任务列表失败')
  }
}
```

- 禁止空 `catch`。
- 不完全吞掉异常。
- 用户可感知错误给出明确提示。
- 不在正式代码保留大量无意义 `console.log()`。

## 23. README 规范

新项目必须在项目根目录创建 `README.md`，并按以下顺序组织内容：

1. **项目背景**：说明项目是什么、解决什么问题。
2. **技术栈**：Vue 3、Vue Router、Pinia、Axios、Element Plus、Vite。
3. **目录结构**：介绍 `src/` 各目录职责。
4. **环境变量**：说明 `.env.dev`、`.env.pre`、`.env.prd` 及主要变量。
5. **快速开始**：安装依赖、启动开发服务器。
6. **构建命令**：dev、build、build:pre、preview。
7. **后端接口**：说明 API 基地址由 `VITE_API_BASE_URL` 配置。

推荐结构：

````markdown
# 项目名称

## 项目背景

简要说明项目用途。

## 技术栈

- Vue 3
- Vue Router
- Pinia
- Axios
- Element Plus
- Vite

## 目录结构

```text
src/
├── assets/
├── components/
├── router/
├── stores/
├── utils/
└── views/
```

## 环境变量

| 文件 | 说明 |
|------|------|
| `.env.dev` | 开发环境 |
| `.env.pre` | 预发布环境 |
| `.env.prd` | 生产环境 |

主要变量：

- `VITE_APP_ENV`：当前环境标识
- `VITE_API_BASE_URL`：后端 API 地址

## 快速开始

```bash
npm install
npm run dev
```

## 构建命令

```bash
npm run dev
npm run build
npm run build:pre
npm run preview
```

## 后端接口

后端 API 基地址由 `VITE_API_BASE_URL` 配置。
````

规则：

- README 使用中文。
- 包含项目背景、技术栈、目录结构、环境变量、启动和构建命令。
- 不写入密钥、Token 等敏感信息。
- 修改已有项目时，除非用户明确要求，否则不覆盖已有 README。

## 24. 修改已有项目

1. 修改前先理解相关目录、组件、Store、Router 和请求逻辑。
2. 优先遵循已有架构。
3. 只修改完成当前需求所必要的代码。
4. 不顺手重构无关代码。
5. 不擅自更换依赖。
6. 不擅自改变目录结构。
7. 不因为认为其他写法更优而大规模改造。
8. 新增代码遵循本 Skill。
9. 修改公共组件、Store、Router、`requests.js` 前考虑其他引用。
10. 删除代码或文件前确认没有其他引用。
11. 不为了格式统一制造大量无关 diff。
12. 用户明确要求重构时再进行结构调整。

## 25. 代码位置决策

- 页面专属 UI / 状态 → `views/`
- 可复用 UI → `components/`
- 跨页面共享状态 → `stores/`
- 路由 → `router/`
- Axios / 通用工具 → `utils/`

简单功能不需要额外抽象时，不为了架构完整性创建新层。

## 26. 禁止事项

除非用户明确要求，否则禁止：

- 在业务代码中硬编码 API 地址。
- 擅自使用 TypeScript。
- 擅自加入 ESLint / Prettier。
- 擅自安装未要求的依赖。
- 擅自将 Hash Router 改为 History Router。
- 擅自使用 Options API。
- Vue 业务代码使用普通 `function` 声明自行定义函数。
- Pinia Store 顶部定义业务辅助函数。
- 业务代码重复创建 Axios 实例。
- 混用 Axios、Fetch 等请求方式。
- 擅自改变已有目录结构。
- 简单功能拆出大量无意义组件。
- 简单项目创建大量架构层。
- 重构当前任务无关代码。
- 删除仍被其他模块引用的代码。
- 保留 Vue 初始化项目无用 Demo。
- 使用无业务含义的正式文件名、变量名或函数名。
- 正式代码保留大量调试 `console.log()`。

## 27. 执行优先级

1. 用户当前明确要求。
2. 当前项目架构和业务约束。
3. 本 Skill。
4. 通用 Vue 开发惯例。

如果用户当前明确要求与本 Skill 不同，以用户当前要求为准。
