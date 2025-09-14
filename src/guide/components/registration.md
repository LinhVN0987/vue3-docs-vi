# Component Registration {#component-registration}

> Trang này giả định bạn đã đọc [Components Basics](/guide/essentials/component-basics). Nếu bạn mới với components, hãy đọc phần đó trước.

<VueSchoolLink href="https://vueschool.io/lessons/vue-3-global-vs-local-vue-components" title="Free Vue.js Component Registration Lesson"/>

Một Vue component cần được "đăng ký" để Vue biết nơi tìm phần triển khai của nó khi gặp trong template. Có hai cách đăng ký component: global và local.

## Global Registration {#global-registration}

Ta có thể làm cho component khả dụng toàn cục trong [Vue application](/guide/essentials/application) hiện tại bằng phương thức `.component()`:

```js
import { createApp } from 'vue'

const app = createApp({})

app.component(
  // the registered name
  'MyComponent',
  // the implementation
  {
    /* ... */
  }
)
```

Nếu dùng SFC, bạn sẽ đăng ký các file `.vue` được import:

```js
import MyComponent from './App.vue'

app.component('MyComponent', MyComponent)
```

Phương thức `.component()` có thể chain:

```js
app
  .component('ComponentA', ComponentA)
  .component('ComponentB', ComponentB)
  .component('ComponentC', ComponentC)
```

Component đăng ký toàn cục có thể dùng trong template của bất kỳ component nào trong ứng dụng:

```vue-html
<!-- this will work in any component inside the app -->
<ComponentA/>
<ComponentB/>
<ComponentC/>
```

Điều này áp dụng cho cả subcomponent: nghĩa là cả ba component này cũng khả dụng _bên trong nhau_.

## Local Registration {#local-registration}

Mặc dù tiện lợi, đăng ký toàn cục có vài nhược điểm:

1. Đăng ký toàn cục ngăn build system loại bỏ component không dùng ("tree‑shaking"). Nếu bạn đăng ký toàn cục một component nhưng không dùng ở đâu, nó vẫn sẽ nằm trong bundle cuối.

2. Đăng ký toàn cục làm mối quan hệ phụ thuộc kém rõ ràng trong ứng dụng lớn. Việc tìm phần triển khai của child component từ parent component đang dùng nó trở nên khó khăn hơn, ảnh hưởng đến khả năng bảo trì dài hạn (tương tự dùng quá nhiều biến global).

Đăng ký local giới hạn phạm vi khả dụng của component đã đăng ký trong component hiện tại. Nó giúp mối quan hệ phụ thuộc rõ ràng hơn và thân thiện với tree‑shaking hơn.

<div class="composition-api">

Khi dùng SFC với `<script setup>`, component được import có thể dùng cục bộ mà không cần đăng ký:

```vue
<script setup>
import ComponentA from './ComponentA.vue'
</script>

<template>
  <ComponentA />
</template>
```

Với trường hợp không dùng `<script setup>`, bạn cần dùng tùy chọn `components`:

```js
import ComponentA from './ComponentA.js'

export default {
  components: {
    ComponentA
  },
  setup() {
    // ...
  }
}
```

</div>
<div class="options-api">

Đăng ký local được thực hiện bằng tùy chọn `components`:

```vue
<script>
import ComponentA from './ComponentA.vue'

export default {
  components: {
    ComponentA
  }
}
</script>

<template>
  <ComponentA />
</template>
```

</div>

Với mỗi thuộc tính trong object `components`, key sẽ là tên đăng ký của component, còn value chứa phần triển khai của component. Ví dụ trên dùng property shorthand (ES2015) và tương đương với:

```js
export default {
  components: {
    ComponentA: ComponentA
  }
  // ...
}
```

Lưu ý **component đăng ký local _không_ khả dụng trong descendant component**. Ở đây, `ComponentA` chỉ khả dụng trong component hiện tại, không khả dụng trong child/descendant của nó.

## Component Name Casing {#component-name-casing}

Trong suốt hướng dẫn, chúng tôi dùng tên PascalCase khi đăng ký component. Lý do:

1. Tên PascalCase là JavaScript identifier hợp lệ. Điều này giúp import và đăng ký component dễ hơn trong JavaScript, đồng thời hỗ trợ IDE auto‑completion.

2. `<PascalCase />` giúp dễ nhận ra đây là Vue component thay vì phần tử HTML gốc trong template. Nó cũng phân biệt Vue component với custom element (web component).

Đây là phong cách được khuyến nghị khi làm việc với SFC hoặc string template. Tuy nhiên, như đã bàn trong [in-DOM Template Parsing Caveats](/guide/essentials/component-basics#in-dom-template-parsing-caveats), thẻ PascalCase không dùng được trong in‑DOM template.

May mắn là Vue hỗ trợ ánh xạ thẻ kebab‑case tới component đăng ký bằng PascalCase. Nghĩa là component đăng ký là `MyComponent` có thể được tham chiếu trong template của Vue (hoặc trong phần tử HTML do Vue render) qua cả `<MyComponent>` và `<my-component>`. Điều này cho phép dùng cùng một code đăng ký JavaScript bất kể nguồn template.
