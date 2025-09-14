# Reactivity Transform {#reactivity-transform}

:::danger Removed Experimental Feature
Reactivity Transform là tính năng thử nghiệm và đã bị gỡ trong bản 3.4. Vui lòng xem [lý do ở đây](https://github.com/vuejs/rfcs/discussions/369#discussioncomment-5059028).

Nếu vẫn muốn dùng, hãy sử dụng qua plugin [Vue Macros](https://vue-macros.sxzz.moe/features/reactivity-transform.html).
:::

:::tip Composition-API-specific
Reactivity Transform là tính năng dành riêng cho Composition API và cần build step.
:::

## Refs vs. Reactive Variables {#refs-vs-reactive-variables}

Kể từ khi Composition API ra đời, một câu hỏi lớn là dùng ref hay reactive object. Rất dễ mất reactivity khi destructure reactive object, trong khi với ref thì việc dùng `.value` ở khắp nơi khá vướng víu — và dễ bỏ sót nếu không có hệ thống kiểu.

[Vue Reactivity Transform](https://github.com/vuejs/core/tree/main/packages/reactivity-transform) is a compile-time transform that allows us to write code like this:

```vue
<script setup>
let count = $ref(0)

console.log(count)

function increment() {
  count++
}
</script>

<template>
  <button @click="increment">{{ count }}</button>
</template>
```

`$ref()` ở đây là **macro compile‑time**: không phải hàm thực thi lúc chạy. Compiler của Vue dùng nó như một gợi ý để coi biến `count` là **reactive variable.**

Reactive variable có thể truy cập và gán lại như biến thường, nhưng các thao tác này được biên dịch thành ref với `.value`. Ví dụ, phần `<script>` ở trên được biên dịch thành:

```js{5,8}
import { ref } from 'vue'

let count = ref(0)

console.log(count.value)

function increment() {
  count.value++
}
```

Every reactivity API that returns refs will have a `$`-prefixed macro equivalent. These APIs include:

- [`ref`](/api/reactivity-core#ref) -> `$ref`
- [`computed`](/api/reactivity-core#computed) -> `$computed`
- [`shallowRef`](/api/reactivity-advanced#shallowref) -> `$shallowRef`
- [`customRef`](/api/reactivity-advanced#customref) -> `$customRef`
- [`toRef`](/api/reactivity-utilities#toref) -> `$toRef`

Các macro này khả dụng toàn cục khi bật Reactivity Transform, không cần import. Bạn có thể import từ `vue/macros` nếu muốn tường minh:

```js
import { $ref } from 'vue/macros'

let count = $ref(0)
```

## Destructuring with `$()` {#destructuring-with}

Một composition function thường trả về object các ref, và ta dùng destructuring để lấy chúng. Reactivity transform cung cấp macro **`$()`** cho mục đích này:

```js
import { useMouse } from '@vueuse/core'

const { x, y } = $(useMouse())

console.log(x, y)
```

Compiled output:

```js
import { toRef } from 'vue'
import { useMouse } from '@vueuse/core'

const __temp = useMouse(),
  x = toRef(__temp, 'x'),
  y = toRef(__temp, 'y')

console.log(x.value, y.value)
```

Lưu ý nếu `x` đã là ref, `toRef(__temp, 'x')` sẽ trả lại nguyên si, không tạo ref mới. Nếu giá trị destructure không phải ref (vd một hàm), nó vẫn hoạt động — giá trị sẽ được bọc trong ref để phần còn lại của code chạy đúng.

`$()` destructure works on both reactive objects **and** plain objects containing refs.

## Convert Existing Refs to Reactive Variables with `$()` {#convert-existing-refs-to-reactive-variables-with}

Đôi khi ta có các hàm bọc bên ngoài cũng trả về ref. Compiler không thể biết trước một hàm sẽ trả về ref. Khi đó, macro `$()` cũng dùng để chuyển ref hiện có thành reactive variable:

```js
function myCreateRef() {
  return ref(0)
}

let count = $(myCreateRef())
```

## Reactive Props Destructure {#reactive-props-destructure}

Có hai điểm khó với `defineProps()` trong `<script setup>`:

1. Similar to `.value`, you need to always access props as `props.x` in order to retain reactivity. This means you cannot destructure `defineProps` because the resulting destructured variables are not reactive and will not update.

2. When using the [type-only props declaration](/api/sfc-script-setup#type-only-props-emit-declarations), there is no easy way to declare default values for the props. We introduced the `withDefaults()` API for this exact purpose, but it's still clunky to use.

Ta có thể giải quyết bằng transform compile‑time khi `defineProps` được dùng với destructuring, tương tự `$()`:

```html
<script setup lang="ts">
  interface Props {
    msg: string
    count?: number
    foo?: string
  }

  const {
    msg,
    // default value just works
    count = 1,
    // local aliasing also just works
    // here we are aliasing `props.foo` to `bar`
    foo: bar
  } = defineProps<Props>()

  watchEffect(() => {
    // will log whenever the props change
    console.log(msg, count, bar)
  })
</script>
```

The above will be compiled into the following runtime declaration equivalent:

```js
export default {
  props: {
    msg: { type: String, required: true },
    count: { type: Number, default: 1 },
    foo: String
  },
  setup(props) {
    watchEffect(() => {
      console.log(props.msg, props.count, props.foo)
    })
  }
}
```

## Retaining Reactivity Across Function Boundaries {#retaining-reactivity-across-function-boundaries}

Dù reactive variable giúp tránh dùng `.value` ở khắp nơi, nó lại gây vấn đề “mất reactivity” khi truyền qua ranh giới hàm. Có hai trường hợp:

### Passing into function as argument {#passing-into-function-as-argument}

Given a function that expects a ref as an argument, e.g.:

```ts
function trackChange(x: Ref<number>) {
  watch(x, (x) => {
    console.log('x changed!')
  })
}

let count = $ref(0)
trackChange(count) // doesn't work!
```

Trường hợp trên không hoạt động như kỳ vọng vì nó biên dịch thành:

```ts
let count = ref(0)
trackChange(count.value)
```

Ở đây `count.value` được truyền như số, trong khi `trackChange` cần ref. Sửa bằng cách bọc `count` với `$$()` trước khi truyền:

```diff
let count = $ref(0)
- trackChange(count)
+ trackChange($$(count))
```

The above compiles to:

```js
import { ref } from 'vue'

let count = ref(0)
trackChange(count)
```

Như bạn thấy, `$$()` là macro đóng vai trò **escape hint**: các reactive variable bên trong `$$()` sẽ không bị thêm `.value`.

### Returning inside function scope {#returning-inside-function-scope}

Reactivity cũng bị mất nếu reactive variable được dùng trực tiếp trong biểu thức trả về:

```ts
function useMouse() {
  let x = $ref(0)
  let y = $ref(0)

  // listen to mousemove...

  // doesn't work!
  return {
    x,
    y
  }
}
```

The above return statement compiles to:

```ts
return {
  x: x.value,
  y: y.value
}
```

Để giữ reactivity, ta nên trả về chính ref, không phải giá trị tại thời điểm trả về.

Again, we can use `$$()` to fix this. In this case, `$$()` can be used directly on the returned object - any reference to reactive variables inside the `$$()` call will retain the reference to their underlying refs:

```ts
function useMouse() {
  let x = $ref(0)
  let y = $ref(0)

  // listen to mousemove...

  // fixed
  return $$({
    x,
    y
  })
}
```

### Using `$$()` on destructured props {#using-on-destructured-props}

`$$()` cũng hoạt động trên props đã destructure vì chúng cũng là reactive variable. Compiler sẽ chuyển bằng `toRef` để hiệu quả hơn:

```ts
const { count } = defineProps<{ count: number }>()

passAsRef($$(count))
```

compiles to:

```js
setup(props) {
  const __props_count = toRef(props, 'count')
  passAsRef(__props_count)
}
```

## TypeScript Integration <sup class="vt-badge ts" /> {#typescript-integration}

Vue cung cấp kiểu cho các macro này (khả dụng toàn cục) và mọi kiểu hoạt động như mong đợi. Không có xung đột với ngữ nghĩa TypeScript chuẩn, nên cú pháp hoạt động với toàn bộ tooling hiện có.

Điều này cũng có nghĩa macro có thể dùng ở bất kỳ file JS/TS hợp lệ — không chỉ trong Vue SFC.

Vì macro khả dụng toàn cục, kiểu của chúng cần được tham chiếu tường minh (vd trong file `env.d.ts`):

```ts
/// <reference types="vue/macros-global" />
```

Khi import tường minh macro từ `vue/macros`, kiểu sẽ hoạt động mà không cần khai báo global.

## Explicit Opt-in {#explicit-opt-in}

:::danger No longer supported in core
Phần dưới chỉ áp dụng tới Vue 3.3 trở xuống. Hỗ trợ đã bị gỡ trong Vue core 3.4+ và `@vitejs/plugin-vue` 5.0+. Nếu muốn tiếp tục dùng transform, hãy chuyển sang [Vue Macros](https://vue-macros.sxzz.moe/features/reactivity-transform.html).
:::

### Vite {#vite}

- Cần `@vitejs/plugin-vue@>=2.0.0`
- Áp dụng cho SFC và file js(x)/ts(x). Có kiểm tra nhanh trước khi áp transform nên không tốn hiệu năng với file không dùng macro.
- Lưu ý `reactivityTransform` nay là tùy chọn ở cấp plugin root thay vì lồng trong `script.refSugar`, vì nó ảnh hưởng không chỉ SFC.

```js [vite.config.js]
export default {
  plugins: [
    vue({
      reactivityTransform: true
    })
  ]
}
```

### `vue-cli` {#vue-cli}

- Hiện chỉ ảnh hưởng SFC
- Cần `vue-loader@>=17.0.0`

```js [vue.config.js]
module.exports = {
  chainWebpack: (config) => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .tap((options) => {
        return {
          ...options,
          reactivityTransform: true
        }
      })
  }
}
```

### Plain `webpack` + `vue-loader` {#plain-webpack-vue-loader}

- Hiện chỉ ảnh hưởng SFC
- Cần `vue-loader@>=17.0.0`

```js [webpack.config.js]
module.exports = {
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          reactivityTransform: true
        }
      }
    ]
  }
}
```
