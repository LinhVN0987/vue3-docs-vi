# Conditional Rendering {#conditional-rendering}

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/conditional-rendering-in-vue-3" title="Free Vue.js Conditional Rendering Lesson"/>
</div>

<div class="composition-api">
  <VueSchoolLink href="https://vueschool.io/lessons/vue-fundamentals-capi-conditionals-in-vue" title="Free Vue.js Conditional Rendering Lesson"/>
</div>

<script setup>
import { ref } from 'vue'
const awesome = ref(true)
</script>

## `v-if` {#v-if}

Directive `v-if` dùng để render có điều kiện một khối. Khối chỉ được render nếu biểu thức của directive trả về giá trị truthy.

```vue-html
<h1 v-if="awesome">Vue is awesome!</h1>
```

## `v-else` {#v-else}

Bạn có thể dùng directive `v-else` để chỉ định một "else block" cho `v-if`:

```vue-html
<button @click="awesome = !awesome">Toggle</button>

<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```

<div class="demo">
  <button @click="awesome = !awesome">Toggle</button>
  <h1 v-if="awesome">Vue is awesome!</h1>
  <h1 v-else>Oh no 😢</h1>
</div>

<div class="composition-api">

[Try it in the Playground](https://play.vuejs.org/#eNpFjkEOgjAQRa8ydIMulLA1hegJ3LnqBskAjdA27RQXhHu4M/GEHsEiKLv5mfdf/sBOxux7j+zAuCutNAQOyZtcKNkZbQkGsFjBCJXVHcQBjYUSqtTKERR3dLpDyCZmQ9bjViiezKKgCIGwM21BGBIAv3oireBYtrK8ZYKtgmg5BctJ13WLPJnhr0YQb1Lod7JaS4G8eATpfjMinjTphC8wtg7zcwNKw/v5eC1fnvwnsfEDwaha7w==)

</div>
<div class="options-api">

[Try it in the Playground](https://play.vuejs.org/#eNpFjj0OwjAMha9iMsEAFWuVVnACNqYsoXV/RJpEqVOQqt6DDYkTcgRSWoplWX7y56fXs6O1u84jixlvM1dbSoXGuzWOIMdCekXQCw2QS5LrzbQLckje6VEJglDyhq1pMAZyHidkGG9hhObRYh0EYWOVJAwKgF88kdFwyFSdXRPBZidIYDWvgqVkylIhjyb4ayOIV3votnXxfwrk2SPU7S/PikfVfsRnGFWL6akCbeD9fLzmK4+WSGz4AA5dYQY=)

</div>

Phần tử `v-else` phải đứng ngay sau một phần tử `v-if` hoặc `v-else-if` — nếu không sẽ không được nhận diện.

## `v-else-if` {#v-else-if}

`v-else-if`, như tên gọi, hoạt động như một "else if block" cho `v-if`. Nó cũng có thể được xâu chuỗi nhiều lần:

```vue-html
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

Similar to `v-else`, a `v-else-if` element must immediately follow a `v-if` or a `v-else-if` element.

## `v-if` on `<template>` {#v-if-on-template}

Vì `v-if` là một directive, nó phải gắn vào một phần tử duy nhất. Nhưng nếu ta muốn bật/tắt nhiều phần tử thì sao? Trong trường hợp này, ta có thể dùng `v-if` trên phần tử `<template>`, đóng vai trò wrapper “vô hình”. Kết quả render cuối cùng sẽ không chứa phần tử `<template>`.

```vue-html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

`v-else` and `v-else-if` can also be used on `<template>`.

## `v-show` {#v-show}

Một lựa chọn khác để hiển thị phần tử có điều kiện là directive `v-show`. Cách dùng về cơ bản giống nhau:

```vue-html
<h1 v-show="ok">Hello!</h1>
```

Điểm khác là phần tử với `v-show` sẽ luôn được render và giữ trong DOM; `v-show` chỉ bật/tắt thuộc tính CSS `display` của phần tử.

`v-show` không hỗ trợ phần tử `<template>`, cũng không hoạt động với `v-else`.

## `v-if` vs. `v-show` {#v-if-vs-v-show}

`v-if` là render có điều kiện “thực sự” vì nó đảm bảo các event listener và child component bên trong khối điều kiện được hủy và tạo lại đúng cách khi bật/tắt.

`v-if` cũng **lười**: nếu điều kiện là false ở lần render đầu, nó sẽ không làm gì — khối điều kiện sẽ không được render cho đến khi điều kiện lần đầu tiên trở thành true.

So với nó, `v-show` đơn giản hơn nhiều — phần tử luôn được render bất kể điều kiện ban đầu, chỉ bật/tắt bằng CSS.

Nói chung, `v-if` tốn chi phí khi bật/tắt nhiều hơn, còn `v-show` tốn chi phí render ban đầu cao hơn. Hãy ưu tiên `v-show` nếu bạn cần bật/tắt rất thường xuyên, và ưu tiên `v-if` nếu điều kiện ít có khả năng thay đổi lúc chạy.

## `v-if` with `v-for` {#v-if-with-v-for}

Khi `v-if` và `v-for` cùng dùng trên một phần tử, `v-if` sẽ được đánh giá trước. Xem [hướng dẫn render danh sách](list#v-for-with-v-if) để biết chi tiết.

::: warning Note
**Không** khuyến nghị dùng `v-if` và `v-for` trên cùng một phần tử do thứ tự ưu tiên ngầm định. Tham khảo [hướng dẫn render danh sách](list#v-for-with-v-if) để biết chi tiết.
:::
