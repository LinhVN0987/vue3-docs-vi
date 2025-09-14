# Teleport {#teleport}

 <VueSchoolLink href="https://vueschool.io/lessons/vue-3-teleport" title="Free Vue.js Teleport Lesson"/>

`<Teleport>` là built‑in component cho phép “teleport” một phần template của component vào một DOM node nằm ngoài cây DOM của chính component đó.

## Basic Usage {#basic-usage}

Đôi khi một phần template về mặt logic thuộc về component, nhưng về hiển thị, nó nên hiện ở nơi khác trong DOM, thậm chí ngoài ứng dụng Vue.

Ví dụ phổ biến nhất là khi xây modal toàn màn hình. Lý tưởng là code cho nút mở modal và bản thân modal ở cùng một single‑file component, vì chúng cùng liên quan đến trạng thái mở/đóng của modal. Nhưng điều đó khiến modal được render cạnh nút, lồng sâu trong cây DOM của ứng dụng. Việc định vị modal bằng CSS có thể phát sinh vấn đề.

Xem cấu trúc HTML sau.

```vue-html
<div class="outer">
  <h3>Vue Teleport Example</h3>
  <div>
    <MyModal />
  </div>
</div>
```

Và đây là phần triển khai `<MyModal>`:

<div class="composition-api">

```vue
<script setup>
import { ref } from 'vue'

const open = ref(false)
</script>

<template>
  <button @click="open = true">Open Modal</button>

  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</template>

<style scoped>
.modal {
  position: fixed;
  z-index: 999;
  top: 20%;
  left: 50%;
  width: 300px;
  margin-left: -150px;
}
</style>
```

</div>
<div class="options-api">

```vue
<script>
export default {
  data() {
    return {
      open: false
    }
  }
}
</script>

<template>
  <button @click="open = true">Open Modal</button>

  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</template>

<style scoped>
.modal {
  position: fixed;
  z-index: 999;
  top: 20%;
  left: 50%;
  width: 300px;
  margin-left: -150px;
}
</style>
```

</div>

Component chứa một `<button>` để mở modal, và một `<div>` với class `.modal`, chứa nội dung của modal và nút đóng.

Khi dùng component này trong cấu trúc HTML ban đầu, có một số vấn đề tiềm ẩn:

- `position: fixed` only places the element relative to the viewport when no ancestor element has `transform`, `perspective` or `filter` property set. If, for example, we intend to animate the ancestor `<div class="outer">` with a CSS transform, it would break the modal layout!

- The modal's `z-index` is constrained by its containing elements. If there is another element that overlaps with `<div class="outer">` and has a higher `z-index`, it would cover our modal.

`<Teleport>` cung cấp cách gọn gàng để xử lý: cho phép thoát khỏi cấu trúc DOM lồng nhau. Hãy sửa `<MyModal>` để dùng `<Teleport>`:

```vue-html{3,8}
<button @click="open = true">Open Modal</button>

<Teleport to="body">
  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</Teleport>
```

`to` của `<Teleport>` nhận một CSS selector hoặc một DOM node. Ở đây, chúng ta đang nói với Vue “**teleport** đoạn template này **đến** thẻ **`body`**”.

Bạn có thể bấm nút bên dưới và kiểm tra thẻ `<body>` bằng devtools của trình duyệt:

<script setup>
import { ref } from 'vue'
const open = ref(false)
</script>

<div class="demo">
  <button @click="open = true">Open Modal</button>
  <ClientOnly>
    <Teleport to="body">
      <div v-if="open" class="demo modal-demo">
        <p style="margin-bottom:20px">Hello from the modal!</p>
        <button @click="open = false">Close</button>
      </div>
    </Teleport>
  </ClientOnly>
</div>

<style>
.modal-demo {
  position: fixed;
  z-index: 999;
  top: 20%;
  left: 50%;
  width: 300px;
  margin-left: -150px;
  background-color: var(--vt-c-bg);
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}
</style>

Bạn có thể kết hợp `<Teleport>` với [`<Transition>`](./transition) để tạo modal có animation — xem [ví dụ](/examples/#modal).

:::tip
Mục tiêu `to` phải có sẵn trong DOM khi component `<Teleport>` được mount. Lý tưởng là một phần tử nằm ngoài toàn bộ ứng dụng Vue. Nếu nhắm tới phần tử do Vue render, cần đảm bảo phần tử đó được mount trước `<Teleport>`.
:::

## Using with Components {#using-with-components}

`<Teleport>` chỉ thay đổi cấu trúc DOM render — không ảnh hưởng hệ phân cấp logic của component. Tức là, nếu `<Teleport>` chứa một component, component đó vẫn là con logic của component cha chứa `<Teleport>`. Truyền props và emit event vẫn hoạt động như cũ.

Điều này cũng có nghĩa injection từ parent hoạt động như mong đợi, và child component vẫn nằm dưới parent trong Vue Devtools, thay vì ở vị trí nội dung thực tế được chuyển tới.

## Disabling Teleport {#disabling-teleport}

Một số trường hợp, ta muốn tắt `<Teleport>` có điều kiện. Ví dụ, render component dạng overlay trên desktop, nhưng inline trên mobile. `<Teleport>` hỗ trợ prop `disabled` có thể bật/tắt động:

```vue-html
<Teleport :disabled="isMobile">
  ...
</Teleport>
```

We could then dynamically update `isMobile`.

## Multiple Teleports on the Same Target {#multiple-teleports-on-the-same-target}

Một use case phổ biến là `<Modal>` có thể tái sử dụng, với khả năng nhiều instance hoạt động đồng thời. Với kịch bản này, nhiều `<Teleport>` có thể mount nội dung vào cùng một phần tử đích. Thứ tự là append đơn giản: mount sau sẽ nằm sau mount trước, tất cả trong phần tử đích.

Given the following usage:

```vue-html
<Teleport to="#modals">
  <div>A</div>
</Teleport>
<Teleport to="#modals">
  <div>B</div>
</Teleport>
```

The rendered result would be:

```html
<div id="modals">
  <div>A</div>
  <div>B</div>
</div>
```

## Deferred Teleport <sup class="vt-badge" data-text="3.5+" /> {#deferred-teleport}

Trong Vue 3.5 trở lên, ta có thể dùng prop `defer` để trì hoãn việc resolve mục tiêu của Teleport cho đến khi các phần khác của ứng dụng được mount. Điều này cho phép Teleport nhắm tới phần tử container do Vue render, nhưng ở phần sâu hơn của cây component:

```vue-html
<Teleport defer to="#late-div">...</Teleport>

<!-- somewhere later in the template -->
<div id="late-div"></div>
```

Lưu ý phần tử đích phải được render trong cùng tick mount/update với Teleport — tức nếu `<div>` được mount chậm hơn, Teleport vẫn báo lỗi. `defer` hoạt động tương tự hook `mounted`.

---

**Related**

- [`<Teleport>` API reference](/api/built-in-components#teleport)
- [Handling Teleports in SSR](/guide/scaling-up/ssr#teleports)
