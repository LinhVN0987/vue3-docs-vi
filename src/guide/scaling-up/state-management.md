# State Management {#state-management}

## What is State Management? {#what-is-state-management}

Về mặt kỹ thuật, mỗi component instance của Vue đã "quản lý" state reactive của riêng nó. Lấy ví dụ component counter đơn giản:

<div class="composition-api">

```vue
<script setup>
import { ref } from 'vue'

// state
const count = ref(0)

// actions
function increment() {
  count.value++
}
</script>

<!-- view -->
<template>{{ count }}</template>
```

</div>
<div class="options-api">

```vue
<script>
export default {
  // state
  data() {
    return {
      count: 0
    }
  },
  // actions
  methods: {
    increment() {
      this.count++
    }
  }
}
</script>

<!-- view -->
<template>{{ count }}</template>
```

</div>

Đây là một đơn vị tự chứa với các phần:

- **state**: nguồn sự thật điều khiển ứng dụng;
- **view**: ánh xạ khai báo từ **state**;
- **actions**: các cách state có thể thay đổi để phản ứng với input người dùng từ **view**.

Đây là biểu diễn đơn giản của khái niệm "one‑way data flow":

<p style="text-align: center">
  <img alt="state flow diagram" src="./images/state-flow.png" width="252px" style="margin: 40px auto">
</p>

Tuy nhiên sự đơn giản bắt đầu sụp đổ khi có **nhiều component chia sẻ chung một state**:

1. Nhiều view có thể phụ thuộc cùng một phần state.
2. Actions từ các view khác nhau có thể cần mutate cùng một phần state.

Với trường hợp thứ nhất, có thể "nâng" state dùng chung lên ancestor chung và truyền xuống qua props. Tuy nhiên, điều này nhanh chóng trở nên mệt mỏi trong cây component sâu, dẫn đến vấn đề [Prop Drilling](/guide/components/provide-inject#prop-drilling).

Với trường hợp thứ hai, ta thường dùng các giải pháp như thao tác trực tiếp parent/child qua template ref, hoặc cố mutate và đồng bộ nhiều bản sao state qua event emit. Cả hai đều mong manh và nhanh chóng dẫn đến code khó bảo trì.

Giải pháp đơn giản hơn là tách state dùng chung ra khỏi component và quản lý nó trong một singleton toàn cục. Khi đó cây component trở thành một "view" lớn, và bất kỳ component nào cũng có thể truy cập state hoặc kích hoạt action, bất kể ở đâu trong cây!

## Simple State Management with Reactivity API {#simple-state-management-with-reactivity-api}

<div class="options-api">

Trong Options API, dữ liệu reactive được khai báo bằng `data()`. Nội bộ, object trả về từ `data()` được làm reactive qua hàm [`reactive()`](/api/reactivity-core#reactive), cũng là public API.

</div>

If you have a piece of state that should be shared by multiple instances, you can use [`reactive()`](/api/reactivity-core#reactive) to create a reactive object, and then import it into multiple components:

```js [store.js]
import { reactive } from 'vue'

export const store = reactive({
  count: 0
})
```

<div class="composition-api">

```vue [ComponentA.vue]
<script setup>
import { store } from './store.js'
</script>

<template>From A: {{ store.count }}</template>
```

```vue [ComponentB.vue]
<script setup>
import { store } from './store.js'
</script>

<template>From B: {{ store.count }}</template>
```

</div>
<div class="options-api">

```vue [ComponentA.vue]
<script>
import { store } from './store.js'

export default {
  data() {
    return {
      store
    }
  }
}
</script>

<template>From A: {{ store.count }}</template>
```

```vue [ComponentB.vue]
<script>
import { store } from './store.js'

export default {
  data() {
    return {
      store
    }
  }
}
</script>

<template>From B: {{ store.count }}</template>
```

</div>

Giờ bất cứ khi nào object `store` bị mutate, cả `<ComponentA>` và `<ComponentB>` sẽ tự động cập nhật view — ta đã có một nguồn sự thật duy nhất.

Tuy nhiên, điều này cũng có nghĩa bất kỳ component nào import `store` đều có thể mutate nó tùy ý:

```vue-html{2}
<template>
  <button @click="store.count++">
    From B: {{ store.count }}
  </button>
</template>
```

Mặc dù hoạt động trong trường hợp đơn giản, state toàn cục có thể bị mutate tùy ý sẽ khó bảo trì về lâu dài. Để đảm bảo logic mutate state được tập trung giống như state, nên định nghĩa các method trên store với tên thể hiện rõ mục đích của action:

```js{5-7} [store.js]
import { reactive } from 'vue'

export const store = reactive({
  count: 0,
  increment() {
    this.count++
  }
})
```

```vue-html{2}
<template>
  <button @click="store.increment()">
    From B: {{ store.count }}
  </button>
</template>
```

<div class="composition-api">

[Try it in the Playground](https://play.vuejs.org/#eNrNkk1uwyAQha8yYpNEiUzXllPVrtRTeJNSqtLGgGBsVbK4ewdwnT9FWWSTFczwmPc+xMhqa4uhl6xklRdOWQQvsbfPrVadNQ7h1dCqpcYaPp3pYFHwQyteXVxKm0tpM0krnm3IgAqUnd3vUFIFUB1Z8bNOkzoVny+wDTuNcZ1gBI/GSQhzqlQX3/5Gng81pA1t33tEo+FF7JX42bYsT1BaONlRguWqZZMU4C261CWMk3EhTK8RQphm8Twse/BscoUsvdqDkTX3kP3nI6aZwcmdQDUcMPJPabX8TQphtCf0RLqd1csxuqQAJTxtYnEUGtIpAH4pn1Ou17FDScOKhT+QNAVM)

</div>
<div class="options-api">

[Try it in the Playground](https://play.vuejs.org/#eNrdU8FqhDAU/JVHLruyi+lZ3FIt9Cu82JilaTWR5CkF8d8bE5O1u1so9FYQzAyTvJnRTKTo+3QcOMlIbpgWPT5WUnS90gjPyr4ll1jAWasOdim9UMum3a20vJWWqxSgkvzTyRt+rocWYVpYFoQm8wRsJh+viHLBcyXtk9No2ALkXd/WyC0CyDfW6RVTOiancQM5ku+x7nUxgUGlOcwxn8Ppu7HJ7udqaqz3SYikOQ5aBgT+OA9slt9kasToFnb5OiAqCU+sFezjVBHvRUimeWdT7JOKrFKAl8VvYatdI6RMDRJhdlPtWdQf5mdQP+SHdtyX/IftlH9pJyS1vcQ2NK8ZivFSiL8BsQmmpMG1s1NU79frYA1k8OD+/I3pUA6+CeNdHg6hmoTMX9pPSnk=)

</div>

:::tip
Lưu ý click handler dùng `store.increment()` có dấu ngoặc — cần thiết để gọi method với ngữ cảnh `this` đúng vì đây không phải method của component.
:::

Mặc dù ở đây dùng một reactive object làm store, bạn cũng có thể chia sẻ state reactive tạo bởi các [Reactivity API](/api/reactivity-core) khác như `ref()` hoặc `computed()`, hoặc thậm chí trả về state toàn cục từ một [Composable](/guide/reusability/composables):

```js
import { ref } from 'vue'

// global state, created in module scope
const globalCount = ref(1)

export function useCount() {
  // local state, created per-component
  const localCount = ref(1)

  return {
    globalCount,
    localCount
  }
}
```

Việc hệ thống reactivity của Vue tách rời khỏi mô hình component khiến nó cực kỳ linh hoạt.

## SSR Considerations {#ssr-considerations}

Nếu bạn xây dựng ứng dụng dùng [Server‑Side Rendering (SSR)](./ssr), mẫu trên có thể gây vấn đề do store là singleton dùng chung giữa nhiều request. Điều này được bàn [chi tiết hơn](./ssr#cross-request-state-pollution) trong hướng dẫn SSR.

## Pinia {#pinia}

Mặc dù giải pháp tự viết đủ dùng trong bối cảnh đơn giản, còn nhiều điều cần cân nhắc ở ứng dụng production quy mô lớn:

- Stronger conventions for team collaboration
- Integrating with the Vue DevTools, including timeline, in-component inspection, and time-travel debugging
- Hot Module Replacement
- Server-Side Rendering support

[Pinia](https://pinia.vuejs.org) là thư viện quản lý state hiện thực tất cả số trên. Nó được team core Vue duy trì, và hoạt động với cả Vue 2 lẫn Vue 3.

Người dùng hiện tại có thể quen với [Vuex](https://vuex.vuejs.org/), thư viện quản lý state chính thức trước đây. Với Pinia đảm nhận vai trò tương tự trong hệ sinh thái, Vuex nay ở chế độ bảo trì. Nó vẫn hoạt động nhưng không nhận tính năng mới. Khuyến nghị dùng Pinia cho ứng dụng mới.

Pinia bắt đầu như một thử nghiệm cho thế hệ kế tiếp của Vuex, tích hợp nhiều ý tưởng từ các thảo luận về Vuex 5. Cuối cùng, chúng tôi nhận ra Pinia đã hiện thực phần lớn những gì mong muốn ở Vuex 5, và quyết định đưa nó thành khuyến nghị mới.

So với Vuex, Pinia có API đơn giản hơn, “ít nghi thức” hơn, cung cấp API theo phong cách Composition API, và quan trọng nhất, hỗ trợ suy luận kiểu vững chắc khi dùng với TypeScript.
