---
outline: deep
---

# Fallthrough Attributes {#fallthrough-attributes}

> Trang này giả định bạn đã đọc [Components Basics](/guide/essentials/component-basics). Nếu bạn mới với components, hãy đọc phần đó trước.

## Attribute Inheritance {#attribute-inheritance}

"Fallthrough attribute" là một attribute hoặc `v-on` listener được truyền vào component nhưng không được khai báo tường minh trong [props](./props) hoặc [emits](./events#declaring-emitted-events) của component nhận. Ví dụ thường gặp gồm các attribute `class`, `style`, `id`.

Khi một component render một root element duy nhất, các fallthrough attribute sẽ được tự động thêm vào attribute của root element. Ví dụ, với component `<MyButton>` có template sau:

```vue-html
<!-- template of <MyButton> -->
<button>Click Me</button>
```

Và parent dùng component này như sau:

```vue-html
<MyButton class="large" />
```

DOM render cuối cùng sẽ là:

```html
<button class="large">Click Me</button>
```

Ở đây, `<MyButton>` không khai báo `class` là prop được chấp nhận. Do đó, `class` được coi là fallthrough attribute và tự động thêm vào root element của `<MyButton>`.

### `class` and `style` Merging {#class-and-style-merging}

Nếu root element của child component đã có `class` hoặc `style`, chúng sẽ được merge với giá trị `class` và `style` kế thừa từ parent. Giả sử ta đổi template của `<MyButton>` như sau:

```vue-html
<!-- template of <MyButton> -->
<button class="btn">Click Me</button>
```

Khi đó DOM render cuối cùng sẽ trở thành:

```html
<button class="btn large">Click Me</button>
```

### `v-on` Listener Inheritance {#v-on-listener-inheritance}

Quy tắc tương tự áp dụng cho `v-on` listener:

```vue-html
<MyButton @click="onClick" />
```

Listener `click` sẽ được thêm vào root element của `<MyButton>`, tức phần tử `<button>` gốc. Khi `<button>` gốc được click, nó sẽ gọi method `onClick` của parent. Nếu `<button>` gốc đã có listener `click` gắn bằng `v-on`, thì cả hai listener sẽ cùng chạy.

### Nested Component Inheritance {#nested-component-inheritance}

Nếu một component render một component khác làm root node — ví dụ, ta refactor `<MyButton>` để render `<BaseButton>` làm root:

```vue-html
<!-- template of <MyButton/> that simply renders another component -->
<BaseButton />
```

Khi đó các fallthrough attribute mà `<MyButton>` nhận sẽ tự động được forward tới `<BaseButton>`.

Lưu ý:

1. Forwarded attributes do not include any attributes that are declared as props, or `v-on` listeners of declared events by `<MyButton>` - in other words, the declared props and listeners have been "consumed" by `<MyButton>`.

2. Forwarded attributes may be accepted as props by `<BaseButton>`, if declared by it.

## Disabling Attribute Inheritance {#disabling-attribute-inheritance}

Nếu bạn **không** muốn component tự động thừa kế attributes, hãy đặt `inheritAttrs: false` trong tùy chọn của component.

<div class="composition-api">

 Từ 3.3 bạn cũng có thể dùng [`defineOptions`](/api/sfc-script-setup#defineoptions) trực tiếp trong `<script setup>`:

```vue
<script setup>
defineOptions({
  inheritAttrs: false
})
// ...setup logic
</script>
```

</div>

Kịch bản thường gặp khi tắt attribute inheritance là khi các attribute cần áp dụng lên phần tử khác ngoài root node. Bằng cách đặt `inheritAttrs: false`, bạn có toàn quyền kiểm soát nơi áp dụng fallthrough attributes.

Các fallthrough attribute có thể truy cập trực tiếp trong biểu thức template dưới tên `$attrs`:

```vue-html
<span>Fallthrough attributes: {{ $attrs }}</span>
```

Object `$attrs` bao gồm tất cả attribute không được khai báo bởi `props` hoặc `emits` của component (ví dụ `class`, `style`, `v-on` listener, ...).

Một vài lưu ý:

- Unlike props, fallthrough attributes preserve their original casing in JavaScript, so an attribute like `foo-bar` needs to be accessed as `$attrs['foo-bar']`.

- A `v-on` event listener like `@click` will be exposed on the object as a function under `$attrs.onClick`.

Với ví dụ `<MyButton>` ở [phần trước](#attribute-inheritance) — đôi khi ta cần bọc `<button>` thực bằng một `<div>` để phục vụ styling:

```vue-html
<div class="btn-wrapper">
  <button class="btn">Click Me</button>
</div>
```

Ta muốn tất cả fallthrough attribute như `class` và `v-on` listener được áp dụng cho `<button>` bên trong, không phải `<div>` bên ngoài. Có thể đạt được điều này bằng `inheritAttrs: false` và `v-bind="$attrs"`:

```vue-html{2}
<div class="btn-wrapper">
  <button class="btn" v-bind="$attrs">Click Me</button>
</div>
```

Nhớ rằng [`v-bind` không có tham số](/guide/essentials/template-syntax#dynamically-binding-multiple-attributes) sẽ bind tất cả thuộc tính của một object làm attribute của phần tử đích.

## Attribute Inheritance on Multiple Root Nodes {#attribute-inheritance-on-multiple-root-nodes}

Khác với component có một root node, component có nhiều root node không có hành vi fallthrough attribute tự động. Nếu `$attrs` không được bind tường minh, sẽ có cảnh báo lúc chạy.

```vue-html
<CustomLayout id="custom-layout" @click="changeValue" />
```

Nếu `<CustomLayout>` có template nhiều root như dưới đây, sẽ có cảnh báo vì Vue không thể biết áp dụng fallthrough attribute ở đâu:

```vue-html
<header>...</header>
<main>...</main>
<footer>...</footer>
```

Cảnh báo sẽ biến mất nếu `$attrs` được bind tường minh:

```vue-html{2}
<header>...</header>
<main v-bind="$attrs">...</main>
<footer>...</footer>
```

## Accessing Fallthrough Attributes in JavaScript {#accessing-fallthrough-attributes-in-javascript}

<div class="composition-api">

Nếu cần, bạn có thể truy cập fallthrough attribute của component trong `<script setup>` bằng API `useAttrs()`:

```vue
<script setup>
import { useAttrs } from 'vue'

const attrs = useAttrs()
</script>
```

Nếu không dùng `<script setup>`, `attrs` sẽ xuất hiện như một thuộc tính của context `setup()`:

```js
export default {
  setup(props, ctx) {
    // fallthrough attributes are exposed as ctx.attrs
    console.log(ctx.attrs)
  }
}
```

Lưu ý dù object `attrs` ở đây luôn phản ánh fallthrough attribute mới nhất, nó không reactive (vì lý do hiệu năng). Bạn không thể dùng watcher để theo dõi thay đổi của nó. Nếu cần reactive, hãy dùng một prop. Hoặc, bạn có thể dùng `onUpdated()` để thực hiện side effect với `attrs` mới nhất ở mỗi lần cập nhật.

</div>

<div class="options-api">

Nếu cần, bạn có thể truy cập fallthrough attribute qua thuộc tính instance `$attrs`:

```js
export default {
  created() {
    console.log(this.$attrs)
  }
}
```

</div>
