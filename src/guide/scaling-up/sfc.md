# Single-File Components {#single-file-components}

## Introduction {#introduction}

Vue Single‑File Components (hay `*.vue`, viết tắt **SFC**) là định dạng file đặc biệt cho phép đóng gói template, logic **và** styling của một Vue component trong một file duy nhất. Ví dụ SFC:

<div class="options-api">

```vue
<script>
export default {
  data() {
    return {
      greeting: 'Hello World!'
    }
  }
}
</script>

<template>
  <p class="greeting">{{ greeting }}</p>
</template>

<style>
.greeting {
  color: red;
  font-weight: bold;
}
</style>
```

</div>

<div class="composition-api">

```vue
<script setup>
import { ref } from 'vue'
const greeting = ref('Hello World!')
</script>

<template>
  <p class="greeting">{{ greeting }}</p>
</template>

<style>
.greeting {
  color: red;
  font-weight: bold;
}
</style>
```

</div>

Như bạn thấy, SFC là phần mở rộng tự nhiên của bộ ba HTML, CSS và JavaScript. Các khối `<template>`, `<script>`, và `<style>` đóng gói và đặt chung phần view, logic và styling của component trong cùng một file. Cú pháp đầy đủ được định nghĩa ở [SFC Syntax Specification](/api/sfc-spec).

## Why SFC {#why-sfc}

Mặc dù SFC cần build step, bù lại có rất nhiều lợi ích:

- Author modularized components using familiar HTML, CSS and JavaScript syntax
- [Colocation of inherently coupled concerns](#what-about-separation-of-concerns)
- Pre-compiled templates without runtime compilation cost
- [Component-scoped CSS](/api/sfc-css-features)
- [More ergonomic syntax when working with Composition API](/api/sfc-script-setup)
- More compile-time optimizations by cross-analyzing template and script
- [IDE support](/guide/scaling-up/tooling#ide-support) with auto-completion and type-checking for template expressions
- Out-of-the-box Hot-Module Replacement (HMR) support

SFC là đặc trưng quan trọng của Vue như một framework, và là cách được khuyến nghị dùng Vue trong các trường hợp:

- Single-Page Applications (SPA)
- Static Site Generation (SSG)
- Any non-trivial frontend where a build step can be justified for better development experience (DX).

Tuy vậy, có những bối cảnh SFC có thể là “quá tay”. Vì thế Vue vẫn có thể dùng bằng JavaScript thuần không cần build step. Nếu bạn chỉ muốn tăng cường HTML tĩnh với tương tác nhẹ, hãy xem [petite-vue](https://github.com/vuejs/petite-vue), phiên bản 6 kB của Vue tối ưu cho progressive enhancement.

## How It Works {#how-it-works}

Vue SFC là định dạng file dành riêng cho framework và phải được tiền biên dịch bởi [@vue/compiler-sfc](https://github.com/vuejs/core/tree/main/packages/compiler-sfc) thành JavaScript và CSS chuẩn. Một SFC sau biên dịch là một module JavaScript (ES) chuẩn — tức với build setup phù hợp bạn có thể import SFC như một module:

```js
import MyComponent from './MyComponent.vue'

export default {
  components: {
    MyComponent
  }
}
```

Thẻ `<style>` bên trong SFC thường được chèn thành thẻ `<style>` gốc khi phát triển để hỗ trợ hot update. Ở production, chúng có thể được tách và gộp vào một file CSS.

Bạn có thể thử SFC và xem cách chúng được biên dịch tại [Vue SFC Playground](https://play.vuejs.org/).

Trong dự án thực tế, ta thường tích hợp SFC compiler với build tool như [Vite](https://vitejs.dev/) hoặc [Vue CLI](http://cli.vuejs.org/) (dựa trên [webpack](https://webpack.js.org/)), và Vue cung cấp công cụ scaffold chính thức để bắt đầu với SFC nhanh nhất. Xem chi tiết ở phần [SFC Tooling](/guide/scaling-up/tooling).

## What About Separation of Concerns? {#what-about-separation-of-concerns}

Vài người đến từ nền tảng web truyền thống có thể lo rằng SFC trộn nhiều mối quan tâm vào cùng một chỗ — trong khi HTML/CSS/JS vốn tách biệt!

Để trả lời, cần thống nhất rằng **tách mối quan tâm không đồng nghĩa tách loại file**. Mục tiêu cuối cùng của nguyên tắc kỹ thuật là cải thiện khả năng bảo trì. Việc áp dụng máy móc “tách mối quan tâm” thành “tách loại file” không giúp đạt mục tiêu đó trong bối cảnh frontend ngày càng phức tạp.

Trong phát triển UI hiện đại, thay vì chia codebase thành ba lớp lớn đan xen nhau, chia thành các component ít phụ thuộc và lắp ghép chúng hợp lý hơn. Bên trong một component, template, logic và styles vốn gắn kết, và đặt chúng cạnh nhau khiến component gọn ghẽ, dễ bảo trì hơn.

Ngay cả khi bạn không thích Single‑File Components, bạn vẫn có thể tận dụng hot‑reloading và tiền biên dịch bằng cách tách JavaScript và CSS vào file riêng qua [Src Imports](/api/sfc-spec#src-imports).
