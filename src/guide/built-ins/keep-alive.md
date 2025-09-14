<script setup>
import SwitchComponent from './keep-alive-demos/SwitchComponent.vue'
</script>

# KeepAlive {#keepalive}

`<KeepAlive>` là built‑in component cho phép cache có điều kiện các instance của component khi chuyển đổi động giữa nhiều component.

## Basic Usage {#basic-usage}

Trong chương Component Basics, chúng ta đã giới thiệu cú pháp cho [Dynamic Components](/guide/essentials/component-basics#dynamic-components) bằng phần tử đặc biệt `<component>`:

```vue-html
<component :is="activeComponent" />
```

Mặc định, một component instance đang hoạt động sẽ bị unmount khi chuyển sang component khác. Điều này làm mất mọi state đã thay đổi. Khi component được hiển thị lại, một instance mới sẽ được tạo với state ban đầu.

Trong ví dụ dưới, có hai component có state — A chứa counter, B chứa message đồng bộ với input qua `v-model`. Hãy thử cập nhật state của một cái, chuyển sang cái kia, rồi chuyển lại:

<SwitchComponent />

Bạn sẽ thấy khi chuyển lại, state đã thay đổi trước đó bị đặt lại.

Việc tạo instance mới khi chuyển đổi thường là hành vi hữu ích, nhưng ở đây, chúng ta muốn hai instance được giữ lại ngay cả khi không hoạt động. Để giải quyết, hãy bọc dynamic component bằng built‑in component `<KeepAlive>`:

```vue-html
<!-- Inactive components will be cached! -->
<KeepAlive>
  <component :is="activeComponent" />
</KeepAlive>
```

Giờ state sẽ được giữ nguyên khi chuyển đổi component:

<SwitchComponent use-KeepAlive />

<div class="composition-api">

[Try it in the Playground](https://play.vuejs.org/#eNqtUsFOwzAM/RWrl4IGC+cqq2h3RFw495K12YhIk6hJi1DVf8dJSllBaAJxi+2XZz8/j0lhzHboeZIl1NadMA4sd73JKyVaozsHI9hnJqV+feJHmODY6RZS/JEuiL1uTTEXtiREnnINKFeAcgZUqtbKOqj7ruPKwe6s2VVguq4UJXEynAkDx1sjmeMYAdBGDFBLZu2uShre6ioJeaxIduAyp0KZ3oF7MxwRHWsEQmC4bXXDJWbmxpjLBiZ7DwptMUFyKCiJNP/BWUbO8gvnA+emkGKIgkKqRrRWfh+Z8MIWwpySpfbxn6wJKMGV4IuSs0UlN1HVJae7bxYvBuk+2IOIq7sLnph8P9u5DJv5VfpWWLaGqTzwZTCOM/M0IaMvBMihd04ruK+lqF/8Ajxms8EFbCiJxR8khsP6ncQosLWnWV6a/kUf2nqu75Fby04chA0iPftaYryhz6NBRLjdtajpHZTWPio=)

</div>
<div class="options-api">

[Try it in the Playground](https://play.vuejs.org/#eNqtU8tugzAQ/JUVl7RKWveMXFTIseofcHHAiawasPxArRD/3rVNSEhbpVUrIWB3x7PM7jAkuVL3veNJmlBTaaFsVraiUZ22sO0alcNedw2s7kmIPHS1ABQLQDEBAMqWvwVQzffMSQuDz1aI6VreWpPCEBtsJppx4wE1s+zmNoIBNLdOt8cIjzut8XAKq3A0NAIY/QNveFEyi8DA8kZJZjlGALQWPVSSGfNYJjVvujIJeaxItuMyo6JVzoJ9VxwRmtUCIdDfNV3NJWam5j7HpPOY8BEYkwxySiLLP1AWkbK4oHzmXOVS9FFOSM3jhFR4WTNfRslcO54nSwJKcCD4RsnZmJJNFPXJEl8t88quOuc39fCrHalsGyWcnJL62apYNoq12UQ8DLEFjCMy+kKA7Jy1XQtPlRTVqx+Jx6zXOJI1JbH4jejg3T+KbswBzXnFlz9Tjes/V/3CjWEHDsL/OYNvdCE8Wu3kLUQEhy+ljh+brFFu)

</div>

:::tip
Khi dùng trong [in‑DOM template](/guide/essentials/component-basics#in-dom-template-parsing-caveats), nên tham chiếu là `<keep-alive>`.
:::

## Include / Exclude {#include-exclude}

Mặc định, `<KeepAlive>` sẽ cache mọi component instance bên trong. Ta có thể tùy biến hành vi này qua các prop `include` và `exclude`. Cả hai có thể là chuỗi phân tách bằng dấu phẩy, một `RegExp`, hoặc một mảng chứa các kiểu đó:

```vue-html
<!-- comma-delimited string -->
<KeepAlive include="a,b">
  <component :is="view" />
</KeepAlive>

<!-- regex (use `v-bind`) -->
<KeepAlive :include="/a|b/">
  <component :is="view" />
</KeepAlive>

<!-- Array (use `v-bind`) -->
<KeepAlive :include="['a', 'b']">
  <component :is="view" />
</KeepAlive>
```

Việc khớp được kiểm tra với tùy chọn [`name`](/api/options-misc#name) của component, nên các component cần cache có điều kiện bởi `KeepAlive` phải khai báo `name` một cách tường minh.

:::tip
Từ phiên bản 3.2.34, single‑file component dùng `<script setup>` sẽ tự suy ra tùy chọn `name` dựa trên tên file, không cần khai báo thủ công.
:::

## Max Cached Instances {#max-cached-instances}

Ta có thể giới hạn số instance tối đa được cache qua prop `max`. Khi có `max`, `<KeepAlive>` hoạt động như [LRU cache](<https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)>): nếu số instance cache sắp vượt quá giới hạn, instance được truy cập gần nhất ít nhất sẽ bị hủy để nhường chỗ cho instance mới.

```vue-html
<KeepAlive :max="10">
  <component :is="activeComponent" />
</KeepAlive>
```

## Lifecycle of Cached Instance {#lifecycle-of-cached-instance}

Khi một component instance bị gỡ khỏi DOM nhưng thuộc một cây component được `<KeepAlive>` cache, nó chuyển sang trạng thái **deactivated** thay vì bị unmount. Khi một instance được chèn vào DOM như một phần của cây được cache, nó **activated**.

<div class="composition-api">

Một component được keep‑alive có thể đăng ký lifecycle hook cho hai trạng thái này qua [`onActivated()`](/api/composition-api-lifecycle#onactivated) và [`onDeactivated()`](/api/composition-api-lifecycle#ondeactivated):

```vue
<script setup>
import { onActivated, onDeactivated } from 'vue'

onActivated(() => {
  // called on initial mount
  // and every time it is re-inserted from the cache
})

onDeactivated(() => {
  // called when removed from the DOM into the cache
  // and also when unmounted
})
</script>
```

</div>
<div class="options-api">

Một component được keep‑alive có thể đăng ký lifecycle hook cho hai trạng thái này qua hook [`activated`](/api/options-lifecycle#activated) và [`deactivated`](/api/options-lifecycle#deactivated):

```js
export default {
  activated() {
    // called on initial mount
    // and every time it is re-inserted from the cache
  },
  deactivated() {
    // called when removed from the DOM into the cache
    // and also when unmounted
  }
}
```

</div>

Lưu ý:

- <span class="composition-api">`onActivated`</span><span class="options-api">`activated`</span> cũng được gọi khi mount, và <span class="composition-api">`onDeactivated`</span><span class="options-api">`deactivated`</span> khi unmount.

- Cả hai hook hoạt động không chỉ cho root component được `<KeepAlive>` cache, mà còn cho các descendant trong cây được cache.
---

**Related**

- [`<KeepAlive>` API reference](/api/built-in-components#keepalive)
