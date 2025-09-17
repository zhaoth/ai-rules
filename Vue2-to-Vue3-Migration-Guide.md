# 🚀 Vue 2 升级 Vue 3 + Element 升级 Element Plus 迁移指南

> 基于 AI 的 Vue 2 到 Vue 3 完整迁移规则，包含 Element UI 到 Element Plus 的升级方案

## 📋 目录

- [迁移概览](#迁移概览)
- [环境准备](#环境准备)
- [Vue 3 核心变更](#vue-3-核心变更)
- [Element Plus 迁移](#element-plus-迁移)
- [自动化迁移工具](#自动化迁移工具)
- [测试策略](#测试策略)
- [性能优化](#性能优化)
- [常见问题](#常见问题)

## 🎯 迁移概览

### 迁移收益
- **性能提升** - Vue 3 提供更好的性能和更小的包体积
- **Composition API** - 更好的逻辑复用和类型推导
- **TypeScript 支持** - 原生 TypeScript 支持
- **现代化 UI** - Element Plus 提供更现代的组件设计
- **Tree Shaking** - 更好的按需引入支持

### 迁移风险评估
| 风险等级 | 影响范围 | 预估工作量 | 建议策略 |
|---------|---------|-----------|---------|
| 🔴 高风险 | 全局 API、生命周期 | 2-4 周 | 分阶段迁移 |
| 🟡 中风险 | 组件库、第三方依赖 | 1-2 周 | 兼容性测试 |
| 🟢 低风险 | 样式、配置 | 3-5 天 | 批量处理 |

### 迁移时间线
```
第1周：环境准备 + 依赖升级
第2周：核心 API 迁移 + 组件库升级
第3周：业务组件适配 + 测试验证
第4周：性能优化 + 部署上线
```

## 🛠️ 环境准备

### 1. 依赖版本要求

#### 核心依赖
```json
{
  "vue": "^3.4.0",
  "@vue/compat": "^3.4.0",
  "element-plus": "^2.4.0",
  "vue-router": "^4.0.0",
  "vuex": "^4.0.0",
  "@vitejs/plugin-vue": "^4.0.0"
}
```

#### 开发工具
```json
{
  "@vue/cli": "^5.0.0",
  "vite": "^5.0.0",
  "typescript": "^5.0.0",
  "@vue/tsconfig": "^0.4.0",
  "unplugin-vue-components": "^0.25.0",
  "unplugin-auto-import": "^0.16.0"
}
```

### 2. 构建工具迁移

#### 从 Vue CLI 迁移到 Vite
```bash
# 安装 Vite
npm install -D vite @vitejs/plugin-vue

# 创建 vite.config.ts
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      resolvers: [ElementPlusResolver()],
      imports: ['vue', 'vue-router'],
      dts: true
    }),
    Components({
      resolvers: [ElementPlusResolver()],
      dts: true
    })
  ],
  resolve: {
    alias: {
      '@': '/src'
    }
  },
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@use "@/styles/element/index.scss" as *;`
      }
    }
  }
})
```

### 3. TypeScript 配置

```json
// tsconfig.json
{
  "extends": "@vue/tsconfig/tsconfig.dom.json",
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "jsx": "preserve",
    "importHelpers": true,
    "experimentalDecorators": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "useDefineForClassFields": true,
    "sourceMap": true,
    "baseUrl": ".",
    "types": ["vite/client", "element-plus/global"],
    "paths": {
      "@/*": ["src/*"]
    },
    "lib": ["ES2020", "DOM", "DOM.Iterable"]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "auto-imports.d.ts",
    "components.d.ts"
  ],
  "exclude": ["node_modules", "dist"]
}
```

## 🔄 Vue 3 核心变更

### 1. 应用实例创建

#### Vue 2 写法
```javascript
// main.js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

Vue.use(ElementUI)
Vue.config.productionTip = false

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')
```

#### Vue 3 写法
```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import zhCn from 'element-plus/dist/locale/zh-cn.mjs'

const app = createApp(App)

app.use(ElementPlus, {
  locale: zhCn,
})
app.use(store)
app.use(router)

app.mount('#app')
```

### 2. 组件定义方式

#### Vue 2 Options API
```vue
<template>
  <div class="user-profile">
    <el-card>
      <el-form :model="form" :rules="rules" ref="formRef">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="form.username" />
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="form.email" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm">提交</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'UserProfile',
  data() {
    return {
      form: {
        username: '',
        email: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    submitForm() {
      this.$refs.formRef.validate((valid) => {
        if (valid) {
          console.log('提交表单:', this.form)
        }
      })
    }
  }
}
</script>
```

#### Vue 3 Composition API
```vue
<template>
  <div class="user-profile">
    <el-card>
      <el-form :model="form" :rules="rules" ref="formRef">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="form.username" />
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="form.email" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm">提交</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'

interface UserForm {
  username: string
  email: string
}

const formRef = ref<FormInstance>()

const form = reactive<UserForm>({
  username: '',
  email: ''
})

const rules = reactive<FormRules<UserForm>>({
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
  ]
})

const submitForm = async () => {
  if (!formRef.value) return
  
  await formRef.value.validate((valid) => {
    if (valid) {
      console.log('提交表单:', form)
    }
  })
}
</script>
```

### 3. 生命周期钩子变更

#### Vue 2 生命周期
```javascript
export default {
  beforeCreate() {},
  created() {},
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {}
}
```

#### Vue 3 生命周期
```typescript
import { 
  onBeforeMount, 
  onMounted, 
  onBeforeUpdate, 
  onUpdated, 
  onBeforeUnmount, 
  onUnmounted 
} from 'vue'

export default {
  setup() {
    // beforeCreate 和 created 的逻辑直接写在 setup 中
    
    onBeforeMount(() => {
      console.log('组件挂载前')
    })
    
    onMounted(() => {
      console.log('组件已挂载')
    })
    
    onBeforeUpdate(() => {
      console.log('组件更新前')
    })
    
    onUpdated(() => {
      console.log('组件已更新')
    })
    
    onBeforeUnmount(() => {
      console.log('组件卸载前')
    })
    
    onUnmounted(() => {
      console.log('组件已卸载')
    })
  }
}
```

### 4. 响应式系统变更

#### Vue 2 响应式
```javascript
export default {
  data() {
    return {
      count: 0,
      user: {
        name: 'John',
        age: 25
      },
      list: []
    }
  },
  methods: {
    updateUser() {
      // Vue 2 需要使用 $set 来添加新属性
      this.$set(this.user, 'email', 'john@example.com')
      
      // 数组操作
      this.list.push(newItem)
      this.$set(this.list, index, newValue)
    }
  }
}
```

#### Vue 3 响应式
```typescript
import { ref, reactive, computed, watch } from 'vue'

export default {
  setup() {
    // 基本类型使用 ref
    const count = ref(0)
    
    // 对象类型使用 reactive
    const user = reactive({
      name: 'John',
      age: 25
    })
    
    const list = reactive<any[]>([])
    
    // 计算属性
    const doubleCount = computed(() => count.value * 2)
    
    // 侦听器
    watch(count, (newVal, oldVal) => {
      console.log(`count changed from ${oldVal} to ${newVal}`)
    })
    
    const updateUser = () => {
      // Vue 3 可以直接添加属性
      user.email = 'john@example.com'
      
      // 数组操作更简单
      list.push(newItem)
      list[index] = newValue
    }
    
    return {
      count,
      user,
      list,
      doubleCount,
      updateUser
    }
  }
}
```

### 5. 全局 API 变更

#### Vue 2 全局配置
```javascript
import Vue from 'vue'

// 全局组件
Vue.component('MyComponent', MyComponent)

// 全局指令
Vue.directive('focus', {
  inserted: function (el) {
    el.focus()
  }
})

// 全局混入
Vue.mixin({
  methods: {
    $log(message) {
      console.log(message)
    }
  }
})

// 全局属性
Vue.prototype.$http = axios
```

#### Vue 3 应用实例配置
```typescript
import { createApp } from 'vue'

const app = createApp(App)

// 全局组件
app.component('MyComponent', MyComponent)

// 全局指令
app.directive('focus', {
  mounted(el) {
    el.focus()
  }
})

// 全局混入
app.mixin({
  methods: {
    $log(message: string) {
      console.log(message)
    }
  }
})

// 全局属性
app.config.globalProperties.$http = axios

// 或者使用 provide/inject
app.provide('http', axios)
```

## 🎨 Element Plus 迁移

### 1. 安装和配置

#### 完整引入
```typescript
// main.ts
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import zhCn from 'element-plus/dist/locale/zh-cn.mjs'

const app = createApp(App)
app.use(ElementPlus, {
  locale: zhCn,
})
```

#### 按需引入（推荐）
```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    AutoImport({
      resolvers: [ElementPlusResolver()],
    }),
    Components({
      resolvers: [ElementPlusResolver()],
    }),
  ],
})
```

### 2. 组件名称变更

| Element UI | Element Plus | 说明 |
|------------|--------------|------|
| `el-col` | `el-col` | 无变更 |
| `el-row` | `el-row` | 无变更 |
| `el-button` | `el-button` | 无变更 |
| `el-input` | `el-input` | 无变更 |
| `el-select` | `el-select` | 无变更 |
| `el-date-picker` | `el-date-picker` | 无变更 |
| `el-table` | `el-table` | 无变更 |
| `el-pagination` | `el-pagination` | 无变更 |

### 3. 属性和事件变更

#### Table 组件变更
```vue
<!-- Element UI -->
<template>
  <el-table
    :data="tableData"
    @selection-change="handleSelectionChange"
    @sort-change="handleSortChange"
  >
    <el-table-column type="selection" width="55" />
    <el-table-column prop="name" label="姓名" sortable />
    <el-table-column prop="age" label="年龄" />
  </el-table>
</template>

<!-- Element Plus -->
<template>
  <el-table
    :data="tableData"
    @selection-change="handleSelectionChange"
    @sort-change="handleSortChange"
  >
    <el-table-column type="selection" width="55" />
    <el-table-column prop="name" label="姓名" sortable />
    <el-table-column prop="age" label="年龄" />
  </el-table>
</template>
```

#### Form 组件变更
```vue
<!-- Element UI -->
<template>
  <el-form :model="form" :rules="rules" ref="form">
    <el-form-item label="用户名" prop="username">
      <el-input v-model="form.username" />
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  methods: {
    submitForm() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          // 提交逻辑
        }
      })
    }
  }
}
</script>

<!-- Element Plus -->
<template>
  <el-form :model="form" :rules="rules" ref="formRef">
    <el-form-item label="用户名" prop="username">
      <el-input v-model="form.username" />
    </el-form-item>
  </el-form>
</template>

<script setup lang="ts">
import type { FormInstance } from 'element-plus'

const formRef = ref<FormInstance>()

const submitForm = async () => {
  if (!formRef.value) return
  
  await formRef.value.validate((valid) => {
    if (valid) {
      // 提交逻辑
    }
  })
}
</script>
```

#### Message 和 MessageBox 变更
```typescript
// Element UI
this.$message.success('操作成功')
this.$confirm('确定删除吗？', '提示', {
  confirmButtonText: '确定',
  cancelButtonText: '取消',
  type: 'warning'
})

// Element Plus
import { ElMessage, ElMessageBox } from 'element-plus'

ElMessage.success('操作成功')
ElMessageBox.confirm('确定删除吗？', '提示', {
  confirmButtonText: '确定',
  cancelButtonText: '取消',
  type: 'warning'
})

// 或者使用自动导入
ElMessage.success('操作成功')
ElMessageBox.confirm('确定删除吗？', '提示')
```

### 4. 样式和主题变更

#### CSS 变量系统
```scss
// Element UI 主题定制
$--color-primary: #409EFF;
$--color-success: #67C23A;
$--color-warning: #E6A23C;
$--color-danger: #F56C6C;
$--color-info: #909399;

// Element Plus 主题定制
:root {
  --el-color-primary: #409EFF;
  --el-color-success: #67C23A;
  --el-color-warning: #E6A23C;
  --el-color-danger: #F56C6C;
  --el-color-info: #909399;
}

// 或者使用 SCSS
@forward 'element-plus/theme-chalk/src/common/var.scss' with (
  $colors: (
    'primary': (
      'base': #409EFF,
    ),
  ),
);
```

#### 暗黑模式支持
```typescript
// 暗黑模式切换
import { useDark, useToggle } from '@vueuse/core'

const isDark = useDark()
const toggleDark = useToggle(isDark)

// 在模板中使用
<el-switch v-model="isDark" @change="toggleDark" />
```

### 5. 图标系统变更

#### Element UI 图标
```vue
<template>
  <el-button icon="el-icon-search">搜索</el-button>
  <i class="el-icon-edit"></i>
</template>
```

#### Element Plus 图标
```vue
<template>
  <el-button :icon="Search">搜索</el-button>
  <el-icon><Edit /></el-icon>
</template>

<script setup lang="ts">
import { Search, Edit } from '@element-plus/icons-vue'
</script>
```

## 🤖 自动化迁移工具

### 1. Vue 官方迁移工具

#### @vue/compat 兼容模式
```bash
# 安装兼容包
npm install @vue/compat

# 配置别名
# vite.config.ts
export default {
  resolve: {
    alias: {
      vue: '@vue/compat'
    }
  }
}
```

#### 迁移构建工具
```typescript
// vue.config.js (Vue CLI)
module.exports = {
  chainWebpack: config => {
    config.resolve.alias.set('vue', '@vue/compat')
    
    config.plugin('define').tap(definitions => {
      Object.assign(definitions[0], {
        __VUE_OPTIONS_API__: 'true',
        __VUE_PROD_DEVTOOLS__: 'false',
        __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: 'false'
      })
      return definitions
    })
  }
}
```

### 2. 自定义迁移脚本

#### 组件迁移脚本
```javascript
// migrate-components.js
const fs = require('fs')
const path = require('path')

const migrations = [
  // Vue 2 to Vue 3 API 迁移
  {
    from: /this\.\$refs\.(\w+)\.validate\(/g,
    to: 'await $1Ref.value?.validate('
  },
  {
    from: /this\.\$message\.(\w+)\(/g,
    to: 'ElMessage.$1('
  },
  {
    from: /this\.\$confirm\(/g,
    to: 'ElMessageBox.confirm('
  },
  // Element UI to Element Plus 迁移
  {
    from: /el-icon-(\w+)/g,
    to: (match, iconName) => {
      const iconMap = {
        'search': 'Search',
        'edit': 'Edit',
        'delete': 'Delete',
        'plus': 'Plus'
      }
      return iconMap[iconName] || iconName
    }
  }
]

function migrateFile(filePath) {
  let content = fs.readFileSync(filePath, 'utf8')
  
  migrations.forEach(migration => {
    content = content.replace(migration.from, migration.to)
  })
  
  fs.writeFileSync(filePath, content)
  console.log(`Migrated: ${filePath}`)
}

function migrateDirectory(dirPath) {
  const files = fs.readdirSync(dirPath)
  
  files.forEach(file => {
    const filePath = path.join(dirPath, file)
    const stat = fs.statSync(filePath)
    
    if (stat.isDirectory()) {
      migrateDirectory(filePath)
    } else if (file.endsWith('.vue') || file.endsWith('.js') || file.endsWith('.ts')) {
      migrateFile(filePath)
    }
  })
}

// 执行迁移
migrateDirectory('./src')
```

#### 依赖升级脚本
```bash
#!/bin/bash
# upgrade-dependencies.sh

echo "开始升级 Vue 3 和 Element Plus..."

# 卸载旧依赖
npm uninstall vue@2 element-ui vue-template-compiler

# 安装新依赖
npm install vue@^3.4.0 element-plus@^2.4.0
npm install -D @vitejs/plugin-vue unplugin-vue-components unplugin-auto-import

# 安装图标库
npm install @element-plus/icons-vue

# 安装类型定义
npm install -D @types/node

echo "依赖升级完成！"
```

### 3. ESLint 迁移规则

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'plugin:vue/vue3-essential',
    '@vue/typescript/recommended'
  ],
  rules: {
    // Vue 3 特定规则
    'vue/no-deprecated-v-on-native-modifier': 'error',
    'vue/no-deprecated-v-bind-sync': 'error',
    'vue/no-deprecated-dollar-listeners-api': 'error',
    'vue/no-deprecated-events-api': 'error',
    'vue/no-deprecated-filter': 'error',
    'vue/no-deprecated-functional-template': 'error',
    'vue/no-deprecated-html-element-is': 'error',
    'vue/no-deprecated-inline-template': 'error',
    'vue/no-deprecated-props-default-this': 'error',
    'vue/no-deprecated-router-link-tag-prop': 'error',
    'vue/no-deprecated-scope-attribute': 'error',
    'vue/no-deprecated-slot-attribute': 'error',
    'vue/no-deprecated-slot-scope-attribute': 'error',
    'vue/no-deprecated-v-is': 'error',
    'vue/no-lifecycle-after-await': 'error',
    'vue/no-watch-after-await': 'error',
    'vue/prefer-import-from-vue': 'error',
    'vue/require-slots-as-functions': 'error',
    'vue/require-toggle-inside-transition': 'error',
    'vue/valid-next-tick': 'error'
  }
}
```

## 🧪 测试策略

### 1. 单元测试迁移

#### Vue Test Utils 升级
```bash
# 安装新版本测试工具
npm install -D @vue/test-utils@next vitest jsdom
```

#### 测试配置
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./tests/setup.ts']
  }
})
```

#### 测试用例迁移
```typescript
// Vue 2 测试
import { shallowMount } from '@vue/test-utils'
import Component from '@/components/MyComponent.vue'

describe('MyComponent', () => {
  it('renders correctly', () => {
    const wrapper = shallowMount(Component, {
      propsData: {
        title: 'Test Title'
      }
    })
    expect(wrapper.text()).toContain('Test Title')
  })
})

// Vue 3 测试
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import Component from '@/components/MyComponent.vue'

describe('MyComponent', () => {
  it('renders correctly', () => {
    const wrapper = mount(Component, {
      props: {
        title: 'Test Title'
      }
    })
    expect(wrapper.text()).toContain('Test Title')
  })
})
```

### 2. E2E 测试更新

#### Cypress 配置
```typescript
// cypress.config.ts
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:5173',
    supportFile: 'cypress/support/e2e.ts',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}'
  },
  component: {
    devServer: {
      framework: 'vue',
      bundler: 'vite'
    }
  }
})
```

### 3. 兼容性测试清单

#### 功能测试清单
- [ ] 路由导航正常
- [ ] 表单提交和验证
- [ ] 数据列表展示和操作
- [ ] 弹窗和消息提示
- [ ] 文件上传下载
- [ ] 权限控制
- [ ] 国际化切换
- [ ] 主题切换

#### 性能测试清单
- [ ] 首屏加载时间
- [ ] 路由切换速度
- [ ] 大数据列表渲染
- [ ] 内存使用情况
- [ ] Bundle 大小对比

## ⚡ 性能优化

### 1. 构建优化

#### Vite 配置优化
```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'element-plus': ['element-plus'],
          'vue-vendor': ['vue', 'vue-router'],
          'utils': ['lodash', 'dayjs']
        }
      }
    },
    chunkSizeWarningLimit: 1000
  },
  optimizeDeps: {
    include: ['element-plus/es/locale/lang/zh-cn']
  }
})
```

### 2. 组件优化

#### 异步组件
```typescript
// Vue 2
const AsyncComponent = () => import('./AsyncComponent.vue')

// Vue 3
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() => 
  import('./AsyncComponent.vue')
)

// 带加载状态的异步组件
const AsyncComponentWithOptions = defineAsyncComponent({
  loader: () => import('./AsyncComponent.vue'),
  loadingComponent: LoadingComponent,
  errorComponent: ErrorComponent,
  delay: 200,
  timeout: 3000
})
```

#### 组件缓存
```vue
<template>
  <!-- Vue 2 -->
  <keep-alive>
    <component :is="currentComponent" />
  </keep-alive>

  <!-- Vue 3 -->
  <keep-alive>
    <component :is="currentComponent" />
  </keep-alive>
</template>
```

### 3. 按需加载优化

#### Element Plus 按需引入
```typescript
// 自动按需引入配置
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

Components({
  resolvers: [
    ElementPlusResolver({
      importStyle: 'sass' // 或 'css'
    })
  ]
})
```

## ❓ 常见问题

### 1. 迁移过程中的常见错误

#### 错误：Cannot read property '$refs' of undefined
```typescript
// 问题代码
this.$refs.formRef.validate()

// 解决方案
const formRef = ref<FormInstance>()
await formRef.value?.validate()
```

#### 错误：Property 'xxx' does not exist on type 'ComponentPublicInstance'
```typescript
// 问题：全局属性访问
this.$http.get('/api/users')

// 解决方案1：使用 provide/inject
// main.ts
app.provide('http', axios)

// 组件中
const http = inject('http')

// 解决方案2：类型声明
declare module '@vue/runtime-core' {
  interface ComponentCustomProperties {
    $http: typeof axios
  }
}
```

#### 错误：Element Plus 组件样式丢失
```typescript
// 问题：按需引入时样式未加载

// 解决方案：确保样式正确引入
import 'element-plus/dist/index.css'

// 或者使用 SCSS
@use 'element-plus/theme-chalk/src/index.scss' as *;
```

### 2. 性能问题排查

#### Bundle 分析
```bash
# 安装分析工具
npm install -D rollup-plugin-visualizer

# 生成分析报告
npm run build -- --report
```

#### 内存泄漏检查
```typescript
// 确保正确清理定时器和事件监听
import { onUnmounted } from 'vue'

export default {
  setup() {
    const timer = setInterval(() => {
      // 定时任务
    }, 1000)
    
    onUnmounted(() => {
      clearInterval(timer)
    })
  }
}
```

### 3. 兼容性问题

#### 第三方库兼容性
```typescript
// 检查第三方库 Vue 3 兼容性
// 使用兼容性列表：https://github.com/vue3/awesome-vue3

// 不兼容的库替代方案
// vue-class-component → @vue/composition-api
// vue-property-decorator → 使用 Composition API
// vuex-class → 直接使用 Vuex 4
```

## 📚 参考资源

### 官方文档
- [Vue 3 迁移指南](https://v3-migration.vuejs.org/)
- [Element Plus 官方文档](https://element-plus.org/)
- [Vue 3 Composition API](https://composition-api.vuejs.org/)

### 工具和插件
- [@vue/compat](https://www.npmjs.com/package/@vue/compat) - Vue 2/3 兼容层
- [unplugin-vue-components](https://github.com/antfu/unplugin-vue-components) - 自动导入组件
- [unplugin-auto-import](https://github.com/antfu/unplugin-auto-import) - 自动导入 API

### 社区资源
- [Vue 3 迁移最佳实践](https://github.com/vuejs/rfcs)
- [Element Plus 迁移案例](https://github.com/element-plus/element-plus)

---

## 🎯 AI 开发指导

### 代码生成提示
在使用 AI 工具进行 Vue 2 到 Vue 3 迁移时，请遵循以下提示模板：

```
请帮我将以下 Vue 2 代码迁移到 Vue 3，要求：
1. 使用 Composition API 和 <script setup> 语法
2. 将 Element UI 组件替换为 Element Plus
3. 添加完整的 TypeScript 类型定义
4. 保持原有功能不变
5. 优化性能和代码结构
6. 包含必要的导入语句

原代码：
[粘贴 Vue 2 代码]
```

### 迁移审查要点
- Composition API 使用是否正确
- TypeScript 类型定义是否完整
- Element Plus 组件是否正确替换
- 响应式数据处理是否合理
- 生命周期钩子是否正确迁移
- 性能优化是否到位