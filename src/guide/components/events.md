<script setup>
import { onMounted } from 'vue'

if (typeof window !== 'undefined') {
  const hash = window.location.hash

  // The docs for v-model used to be part of this page. Attempt to redirect outdated links.
  if ([
    '#usage-with-v-model',
    '#v-model-arguments',
    '#multiple-v-model-bindings',
    '#handling-v-model-modifiers'
  ].includes(hash)) {
    onMounted(() => {
      window.location = './v-model.html' + hash
    })
  }
}
</script>

# Component Events {#component-events}

> Trang này giả định bạn đã đọc [Components Basics](/guide/essentials/component-basics). Nếu bạn mới với components, hãy đọc phần đó trước.

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/defining-custom-events-emits" title="Free Vue.js Lesson on Defining Custom Events"/>
</div>

## Emitting and Listening to Events {#emitting-and-listening-to-events}

Một component có thể emit custom event trực tiếp trong biểu thức template (ví dụ trong handler `v-on`) bằng method `$emit` built‑in:

```vue-html
<!-- MyComponent -->
<button @click="$emit('someEvent')">Click Me</button>
```

<div class="options-api">

Method `$emit()` cũng khả dụng trên component instance dưới dạng `this.$emit()`:

```js
export default {
  methods: {
    submit() {
      this.$emit('someEvent')
    }
  }
}
```

</div>

Parent có thể lắng nghe bằng `v-on`:

```vue-html
<MyComponent @some-event="callback" />
```

Modifier `.once` cũng được hỗ trợ trên listener của component events:

```vue-html
<MyComponent @some-event.once="callback" />
```

Tương tự component và props, tên event hỗ trợ chuyển đổi kiểu chữ tự động. Lưu ý ta emit event dạng camelCase nhưng có thể lắng nghe bằng listener dạng kebab‑case ở parent. Như [props casing](/guide/components/props#prop-name-casing), chúng tôi khuyến nghị dùng listener dạng kebab‑case trong template.

:::tip
Khác với DOM event gốc, event do component emit **không** bubble. Bạn chỉ có thể lắng nghe các event do child component trực tiếp emit. Nếu cần giao tiếp giữa sibling hoặc component lồng sâu, hãy dùng event bus bên ngoài hoặc [giải pháp quản lý state toàn cục](/guide/scaling-up/state-management).
:::

## Event Arguments {#event-arguments}

Đôi khi hữu ích khi emit một giá trị cụ thể cùng event. Ví dụ, ta muốn component `<BlogPost>` quyết định mức độ phóng to text. Khi đó, ta có thể truyền thêm đối số cho `$emit` để cung cấp giá trị này:

```vue-html
<button @click="$emit('increaseBy', 1)">
  Increase by 1
</button>
```

Sau đó, khi lắng nghe event ở parent, ta có thể dùng inline arrow function làm listener để truy cập đối số event:

```vue-html
<MyButton @increase-by="(n) => count += n" />
```

Hoặc nếu event handler là một method:

```vue-html
<MyButton @increase-by="increaseCount" />
```

Giá trị sẽ được truyền làm tham số đầu tiên của method đó:

<div class="options-api">

```js
methods: {
  increaseCount(n) {
    this.count += n
  }
}
```

</div>
<div class="composition-api">

```js
function increaseCount(n) {
  count.value += n
}
```

</div>

:::tip
Mọi đối số bổ sung truyền cho `$emit()` sau tên event sẽ được chuyển tiếp đến listener. Ví dụ, với `$emit('foo', 1, 2, 3)` hàm listener sẽ nhận ba đối số.
:::

## Declaring Emitted Events {#declaring-emitted-events}

Một component có thể khai báo rõ ràng các event mà nó sẽ emit bằng <span class="composition-api">macro [`defineEmits()`](/api/sfc-script-setup#defineprops-defineemits)</span><span class="options-api">tùy chọn [`emits`](/api/options-state#emits)</span>:

<div class="composition-api">

```vue
<script setup>
defineEmits(['inFocus', 'submit'])
</script>
```

Method `$emit` dùng trong `<template>` không thể truy cập trong phần `<script setup>` của component, nhưng `defineEmits()` trả về một hàm tương đương để dùng thay thế:

```vue
<script setup>
const emit = defineEmits(['inFocus', 'submit'])

function buttonClick() {
  emit('submit')
}
</script>
```

Macro `defineEmits()` **không** thể dùng bên trong một hàm; nó phải được đặt trực tiếp trong `<script setup>`, như ví dụ trên.

Nếu bạn dùng hàm `setup` thay vì `<script setup>`, event nên được khai báo bằng tùy chọn [`emits`](/api/options-state#emits), và hàm `emit` được cung cấp qua context của `setup()`:

```js
export default {
  emits: ['inFocus', 'submit'],
  setup(props, ctx) {
    ctx.emit('submit')
  }
}
```

Giống các thuộc tính khác của context `setup()`, có thể destructure `emit` một cách an toàn:

```js
export default {
  emits: ['inFocus', 'submit'],
  setup(props, { emit }) {
    emit('submit')
  }
}
```

</div>
<div class="options-api">

```js
export default {
  emits: ['inFocus', 'submit']
}
```

</div>

Tùy chọn `emits` và macro `defineEmits()` cũng hỗ trợ object syntax. Nếu dùng TypeScript, bạn có thể định kiểu cho đối số, cho phép kiểm tra payload của event emit ở runtime:

<div class="composition-api">

```vue
<script setup lang="ts">
const emit = defineEmits({
  submit(payload: { email: string, password: string }) {
    // return `true` or `false` to indicate
    // validation pass / fail
  }
})
</script>
```

Nếu bạn dùng TypeScript với `<script setup>`, bạn cũng có thể khai báo emitted events chỉ với type annotation:

```vue
<script setup lang="ts">
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()
</script>
```

Chi tiết: [Typing Component Emits](/guide/typescript/composition-api#typing-component-emits) <sup class="vt-badge ts" />

</div>
<div class="options-api">

```js
export default {
  emits: {
    submit(payload: { email: string, password: string }) {
      // return `true` or `false` to indicate
      // validation pass / fail
    }
  }
}
```

See also: [Typing Component Emits](/guide/typescript/options-api#typing-component-emits) <sup class="vt-badge ts" />

</div>

Although optional, it is recommended to define all emitted events in order to better document how a component should work. It also allows Vue to exclude known listeners from [fallthrough attributes](/guide/components/attrs#v-on-listener-inheritance), avoiding edge cases caused by DOM events manually dispatched by 3rd party code.

:::tip
If a native event (e.g., `click`) is defined in the `emits` option, the listener will now only listen to component-emitted `click` events and no longer respond to native `click` events.
:::

## Events Validation {#events-validation}

Similar to prop type validation, an emitted event can be validated if it is defined with the object syntax instead of the array syntax.

To add validation, the event is assigned a function that receives the arguments passed to the <span class="options-api">`this.$emit`</span><span class="composition-api">`emit`</span> call and returns a boolean to indicate whether the event is valid or not.

<div class="composition-api">

```vue
<script setup>
const emit = defineEmits({
  // No validation
  click: null,

  // Validate submit event
  submit: ({ email, password }) => {
    if (email && password) {
      return true
    } else {
      console.warn('Invalid submit event payload!')
      return false
    }
  }
})

function submitForm(email, password) {
  emit('submit', { email, password })
}
</script>
```

</div>
<div class="options-api">

```js
export default {
  emits: {
    // No validation
    click: null,

    // Validate submit event
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Invalid submit event payload!')
        return false
      }
    }
  },
  methods: {
    submitForm(email, password) {
      this.$emit('submit', { email, password })
    }
  }
}
```

</div>
