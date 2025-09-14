# Provide / Inject {#provide-inject}

> Trang này giả định bạn đã đọc [Components Basics](/guide/essentials/component-basics). Nếu bạn mới với components, hãy đọc phần đó trước.

## Prop Drilling {#prop-drilling}

Thông thường, khi cần truyền dữ liệu từ parent xuống child component, ta dùng [props](/guide/components/props). Tuy nhiên, hãy tưởng tượng chúng ta có một cây component lớn, và một component lồng sâu cần thứ gì đó từ ancestor xa. Nếu chỉ dùng props, chúng ta phải truyền cùng một prop xuyên suốt cả chuỗi parent:

![prop drilling diagram](./images/prop-drilling.png)

<!-- https://www.figma.com/file/yNDTtReM2xVgjcGVRzChss/prop-drilling -->

Lưu ý dù component `<Footer>` có thể không quan tâm các props này, nó vẫn phải khai báo và chuyền tiếp để `<DeepChild>` có thể truy cập. Nếu chuỗi parent dài hơn, nhiều component sẽ bị ảnh hưởng. Đây gọi là "props drilling" và không hề dễ chịu.

Chúng ta có thể giải quyết props drilling bằng `provide` và `inject`. Parent component có thể đóng vai trò **dependency provider** cho toàn bộ descendants. Bất kỳ component nào trong cây descendants, bất kể sâu đến đâu, đều có thể **inject** dependency do các component phía trên cung cấp.

![Provide/inject scheme](./images/provide-inject.png)

<!-- https://www.figma.com/file/PbTJ9oXis5KUawEOWdy2cE/provide-inject -->

## Provide {#provide}

<div class="composition-api">

Để cung cấp dữ liệu cho descendants của component, dùng hàm [`provide()`](/api/composition-api-dependency-injection#provide):

```vue
<script setup>
import { provide } from 'vue'

provide(/* key */ 'message', /* value */ 'hello!')
</script>
```

Nếu không dùng `<script setup>`, đảm bảo gọi `provide()` một cách đồng bộ bên trong `setup()`:

```js
import { provide } from 'vue'

export default {
  setup() {
    provide(/* key */ 'message', /* value */ 'hello!')
  }
}
```

Hàm `provide()` nhận hai đối số. Đối số đầu tiên gọi là **injection key**, có thể là chuỗi hoặc `Symbol`. Injection key được descendants dùng để tra cứu giá trị cần inject. Một component có thể gọi `provide()` nhiều lần với các injection key khác nhau để cung cấp các giá trị khác nhau.

Đối số thứ hai là giá trị cung cấp. Giá trị có thể thuộc bất kỳ kiểu nào, bao gồm state reactive như ref:

```js
import { ref, provide } from 'vue'

const count = ref(0)
provide('key', count)
```

Cung cấp giá trị reactive cho phép các descendant dùng giá trị đó thiết lập kết nối reactive với component provider.

</div>

<div class="options-api">

Để cung cấp dữ liệu cho descendants, dùng tùy chọn [`provide`](/api/options-composition#provide):

```js
export default {
  provide: {
    message: 'hello!'
  }
}
```

Với mỗi thuộc tính trong object `provide`, key được child component dùng để tìm giá trị cần inject, còn value là giá trị thực sự được inject.

Nếu cần cung cấp state theo từng instance, ví dụ dữ liệu khai báo qua `data()`, thì `provide` phải dùng giá trị là một hàm:

```js{7-12}
export default {
  data() {
    return {
      message: 'hello!'
    }
  },
  provide() {
    // use function syntax so that we can access `this`
    return {
      message: this.message
    }
  }
}
```

Tuy nhiên, lưu ý điều này **không** khiến injection trở thành reactive. Chúng ta sẽ bàn về [làm injection reactive](#working-with-reactivity) bên dưới.

</div>

## App-level Provide {#app-level-provide}

Ngoài việc provide trong component, ta cũng có thể provide ở cấp app:

```js
import { createApp } from 'vue'

const app = createApp({})

app.provide(/* key */ 'message', /* value */ 'hello!')
```

Provide cấp app khả dụng cho mọi component được render trong ứng dụng. Điều này đặc biệt hữu ích khi viết [plugins](/guide/reusability/plugins), vì plugin thường không thể provide giá trị thông qua component.

## Inject {#inject}

<div class="composition-api">

Để inject dữ liệu được ancestor cung cấp, dùng hàm [`inject()`](/api/composition-api-dependency-injection#inject):

```vue
<script setup>
import { inject } from 'vue'

const message = inject('message')
</script>
```

Nếu nhiều parent provide dữ liệu với cùng key, inject sẽ lấy giá trị từ parent gần nhất trong chuỗi parent của component.

Nếu giá trị cung cấp là một ref, nó sẽ được inject nguyên trạng và **không** tự động unwrapped. Điều này cho phép component inject giữ kết nối reactive với component provider.

[Full provide + inject Example with Reactivity](https://play.vuejs.org/#eNqFUUFugzAQ/MrKF1IpxfeIVKp66Kk/8MWFDXYFtmUbpArx967BhURRU9/WOzO7MzuxV+fKcUB2YlWovXYRAsbBvQije2d9hAk8Xo7gvB11gzDDxdseCuIUG+ZN6a7JjZIvVRIlgDCcw+d3pmvTglz1okJ499I0C3qB1dJQT9YRooVaSdNiACWdQ5OICj2WwtTWhAg9hiBbhHNSOxQKu84WT8LkNQ9FBhTHXyg1K75aJHNUROxdJyNSBVBp44YI43NvG+zOgmWWYGt7dcipqPhGZEe2ef07wN3lltD+lWN6tNkV/37+rdKjK2rzhRTt7f3u41xhe37/xJZGAL2PLECXa9NKdD/a6QTTtGnP88LgiXJtYv4BaLHhvg==)

Tương tự, nếu không dùng `<script setup>`, `inject()` chỉ nên được gọi đồng bộ trong `setup()`:

```js
import { inject } from 'vue'

export default {
  setup() {
    const message = inject('message')
    return { message }
  }
}
```

</div>

<div class="options-api">

Để inject dữ liệu do ancestor cung cấp, dùng tùy chọn [`inject`](/api/options-composition#inject):

```js
export default {
  inject: ['message'],
  created() {
    console.log(this.message) // injected value
  }
}
```

Injection được resolve **trước** state riêng của component, vì vậy bạn có thể truy cập thuộc tính được inject trong `data()`:

```js
export default {
  inject: ['message'],
  data() {
    return {
      // initial data based on injected value
      fullMessage: this.message
    }
  }
}
```

If multiple parents provide data with the same key, inject will resolve to the value from the closest parent in component's parent chain.

[Full provide + inject example](https://play.vuejs.org/#eNqNkcFqwzAQRH9l0EUthOhuRKH00FO/oO7B2JtERZaEvA4F43+vZCdOTAIJCImRdpi32kG8h7A99iQKobs6msBvpTNt8JHxcTC2wS76FnKrJpVLZelKR39TSUO7qreMoXRA7ZPPkeOuwHByj5v8EqI/moZeXudCIBL30Z0V0FLXVXsqIA9krU8R+XbMR9rS0mqhS4KpDbZiSgrQc5JKQqvlRWzEQnyvuc9YuWbd4eXq+TZn0IvzOeKr8FvsNcaK/R6Ocb9Uc4FvefpE+fMwP0wH8DU7wB77nIo6x6a2hvNEME5D0CpbrjnHf+8excI=)

### Injection Aliasing \* {#injection-aliasing}

Khi dùng cú pháp mảng cho `inject`, các thuộc tính inject sẽ xuất hiện trên instance với cùng key. Trong ví dụ trên, thuộc tính được provide với key `"message"` và inject thành `this.message`. Local key giống injection key.

Nếu muốn inject thuộc tính với local key khác, ta cần dùng cú pháp object cho tùy chọn `inject`:

```js
export default {
  inject: {
    /* local key */ localMessage: {
      from: /* injection key */ 'message'
    }
  }
}
```

Ở đây, component sẽ tìm thuộc tính được provide với key `"message"`, rồi expose nó là `this.localMessage`.

</div>

### Injection Default Values {#injection-default-values}

Mặc định, `inject` giả định key cần inject được provide ở đâu đó trong chuỗi parent. Nếu key không được provide, sẽ có cảnh báo lúc chạy.

Nếu muốn thuộc tính inject hoạt động với provider tùy chọn, ta cần khai báo giá trị mặc định, tương tự props:

<div class="composition-api">

```js
// `value` will be "default value"
// if no data matching "message" was provided
const value = inject('message', 'default value')
```

Một số trường hợp, giá trị mặc định có thể cần được tạo bằng cách gọi hàm hoặc khởi tạo class mới. Để tránh tính toán không cần thiết hoặc side effect khi giá trị tùy chọn không được dùng, ta có thể dùng factory function để tạo giá trị mặc định:

```js
const value = inject('key', () => new ExpensiveClass(), true)
```

Tham số thứ ba cho biết giá trị mặc định sẽ được coi là một factory function.

</div>

<div class="options-api">

```js
export default {
  // object syntax is required
  // when declaring default values for injections
  inject: {
    message: {
      from: 'message', // this is optional if using the same key for injection
      default: 'default value'
    },
    user: {
      // use a factory function for non-primitive values that are expensive
      // to create, or ones that should be unique per component instance.
      default: () => ({ name: 'John' })
    }
  }
}
```

</div>

## Working with Reactivity {#working-with-reactivity}

<div class="composition-api">

When using reactive provide / inject values, **it is recommended to keep any mutations to reactive state inside of the _provider_ whenever possible**. This ensures that the provided state and its possible mutations are co-located in the same component, making it easier to maintain in the future.

There may be times when we need to update the data from an injector component. In such cases, we recommend providing a function that is responsible for mutating the state:

```vue{7-9,13}
<!-- inside provider component -->
<script setup>
import { provide, ref } from 'vue'

const location = ref('North Pole')

function updateLocation() {
  location.value = 'South Pole'
}

provide('location', {
  location,
  updateLocation
})
</script>
```

```vue{5}
<!-- in injector component -->
<script setup>
import { inject } from 'vue'

const { location, updateLocation } = inject('location')
</script>

<template>
  <button @click="updateLocation">{{ location }}</button>
</template>
```

Finally, you can wrap the provided value with [`readonly()`](/api/reactivity-core#readonly) if you want to ensure that the data passed through `provide` cannot be mutated by the injector component.

```vue
<script setup>
import { ref, provide, readonly } from 'vue'

const count = ref(0)
provide('read-only-count', readonly(count))
</script>
```

</div>

<div class="options-api">

In order to make injections reactively linked to the provider, we need to provide a computed property using the [computed()](/api/reactivity-core#computed) function:

```js{12}
import { computed } from 'vue'

export default {
  data() {
    return {
      message: 'hello!'
    }
  },
  provide() {
    return {
      // explicitly provide a computed property
      message: computed(() => this.message)
    }
  }
}
```

[Full provide + inject Example with Reactivity](https://play.vuejs.org/#eNqNUctqwzAQ/JVFFyeQxnfjBEoPPfULqh6EtYlV9EKWTcH43ytZtmPTQA0CsdqZ2dlRT16tPXctkoKUTeWE9VeqhbLGeXirheRwc0ZBds7HKkKzBdBDZZRtPXIYJlzqU40/I4LjjbUyIKmGEWw0at8UgZrUh1PscObZ4ZhQAA596/RcAShsGnbHArIapTRBP74O8Up060wnOO5QmP0eAvZyBV+L5jw1j2tZqsMp8yWRUHhUVjKPoQIohQ460L0ow1FeKJlEKEnttFweijJfiORElhCf5f3umObb0B9PU/I7kk17PJj7FloN/2t7a2Pj/Zkdob+x8gV8ZlMs2de/8+14AXwkBngD9zgVqjg2rNXPvwjD+EdlHilrn8MvtvD1+Q==)

The `computed()` function is typically used in Composition API components, but can also be used to complement certain use cases in Options API. You can learn more about its usage by reading the [Reactivity Fundamentals](/guide/essentials/reactivity-fundamentals) and [Computed Properties](/guide/essentials/computed) with the API Preference set to Composition API.

</div>

## Working with Symbol Keys {#working-with-symbol-keys}

So far, we have been using string injection keys in the examples. If you are working in a large application with many dependency providers, or you are authoring components that are going to be used by other developers, it is best to use [Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol) injection keys to avoid potential collisions.

It's recommended to export the Symbols in a dedicated file:

```js [keys.js]
export const myInjectionKey = Symbol()
```

<div class="composition-api">

```js
// in provider component
import { provide } from 'vue'
import { myInjectionKey } from './keys.js'

provide(myInjectionKey, {
  /* data to provide */
})
```

```js
// in injector component
import { inject } from 'vue'
import { myInjectionKey } from './keys.js'

const injected = inject(myInjectionKey)
```

See also: [Typing Provide / Inject](/guide/typescript/composition-api#typing-provide-inject) <sup class="vt-badge ts" />

</div>

<div class="options-api">

```js
// in provider component
import { myInjectionKey } from './keys.js'

export default {
  provide() {
    return {
      [myInjectionKey]: {
        /* data to provide */
      }
    }
  }
}
```

```js
// in injector component
import { myInjectionKey } from './keys.js'

export default {
  inject: {
    injected: { from: myInjectionKey }
  }
}
```

</div>
