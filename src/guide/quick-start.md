---
footer: false
---

<script setup>
import { VTCodeGroup, VTCodeGroupTab } from '@vue/theme'
</script>

# Quick Start {#quick-start}

## Try Vue Online {#try-vue-online}

- Để nhanh chóng làm quen với Vue, bạn có thể thử trực tiếp trong [Playground](https://play.vuejs.org/#eNo9jcEKwjAMhl/lt5fpQYfXUQfefAMvvRQbddC1pUuHUPrudg4HIcmXjyRZXEM4zYlEJ+T0iEPgXjn6BB8Zhp46WUZWDjCa9f6w9kAkTtH9CRinV4fmRtZ63H20Ztesqiylphqy3R5UYBqD1UyVAPk+9zkvV1CKbCv9poMLiTEfR2/IXpSoXomqZLtti/IFwVtA9A==).

- Nếu bạn thích thiết lập HTML thuần mà không cần build step, bạn có thể dùng [JSFiddle](https://jsfiddle.net/yyx990803/2ke1ab0z/) này làm điểm bắt đầu.

- Nếu bạn đã quen với Node.js và khái niệm build tools, bạn cũng có thể thử một thiết lập build hoàn chỉnh ngay trong trình duyệt với [StackBlitz](https://vite.new/vue).

- Để có phần hướng dẫn từng bước cho thiết lập khuyến nghị, hãy xem khóa học tương tác trên [Scrimba](http://scrimba.com/links/vue-quickstart) cho thấy cách chạy, chỉnh sửa và deploy ứng dụng Vue đầu tiên của bạn.

## Creating a Vue Application {#creating-a-vue-application}

:::tip Prerequisites

- Quen thuộc với command line
- Cài đặt [Node.js](https://nodejs.org/) phiên bản `^20.19.0 || >=22.12.0`
  :::

Trong phần này, chúng ta sẽ giới thiệu cách scaffold một Vue [Single Page Application](/guide/extras/ways-of-using-vue#single-page-application-spa) trên máy của bạn. Project được tạo sẽ dùng build setup dựa trên [Vite](https://vitejs.dev) và cho phép chúng ta dùng Vue [Single-File Components](/guide/scaling-up/sfc) (SFCs).

Hãy đảm bảo bạn đã cài [Node.js](https://nodejs.org/) bản mới và thư mục làm việc hiện tại là nơi bạn muốn tạo project. Chạy lệnh sau trong command line (không gõ ký hiệu `$`):

::: code-group

```sh [npm]
$ npm create vue@latest
```

```sh [pnpm]
$ pnpm create vue@latest
```

```sh [yarn]
# For Yarn (v1+)
$ yarn create vue

# For Yarn Modern (v2+)
$ yarn create vue@latest
  
# For Yarn ^v4.11
$ yarn dlx create-vue@latest
```

```sh [bun]
$ bun create vue@latest
```
:::

Lệnh này sẽ cài đặt và chạy [create-vue](https://github.com/vuejs/create-vue), công cụ scaffold project chính thức của Vue. Bạn sẽ được hỏi về một số tùy chọn như TypeScript và hỗ trợ testing:

<div class="language-sh"><pre><code><span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Project name: <span style="color:#888;">… <span style="color:#89DDFF;">&lt;</span><span style="color:#888;">your-project-name</span><span style="color:#89DDFF;">&gt;</span></span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add TypeScript? <span style="color:#888;">… <span style="color:#89DDFF;text-decoration:underline">No</span> / Yes</span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add JSX Support? <span style="color:#888;">… <span style="color:#89DDFF;text-decoration:underline">No</span> / Yes</span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add Vue Router for Single Page Application development? <span style="color:#888;">… <span style="color:#89DDFF;text-decoration:underline">No</span> / Yes</span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add Pinia for state management? <span style="color:#888;">… <span style="color:#89DDFF;text-decoration:underline">No</span> / Yes</span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add Vitest for Unit testing? <span style="color:#888;">… <span style="color:#89DDFF;text-decoration:underline">No</span> / Yes</span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add an End-to-End Testing Solution? <span style="color:#888;">… <span style="color:#89DDFF;text-decoration:underline">No</span> / Cypress / Nightwatch / Playwright</span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add ESLint for code quality? <span style="color:#888;">… No / <span style="color:#89DDFF;text-decoration:underline">Yes</span></span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add Prettier for code formatting? <span style="color:#888;">… <span style="color:#89DDFF;text-decoration:underline">No</span> / Yes</span></span>
<span style="color:var(--vt-c-green);">✔</span> <span style="color:#A6ACCD;">Add Vue DevTools 7 extension for debugging? (experimental) <span style="color:#888;">… <span style="color:#89DDFF;text-decoration:underline">No</span> / Yes</span></span>
<span></span>
<span style="color:#A6ACCD;">Scaffolding project in ./<span style="color:#89DDFF;">&lt;</span><span style="color:#888;">your-project-name</span><span style="color:#89DDFF;">&gt;</span>...</span>
<span style="color:#A6ACCD;">Done.</span></code></pre></div>

Nếu bạn chưa chắc về một tùy chọn nào đó, tạm thời cứ chọn `No` bằng cách nhấn Enter. Khi project đã được tạo, làm theo hướng dẫn để cài dependencies và khởi động dev server:

::: code-group

```sh-vue [npm]
$ cd {{'<your-project-name>'}}
$ npm install
$ npm run dev
```

```sh-vue [pnpm]
$ cd {{'<your-project-name>'}}
$ pnpm install
$ pnpm run dev
```

```sh-vue [yarn]
$ cd {{'<your-project-name>'}}
$ yarn
$ yarn dev
```

```sh-vue [bun]
$ cd {{'<your-project-name>'}}
$ bun install
$ bun run dev
```

:::


Giờ đây bạn đã chạy được project Vue đầu tiên! Lưu ý các component ví dụ trong project được tạo được viết bằng [Composition API](/guide/introduction#composition-api) và `<script setup>`, thay vì [Options API](/guide/introduction#options-api). Một số gợi ý thêm:

- Thiết lập IDE khuyến nghị là [Visual Studio Code](https://code.visualstudio.com/) + [Vue - Official extension](https://marketplace.visualstudio.com/items?itemName=Vue.volar). Nếu dùng editor khác, xem phần [IDE support](/guide/scaling-up/tooling#ide-support).
- Thêm chi tiết về tooling, bao gồm tích hợp với backend frameworks, có trong [Tooling Guide](/guide/scaling-up/tooling).
- Để tìm hiểu thêm về build tool nền tảng là Vite, xem [Vite docs](https://vitejs.dev).
- Nếu bạn chọn dùng TypeScript, xem [TypeScript Usage Guide](typescript/overview).

Khi bạn sẵn sàng đưa ứng dụng lên production, chạy các lệnh sau:

::: code-group

```sh [npm]
$ npm run build
```

```sh [pnpm]
$ pnpm run build
```

```sh [yarn]
$ yarn build
```

```sh [bun]
$ bun run build
```

:::


Lệnh này sẽ tạo build sẵn sàng cho production của ứng dụng trong thư mục `./dist` của project. Xem [Production Deployment Guide](/guide/best-practices/production-deployment) để biết thêm về việc đưa ứng dụng lên production.

[Next Steps >](#next-steps)

## Using Vue from CDN {#using-vue-from-cdn}

Bạn có thể dùng Vue trực tiếp từ CDN qua thẻ script:

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```

Ở đây chúng ta dùng [unpkg](https://unpkg.com/), nhưng bạn cũng có thể dùng bất kỳ CDN nào phục vụ gói npm, ví dụ [jsdelivr](https://www.jsdelivr.com/package/npm/vue) hoặc [cdnjs](https://cdnjs.com/libraries/vue). Tất nhiên, bạn cũng có thể tải file này về và tự phục vụ.

Khi dùng Vue từ CDN, sẽ không có "build step". Thiết lập vì thế trở nên đơn giản hơn nhiều và phù hợp để tăng cường HTML tĩnh hoặc tích hợp với backend framework. Tuy nhiên, bạn sẽ không thể dùng cú pháp Single‑File Component (SFC).

### Using the Global Build {#using-the-global-build}

Liên kết trên tải _global build_ của Vue, nơi tất cả top‑level API được lộ ra như các thuộc tính trên object toàn cục `Vue`. Dưới đây là một ví dụ đầy đủ dùng global build:

<div class="options-api">

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

<div id="app">{{ message }}</div>

<script>
  const { createApp } = Vue

  createApp({
    data() {
      return {
        message: 'Hello Vue!'
      }
    }
  }).mount('#app')
</script>
```

[CodePen Demo >](https://codepen.io/vuejs-examples/pen/QWJwJLp)

</div>

<div class="composition-api">

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

<div id="app">{{ message }}</div>

<script>
  const { createApp, ref } = Vue

  createApp({
    setup() {
      const message = ref('Hello vue!')
      return {
        message
      }
    }
  }).mount('#app')
</script>
```

[CodePen Demo >](https://codepen.io/vuejs-examples/pen/eYQpQEG)

:::tip
Nhiều ví dụ về Composition API trong suốt phần hướng dẫn sẽ dùng cú pháp `<script setup>`, vốn yêu cầu build tools. Nếu bạn định dùng Composition API mà không có build step, tham khảo cách dùng [`setup()` option](/api/composition-api-setup).
:::

</div>

### Using the ES Module Build {#using-the-es-module-build}

Trong phần còn lại của tài liệu, chúng ta chủ yếu dùng cú pháp [ES modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules). Hầu hết trình duyệt hiện đại đã hỗ trợ ES modules một cách nguyên bản, vì vậy ta có thể dùng Vue từ CDN thông qua ES modules như sau:

<div class="options-api">

```html{3,4}
<div id="app">{{ message }}</div>

<script type="module">
  import { createApp } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'

  createApp({
    data() {
      return {
        message: 'Hello Vue!'
      }
    }
  }).mount('#app')
</script>
```

</div>

<div class="composition-api">

```html{3,4}
<div id="app">{{ message }}</div>

<script type="module">
  import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'

  createApp({
    setup() {
      const message = ref('Hello Vue!')
      return {
        message
      }
    }
  }).mount('#app')
</script>
```

</div>

Lưu ý chúng ta đang dùng `<script type="module">`, và URL CDN được import trỏ đến **ES modules build** của Vue.

<div class="options-api">

[CodePen Demo >](https://codepen.io/vuejs-examples/pen/VwVYVZO)

</div>
<div class="composition-api">

[CodePen Demo >](https://codepen.io/vuejs-examples/pen/MWzazEv)

</div>

### Enabling Import maps {#enabling-import-maps}

Trong ví dụ trên, chúng ta import từ CDN URL đầy đủ, nhưng ở phần còn lại của tài liệu bạn sẽ thấy code như sau:

```js
import { createApp } from 'vue'
```

Ta có thể “chỉ” cho trình duyệt biết `vue` được import từ đâu bằng [Import Maps](https://caniuse.com/import-maps):

<div class="options-api">

```html{1-7,12}
<script type="importmap">
  {
    "imports": {
      "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
    }
  }
</script>

<div id="app">{{ message }}</div>

<script type="module">
  import { createApp } from 'vue'

  createApp({
    data() {
      return {
        message: 'Hello Vue!'
      }
    }
  }).mount('#app')
</script>
```

[CodePen Demo >](https://codepen.io/vuejs-examples/pen/wvQKQyM)

</div>

<div class="composition-api">

```html{1-7,12}
<script type="importmap">
  {
    "imports": {
      "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
    }
  }
</script>

<div id="app">{{ message }}</div>

<script type="module">
  import { createApp, ref } from 'vue'

  createApp({
    setup() {
      const message = ref('Hello Vue!')
      return {
        message
      }
    }
  }).mount('#app')
</script>
```

[CodePen Demo >](https://codepen.io/vuejs-examples/pen/YzRyRYM)

</div>

Bạn cũng có thể thêm các entry cho dependencies khác vào import map — nhưng đảm bảo chúng trỏ tới phiên bản ES modules của thư viện bạn muốn dùng.

:::tip Import Maps Browser Support
Import Maps là một tính năng trình duyệt còn khá mới. Hãy dùng trình duyệt nằm trong [phạm vi hỗ trợ](https://caniuse.com/import-maps). Đặc biệt, nó chỉ được hỗ trợ trên Safari 16.4+.
:::

:::warning Notes on Production Use
Các ví dụ đến đây dùng development build của Vue — nếu bạn định dùng Vue từ CDN trong production, nhớ xem [Production Deployment Guide](/guide/best-practices/production-deployment#without-build-tools).

Mặc dù có thể dùng Vue mà không cần build system, một phương án khác đáng cân nhắc là dùng [`vuejs/petite-vue`](https://github.com/vuejs/petite-vue), có thể phù hợp hơn với bối cảnh nơi trước đây dùng [`jquery/jquery`](https://github.com/jquery/jquery) hoặc hiện nay dùng [`alpinejs/alpine`](https://github.com/alpinejs/alpine).
:::

### Splitting Up the Modules {#splitting-up-the-modules}

Khi đi sâu hơn vào phần hướng dẫn, chúng ta có thể cần tách code thành các file JavaScript riêng để dễ quản lý hơn. Ví dụ:

```html [index.html]
<div id="app"></div>

<script type="module">
  import { createApp } from 'vue'
  import MyComponent from './my-component.js'

  createApp(MyComponent).mount('#app')
</script>
```

<div class="options-api">

```js [my-component.js]
export default {
  data() {
    return { count: 0 }
  },
  template: `<div>Count is: {{ count }}</div>`
}
```

</div>
<div class="composition-api">

```js [my-component.js]
import { ref } from 'vue'
export default {
  setup() {
    const count = ref(0)
    return { count }
  },
  template: `<div>Count is: {{ count }}</div>`
}
```

</div>

Nếu bạn mở trực tiếp `index.html` ở trên trong trình duyệt, bạn sẽ thấy lỗi vì ES modules không thể hoạt động qua giao thức `file://`, vốn là giao thức trình duyệt dùng khi mở file cục bộ.

Vì lý do bảo mật, ES modules chỉ hoạt động qua giao thức `http://`, là giao thức trình duyệt dùng khi mở trang web. Để ES modules hoạt động trên máy cục bộ, ta cần phục vụ `index.html` qua `http://` bằng một HTTP server cục bộ.

Để khởi động HTTP server cục bộ, trước tiên đảm bảo bạn đã cài [Node.js](https://nodejs.org/en/), sau đó chạy `npx serve` trong command line tại cùng thư mục với file HTML của bạn. Bạn cũng có thể dùng bất kỳ HTTP server nào khác có thể phục vụ static files với đúng MIME type.

Bạn có thể để ý template của component được import đang được viết inline dưới dạng chuỗi JavaScript. Nếu dùng VS Code, bạn có thể cài extension [es6-string-html](https://marketplace.visualstudio.com/items?itemName=Tobermory.es6-string-html) và thêm tiền tố comment `/*html*/` trước chuỗi để có syntax highlighting.

## Next Steps {#next-steps}

Nếu bạn đã bỏ qua phần [Introduction](/guide/introduction), chúng tôi rất khuyến nghị đọc nó trước khi tiếp tục với phần còn lại của tài liệu.

<div class="vt-box-container next-steps">
  <a class="vt-box" href="/guide/essentials/application.html">
    <p class="next-steps-link">Continue with the Guide</p>
    <p class="next-steps-caption">Hướng dẫn này đưa bạn qua mọi khía cạnh của framework một cách chi tiết.</p>
  </a>
  <a class="vt-box" href="/tutorial/">
    <p class="next-steps-link">Try the Tutorial</p>
    <p class="next-steps-caption">Dành cho những ai thích học bằng thực hành.</p>
  </a>
  <a class="vt-box" href="/examples/">
    <p class="next-steps-link">Check out the Examples</p>
    <p class="next-steps-caption">Khám phá các ví dụ về tính năng cốt lõi và các tác vụ UI phổ biến.</p>
  </a>
</div>
