---
outline: deep
---

# Using Vue with TypeScript {#using-vue-with-typescript}

Một hệ thống kiểu như TypeScript có thể phát hiện nhiều lỗi phổ biến qua phân tích tĩnh ở thời điểm build. Điều này giảm khả năng lỗi runtime ở production, và giúp tự tin refactor trong ứng dụng lớn. TypeScript cũng cải thiện trải nghiệm phát triển qua tự động hoàn thành dựa trên kiểu trong IDE.

Vue được viết bằng TypeScript và cung cấp hỗ trợ TypeScript hạng nhất. Tất cả gói chính thức của Vue đều đi kèm khai báo kiểu, hoạt động ngay.

## Project Setup {#project-setup}

[`create-vue`](https://github.com/vuejs/create-vue), công cụ scaffold chính thức, cung cấp tùy chọn tạo dự án Vue dùng [Vite](https://vitejs.dev/) với TypeScript sẵn sàng.

### Overview {#overview}

Với thiết lập dựa trên Vite, dev server và bundler chỉ transpile, không kiểm tra kiểu. Điều này giúp dev server của Vite vẫn rất nhanh ngay cả khi dùng TypeScript.

- Trong phát triển, khuyến nghị dựa vào [thiết lập IDE](#ide-support) tốt để nhận phản hồi lỗi kiểu tức thì.

- Nếu dùng SFC, dùng tiện ích [`vue-tsc`](https://github.com/vuejs/language-tools/tree/master/packages/tsc) để kiểm tra kiểu và tạo khai báo kiểu từ dòng lệnh. `vue-tsc` là wrapper quanh `tsc`, hoạt động gần như tương tự nhưng hỗ trợ thêm SFC. Bạn có thể chạy `vue-tsc` ở watch mode song song dev server Vite, hoặc dùng plugin như [vite-plugin-checker](https://vite-plugin-checker.netlify.app/) chạy kiểm tra ở worker riêng.

- Vue CLI cũng hỗ trợ TypeScript, nhưng không còn khuyến nghị. Xem [ghi chú](#note-on-vue-cli-and-ts-loader).

### IDE Support {#ide-support}

- [Visual Studio Code](https://code.visualstudio.com/) (VS Code) is strongly recommended for its great out-of-the-box support for TypeScript.

  - [Vue - Official](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (previously Volar) is the official VS Code extension that provides TypeScript support inside Vue SFCs, along with many other great features.

    :::tip
    Vue - Official extension replaces [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur), our previous official VS Code extension for Vue 2. If you have Vetur currently installed, make sure to disable it in Vue 3 projects.
    :::

- [WebStorm](https://www.jetbrains.com/webstorm/) also provides out-of-the-box support for both TypeScript and Vue. Other JetBrains IDEs support them too, either out of the box or via [a free plugin](https://plugins.jetbrains.com/plugin/9442-vue-js). As of version 2023.2, WebStorm and the Vue Plugin come with built-in support for the Vue Language Server. You can set the Vue service to use Volar integration on all TypeScript versions, under Settings > Languages & Frameworks > TypeScript > Vue. By default, Volar will be used for TypeScript versions 5.0 and higher.

### Configuring `tsconfig.json` {#configuring-tsconfig-json}

Các dự án scaffold qua `create-vue` gồm sẵn `tsconfig.json`. Cấu hình cơ sở được trừu tượng trong gói [`@vue/tsconfig`](https://github.com/vuejs/tsconfig). Trong dự án, ta dùng [Project References](https://www.typescriptlang.org/docs/handbook/project-references.html) để đảm bảo kiểu đúng cho code chạy ở môi trường khác nhau (ví dụ code app và code test có biến global khác nhau).

Khi tự cấu hình `tsconfig.json`, một số tùy chọn đáng chú ý:

- [`compilerOptions.isolatedModules`](https://www.typescriptlang.org/tsconfig#isolatedModules) is set to `true` because Vite uses [esbuild](https://esbuild.github.io/) for transpiling TypeScript and is subject to single-file transpile limitations. [`compilerOptions.verbatimModuleSyntax`](https://www.typescriptlang.org/tsconfig#verbatimModuleSyntax) is [a superset of `isolatedModules`](https://github.com/microsoft/TypeScript/issues/53601) and is a good choice, too - it's what [`@vue/tsconfig`](https://github.com/vuejs/tsconfig) uses.

- Nếu bạn dùng Options API, cần đặt [`compilerOptions.strict`](https://www.typescriptlang.org/tsconfig#strict) thành `true` (hoặc ít nhất bật [`compilerOptions.noImplicitThis`](https://www.typescriptlang.org/tsconfig#noImplicitThis), một phần của `strict`) để tận dụng kiểm tra kiểu của `this` trong options. Nếu không `this` sẽ là `any`.

- Nếu bạn cấu hình alias resolver trong build tool (ví dụ alias `@/*` mặc định trong `create-vue`), bạn cũng cần cấu hình cho TypeScript qua [`compilerOptions.paths`](https://www.typescriptlang.org/tsconfig#paths).

- Nếu bạn định dùng TSX với Vue, đặt [`compilerOptions.jsx`](https://www.typescriptlang.org/tsconfig#jsx) là `"preserve"`, và [`compilerOptions.jsxImportSource`](https://www.typescriptlang.org/tsconfig#jsxImportSource) là `"vue"`.

See also:

- [Official TypeScript compiler options docs](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
- [esbuild TypeScript compilation caveats](https://esbuild.github.io/content-types/#typescript-caveats)

### Note on Vue CLI and `ts-loader` {#note-on-vue-cli-and-ts-loader}

Trong thiết lập dựa trên webpack như Vue CLI, thường kiểm tra kiểu là một phần pipeline transform module (ví dụ với `ts-loader`). Tuy nhiên, đây không phải giải pháp sạch vì hệ thống kiểu cần biết toàn bộ đồ thị module. Bước transform của từng module không phải nơi phù hợp. Nó dẫn tới:

- `ts-loader` can only type check post-transform code. This doesn't align with the errors we see in IDEs or from `vue-tsc`, which map directly back to the source code.

- Type checking can be slow. When it is performed in the same thread / process with code transformations, it significantly affects the build speed of the entire application.

- We already have type checking running right in our IDE in a separate process, so the cost of dev experience slow down simply isn't a good trade-off.

Nếu bạn đang dùng Vue 3 + TypeScript qua Vue CLI, chúng tôi khuyến nghị chuyển sang Vite. Chúng tôi cũng đang làm việc để hỗ trợ chỉ transpile TS, cho phép bạn chuyển sang `vue-tsc` để kiểm tra kiểu.

## General Usage Notes {#general-usage-notes}

### `defineComponent()` {#definecomponent}

To let TypeScript properly infer types inside component options, we need to define components with [`defineComponent()`](/api/general#definecomponent):

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  // type inference enabled
  props: {
    name: String,
    msg: { type: String, required: true }
  },
  data() {
    return {
      count: 1
    }
  },
  mounted() {
    this.name // type: string | undefined
    this.msg // type: string
    this.count // type: number
  }
})
```

`defineComponent()` also supports inferring the props passed to `setup()` when using Composition API without `<script setup>`:

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  // type inference enabled
  props: {
    message: String
  },
  setup(props) {
    props.message // type: string | undefined
  }
})
```

See also:

- [Note on webpack Treeshaking](/api/general#note-on-webpack-treeshaking)
- [type tests for `defineComponent`](https://github.com/vuejs/core/blob/main/packages-private/dts-test/defineComponent.test-d.tsx)

:::tip
`defineComponent()` also enables type inference for components defined in plain JavaScript.
:::

### Usage in Single-File Components {#usage-in-single-file-components}

To use TypeScript in SFCs, add the `lang="ts"` attribute to `<script>` tags. When `lang="ts"` is present, all template expressions also enjoy stricter type checking.

```vue
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  data() {
    return {
      count: 1
    }
  }
})
</script>

<template>
  <!-- type checking and auto-completion enabled -->
  {{ count.toFixed(2) }}
</template>
```

`lang="ts"` can also be used with `<script setup>`:

```vue
<script setup lang="ts">
// TypeScript enabled
import { ref } from 'vue'

const count = ref(1)
</script>

<template>
  <!-- type checking and auto-completion enabled -->
  {{ count.toFixed(2) }}
</template>
```

### TypeScript in Templates {#typescript-in-templates}

The `<template>` also supports TypeScript in binding expressions when `<script lang="ts">` or `<script setup lang="ts">` is used. This is useful in cases where you need to perform type casting in template expressions.

Here's a contrived example:

```vue
<script setup lang="ts">
let x: string | number = 1
</script>

<template>
  <!-- error because x could be a string -->
  {{ x.toFixed(2) }}
</template>
```

This can be worked around with an inline type cast:

```vue{6}
<script setup lang="ts">
let x: string | number = 1
</script>

<template>
  {{ (x as number).toFixed(2) }}
</template>
```

:::tip
If using Vue CLI or a webpack-based setup, TypeScript in template expressions requires `vue-loader@^16.8.0`.
:::

### Usage with TSX {#usage-with-tsx}

Vue also supports authoring components with JSX / TSX. Details are covered in the [Render Function & JSX](/guide/extras/render-function.html#jsx-tsx) guide.

## Generic Components {#generic-components}

Generic components are supported in two cases:

- In SFCs: [`<script setup>` with the `generic` attribute](/api/sfc-script-setup.html#generics)
- Render function / JSX components: [`defineComponent()`'s function signature](/api/general.html#function-signature)

## API-Specific Recipes {#api-specific-recipes}

- [TS with Composition API](./composition-api)
- [TS with Options API](./options-api)
