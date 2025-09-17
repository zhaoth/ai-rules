# Vue 3 + TailwindCSS 开发规范

你是 TypeScript、Node.js、Vite、Vue.js、Vue Router、Pinia、VueUse、Headless UI、Element Plus 和 TailwindCSS 的专家，深度理解这些技术的最佳实践和性能优化技巧。

## 代码风格和结构

- 编写简洁、可维护、技术准确的 TypeScript 代码，提供相关示例
- 使用函数式和声明式编程模式，避免使用类
- 优先使用迭代和模块化，遵循 DRY 原则，避免代码重复
- 使用描述性变量名，带有辅助动词（如 isLoading、hasError）
- 系统化组织文件：每个文件只包含相关内容，如导出组件、子组件、辅助函数、静态内容和类型

## 命名约定

- 目录使用小写加短横线（如 components/auth-wizard）
- 函数优先使用命名导出

## TypeScript 使用

- 所有代码使用 TypeScript；优先使用 interface 而非 type，因为其可扩展性和合并能力
- 避免使用枚举；使用映射对象以获得更好的类型安全性和灵活性
- 使用带有 TypeScript 接口的函数组件

## 语法和格式

- 纯函数使用 "function" 关键字，以利用提升和清晰性
- 始终使用 Vue Composition API 的 script setup 风格

## UI 和样式

- 使用 Headless UI、Element Plus 和 TailwindCSS 进行组件和样式设计
- 使用 TailwindCSS 实现响应式设计；采用移动优先方法

## 性能优化

- 在适用的地方使用 VueUse 函数来增强响应性和性能
- 使用 Suspense 包装异步组件，提供回退 UI
- 对非关键组件使用动态加载
- 优化图片：使用 WebP 格式，包含尺寸数据，实现懒加载
- 在 Vite 构建过程中实现优化的分块策略，如代码分割，以生成更小的包大小

## 核心约定

- 使用 Lighthouse 或 WebPageTest 等工具优化 Web Vitals（LCP、CLS、FID）

## Vue.js 最佳实践

### 组件结构
```vue
<template>
  <div class="组件根元素类名">
    <!-- 使用语义化 HTML -->
    <header v-if="showHeader">
      <h1>{{ title }}</h1>
    </header>
    <main>
      <slot />
    </main>
  </div>
</template>

<script setup lang="ts">
// 导入
import { ref, computed, onMounted } from 'vue'
import type { ComponentProps } from './types'

// Props 定义
interface Props {
  title?: string
  showHeader?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  title: '',
  showHeader: true
})

// Emits 定义
const emit = defineEmits<{
  update: [value: string]
  close: []
}>()

// 响应式状态
const isVisible = ref(false)
const count = ref(0)

// 计算属性
const displayText = computed(() => 
  `${props.title} - ${count.value}`
)

// 方法
function handleClick() {
  count.value++
  emit('update', displayText.value)
}

// 生命周期
onMounted(() => {
  isVisible.value = true
})
</script>
```

### 状态管理 (Pinia)
```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
  // 状态
  const user = ref<User | null>(null)
  const isLoading = ref(false)

  // 计算属性
  const isAuthenticated = computed(() => !!user.value)
  const userName = computed(() => user.value?.name ?? '游客')

  // 操作
  async function login(credentials: LoginCredentials) {
    isLoading.value = true
    try {
      user.value = await authApi.login(credentials)
    } finally {
      isLoading.value = false
    }
  }

  function logout() {
    user.value = null
  }

  return {
    user: readonly(user),
    isLoading: readonly(isLoading),
    isAuthenticated,
    userName,
    login,
    logout
  }
})
```

### 组合式函数 (Composables)
```typescript
// composables/useApi.ts
import { ref } from 'vue'

export function useApi<T>(apiCall: () => Promise<T>) {
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  const isLoading = ref(false)

  async function execute() {
    isLoading.value = true
    error.value = null
    
    try {
      data.value = await apiCall()
    } catch (err) {
      error.value = err as Error
    } finally {
      isLoading.value = false
    }
  }

  return {
    data: readonly(data),
    error: readonly(error),
    isLoading: readonly(isLoading),
    execute
  }
}
```

## TailwindCSS 最佳实践

### 响应式设计
```html
<!-- 移动优先 -->
<div class="w-full md:w-1/2 lg:w-1/3">
  <p class="text-sm md:text-base lg:text-lg">响应式文本</p>
</div>

<!-- 网格布局 -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
  <!-- 网格项 -->
</div>
```

### 组件样式
```css
/* 在 CSS 中定义可复用组件 */
@layer components {
  .btn {
    @apply px-4 py-2 rounded-lg font-medium transition-colors;
  }
  
  .btn-primary {
    @apply bg-blue-600 hover:bg-blue-700 text-white;
  }
  
  .card {
    @apply bg-white rounded-xl shadow-sm border p-6;
  }
}
```

## 项目结构

```
src/
├── components/          # 组件
│   ├── ui/             # 基础 UI 组件
│   ├── layout/         # 布局组件
│   └── business/       # 业务组件
├── composables/        # 组合式函数
├── stores/             # Pinia 状态管理
├── utils/              # 工具函数
├── types/              # 类型定义
├── views/              # 页面组件
└── assets/             # 静态资源
```

## 类型定义

```typescript
// types/api.ts
export interface ApiResponse<T> {
  data: T
  message: string
  success: boolean
}

export interface User {
  id: string
  name: string
  email: string
  avatar?: string
}

export interface PaginatedResponse<T> {
  items: T[]
  total: number
  page: number
  pageSize: number
}
```

## 性能优化技巧

### 懒加载组件
```typescript
// 路由懒加载
const Home = () => import('@/views/Home.vue')

// 组件懒加载
const HeavyComponent = defineAsyncComponent(() => 
  import('@/components/HeavyComponent.vue')
)
```

### 图片优化
```vue
<template>
  <img
    :src="optimizedImageSrc"
    :alt="imageAlt"
    class="w-full h-auto"
    loading="lazy"
    width="800"
    height="600"
  />
</template>

<script setup lang="ts">
const optimizedImageSrc = computed(() => 
  `${props.imageSrc}?format=webp&quality=80`
)
</script>
```

## 测试

```typescript
// 组件测试
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import Button from '@/components/ui/Button.vue'

describe('Button', () => {
  it('渲染正确的文本', () => {
    const wrapper = mount(Button, {
      props: { variant: 'primary' },
      slots: { default: '点击我' }
    })
    
    expect(wrapper.text()).toBe('点击我')
    expect(wrapper.classes()).toContain('btn-primary')
  })
})
```

| 技术栈 | 版本要求 | 说明 |
|--------|----------|------|
| **Vue** | `^3.4.0` | 使用最新稳定版，支持 Composition API |
| **TailwindCSS** | `^3.4.0` | 原子化 CSS 框架，支持 JIT 模式 |
| **TypeScript** | `^5.0.0` | 强类型支持，提升开发体验 |
| **Vite** | `^5.0.0` | 现代化构建工具，快速热更新 |

#### 🔧 配套工具

```json
{
  "vue-router": "^4.2.0",
  "pinia": "^2.1.0",
  "@vueuse/core": "^10.0.0",
  "autoprefixer": "^10.4.0",
  "postcss": "^8.4.0"
}
```

#### 🛠️ 开发工具

```json
{
  "@vitejs/plugin-vue": "^5.0.0",
  "@vue/eslint-config-typescript": "^12.0.0",
  "eslint": "^8.0.0",
  "prettier": "^3.0.0",
  "tailwindcss": "^3.4.0",
  "@tailwindcss/typography": "^0.5.0",
  "@tailwindcss/forms": "^0.5.0"
}
```

### 1.2 环境配置

#### Vite 配置 (`vite.config.ts`)

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@views': resolve(__dirname, 'src/views'),
      '@utils': resolve(__dirname, 'src/utils'),
      '@assets': resolve(__dirname, 'src/assets')
    }
  },
  css: {
    postcss: {
      plugins: [
        require('tailwindcss'),
        require('autoprefixer')
      ]
    }
  }
})
```

#### TailwindCSS 配置 (`tailwind.config.js`)

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        }
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      }
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
}
```

---

## 2. 项目结构规范

### 2.1 目录结构

```
src/
├── assets/                 # 静态资源
│   ├── images/            # 图片资源
│   ├── icons/             # 图标文件
│   └── styles/            # 全局样式
│       ├── main.css       # TailwindCSS 入口
│       ├── components.css # 组件样式
│       └── utilities.css  # 工具类样式
├── components/            # 可复用组件
│   ├── ui/               # 基础 UI 组件
│   │   ├── Button/
│   │   │   ├── Button.vue
│   │   │   ├── Button.types.ts
│   │   │   └── index.ts
│   │   └── Input/
│   ├── layout/           # 布局组件
│   │   ├── Header.vue
│   │   ├── Sidebar.vue
│   │   └── Footer.vue
│   └── business/         # 业务组件
├── composables/          # 组合式函数
│   ├── useAuth.ts
│   ├── useApi.ts
│   └── useTheme.ts
├── stores/               # Pinia 状态管理
│   ├── auth.ts
│   ├── user.ts
│   └── index.ts
├── utils/                # 工具函数
│   ├── helpers.ts
│   ├── constants.ts
│   └── validators.ts
├── views/                # 页面组件
│   ├── Home/
│   │   ├── Home.vue
│   │   └── components/
│   └── About/
├── router/               # 路由配置
│   └── index.ts
├── types/                # TypeScript 类型定义
│   ├── api.ts
│   ├── user.ts
│   └── global.ts
├── App.vue              # 根组件
└── main.ts              # 应用入口
```

### 2.2 组件目录规范

#### 🎯 组件命名规则

- **PascalCase** 命名：`UserProfile.vue`
- **文件夹组织**：复杂组件使用文件夹
- **索引文件**：提供统一导出

```
components/ui/Button/
├── Button.vue           # 主组件
├── Button.types.ts      # 类型定义
├── Button.stories.ts    # Storybook 故事
├── Button.test.ts       # 单元测试
└── index.ts            # 导出文件
```

#### 📁 组件分类

1. **UI 组件** (`components/ui/`)
   - 纯展示组件，无业务逻辑
   - 高度可复用
   - 完整的 props 类型定义

2. **布局组件** (`components/layout/`)
   - 页面布局结构
   - 响应式设计
   - 插槽支持

3. **业务组件** (`components/business/`)
   - 包含业务逻辑
   - 特定场景使用
   - 可能依赖状态管理

### 2.3 样式文件组织

#### 主样式文件 (`src/assets/styles/main.css`)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 自定义基础样式 */
@layer base {
  html {
    @apply scroll-smooth;
  }
  
  body {
    @apply bg-gray-50 text-gray-900 font-sans;
  }
}

/* 组件样式 */
@layer components {
  .btn-primary {
    @apply bg-primary-600 hover:bg-primary-700 text-white font-medium py-2 px-4 rounded-lg transition-colors duration-200;
  }
  
  .card {
    @apply bg-white rounded-xl shadow-sm border border-gray-200 p-6;
  }
}

/* 工具类样式 */
@layer utilities {
  .text-balance {
    text-wrap: balance;
  }
}
```

### 2.4 静态资源管理

#### 🖼️ 图片资源

```
assets/images/
├── common/              # 通用图片
│   ├── logo.svg
│   └── placeholder.svg
├── icons/               # 图标文件
│   ├── user.svg
│   └── settings.svg
└── backgrounds/         # 背景图片
    └── hero-bg.jpg
```

#### 📦 资源导入规范

```typescript
// 推荐：使用 import 导入
import logoSvg from '@/assets/images/common/logo.svg'
import userIcon from '@/assets/icons/user.svg'

// 动态导入
const getImage = (name: string) => {
  return new URL(`../assets/images/${name}`, import.meta.url).href
}
```

---

## 3. 编码规范

### 3.1 Vue 组件编写规范

#### 🎯 组件结构模板

```vue
<template>
  <div class="component-wrapper">
    <!-- 使用语义化的 HTML 结构 -->
    <header v-if="showHeader" class="component-header">
      <h2 class="text-xl font-semibold text-gray-900">
        {{ title }}
      </h2>
    </header>
    
    <main class="component-content">
      <slot />
    </main>
    
    <footer v-if="$slots.footer" class="component-footer">
      <slot name="footer" />
    </footer>
  </div>
</template>

<script setup lang="ts">
import { computed, defineProps, defineEmits } from 'vue'
import type { ComponentProps, ComponentEmits } from './Component.types'

// Props 定义
const props = withDefaults(defineProps<ComponentProps>(), {
  title: '',
  showHeader: true,
  variant: 'default'
})

// Emits 定义
const emit = defineEmits<ComponentEmits>()

// 响应式数据
const isActive = ref(false)

// 计算属性
const componentClasses = computed(() => [
  'component-wrapper',
  `component-wrapper--${props.variant}`,
  {
    'component-wrapper--active': isActive.value
  }
])

// 方法
const handleClick = () => {
  isActive.value = !isActive.value
  emit('toggle', isActive.value)
}

// 生命周期
onMounted(() => {
  console.log('Component mounted')
})
</script>

<style scoped>
/* 仅在必要时使用 scoped 样式 */
.component-wrapper {
  /* 优先使用 TailwindCSS，必要时补充自定义样式 */
}
</style>
```

#### 📝 类型定义 (`Component.types.ts`)

```typescript
export interface ComponentProps {
  title?: string
  showHeader?: boolean
  variant?: 'default' | 'primary' | 'secondary'
  disabled?: boolean
}

export interface ComponentEmits {
  toggle: [value: boolean]
  click: [event: MouseEvent]
}

export interface ComponentSlots {
  default(): any
  footer(): any
}
```

#### ✅ Vue 组件最佳实践

1. **使用 Composition API**
   ```typescript
   // ✅ 推荐
   <script setup lang="ts">
   import { ref, computed } from 'vue'
   
   const count = ref(0)
   const doubleCount = computed(() => count.value * 2)
   </script>
   
   // ❌ 避免使用 Options API（除非维护旧代码）
   ```

2. **Props 验证和默认值**
   ```typescript
   // ✅ 推荐：完整的类型定义和默认值
   interface Props {
     size?: 'sm' | 'md' | 'lg'
     disabled?: boolean
   }
   
   const props = withDefaults(defineProps<Props>(), {
     size: 'md',
     disabled: false
   })
   ```

3. **事件命名规范**
   ```typescript
   // ✅ 推荐：使用 kebab-case
   emit('update:modelValue', value)
   emit('item-selected', item)
   
   // ❌ 避免：camelCase
   emit('updateModelValue', value)
   ```

### 3.2 TailwindCSS 使用指南

#### 🎨 样式编写原则

1. **移动优先设计**
   ```html
   <!-- ✅ 推荐：从小屏幕开始，逐步增强 -->
   <div class="w-full md:w-1/2 lg:w-1/3">
     <p class="text-sm md:text-base lg:text-lg">响应式文本</p>
   </div>
   ```

2. **语义化类名组合**
   ```html
   <!-- ✅ 推荐：逻辑分组 -->
   <button class="
     bg-blue-600 hover:bg-blue-700 
     text-white font-medium 
     py-2 px-4 
     rounded-lg 
     transition-colors duration-200
     focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
   ">
     提交
   </button>
   ```

3. **自定义组件类**
   ```css
   @layer components {
     .btn {
       @apply font-medium py-2 px-4 rounded-lg transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2;
     }
     
     .btn-primary {
       @apply bg-blue-600 hover:bg-blue-700 text-white focus:ring-blue-500;
     }
     
     .btn-secondary {
       @apply bg-gray-200 hover:bg-gray-300 text-gray-900 focus:ring-gray-500;
     }
   }
   ```

#### 🎯 常用样式模式

1. **卡片组件**
   ```html
   <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-6 hover:shadow-md transition-shadow">
     <!-- 卡片内容 -->
   </div>
   ```

2. **表单输入**
   ```html
   <input class="
     w-full px-3 py-2 
     border border-gray-300 rounded-md 
     focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent
     placeholder-gray-400
   " />
   ```

3. **网格布局**
   ```html
   <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
     <!-- 网格项目 -->
   </div>
   ```

### 3.3 响应式设计实现方案

#### 📱 断点系统

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'sm': '640px',   // 小屏幕
      'md': '768px',   // 中等屏幕
      'lg': '1024px',  // 大屏幕
      'xl': '1280px',  // 超大屏幕
      '2xl': '1536px', // 2K屏幕
    }
  }
}
```

#### 🎯 响应式组件示例

```vue
<template>
  <div class="responsive-container">
    <!-- 响应式导航 -->
    <nav class="
      flex flex-col md:flex-row 
      space-y-2 md:space-y-0 md:space-x-4
      p-4 md:p-6
    ">
      <a href="#" class="nav-link">首页</a>
      <a href="#" class="nav-link">产品</a>
      <a href="#" class="nav-link">关于</a>
    </nav>
    
    <!-- 响应式网格 -->
    <main class="
      grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4
      gap-4 md:gap-6 lg:gap-8
      p-4 md:p-6 lg:p-8
    ">
      <div v-for="item in items" :key="item.id" class="card">
        <img 
          :src="item.image" 
          :alt="item.title"
          class="w-full h-48 sm:h-56 lg:h-64 object-cover rounded-lg"
        />
        <h3 class="text-lg md:text-xl font-semibold mt-4">
          {{ item.title }}
        </h3>
        <p class="text-sm md:text-base text-gray-600 mt-2">
          {{ item.description }}
        </p>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useBreakpoints } from '@vueuse/core'

// 响应式断点检测
const breakpoints = useBreakpoints({
  sm: 640,
  md: 768,
  lg: 1024,
  xl: 1280,
})

const isMobile = breakpoints.smaller('md')
const isTablet = breakpoints.between('md', 'lg')
const isDesktop = breakpoints.greater('lg')

// 根据屏幕尺寸调整数据
const items = ref([])

onMounted(() => {
  // 根据屏幕尺寸加载不同数量的数据
  const itemCount = isMobile.value ? 4 : isTablet.value ? 8 : 12
  loadItems(itemCount)
})
</script>
```

---

## 4. AI 开发指导

### 4.1 组件生成规则

#### 🤖 AI 组件生成提示词模板

```
请基于以下规范生成 Vue 3 组件：

**技术要求：**
- 使用 Vue 3 Composition API 和 <script setup>
- 集成 TailwindCSS 进行样式设计
- 完整的 TypeScript 类型定义
- 响应式设计支持

**组件规范：**
- 组件名称：{ComponentName}
- 功能描述：{功能说明}
- Props 接口：{props 列表}
- 事件定义：{events 列表}
- 插槽支持：{slots 说明}

**样式要求：**
- 移动优先设计
- 支持主题色彩系统
- 包含 hover、focus、active 状态
- 无障碍访问支持

**文件结构：**
- ComponentName.vue（主组件）
- ComponentName.types.ts（类型定义）
- index.ts（导出文件）
```

#### 📋 组件生成检查清单

- [ ] 使用 `<script setup lang="ts">`
- [ ] 完整的 Props 类型定义和默认值
- [ ] 正确的 Emits 声明
- [ ] TailwindCSS 类名规范使用
- [ ] 响应式断点支持
- [ ] 无障碍访问属性（aria-*）
- [ ] 语义化 HTML 结构
- [ ] 适当的插槽设计
- [ ] 组件文档注释

### 4.2 样式处理逻辑

#### 🎨 样式生成规则

1. **基础样式模板**
   ```typescript
   // 样式计算函数模板
   const computedClasses = computed(() => {
     const baseClasses = 'base-component-class'
     const variantClasses = {
       primary: 'bg-blue-600 text-white',
       secondary: 'bg-gray-200 text-gray-900',
       danger: 'bg-red-600 text-white'
     }
     const sizeClasses = {
       sm: 'px-2 py-1 text-sm',
       md: 'px-4 py-2 text-base',
       lg: 'px-6 py-3 text-lg'
     }
     
     return [
       baseClasses,
       variantClasses[props.variant],
       sizeClasses[props.size],
       {
         'opacity-50 cursor-not-allowed': props.disabled,
         'ring-2 ring-blue-500': props.focused
       }
     ]
   })
   ```

2. **响应式样式处理**
   ```typescript
   // 响应式样式生成
   const responsiveClasses = computed(() => {
     const mobile = 'w-full text-sm'
     const tablet = 'md:w-auto md:text-base'
     const desktop = 'lg:text-lg xl:text-xl'
     
     return `${mobile} ${tablet} ${desktop}`
   })
   ```

### 4.3 最佳实践示例

#### 🎯 完整组件示例：按钮组件

```vue
<!-- Button.vue -->
<template>
  <button
    :class="buttonClasses"
    :disabled="disabled || loading"
    :type="type"
    :aria-label="ariaLabel"
    @click="handleClick"
  >
    <span v-if="loading" class="animate-spin mr-2">
      <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"/>
      </svg>
    </span>
    
    <span v-if="$slots.icon && !loading" class="mr-2">
      <slot name="icon" />
    </span>
    
    <span>
      <slot />
    </span>
  </button>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { ButtonProps, ButtonEmits } from './Button.types'

const props = withDefaults(defineProps<ButtonProps>(), {
  variant: 'primary',
  size: 'md',
  type: 'button',
  disabled: false,
  loading: false,
  fullWidth: false
})

const emit = defineEmits<ButtonEmits>()

const buttonClasses = computed(() => [
  // 基础样式
  'inline-flex items-center justify-center font-medium rounded-lg transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2',
  
  // 尺寸样式
  {
    'px-2 py-1 text-sm': props.size === 'sm',
    'px-4 py-2 text-base': props.size === 'md',
    'px-6 py-3 text-lg': props.size === 'lg'
  },
  
  // 变体样式
  {
    'bg-blue-600 hover:bg-blue-700 text-white focus:ring-blue-500': props.variant === 'primary',
    'bg-gray-200 hover:bg-gray-300 text-gray-900 focus:ring-gray-500': props.variant === 'secondary',
    'bg-red-600 hover:bg-red-700 text-white focus:ring-red-500': props.variant === 'danger',
    'bg-transparent hover:bg-gray-100 text-gray-700 focus:ring-gray-500': props.variant === 'ghost'
  },
  
  // 状态样式
  {
    'w-full': props.fullWidth,
    'opacity-50 cursor-not-allowed': props.disabled || props.loading,
    'cursor-wait': props.loading
  }
])

const handleClick = (event: MouseEvent) => {
  if (!props.disabled && !props.loading) {
    emit('click', event)
  }
}
</script>
```

```typescript
// Button.types.ts
export interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  type?: 'button' | 'submit' | 'reset'
  disabled?: boolean
  loading?: boolean
  fullWidth?: boolean
  ariaLabel?: string
}

export interface ButtonEmits {
  click: [event: MouseEvent]
}
```

```typescript
// index.ts
export { default as Button } from './Button.vue'
export type { ButtonProps, ButtonEmits } from './Button.types'
```

---

## 5. 最佳实践示例

### 5.1 表单组件示例

```vue
<!-- ContactForm.vue -->
<template>
  <form @submit.prevent="handleSubmit" class="space-y-6">
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <div>
        <label for="firstName" class="block text-sm font-medium text-gray-700 mb-2">
          姓名 *
        </label>
        <input
          id="firstName"
          v-model="form.firstName"
          type="text"
          required
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          :class="{ 'border-red-500': errors.firstName }"
        />
        <p v-if="errors.firstName" class="mt-1 text-sm text-red-600">
          {{ errors.firstName }}
        </p>
      </div>
      
      <div>
        <label for="lastName" class="block text-sm font-medium text-gray-700 mb-2">
          姓氏 *
        </label>
        <input
          id="lastName"
          v-model="form.lastName"
          type="text"
          required
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          :class="{ 'border-red-500': errors.lastName }"
        />
        <p v-if="errors.lastName" class="mt-1 text-sm text-red-600">
          {{ errors.lastName }}
        </p>
      </div>
    </div>
    
    <div>
      <label for="email" class="block text-sm font-medium text-gray-700 mb-2">
        邮箱地址 *
      </label>
      <input
        id="email"
        v-model="form.email"
        type="email"
        required
        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
        :class="{ 'border-red-500': errors.email }"
      />
      <p v-if="errors.email" class="mt-1 text-sm text-red-600">
        {{ errors.email }}
      </p>
    </div>
    
    <div>
      <label for="message" class="block text-sm font-medium text-gray-700 mb-2">
        留言内容
      </label>
      <textarea
        id="message"
        v-model="form.message"
        rows="4"
        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent resize-none"
      ></textarea>
    </div>
    
    <div class="flex flex-col sm:flex-row gap-4">
      <Button
        type="submit"
        :loading="isSubmitting"
        class="flex-1 sm:flex-none"
      >
        {{ isSubmitting ? '提交中...' : '提交表单' }}
      </Button>
      
      <Button
        variant="secondary"
        type="button"
        @click="resetForm"
        :disabled="isSubmitting"
      >
        重置
      </Button>
    </div>
  </form>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { Button } from '@/components/ui'
import { useFormValidation } from '@/composables/useFormValidation'

interface ContactForm {
  firstName: string
  lastName: string
  email: string
  message: string
}

const form = reactive<ContactForm>({
  firstName: '',
  lastName: '',
  email: '',
  message: ''
})

const isSubmitting = ref(false)

const { errors, validate, resetErrors } = useFormValidation(form, {
  firstName: { required: true, minLength: 2 },
  lastName: { required: true, minLength: 2 },
  email: { required: true, email: true }
})

const handleSubmit = async () => {
  if (!validate()) return
  
  isSubmitting.value = true
  
  try {
    // 提交表单逻辑
    await submitForm(form)
    resetForm()
    // 显示成功消息
  } catch (error) {
    // 处理错误
    console.error('表单提交失败:', error)
  } finally {
    isSubmitting.value = false
  }
}

const resetForm = () => {
  Object.assign(form, {
    firstName: '',
    lastName: '',
    email: '',
    message: ''
  })
  resetErrors()
}
</script>
```

### 5.2 数据展示组件

```vue
<!-- ProductCard.vue -->
<template>
  <article class="group relative bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden hover:shadow-lg transition-all duration-300">
    <!-- 产品图片 -->
    <div class="relative aspect-w-16 aspect-h-9 bg-gray-100">
      <img
        :src="product.image"
        :alt="product.name"
        class="w-full h-48 object-cover group-hover:scale-105 transition-transform duration-300"
        loading="lazy"
      />
      
      <!-- 折扣标签 -->
      <div
        v-if="product.discount"
        class="absolute top-3 left-3 bg-red-500 text-white text-xs font-medium px-2 py-1 rounded-full"
      >
        -{{ product.discount }}%
      </div>
      
      <!-- 收藏按钮 -->
      <button
        @click="toggleFavorite"
        class="absolute top-3 right-3 p-2 bg-white/80 backdrop-blur-sm rounded-full hover:bg-white transition-colors"
        :class="{ 'text-red-500': isFavorite, 'text-gray-400': !isFavorite }"
      >
        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
          <path d="M3.172 5.172a4 4 0 015.656 0L10 6.343l1.172-1.171a4 4 0 115.656 5.656L10 17.657l-6.828-6.829a4 4 0 010-5.656z"/>
        </svg>
      </button>
    </div>
    
    <!-- 产品信息 -->
    <div class="p-6">
      <div class="flex items-start justify-between mb-2">
        <h3 class="text-lg font-semibold text-gray-900 line-clamp-2">
          {{ product.name }}
        </h3>
        
        <!-- 评分 -->
        <div class="flex items-center ml-2">
          <div class="flex text-yellow-400">
            <svg
              v-for="star in 5"
              :key="star"
              class="w-4 h-4"
              :class="star <= product.rating ? 'fill-current' : 'fill-gray-300'"
              viewBox="0 0 20 20"
            >
              <path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"/>
            </svg>
          </div>
          <span class="ml-1 text-sm text-gray-600">
            ({{ product.reviewCount }})
          </span>
        </div>
      </div>
      
      <p class="text-gray-600 text-sm mb-4 line-clamp-2">
        {{ product.description }}
      </p>
      
      <!-- 价格和操作 -->
      <div class="flex items-center justify-between">
        <div class="flex items-baseline space-x-2">
          <span class="text-2xl font-bold text-gray-900">
            ¥{{ formatPrice(product.price) }}
          </span>
          <span
            v-if="product.originalPrice && product.originalPrice > product.price"
            class="text-sm text-gray-500 line-through"
          >
            ¥{{ formatPrice(product.originalPrice) }}
          </span>
        </div>
        
        <Button
          size="sm"
          @click="addToCart"
          :disabled="!product.inStock"
        >
          {{ product.inStock ? '加入购物车' : '缺货' }}
        </Button>
      </div>
    </div>
  </article>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { Button } from '@/components/ui'
import type { Product } from '@/types/product'

interface Props {
  product: Product
}

const props = defineProps<Props>()

const emit = defineEmits<{
  addToCart: [product: Product]
  toggleFavorite: [product: Product, isFavorite: boolean]
}>()

const isFavorite = ref(false)

const formatPrice = (price: number) => {
  return price.toFixed(2)
}

const toggleFavorite = () => {
  isFavorite.value = !isFavorite.value
  emit('toggleFavorite', props.product, isFavorite.value)
}

const addToCart = () => {
  if (props.product.inStock) {
    emit('addToCart', props.product)
  }
}
</script>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
```

---

## 6. 常见问题解答

### 6.1 性能优化

**Q: 如何优化 TailwindCSS 的包体积？**

A: 使用 PurgeCSS 和 JIT 模式：

```javascript
// tailwind.config.js
module.exports = {
  mode: 'jit',
  purge: [
    './index.html',
    './src/**/*.{vue,js,ts,jsx,tsx}',
  ],
  // ...其他配置
}
```

**Q: Vue 3 组件如何实现懒加载？**

A: 使用动态导入和 Suspense：

```typescript
// 路由懒加载
const Home = () => import('@/views/Home.vue')

// 组件懒加载
const LazyComponent = defineAsyncComponent(() => import('@/components/Heavy.vue'))
```

### 6.2 开发调试

**Q: 如何调试 TailwindCSS 样式？**

A: 使用浏览器开发者工具和 Tailwind CSS IntelliSense 插件：

1. 安装 VS Code 插件：`Tailwind CSS IntelliSense`
2. 使用 `@apply` 指令调试复杂样式
3. 启用 Tailwind 的开发模式查看生成的 CSS

**Q: Vue 3 Composition API 如何调试？**

A: 使用 Vue DevTools 和 console 调试：

```typescript
import { ref, watch } from 'vue'

const count = ref(0)

// 调试响应式数据
watch(count, (newVal, oldVal) => {
  console.log(`count changed from ${oldVal} to ${newVal}`)
}, { immediate: true })
```

### 6.3 最佳实践

**Q: 何时使用 Composition API vs Options API？**

A: 推荐始终使用 Composition API：
- 更好的 TypeScript 支持
- 更灵活的逻辑复用
- 更好的 tree-shaking
- 更接近原生 JavaScript

**Q: 如何组织大型项目的样式？**

A: 采用分层架构：

```css
/* 1. 基础层 - Tailwind 基础样式 */
@tailwind base;

/* 2. 组件层 - 可复用组件样式 */
@tailwind components;

/* 3. 工具层 - 原子化工具类 */
@tailwind utilities;

/* 4. 自定义层 - 项目特定样式 */
@layer components {
  .btn { /* 组件样式 */ }
}

@layer utilities {
  .text-balance { /* 工具样式 */ }
}
```

---

## 📚 参考资源

### 官方文档
- [Vue 3 官方文档](https://vuejs.org/)
- [TailwindCSS 官方文档](https://tailwindcss.com/)
- [Vite 官方文档](https://vitejs.dev/)
- [Pinia 官方文档](https://pinia.vuejs.org/)

### 工具和插件
- [Vue DevTools](https://devtools.vuejs.org/)
- [Tailwind CSS IntelliSense](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss)
- [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur) / [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar)

### 社区资源
- [Vue 3 Awesome](https://github.com/vue3/awesome-vue)
- [Tailwind UI](https://tailwindui.com/)
- [Headless UI](https://headlessui.com/)

---

*本文档将持续更新，以反映最新的最佳实践和技术发展。*

**版本**: v1.0.0  
**更新日期**: 2024年1月  
**维护者**: 前端开发团队