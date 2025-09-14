# Async Components {#async-components}

## Basic Usage {#basic-usage}

Trong các ứng dụng lớn, ta có thể cần chia ứng dụng thành các phần nhỏ hơn và chỉ tải một component từ server khi cần. Để làm điều đó, Vue có hàm [`defineAsyncComponent`](/api/general#defineasynccomponent):

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() => {
  return new Promise((resolve, reject) => {
    // ...load component from server
    resolve(/* loaded component */)
  })
})
// ... use `AsyncComp` like a normal component
```

Như bạn thấy, `defineAsyncComponent` nhận một loader function trả về Promise. Callback `resolve` của Promise nên được gọi khi bạn đã lấy được định nghĩa component từ server. Bạn cũng có thể gọi `reject(reason)` để chỉ ra việc tải thất bại.

[ES module dynamic import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) cũng trả về Promise, vì vậy hầu hết thời gian ta sẽ dùng nó kết hợp với `defineAsyncComponent`. Các bundler như Vite và webpack cũng hỗ trợ cú pháp này (và dùng nó làm điểm tách bundle), nên ta có thể dùng để import Vue SFC:

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
)
```

`AsyncComp` thu được là một wrapper component chỉ gọi loader function khi nó thực sự được render trên trang. Ngoài ra, nó sẽ chuyển tiếp mọi props và slot tới component bên trong, vì vậy bạn có thể dùng async wrapper để thay thế component gốc một cách mượt mà trong khi vẫn đạt được lazy loading.

Tương tự component thông thường, async component có thể được [đăng ký toàn cục](/guide/components/registration#global-registration) bằng `app.component()`:

```js
app.component('MyComponent', defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
))
```

<div class="options-api">

Bạn cũng có thể dùng `defineAsyncComponent` khi [đăng ký component cục bộ](/guide/components/registration#local-registration):

```vue
<script>
import { defineAsyncComponent } from 'vue'

export default {
  components: {
    AdminPage: defineAsyncComponent(() =>
      import('./components/AdminPageComponent.vue')
    )
  }
}
</script>

<template>
  <AdminPage />
</template>
```

</div>

<div class="composition-api">

Chúng cũng có thể được định nghĩa trực tiếp bên trong parent component:

```vue
<script setup>
import { defineAsyncComponent } from 'vue'

const AdminPage = defineAsyncComponent(() =>
  import('./components/AdminPageComponent.vue')
)
</script>

<template>
  <AdminPage />
</template>
```

</div>

## Loading and Error States {#loading-and-error-states}

Các thao tác bất đồng bộ tất yếu liên quan đến trạng thái loading và error — `defineAsyncComponent()` hỗ trợ xử lý các trạng thái này qua các tùy chọn nâng cao:

```js
const AsyncComp = defineAsyncComponent({
  // the loader function
  loader: () => import('./Foo.vue'),

  // A component to use while the async component is loading
  loadingComponent: LoadingComponent,
  // Delay before showing the loading component. Default: 200ms.
  delay: 200,

  // A component to use if the load fails
  errorComponent: ErrorComponent,
  // The error component will be displayed if a timeout is
  // provided and exceeded. Default: Infinity.
  timeout: 3000
})
```

Nếu cung cấp loading component, nó sẽ hiển thị trước trong khi component bên trong đang được tải. Có độ trễ mặc định 200ms trước khi hiển thị loading component — vì trên mạng nhanh, trạng thái loading tức thời có thể bị thay thế quá nhanh và trông như bị nháy.

Nếu cung cấp error component, nó sẽ hiển thị khi Promise do loader function trả về bị reject. Bạn cũng có thể chỉ định timeout để hiển thị error component khi yêu cầu mất quá nhiều thời gian.

## Lazy Hydration <sup class="vt-badge" data-text="3.5+" /> {#lazy-hydration}

> Phần này chỉ áp dụng nếu bạn đang sử dụng [Server-Side Rendering](/guide/scaling-up/ssr).

Trong Vue 3.5+, async component có thể kiểm soát thời điểm hydrate bằng cách cung cấp hydration strategy.

- Vue cung cấp một số hydration strategy built‑in. Các strategy này cần được import riêng lẻ để có thể tree‑shake nếu không dùng.

- Thiết kế cố tình ở mức low‑level để linh hoạt. Sau này có thể có syntax sugar của compiler xây dựng trên cơ chế này, ở core hoặc giải pháp cấp cao hơn (ví dụ Nuxt).

### Hydrate on Idle {#hydrate-on-idle}

Hydrate thông qua `requestIdleCallback`:

```js
import { defineAsyncComponent, hydrateOnIdle } from 'vue'

const AsyncComp = defineAsyncComponent({
  loader: () => import('./Comp.vue'),
  hydrate: hydrateOnIdle(/* optionally pass a max timeout */)
})
```

### Hydrate on Visible {#hydrate-on-visible}

Hydrate khi phần tử trở nên nhìn thấy thông qua `IntersectionObserver`.

```js
import { defineAsyncComponent, hydrateOnVisible } from 'vue'

const AsyncComp = defineAsyncComponent({
  loader: () => import('./Comp.vue'),
  hydrate: hydrateOnVisible()
})
```

Có thể tùy chọn truyền object options cho observer:

```js
hydrateOnVisible({ rootMargin: '100px' })
```

### Hydrate on Media Query {#hydrate-on-media-query}

Hydrate khi media query chỉ định khớp.

```js
import { defineAsyncComponent, hydrateOnMediaQuery } from 'vue'

const AsyncComp = defineAsyncComponent({
  loader: () => import('./Comp.vue'),
  hydrate: hydrateOnMediaQuery('(max-width:500px)')
})
```

### Hydrate on Interaction {#hydrate-on-interaction}

Hydrate khi event được chỉ định được kích hoạt trên phần tử component. Event kích hoạt hydration cũng sẽ được phát lại sau khi hydration hoàn tất.

```js
import { defineAsyncComponent, hydrateOnInteraction } from 'vue'

const AsyncComp = defineAsyncComponent({
  loader: () => import('./Comp.vue'),
  hydrate: hydrateOnInteraction('click')
})
```

Cũng có thể là danh sách nhiều loại event:

```js
hydrateOnInteraction(['wheel', 'mouseover'])
```

### Custom Strategy {#custom-strategy}

```ts
import { defineAsyncComponent, type HydrationStrategy } from 'vue'

const myStrategy: HydrationStrategy = (hydrate, forEachElement) => {
  // forEachElement is a helper to iterate through all the root elements
  // in the component's non-hydrated DOM, since the root can be a fragment
  // instead of a single element
  forEachElement(el => {
    // ...
  })
  // call `hydrate` when ready
  hydrate()
  return () => {
    // return a teardown function if needed
  }
}

const AsyncComp = defineAsyncComponent({
  loader: () => import('./Comp.vue'),
  hydrate: myStrategy
})
```

## Using with Suspense {#using-with-suspense}

Async component có thể dùng với component built‑in `<Suspense>`. Cách tương tác giữa `<Suspense>` và async component được trình bày trong [chương riêng về `<Suspense>`](/guide/built-ins/suspense).
