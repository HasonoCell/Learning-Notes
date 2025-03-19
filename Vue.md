# 1. ref 与 reactive 的响应式数据问题

首先需要知道，`ref` 适用于所有数据类型，但更常用于原始值（如 `number`、`string`）底层原理与 `reactive` 相同；`reactive` 专为对象类型设计，通过 Proxy 代理整个对象。

如果我们对使用 reactive 创建的响应式数据对象进行解构赋值时，会导致响应式特性丢失，原因：**赋值解构操作本身是值的复制，并没有创建对解构后的新数据的代理**，如何解决这个问题？

1. **toRef** 方法：基于响应式对象上的一个属性，创建一个对应的 ref，这样创建的 ref 与其源属性保持同步：改变源属性的值将更新 ref 的值

   ```js
   const person = reactive({ name: 'John', age: 18 })
   const ageRef = toRef(person, 'age')
   ```

2. **toRefs** 方法：将一个响应式对象转换为普通对象，但这个对象的每一个属性都是指向源对象相应属性的 ref，这里有一个注意点：

   ```js
   const person = reactive({ name: "Hasono", age: 18 });
   const personRef = toRefs(person)
   ```

   如果我们直接**通过对象属性访问 `ref`**，模板无法推断 `personRef.age` 是 `ref`，也就不会对其进行自动解包，我们需要显式`.value`

   如果我们直接**绑定解构后的 `ref`**，Vue 在模板编译阶段自动处理 `.value`（如 `age++` → `age.value++`），就不需要显式`.value`，因此**我们建议使用toRefs时直接进行解构赋值**

   总结：Vue 模板中的自动解包仅作用于**直接绑定的顶层 `ref` 变量**。若 `ref` 作为对象属性存在（如 `obj.refKey`），需通过 `.value` 操作

# 2. 如何理解 computed？

**computed** 用于基于其他响应式数据生成新的派生值（派生值具有**响应式特性**），通过以下代码理解

```vue
<script setup>
import { ref, computed } from 'vue'
  const a = ref(1)
  const b = ref(2)
  const sum = a.value + b.value
</script>

<template>
  <h1>{{ sum }}</h1>
  <h3>a = {{ a }}</h3>
  <button @click="a++">plus</button>
</template>
```

"此时 `a` 的值增加，但 `sum` 的值不会更新，视图仍显示初始计算结果，并且**`const sum = ref(a.value + b.value)`**这种写法是不行的，我们需要使 sum 为一个计算属性来保证这一基于其它响应式数据派生出来的新的响应式数据依然保持其响应式
```vue
<script setup>
import { ref, computed } from 'vue'
  const a = ref(1)
  const b = ref(2)
  const sum = computed(() => a.value + b.value)
</script>
```

# 3. Tree-Shaking

**打包工具（Vite，Webpack）**通过 **AST（抽象语法树）**分析模块间的导入导出关系，构建模块之间的依赖关系图，标记未被引用的代码分支，再像摇晃果树一样抖落语法树上未被引用的"dead code"，减少最后打包出的js文件体积，提高加载速度。



# 4. defineProps 与 defineEmits 的两种用法

### 一、`defineProps` 的两种定义方式

#### 1. **运行时声明（对象语法）**

直接在 `defineProps` 中传入一个对象，类似于选项式 API 的 `props` 定义：

```typescript
const props = defineProps({
  // 基础类型检查
  title: String,
  
  // 带默认值和类型验证的复杂配置
  count: {
    type: Number,
    default: 0,
    validator: (value: number) => value >= 0
  }
});
```

**特点**：

- 兼容 JavaScript 和 TypeScript
- 支持运行时类型验证
- 可以直接定义默认值和验证函数

#### 2. **类型声明（TypeScript 泛型语法）**

使用 TypeScript 类型注解，通过泛型参数定义 Props 类型：

```typescript
interface Props {
  title: string;
  count?: number; // 可选属性（等效于默认值）
}

const props = defineProps<Props>();
```

**需要默认值时，需结合 `withDefaults`**：

```typescript
const props = withDefaults(defineProps<Props>(), {
  count: 0 // 定义默认值
});
```

**特点**：

- 纯 TypeScript 类型检查，无运行时验证
- 代码更简洁，类型提示更完善
- 必须使用构建工具支持 TypeScript

---

### 二、`defineEmits` 的两种定义方式

#### 1. **运行时声明（对象语法）**

传入一个对象，定义事件名称和验证函数：

```typescript
const emit = defineEmits({
  // 无验证函数
  'change': null, 

  // 带参数验证的事件
  'update:modelValue': (value: string) => {
    return typeof value === 'string' && value.length > 0;
  }
});
```

**特点**：

- 支持运行时参数验证
- 适合复杂事件逻辑的场景

#### 2. **类型声明（TS 函数签名语法）**

通过 TypeScript 函数签名定义事件类型：

```typescript
interface Emits {
  (e: 'change'): void; // 无参数事件
  (e: 'update:modelValue', value: string): void; // 带参数事件
}

const emit = defineEmits<Emits>();
```

**特点**：

- 严格的类型检查，确保事件触发的参数类型安全
- 无运行时验证，依赖静态类型分析
- 代码更简洁，适合 TypeScript 项目

---

### 三、两种方式对比总结

| 特性             | 运行时声明 (对象语法)       | 类型声明 (TS 语法)               |
| ---------------- | --------------------------- | -------------------------------- |
| **语言支持**     | JS/TS 通用                  | 仅限 TypeScript                  |
| **类型验证**     | 运行时验证 + 静态类型检查   | 仅静态类型检查                   |
| **默认值**       | 直接通过 `default` 字段定义 | 需配合 `withDefaults` 宏         |
| **事件参数验证** | 支持验证函数                | 仅类型定义，无运行时验证         |
| **代码提示**     | 一般                        | 强大的 IDE 类型提示              |
| **适用场景**     | 需要运行时验证的复杂逻辑    | 追求类型安全和简洁代码的 TS 项目 |

---

### 四、完整组件示例

#### 组合式 API + TypeScript 类型声明

```vue
<script setup lang="ts">
interface Props {
  title: string;
  count?: number;
}

interface Emits {
  (e: 'update:title', newTitle: string): void;
}

const props = withDefaults(defineProps<Props>(), {
  count: 0
});

const emit = defineEmits<Emits>();

const handleClick = () => {
  emit('update:title', 'New Title');
};
</script>

<template>
  <div>
    <h1>{{ title }}</h1>
    <p>Count: {{ count }}</p>
    <button @click="handleClick">Change Title</button>
  </div>
</template>
```

#### 运行时声明（兼容 JavaScript）

```vue
<script setup>
const props = defineProps({
  title: { type: String, required: true },
  count: { type: Number, default: 0 }
});

const emit = defineEmits({
  'update:title': (value) => typeof value === 'string'
});

const handleClick = () => {
  emit('update:title', 'New Title');
};
</script>
```

