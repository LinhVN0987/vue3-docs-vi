# Vue and Web Components {#vue-and-web-components}

[Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components) là thuật ngữ bao trùm cho tập API thuần nền tảng giúp tạo custom element có thể tái sử dụng.

Chúng tôi xem Vue và Web Components chủ yếu là công nghệ bổ trợ cho nhau. Vue hỗ trợ rất tốt cả việc tiêu thụ lẫn tạo custom element. Dù bạn tích hợp custom element vào ứng dụng Vue hiện có, hay dùng Vue để xây và phân phối custom element, đều là lựa chọn tốt.

## Using Custom Elements in Vue {#using-custom-elements-in-vue}

Vue [đạt 100% trong bài kiểm tra Custom Elements Everywhere](https://custom-elements-everywhere.com/libraries/vue/results/results.html). Dùng custom element trong ứng dụng Vue nhìn chung giống dùng phần tử HTML gốc, với vài lưu ý:

### Skipping Component Resolution {#skipping-component-resolution}

Mặc định, Vue sẽ cố resolve thẻ không phải HTML gốc thành component Vue đã đăng ký trước khi render nó như custom element. Điều này gây cảnh báo “failed to resolve component” trong quá trình phát triển. Để Vue coi một số thẻ là custom element và bỏ qua resolve, chỉ định [`compilerOptions.isCustomElement`](/api/application#app-config-compileroptions).

Nếu bạn dùng Vue với build setup, tuỳ chọn này nên được cấu hình qua công cụ build vì đây là tuỳ chọn ở compile-time.

#### Example In-Browser Config {#example-in-browser-config}

```js
// Only works if using in-browser compilation.
// If using build tools, see config examples below.
app.config.compilerOptions.isCustomElement = (tag) => tag.includes('-')
```

#### Example Vite Config {#example-vite-config}

```js [vite.config.js]
import vue from '@vitejs/plugin-vue'

export default {
  plugins: [
    vue({
      template: {
        compilerOptions: {
          // treat all tags with a dash as custom elements
          isCustomElement: (tag) => tag.includes('-')
        }
      }
    })
  ]
}
```

#### Example Vue CLI Config {#example-vue-cli-config}

```js [vue.config.js]
module.exports = {
  chainWebpack: (config) => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .tap((options) => ({
        ...options,
        compilerOptions: {
          // treat any tag that starts with ion- as custom elements
          isCustomElement: (tag) => tag.startsWith('ion-')
        }
      }))
  }
}
```

### Passing DOM Properties {#passing-dom-properties}

Vì attribute DOM chỉ là chuỗi, ta cần truyền dữ liệu phức tạp cho custom element dưới dạng DOM property. Khi set prop trên custom element, Vue 3 tự kiểm tra sự tồn tại property DOM bằng toán tử `in` và ưu tiên set dưới dạng property nếu có. Thường thì bạn không cần bận tâm nếu custom element tuân theo [best practice khuyến nghị](https://web.dev/custom-elements-best-practices/).

Tuy nhiên, hiếm khi dữ liệu buộc phải truyền qua DOM property nhưng custom element không định nghĩa/phản chiếu property đúng (khiến kiểm tra `in` thất bại). Khi đó, bạn có thể ép `v-bind` set thành DOM property bằng modifier `.prop`:

```vue-html
<my-element :user.prop="{ name: 'jack' }"></my-element>

<!-- shorthand equivalent -->
<my-element .user="{ name: 'jack' }"></my-element>
```

## Building Custom Elements with Vue {#building-custom-elements-with-vue}

Lợi ích chính của custom element là có thể dùng với bất kỳ framework nào, thậm chí không cần framework. Điều này lý tưởng để phân phối component cho nơi tiêu thụ không dùng cùng stack frontend, hoặc khi bạn muốn cách ly ứng dụng cuối khỏi chi tiết hiện thực của component.

### defineCustomElement {#definecustomelement}

Vue hỗ trợ tạo custom element bằng đúng API component thông qua [`defineCustomElement`](/api/custom-elements#definecustomelement). Hàm nhận tham số giống [`defineComponent`](/api/general#definecomponent), nhưng trả về constructor custom element kế thừa `HTMLElement`:

```vue-html
<my-vue-element></my-vue-element>
```

```js
import { defineCustomElement } from 'vue'

const MyVueElement = defineCustomElement({
  // normal Vue component options here
  props: {},
  emits: {},
  template: `...`,

  // defineCustomElement only: CSS to be injected into shadow root
  styles: [`/* inlined css */`]
})

// Register the custom element.
// After registration, all `<my-vue-element>` tags
// on the page will be upgraded.
customElements.define('my-vue-element', MyVueElement)

// You can also programmatically instantiate the element:
// (can only be done after registration)
document.body.appendChild(
  new MyVueElement({
    // initial props (optional)
  })
)
```

#### Lifecycle {#lifecycle}

- Custom element của Vue sẽ mount một instance component bên trong shadow root khi [`connectedCallback`](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#using_the_lifecycle_callbacks) được gọi lần đầu.

- Khi `disconnectedCallback` được gọi, Vue sẽ kiểm tra xem phần tử có bị tách khỏi document sau một microtask tick hay không.

  - Nếu phần tử vẫn trong document, đó là thao tác di chuyển và instance sẽ được giữ lại;

  - Nếu phần tử bị tách khỏi document, đó là gỡ bỏ và instance sẽ bị unmount.

#### Props {#props}

- Tất cả prop khai báo trong `props` sẽ được định nghĩa trên custom element dưới dạng property. Vue tự xử lý phản chiếu giữa attribute/property khi phù hợp.

  - Attributes are always reflected to corresponding properties.

  - Properties with primitive values (`string`, `boolean` or `number`) are reflected as attributes.

- Vue cũng tự ép kiểu prop khai báo là `Boolean` hoặc `Number` khi chúng được set dưới dạng attribute (luôn là chuỗi). Ví dụ, với khai báo prop sau:

  ```js
  props: {
    selected: Boolean,
    index: Number
  }
  ```

  Và khi dùng custom element:

  ```vue-html
  <my-element selected index="1"></my-element>
  ```

  Trong component, `selected` sẽ được ép kiểu thành `true` (boolean) và `index` sẽ được ép thành `1` (number).

#### Events {#events}

Event emit qua `this.$emit` hoặc `emit` trong setup sẽ được phát dưới dạng [CustomEvent](https://developer.mozilla.org/en-US/docs/Web/Events/Creating_and_triggering_events#adding_custom_data_%E2%80%93_customevent) native trên custom element. Tham số bổ sung (payload) được đưa vào mảng `detail` của CustomEvent.

#### Slots {#slots}

Trong component, slot render bằng phần tử `<slot/>` như thường. Tuy nhiên, khi dùng phần tử kết quả, nó chỉ chấp nhận [cú pháp slot native](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots):

- [Scoped slots](/guide/components/slots#scoped-slots) are not supported.

- Khi truyền named slot, dùng attribute `slot` thay vì directive `v-slot`:

  ```vue-html
  <my-element>
    <div slot="named">hello</div>
  </my-element>
  ```

#### Provide / Inject {#provide-inject}

[Provide / Inject API](/guide/components/provide-inject#provide-inject) và phiên bản [Composition API](/api/composition-api-dependency-injection#provide) cũng hoạt động giữa các custom element định nghĩa bằng Vue. Lưu ý chỉ hoạt động **giữa custom element** — custom element không thể inject từ component Vue không phải custom element.

#### App Level Config <sup class="vt-badge" data-text="3.5+" /> {#app-level-config}

Bạn có thể cấu hình app instance của custom element Vue thông qua tuỳ chọn `configureApp`:

```js
defineCustomElement(MyComponent, {
  configureApp(app) {
    app.config.errorHandler = (err) => {
      /* ... */
    }
  }
})
```

### SFC as Custom Element {#sfc-as-custom-element}

`defineCustomElement` cũng hoạt động với SFC. Tuy nhiên, với thiết lập tooling mặc định, `<style>` trong SFC sẽ bị tách và gộp vào một file CSS khi build production. Khi dùng SFC làm custom element, thường mong muốn inject thẻ `<style>` vào shadow root của custom element thay vì tách ra.

Tooling SFC chính thức hỗ trợ import SFC ở “custom element mode” (cần `@vitejs/plugin-vue@^1.4.0` hoặc `vue-loader@^16.5.0`). SFC ở chế độ này sẽ inline các thẻ `<style>` thành chuỗi CSS và lộ ra dưới tùy chọn `styles` của component. `defineCustomElement` sẽ lấy và inject vào shadow root khi khởi tạo.

Để bật chế độ này, chỉ cần đặt tên file component kết thúc bằng `.ce.vue`:

```js
import { defineCustomElement } from 'vue'
import Example from './Example.ce.vue'

console.log(Example.styles) // ["/* inlined css */"]

// convert into custom element constructor
const ExampleElement = defineCustomElement(Example)

// register
customElements.define('my-example', ExampleElement)
```

Nếu muốn tuỳ biến file nào sẽ được import ở chế độ custom element (ví dụ, coi _mọi_ SFC là custom element), bạn có thể truyền tuỳ chọn `customElement` cho plugin build tương ứng:

- [@vitejs/plugin-vue](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue#using-vue-sfcs-as-custom-elements)
- [vue-loader](https://github.com/vuejs/vue-loader/tree/next#v16-only-options)

### Tips for a Vue Custom Elements Library {#tips-for-a-vue-custom-elements-library}

Khi xây custom element với Vue, phần tử phụ thuộc vào runtime của Vue. Có chi phí baseline ~16kb tùy số tính năng dùng. Điều này nghĩa là không lý tưởng nếu bạn chỉ phát hành một custom element — có thể cân nhắc JavaScript thuần, [petite-vue](https://github.com/vuejs/petite-vue), hoặc framework tối ưu runtime nhỏ. Tuy nhiên, kích thước cơ sở hoàn toàn hợp lý nếu bạn phát hành bộ custom element có logic phức tạp, vì Vue giúp viết ít code hơn. Bạn phát hành càng nhiều phần tử cùng nhau, đánh đổi càng có lợi.

Nếu custom element dùng trong ứng dụng cũng dùng Vue, bạn có thể externalize Vue khỏi bundle để dùng chung một bản Vue từ ứng dụng chính.

Nên export từng constructor phần tử để người dùng linh hoạt import theo nhu cầu và đăng ký với tên thẻ mong muốn. Bạn cũng có thể export hàm tiện ích để đăng ký tất cả. Ví dụ entry point của thư viện custom element Vue:

```js [elements.js]

import { defineCustomElement } from 'vue'
import Foo from './MyFoo.ce.vue'
import Bar from './MyBar.ce.vue'

const MyFoo = defineCustomElement(Foo)
const MyBar = defineCustomElement(Bar)

// export individual elements
export { MyFoo, MyBar }

export function register() {
  customElements.define('my-foo', MyFoo)
  customElements.define('my-bar', MyBar)
}
```

Người dùng có thể dùng các phần tử này trong file Vue:

```vue
<script setup>
import { register } from 'path/to/elements.js'
register()
</script>

<template>
  <my-foo ...>
    <my-bar ...></my-bar>
  </my-foo>
</template>
```

Hoặc trong framework khác như môi trường JSX, và với tên tuỳ chọn:

```jsx
import { MyFoo, MyBar } from 'path/to/elements.js'

customElements.define('some-foo', MyFoo)
customElements.define('some-bar', MyBar)

export function MyComponent() {
  return <>
    <some-foo ... >
      <some-bar ... ></some-bar>
    </some-foo>
  </>
}
```

### Vue-based Web Components and TypeScript {#web-components-and-typescript}

Khi viết template SFC, bạn có thể muốn [kiểm tra kiểu](/guide/scaling-up/tooling.html#typescript) cho component, kể cả những cái định nghĩa thành custom element.

Custom element được đăng ký toàn cục trong trình duyệt bằng API sẵn có, và mặc định không có suy luận kiểu khi dùng trong template Vue. Để hỗ trợ kiểu, ta có thể đăng ký kiểu component toàn cục bằng cách mở rộng [`GlobalComponents`](https://github.com/vuejs/language-tools/wiki/Global-Component-Types) để kiểm tra kiểu trong template (người dùng JSX có thể mở rộng [JSX.IntrinsicElements](https://www.typescriptlang.org/docs/handbook/jsx.html#intrinsic-elements)).

Đây là cách định nghĩa kiểu cho một custom element được tạo bằng Vue:

```typescript
import { defineCustomElement } from 'vue'

// Import the Vue component.
import SomeComponent from './src/components/SomeComponent.ce.vue'

// Turn the Vue component into a Custom Element class.
export const SomeElement = defineCustomElement(SomeComponent)

// Remember to register the element class with the browser.
customElements.define('some-element', SomeElement)

// Add the new element type to Vue's GlobalComponents type.
declare module 'vue' {
  interface GlobalComponents {
    // Be sure to pass in the Vue component type here 
    // (SomeComponent, *not* SomeElement).
    // Custom Elements require a hyphen in their name, 
    // so use the hyphenated element name here.
    'some-element': typeof SomeComponent
  }
}
```

## Non-Vue Web Components and TypeScript {#non-vue-web-components-and-typescript}

Đây là cách khuyến nghị để bật kiểm tra kiểu trong SFC cho Custom Element không xây bằng Vue.

:::tip Note
Cách này là một khả năng, nhưng có thể khác tùy framework tạo custom element.
:::

Suppose we have a custom element with some JS properties and events defined, and it is shipped in a library called `some-lib`:

```ts [some-lib/src/SomeElement.ts]
// Define a class with typed JS properties.
export class SomeElement extends HTMLElement {
  foo: number = 123
  bar: string = 'blah'

  lorem: boolean = false

  // This method should not be exposed to template types.
  someMethod() {
    /* ... */
  }

  // ... implementation details omitted ...
  // ... assume the element dispatches events named "apple-fell" ...
}

customElements.define('some-element', SomeElement)

// This is a list of properties of SomeElement that will be selected for type
// checking in framework templates (f.e. Vue SFC templates). Any other
// properties will not be exposed.
export type SomeElementAttributes = 'foo' | 'bar'

// Define the event types that SomeElement dispatches.
export type SomeElementEvents = {
  'apple-fell': AppleFellEvent
}

export class AppleFellEvent extends Event {
  /* ... details omitted ... */
}
```

The implementation details have been omitted, but the important part is that we have type definitions for two things: prop types and event types.

Let's create a type helper for easily registering custom element type definitions in Vue:

```ts [some-lib/src/DefineCustomElement.ts]
// We can re-use this type helper per each element we need to define.
type DefineCustomElement<
  ElementType extends HTMLElement,
  Events extends EventMap = {},
  SelectedAttributes extends keyof ElementType = keyof ElementType
> = new () => ElementType & {
  // Use $props to define the properties exposed to template type checking. Vue
  // specifically reads prop definitions from the `$props` type. Note that we
  // combine the element's props with the global HTML props and Vue's special
  // props.
  /** @deprecated Do not use the $props property on a Custom Element ref, 
    this is for template prop types only. */
  $props: HTMLAttributes &
    Partial<Pick<ElementType, SelectedAttributes>> &
    PublicProps

  // Use $emit to specifically define event types. Vue specifically reads event
  // types from the `$emit` type. Note that `$emit` expects a particular format
  // that we map `Events` to.
  /** @deprecated Do not use the $emit property on a Custom Element ref, 
    this is for template prop types only. */
  $emit: VueEmit<Events>
}

type EventMap = {
  [event: string]: Event
}

// This maps an EventMap to the format that Vue's $emit type expects.
type VueEmit<T extends EventMap> = EmitFn<{
  [K in keyof T]: (event: T[K]) => void
}>
```

:::tip Lưu ý
Chúng tôi đánh dấu `$props` và `$emit` là deprecated để khi bạn lấy `ref` tới custom element sẽ không dùng nhầm các thuộc tính này — chúng chỉ phục vụ mục đích kiểm tra kiểu trong template. Các thuộc tính này không tồn tại trên instance custom element.
:::

Dùng type helper này, giờ ta có thể chọn những thuộc tính JS sẽ được expose để kiểm tra kiểu trong template Vue:

```ts [some-lib/src/SomeElement.vue.ts]
import {
  SomeElement,
  SomeElementAttributes,
  SomeElementEvents
} from './SomeElement.js'
import type { Component } from 'vue'
import type { DefineCustomElement } from './DefineCustomElement'

// Add the new element type to Vue's GlobalComponents type.
declare module 'vue' {
  interface GlobalComponents {
    'some-element': DefineCustomElement<
      SomeElement,
      SomeElementAttributes,
      SomeElementEvents
    >
  }
}
```

Giả sử `some-lib` build mã nguồn TypeScript vào thư mục `dist/`. Người dùng `some-lib` có thể import `SomeElement` và dùng trong SFC như sau:

```vue [SomeElementImpl.vue]
<script setup lang="ts">
// This will create and register the element with the browser.
import 'some-lib/dist/SomeElement.js'

// A user that is using TypeScript and Vue should additionally import the
// Vue-specific type definition (users of other frameworks may import other
// framework-specific type definitions).
import type {} from 'some-lib/dist/SomeElement.vue.js'

import { useTemplateRef, onMounted } from 'vue'

const el = useTemplateRef('el')

onMounted(() => {
  console.log(
    el.value!.foo,
    el.value!.bar,
    el.value!.lorem,
    el.value!.someMethod()
  )

  // Do not use these props, they are `undefined`
  // IDE will show them crossed out
  el.$props
  el.$emit
})
</script>

<template>
  <!-- Now we can use the element, with type checking: -->
  <some-element
    ref="el"
    :foo="456"
    :blah="'hello'"
    @apple-fell="
      (event) => {
        // The type of `event` is inferred here to be `AppleFellEvent`
      }
    "
  ></some-element>
</template>
```

Nếu một phần tử không có định nghĩa kiểu sẵn, bạn có thể định nghĩa thủ công kiểu props và events như sau:

```vue [SomeElementImpl.vue]
<script setup lang="ts">
// Suppose that `some-lib` is plain JS without type definitions, and TypeScript
// cannot infer the types:
import { SomeElement } from 'some-lib'

// We'll use the same type helper as before.
import { DefineCustomElement } from './DefineCustomElement'

type SomeElementProps = { foo?: number; bar?: string }
type SomeElementEvents = { 'apple-fell': AppleFellEvent }
interface AppleFellEvent extends Event {
  /* ... */
}

// Add the new element type to Vue's GlobalComponents type.
declare module 'vue' {
  interface GlobalComponents {
    'some-element': DefineCustomElement<
      SomeElementProps,
      SomeElementEvents
    >
  }
}

// ... same as before, use a reference to the element ...
</script>

<template>
  <!-- ... same as before, use the element in the template ... -->
</template>
```

Tác giả Custom Element không nên tự động export định nghĩa kiểu đặc thù framework từ thư viện của mình (ví dụ không export từ `index.ts` nơi export các phần còn lại), nếu không người dùng có thể gặp lỗi module augmentation ngoài ý muốn. Người dùng nên import file định nghĩa kiểu dành riêng cho framework mà họ cần.

## Web Components vs. Vue Components {#web-components-vs-vue-components}

Một số lập trình viên tin rằng nên tránh mô hình component đặc thù framework và chỉ dùng Custom Element để “bền vững tương lai”. Ở đây chúng tôi giải thích vì sao đó là cách nhìn quá đơn giản.

Thực sự có phần giao nhau giữa Custom Element và Vue Component: cả hai cho phép định nghĩa component tái sử dụng với truyền dữ liệu, emit event, quản lý vòng đời. Tuy nhiên, API Web Components khá low‑level và tối giản. Để xây ứng dụng thực tế, ta cần nhiều khả năng bổ sung mà nền tảng chưa bao phủ:

- A declarative and efficient templating system;

- A reactive state management system that facilitates cross-component logic extraction and reuse;

- A performant way to render the components on the server and hydrate them on the client (SSR), which is important for SEO and [Web Vitals metrics such as LCP](https://web.dev/vitals/). Native custom elements SSR typically involves simulating the DOM in Node.js and then serializing the mutated DOM, while Vue SSR compiles into string concatenation whenever possible, which is much more efficient.

Mô hình component của Vue được thiết kế với các nhu cầu này như một hệ thống nhất quán.

Với đội ngũ vững, bạn có thể xây tương đương dựa trên Custom Element thuần — nhưng điều đó đồng nghĩa gánh nặng bảo trì lâu dài của “framework nội bộ” và bỏ lỡ hệ sinh thái/ cộng đồng của framework trưởng thành như Vue.

Cũng có framework dùng Custom Element làm nền tảng component model, nhưng rồi vẫn phải đưa ra giải pháp đặc thù cho các vấn đề kể trên. Dùng các framework này đồng nghĩa chấp nhận quyết định kỹ thuật của họ — điều này, dù quảng bá thế nào, không tự động miễn nhiễm với biến động tương lai.

Có những điểm chúng tôi thấy custom element còn hạn chế:

- Eager slot evaluation hinders component composition. Vue's [scoped slots](/guide/components/slots#scoped-slots) are a powerful mechanism for component composition, which can't be supported by custom elements due to native slots' eager nature. Eager slots also mean the receiving component cannot control when or whether to render a piece of slot content.

- Shipping custom elements with shadow DOM scoped CSS today requires embedding the CSS inside JavaScript so that they can be injected into shadow roots at runtime. They also result in duplicated styles in markup in SSR scenarios. There are [platform features](https://github.com/whatwg/html/pull/4898/) being worked on in this area - but as of now they are not yet universally supported, and there are still production performance / SSR concerns to be addressed. In the meanwhile, Vue SFCs provide [CSS scoping mechanisms](/api/sfc-css-features) that support extracting the styles into plain CSS files.

Vue sẽ luôn cập nhật theo tiêu chuẩn mới nhất của nền tảng web và tận dụng những gì nền tảng cung cấp nếu giúp công việc dễ hơn. Tuy nhiên, mục tiêu của chúng tôi là cung cấp giải pháp hoạt động tốt và hoạt động ngay hôm nay. Điều đó có nghĩa phải tiếp nhận tính năng mới của nền tảng với tư duy phản biện — và lấp khoảng trống nơi tiêu chuẩn còn thiếu trong khi nó vẫn còn thiếu.
