---
outline: deep
---

# Suspense {#suspense}

:::warning Experimental Feature
`<Suspense>` là một tính năng thử nghiệm. Không đảm bảo đạt trạng thái ổn định và API có thể thay đổi trước khi ổn định.
:::

`<Suspense>` là built‑in component để điều phối các phụ thuộc bất đồng bộ trong cây component. Nó có thể render trạng thái loading trong khi chờ nhiều phụ thuộc async lồng nhau được resolve.

## Async Dependencies {#async-dependencies}

Để giải thích vấn đề mà `<Suspense>` giải quyết và cách nó tương tác với các phụ thuộc async, hãy tưởng tượng cấu trúc component như sau:

```
<Suspense>
└─ <Dashboard>
   ├─ <Profile>
   │  └─ <FriendStatus> (component with async setup())
   └─ <Content>
      ├─ <ActivityFeed> (async component)
      └─ <Stats> (async component)
```

Trong cây component có nhiều component lồng nhau mà việc render phụ thuộc vào việc resolve tài nguyên async trước. Không có `<Suspense>`, mỗi component phải tự xử lý trạng thái loading/error/loaded. Tệ hơn, có thể thấy nhiều spinner trên trang và nội dung xuất hiện lệch thời điểm.

`<Suspense>` cho phép hiển thị trạng thái loading/error ở cấp cao, trong khi chờ các phụ thuộc async lồng nhau được resolve.

Có hai loại phụ thuộc async mà `<Suspense>` có thể chờ:

1. Components with an async `setup()` hook. This includes components using `<script setup>` with top-level `await` expressions.

2. [Async Components](/guide/components/async).

### `async setup()` {#async-setup}

Hook `setup()` của component theo Composition API có thể là async:

```js
export default {
  async setup() {
    const res = await fetch(...)
    const posts = await res.json()
    return {
      posts
    }
  }
}
```

Khi dùng `<script setup>`, việc có `await` ở top‑level tự động khiến component trở thành phụ thuộc async:

```vue
<script setup>
const res = await fetch(...)
const posts = await res.json()
</script>

<template>
  {{ posts }}
</template>
```

### Async Components {#async-components}

Async component mặc định là **“suspensible”**. Nghĩa là nếu có `<Suspense>` trong chuỗi parent, nó sẽ được coi là phụ thuộc async của `<Suspense>` đó. Khi đó, trạng thái loading sẽ do `<Suspense>` điều khiển, và các tùy chọn loading/error/delay/timeout riêng của component sẽ bị bỏ qua.

Async component có thể không chịu sự điều khiển của `Suspense` và luôn tự kiểm soát trạng thái loading bằng cách đặt `suspensible: false` trong tùy chọn.

## Loading State {#loading-state}

`<Suspense>` có hai slot: `#default` và `#fallback`. Mỗi slot chỉ cho phép **một** node con trực tiếp. Node trong default slot được hiển thị nếu có thể; nếu không, node trong fallback slot sẽ hiển thị.

```vue-html
<Suspense>
  <!-- component with nested async dependencies -->
  <Dashboard />

  <!-- loading state via #fallback slot -->
  <template #fallback>
    Loading...
  </template>
</Suspense>
```

Ở lần render đầu tiên, `<Suspense>` render nội dung default slot trong bộ nhớ. Nếu gặp phụ thuộc async trong quá trình này, nó chuyển sang trạng thái **pending**. Trong lúc pending, nội dung fallback sẽ hiển thị. Khi tất cả phụ thuộc gặp phải đã resolve, `<Suspense>` vào trạng thái **resolved** và hiển thị nội dung default đã resolve.

Nếu không gặp phụ thuộc async nào, `<Suspense>` chuyển thẳng sang trạng thái resolved.

Khi đã ở trạng thái resolved, `<Suspense>` chỉ quay lại pending nếu root node của `#default` slot bị thay thế. Các phụ thuộc async mới lồng sâu hơn **không** khiến `<Suspense>` quay lại pending.

Khi xảy ra revert, nội dung fallback sẽ không hiển thị ngay. Thay vào đó, `<Suspense>` hiển thị `#default` trước đó trong khi chờ nội dung mới và các phụ thuộc của nó được resolve. Hành vi này có thể cấu hình bằng prop `timeout`: `<Suspense>` sẽ chuyển sang fallback nếu mất lâu hơn `timeout` mili‑giây để render default mới. `timeout: 0` sẽ hiển thị fallback ngay khi default bị thay thế.

## Events {#events}

`<Suspense>` emit 3 event: `pending`, `resolve`, và `fallback`. `pending` xảy ra khi vào trạng thái pending. `resolve` được emit khi nội dung mới trong default slot resolve xong. `fallback` được bắn khi nội dung trong fallback slot hiển thị.

Các event này có thể dùng, ví dụ, để hiển thị chỉ báo loading đè lên DOM cũ trong khi component mới đang được tải.

## Error Handling {#error-handling}

Hiện tại `<Suspense>` không cung cấp xử lý lỗi qua chính component — tuy nhiên, bạn có thể dùng tùy chọn [`errorCaptured`](/api/options-lifecycle#errorcaptured) hoặc hook [`onErrorCaptured()`](/api/composition-api-lifecycle#onerrorcaptured) để bắt và xử lý lỗi async ở parent của `<Suspense>`.

## Combining with Other Components {#combining-with-other-components}

Thông thường ta muốn dùng `<Suspense>` kết hợp với [`<Transition>`](./transition) và [`<KeepAlive>`](./keep-alive). Thứ tự lồng nhau của các component này rất quan trọng để chúng hoạt động đúng.

Ngoài ra, các component này thường dùng cùng `<RouterView>` của [Vue Router](https://router.vuejs.org/).

Ví dụ sau cho thấy cách lồng các component để tất cả hoạt động như mong đợi. Với tổ hợp đơn giản hơn bạn có thể bỏ bớt các component không cần:

```vue-html
<RouterView v-slot="{ Component }">
  <template v-if="Component">
    <Transition mode="out-in">
      <KeepAlive>
        <Suspense>
          <!-- main content -->
          <component :is="Component"></component>

          <!-- loading state -->
          <template #fallback>
            Loading...
          </template>
        </Suspense>
      </KeepAlive>
    </Transition>
  </template>
</RouterView>
```

Vue Router hỗ trợ sẵn [lazy‑load component](https://router.vuejs.org/guide/advanced/lazy-loading.html) bằng dynamic import. Đây khác với async component và hiện không kích hoạt `<Suspense>`. Tuy nhiên, chúng có thể chứa async component con và các component đó sẽ kích hoạt `<Suspense>` như thường.

## Nested Suspense {#nested-suspense}

- Only supported in 3.3+

Khi có nhiều async component (phổ biến với route lồng nhau hoặc dựa trên layout) như sau:

```vue-html
<Suspense>
  <component :is="DynamicAsyncOuter">
    <component :is="DynamicAsyncInner" />
  </component>
</Suspense>
```

`<Suspense>` tạo một boundary sẽ resolve tất cả async component bên dưới, như mong đợi. Tuy nhiên, khi đổi `DynamicAsyncOuter`, `<Suspense>` chờ đúng cách, nhưng khi đổi `DynamicAsyncInner`, phần `DynamicAsyncInner` lồng bên trong render node rỗng cho đến khi resolve (thay vì node trước đó hoặc fallback slot).

Để giải quyết, ta có thể lồng một suspense để xử lý patch cho component lồng bên trong, như sau:

```vue-html
<Suspense>
  <component :is="DynamicAsyncOuter">
    <Suspense suspensible> <!-- this -->
      <component :is="DynamicAsyncInner" />
    </Suspense>
  </component>
</Suspense>
```

Nếu không đặt prop `suspensible`, `<Suspense>` con sẽ bị parent coi như component đồng bộ. Nghĩa là nó có fallback slot riêng và nếu cả hai `Dynamic` thay đổi cùng lúc, có thể xuất hiện node rỗng và nhiều vòng patch trong khi `<Suspense>` con tải cây phụ thuộc của chính nó — điều có thể không mong muốn. Khi đặt `suspensible`, toàn bộ xử lý phụ thuộc async được giao cho `<Suspense>` cha (bao gồm cả event), và `<Suspense>` con chỉ đóng vai trò boundary cho việc resolve phụ thuộc và patching.

---

**Related**

- [`<Suspense>` API reference](/api/built-in-components#suspense)
