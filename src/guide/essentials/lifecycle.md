# Lifecycle Hooks {#lifecycle-hooks}

Mỗi component instance trong Vue đi qua một chuỗi bước khởi tạo khi được tạo ra — ví dụ, cần thiết lập quan sát dữ liệu, biên dịch template, mount instance lên DOM, và cập nhật DOM khi dữ liệu thay đổi. Trong quá trình đó, nó cũng chạy các hàm gọi là lifecycle hooks, cho phép bạn chèn code của mình tại các giai đoạn cụ thể.

## Registering Lifecycle Hooks {#registering-lifecycle-hooks}

Ví dụ, hook <span class="composition-api">`onMounted`</span><span class="options-api">`mounted`</span> có thể dùng để chạy code sau khi component hoàn tất render ban đầu và tạo các DOM node:

<div class="composition-api">

```vue
<script setup>
import { onMounted } from 'vue'

onMounted(() => {
  console.log(`the component is now mounted.`)
})
</script>
```

</div>
<div class="options-api">

```js
export default {
  mounted() {
    console.log(`the component is now mounted.`)
  }
}
```

</div>

Cũng có các hook khác được gọi ở những giai đoạn khác nhau của vòng đời instance, phổ biến nhất là <span class="composition-api">[`onMounted`](/api/composition-api-lifecycle#onmounted), [`onUpdated`](/api/composition-api-lifecycle#onupdated), và [`onUnmounted`](/api/composition-api-lifecycle#onunmounted).</span><span class="options-api">[`mounted`](/api/options-lifecycle#mounted), [`updated`](/api/options-lifecycle#updated), và [`unmounted`](/api/options-lifecycle#unmounted).</span>

<div class="options-api">

Tất cả lifecycle hook được gọi với ngữ cảnh `this` trỏ đến instance đang hoạt động hiện tại gọi nó. Lưu ý điều này có nghĩa bạn nên tránh dùng arrow function khi khai báo lifecycle hook, vì khi đó bạn sẽ không thể truy cập instance qua `this`.

</div>

<div class="composition-api">

Khi gọi `onMounted`, Vue tự động liên kết callback đã đăng ký với component instance đang hoạt động. Điều này yêu cầu các hook được đăng ký **đồng bộ** trong quá trình setup của component. Ví dụ, đừng làm như sau:

```js
setTimeout(() => {
  onMounted(() => {
    // this won't work.
  })
}, 100)
```

Cũng lưu ý điều này không có nghĩa lời gọi phải nằm trực tiếp bên trong `setup()` hoặc `<script setup>`. `onMounted()` có thể được gọi trong một hàm bên ngoài miễn là call stack đồng bộ và xuất phát từ bên trong `setup()`.

</div>

## Lifecycle Diagram {#lifecycle-diagram}

Dưới đây là sơ đồ vòng đời của instance. Bạn chưa cần hiểu hết mọi thứ ngay lúc này, nhưng khi học và xây dựng nhiều hơn, nó sẽ là tài liệu tham khảo hữu ích.

![Component lifecycle diagram](./images/lifecycle.png)

<!-- https://www.figma.com/file/Xw3UeNMOralY6NV7gSjWdS/Vue-Lifecycle -->

Tham khảo <span class="composition-api">[Lifecycle Hooks API reference](/api/composition-api-lifecycle)</span><span class="options-api">[Lifecycle Hooks API reference](/api/options-lifecycle)</span> để biết chi tiết về tất cả lifecycle hook và các trường hợp sử dụng tương ứng.
