# Props {#props}

> Trang này giả định bạn đã đọc [Components Basics](/guide/essentials/component-basics). Nếu bạn mới với components, hãy đọc phần đó trước.

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/vue-3-reusable-components-with-props" title="Free Vue.js Props Lesson"/>
</div>

## Props Declaration {#props-declaration}

Vue component cần khai báo props một cách tường minh để Vue biết những props bên ngoài nào truyền vào component nên được coi là fallthrough attributes (sẽ được bàn trong [phần riêng](/guide/components/attrs)).

<div class="composition-api">

In SFCs using `<script setup>`, props can be declared using the `defineProps()` macro:

```vue
<script setup>
const props = defineProps(['foo'])

console.log(props.foo)
</script>
```

Với component không dùng `<script setup>`, props được khai báo bằng tùy chọn [`props`](/api/options-state#props):

```js
export default {
  props: ['foo'],
  setup(props) {
    // setup() receives props as the first argument.
    console.log(props.foo)
  }
}
```

Lưu ý đối số truyền cho `defineProps()` giống với giá trị cung cấp cho tùy chọn `props`: cùng một API tùy chọn props được dùng chung giữa hai cách khai báo.

</div>

<div class="options-api">

Props được khai báo bằng tùy chọn [`props`](/api/options-state#props):

```js
export default {
  props: ['foo'],
  created() {
    // props are exposed on `this`
    console.log(this.foo)
  }
}
```

</div>

Ngoài việc khai báo props bằng mảng các chuỗi, chúng ta cũng có thể dùng object syntax:

<div class="options-api">

```js
export default {
  props: {
    title: String,
    likes: Number
  }
}
```

</div>
<div class="composition-api">

```js
// in <script setup>
defineProps({
  title: String,
  likes: Number
})
```

```js
// in non-<script setup>
export default {
  props: {
    title: String,
    likes: Number
  }
}
```

</div>

Với mỗi thuộc tính trong object syntax, key là tên của prop, còn value là constructor function của kiểu dữ liệu kỳ vọng.

Cách này vừa tài liệu hóa component của bạn, vừa cảnh báo cho developer khác dùng component nếu họ truyền sai kiểu (trong console của trình duyệt). Chúng ta sẽ bàn chi tiết hơn về [prop validation](#prop-validation) ở bên dưới.

<div class="options-api">

See also: [Typing Component Props](/guide/typescript/options-api#typing-component-props) <sup class="vt-badge ts" />

</div>

<div class="composition-api">

Nếu bạn dùng TypeScript với `<script setup>`, bạn cũng có thể khai báo props chỉ với type annotation:

```vue
<script setup lang="ts">
defineProps<{
  title?: string
  likes?: number
}>()
</script>
```

Chi tiết: [Typing Component Props](/guide/typescript/composition-api#typing-component-props) <sup class="vt-badge ts" />

</div>

<div class="composition-api">

## Reactive Props Destructure <sup class="vt-badge" data-text="3.5+" /> \*\* {#reactive-props-destructure}

Hệ thống reactivity của Vue theo dõi việc sử dụng state dựa trên truy cập thuộc tính. Ví dụ, khi bạn truy cập `props.foo` trong computed getter hoặc watcher, prop `foo` sẽ được theo dõi như một dependency.

So, given the following code:

```js
const { foo } = defineProps(['foo'])

watchEffect(() => {
  // runs only once before 3.5
  // re-runs when the "foo" prop changes in 3.5+
  console.log(foo)
})
```

Ở phiên bản 3.4 trở xuống, `foo` là một hằng số thực sự và sẽ không thay đổi. Ở phiên bản 3.5 trở lên, compiler của Vue tự động thêm tiền tố `props.` khi code trong cùng block `<script setup>` truy cập biến được destructure từ `defineProps`. Do đó đoạn code trên tương đương với:

```js {5}
const props = defineProps(['foo'])

watchEffect(() => {
  // `foo` transformed to `props.foo` by the compiler
  console.log(props.foo)
})
```

Ngoài ra, bạn có thể dùng cú pháp giá trị mặc định của JavaScript để khai báo default value cho props. Điều này đặc biệt hữu ích khi dùng khai báo props dựa trên kiểu:

```ts
const { foo = 'hello' } = defineProps<{ foo?: string }>()
```

If you prefer to have more visual distinction between destructured props and normal variables in your IDE, Vue's VSCode extension provides a setting to enable inlay-hints for destructured props.

### Passing Destructured Props into Functions {#passing-destructured-props-into-functions}

Khi chúng ta truyền một destructured prop vào một hàm, ví dụ:

```js
const { foo } = defineProps(['foo'])

watch(foo, /* ... */)
```

Điều này sẽ không hoạt động như mong đợi vì nó tương đương `watch(props.foo, ...)` — chúng ta đang truyền một giá trị thay vì nguồn dữ liệu reactive cho `watch`. Thực tế, compiler của Vue sẽ phát hiện và cảnh báo trường hợp này.

Tương tự cách watch một prop bình thường với `watch(() => props.foo, ...)`, chúng ta cũng có thể watch một destructured prop bằng cách bọc nó trong getter:

```js
watch(() => foo, /* ... */)
```

Ngoài ra, đây là cách khuyến nghị khi cần truyền một destructured prop vào một hàm bên ngoài mà vẫn giữ được reactivity:

```js
useComposable(() => foo)
```

Hàm bên ngoài có thể gọi getter (hoặc chuẩn hóa bằng [toValue](/api/reactivity-utilities.html#tovalue)) khi cần theo dõi thay đổi của prop truyền vào, ví dụ trong computed hoặc watcher getter.

</div>

## Prop Passing Details {#prop-passing-details}

### Prop Name Casing {#prop-name-casing}

Chúng ta khai báo prop name dài theo camelCase để tránh phải dùng dấu nháy khi dùng làm property key, và cho phép tham chiếu trực tiếp trong biểu thức template vì chúng là JavaScript identifier hợp lệ:

<div class="composition-api">

```js
defineProps({
  greetingMessage: String
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    greetingMessage: String
  }
}
```

</div>

```vue-html
<span>{{ greetingMessage }}</span>
```

Về mặt kỹ thuật, bạn cũng có thể dùng camelCase khi truyền props cho child component (trừ [in‑DOM template](/guide/essentials/component-basics#in-dom-template-parsing-caveats)). Tuy nhiên, quy ước chung là dùng kebab‑case trong mọi trường hợp để thống nhất với HTML attributes:

```vue-html
<MyComponent greeting-message="hello" />
```

Chúng ta dùng [PascalCase cho thẻ component](/guide/components/registration#component-name-casing) khi có thể vì nó giúp template dễ đọc hơn bằng cách phân biệt Vue component với phần tử gốc. Tuy nhiên, việc dùng camelCase khi truyền props không mang lại nhiều lợi ích thực tế, nên ta chọn theo quy ước của từng ngôn ngữ.

### Static vs. Dynamic Props {#static-vs-dynamic-props}

Đến đây, bạn đã thấy props được truyền dưới dạng giá trị tĩnh như sau:

```vue-html
<BlogPost title="My journey with Vue" />
```

Bạn cũng đã thấy props được gán động bằng `v-bind` hoặc dạng viết tắt `:`, như sau:

```vue-html
<!-- Dynamically assign the value of a variable -->
<BlogPost :title="post.title" />

<!-- Dynamically assign the value of a complex expression -->
<BlogPost :title="post.title + ' by ' + post.author.name" />
```

### Passing Different Value Types {#passing-different-value-types}

Trong hai ví dụ trên, ta truyền giá trị chuỗi, nhưng _bất kỳ_ kiểu giá trị nào cũng có thể truyền vào prop.

#### Number {#number}

```vue-html
<!-- Even though `42` is static, we need v-bind to tell Vue that -->
<!-- this is a JavaScript expression rather than a string.       -->
<BlogPost :likes="42" />

<!-- Dynamically assign to the value of a variable. -->
<BlogPost :likes="post.likes" />
```

#### Boolean {#boolean}

```vue-html
<!-- Including the prop with no value will imply `true`. -->
<BlogPost is-published />

<!-- Even though `false` is static, we need v-bind to tell Vue that -->
<!-- this is a JavaScript expression rather than a string.          -->
<BlogPost :is-published="false" />

<!-- Dynamically assign to the value of a variable. -->
<BlogPost :is-published="post.isPublished" />
```

#### Array {#array}

```vue-html
<!-- Even though the array is static, we need v-bind to tell Vue that -->
<!-- this is a JavaScript expression rather than a string.            -->
<BlogPost :comment-ids="[234, 266, 273]" />

<!-- Dynamically assign to the value of a variable. -->
<BlogPost :comment-ids="post.commentIds" />
```

#### Object {#object}

```vue-html
<!-- Even though the object is static, we need v-bind to tell Vue that -->
<!-- this is a JavaScript expression rather than a string.             -->
<BlogPost
  :author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
 />

<!-- Dynamically assign to the value of a variable. -->
<BlogPost :author="post.author" />
```

### Binding Multiple Properties Using an Object {#binding-multiple-properties-using-an-object}

Nếu bạn muốn truyền tất cả thuộc tính của một object làm props, bạn có thể dùng [`v-bind` không có tham số](/guide/essentials/template-syntax#dynamically-binding-multiple-attributes) (`v-bind` thay vì `:prop-name`). Ví dụ, với object `post`:

<div class="options-api">

```js
export default {
  data() {
    return {
      post: {
        id: 1,
        title: 'My Journey with Vue'
      }
    }
  }
}
```

</div>
<div class="composition-api">

```js
const post = {
  id: 1,
  title: 'My Journey with Vue'
}
```

</div>

The following template:

```vue-html
<BlogPost v-bind="post" />
```

Will be equivalent to:

```vue-html
<BlogPost :id="post.id" :title="post.title" />
```

## One-Way Data Flow {#one-way-data-flow}

All props form a **one-way-down binding** between the child property and the parent one: when the parent property updates, it will flow down to the child, but not the other way around. This prevents child components from accidentally mutating the parent's state, which can make your app's data flow harder to understand.

In addition, every time the parent component is updated, all props in the child component will be refreshed with the latest value. This means you should **not** attempt to mutate a prop inside a child component. If you do, Vue will warn you in the console:

<div class="composition-api">

```js
const props = defineProps(['foo'])

// ❌ warning, props are readonly!
props.foo = 'bar'
```

</div>
<div class="options-api">

```js
export default {
  props: ['foo'],
  created() {
    // ❌ warning, props are readonly!
    this.foo = 'bar'
  }
}
```

</div>

There are usually two cases where it's tempting to mutate a prop:

1. **The prop is used to pass in an initial value; the child component wants to use it as a local data property afterwards.** In this case, it's best to define a local data property that uses the prop as its initial value:

   <div class="composition-api">

   ```js
   const props = defineProps(['initialCounter'])

   // counter only uses props.initialCounter as the initial value;
   // it is disconnected from future prop updates.
   const counter = ref(props.initialCounter)
   ```

   </div>
   <div class="options-api">

   ```js
   export default {
     props: ['initialCounter'],
     data() {
       return {
         // counter only uses this.initialCounter as the initial value;
         // it is disconnected from future prop updates.
         counter: this.initialCounter
       }
     }
   }
   ```

   </div>

2. **The prop is passed in as a raw value that needs to be transformed.** In this case, it's best to define a computed property using the prop's value:

   <div class="composition-api">

   ```js
   const props = defineProps(['size'])

   // computed property that auto-updates when the prop changes
   const normalizedSize = computed(() => props.size.trim().toLowerCase())
   ```

   </div>
   <div class="options-api">

   ```js
   export default {
     props: ['size'],
     computed: {
       // computed property that auto-updates when the prop changes
       normalizedSize() {
         return this.size.trim().toLowerCase()
       }
     }
   }
   ```

   </div>

### Mutating Object / Array Props {#mutating-object-array-props}

Khi object và array được truyền làm props, dù child component không thể mutate binding của prop, nó **vẫn** có thể mutate các thuộc tính lồng bên trong của object/array. Lý do là trong JavaScript, object và array được truyền theo tham chiếu, và việc ngăn chặn kiểu mutate này là quá tốn kém đối với Vue.

Hạn chế chính của việc mutate như vậy là nó cho phép child component tác động đến state của parent theo cách parent không nhìn thấy rõ, có thể khiến việc suy luận luồng dữ liệu khó khăn về sau. Theo thực hành tốt, bạn nên tránh mutate như vậy trừ khi parent và child được thiết kế gắn kết chặt chẽ. Trong hầu hết trường hợp, child nên [emit event](/guide/components/events) để parent thực hiện mutate.

## Prop Validation {#prop-validation}

Component có thể chỉ định các yêu cầu cho props của chúng, ví dụ các kiểu dữ liệu như bạn đã thấy. Nếu một yêu cầu không được đáp ứng, Vue sẽ cảnh báo trong console của trình duyệt. Điều này đặc biệt hữu ích khi phát triển component dành cho người khác sử dụng.

Để chỉ định prop validation, bạn có thể cung cấp một object chứa yêu cầu kiểm tra cho <span class="composition-api">macro `defineProps()`</span><span class="options-api">tùy chọn `props`</span> thay vì mảng chuỗi. Ví dụ:

<div class="composition-api">

```js
defineProps({
  // Basic type check
  //  (`null` and `undefined` values will allow any type)
  propA: Number,
  // Multiple possible types
  propB: [String, Number],
  // Required string
  propC: {
    type: String,
    required: true
  },
  // Required but nullable string
  propD: {
    type: [String, null],
    required: true
  },
  // Number with a default value
  propE: {
    type: Number,
    default: 100
  },
  // Object with a default value
  propF: {
    type: Object,
    // Object or array defaults must be returned from
    // a factory function. The function receives the raw
    // props received by the component as the argument.
    default(rawProps) {
      return { message: 'hello' }
    }
  },
  // Custom validator function
  // full props passed as 2nd argument in 3.4+
  propG: {
    validator(value, props) {
      // The value must match one of these strings
      return ['success', 'warning', 'danger'].includes(value)
    }
  },
  // Function with a default value
  propH: {
    type: Function,
    // Unlike object or array default, this is not a factory
    // function - this is a function to serve as a default value
    default() {
      return 'Default function'
    }
  }
})
```

:::tip
Code bên trong đối số `defineProps()` **không thể truy cập biến khác được khai báo trong `<script setup>`**, vì toàn bộ biểu thức được chuyển ra phạm vi hàm bên ngoài khi biên dịch.
:::

</div>
<div class="options-api">

```js
export default {
  props: {
    // Basic type check
    //  (`null` and `undefined` values will allow any type)
    propA: Number,
    // Multiple possible types
    propB: [String, Number],
    // Required string
    propC: {
      type: String,
      required: true
    },
    // Required but nullable string
    propD: {
      type: [String, null],
      required: true
    },
    // Number with a default value
    propE: {
      type: Number,
      default: 100
    },
    // Object with a default value
    propF: {
      type: Object,
      // Object or array defaults must be returned from
      // a factory function. The function receives the raw
      // props received by the component as the argument.
      default(rawProps) {
        return { message: 'hello' }
      }
    },
    // Custom validator function
    // full props passed as 2nd argument in 3.4+
    propG: {
      validator(value, props) {
        // The value must match one of these strings
        return ['success', 'warning', 'danger'].includes(value)
      }
    },
    // Function with a default value
    propH: {
      type: Function,
      // Unlike object or array default, this is not a factory
      // function - this is a function to serve as a default value
      default() {
        return 'Default function'
      }
    }
  }
}
```

</div>

Chi tiết bổ sung:

- Mặc định tất cả props là tùy chọn, trừ khi có `required: true`.

- Một optional prop bị thiếu (khác `Boolean`) sẽ có giá trị `undefined`.

- Prop kiểu `Boolean` khi vắng mặt sẽ được ép thành `false`. Bạn có thể thay đổi bằng cách đặt `default` — ví dụ: `default: undefined` để hành xử như prop không phải Boolean.

- Nếu có chỉ định `default`, giá trị này sẽ được dùng khi prop resolve ra `undefined` — bao gồm cả khi prop vắng mặt hoặc truyền `undefined` một cách tường minh.

Khi prop validation thất bại, Vue sẽ tạo cảnh báo trong console (nếu dùng bản development).

<div class="composition-api">

Nếu dùng [Type-based props declarations](/api/sfc-script-setup#type-only-props-emit-declarations) <sup class="vt-badge ts" />, Vue sẽ cố gắng biên dịch type annotation thành khai báo prop tương đương ở runtime. Ví dụ, `defineProps<{ msg: string }>` sẽ được biên dịch thành `{ msg: { type: String, required: true }}`.

</div>
<div class="options-api">

::: tip Note
Lưu ý props được kiểm tra **trước khi** component instance được tạo, vì vậy các thuộc tính của instance (ví dụ `data`, `computed`, ...) sẽ không khả dụng bên trong hàm `default` hoặc `validator`.
:::

</div>

### Runtime Type Checks {#runtime-type-checks}

`type` có thể là một trong các constructor nguyên thủy sau:

- `String`
- `Number`
- `Boolean`
- `Array`
- `Object`
- `Date`
- `Function`
- `Symbol`
- `Error`

Ngoài ra, `type` cũng có thể là custom class hoặc constructor function, và việc kiểm tra sẽ dùng `instanceof`. Ví dụ, với class sau:

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }
}
```

Bạn có thể dùng nó làm kiểu của prop:

<div class="composition-api">

```js
defineProps({
  author: Person
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    author: Person
  }
}
```

</div>

Vue sẽ dùng `instanceof Person` để kiểm tra liệu giá trị của prop `author` có thực sự là instance của lớp `Person` không.

### Nullable Type {#nullable-type}

Nếu kiểu là bắt buộc nhưng có thể null, bạn có thể dùng cú pháp mảng bao gồm `null`:

<div class="composition-api">

```js
defineProps({
  id: {
    type: [String, null],
    required: true
  }
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    id: {
      type: [String, null],
      required: true
    }
  }
}
```

</div>

Lưu ý nếu `type` chỉ là `null` mà không dùng cú pháp mảng, nó sẽ cho phép mọi kiểu.

## Boolean Casting {#boolean-casting}

Props kiểu `Boolean` có quy tắc ép kiểu đặc biệt để mô phỏng hành vi của boolean attribute gốc. Với `<MyComponent>` có khai báo sau:

<div class="composition-api">

```js
defineProps({
  disabled: Boolean
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    disabled: Boolean
  }
}
```

</div>

Component có thể dùng như sau:

```vue-html
<!-- equivalent of passing :disabled="true" -->
<MyComponent disabled />

<!-- equivalent of passing :disabled="false" -->
<MyComponent />
```

Khi một prop được khai báo cho phép nhiều kiểu, quy tắc ép kiểu cho `Boolean` cũng được áp dụng. Tuy nhiên có một trường hợp biên khi cả `String` và `Boolean` đều được phép — quy tắc ép kiểu Boolean chỉ áp dụng nếu Boolean xuất hiện trước String:

<div class="composition-api">

```js
// disabled will be casted to true
defineProps({
  disabled: [Boolean, Number]
})

// disabled will be casted to true
defineProps({
  disabled: [Boolean, String]
})

// disabled will be casted to true
defineProps({
  disabled: [Number, Boolean]
})

// disabled will be parsed as an empty string (disabled="")
defineProps({
  disabled: [String, Boolean]
})
```

</div>
<div class="options-api">

```js
// disabled will be casted to true
export default {
  props: {
    disabled: [Boolean, Number]
  }
}

// disabled will be casted to true
export default {
  props: {
    disabled: [Boolean, String]
  }
}

// disabled will be casted to true
export default {
  props: {
    disabled: [Number, Boolean]
  }
}

// disabled will be parsed as an empty string (disabled="")
export default {
  props: {
    disabled: [String, Boolean]
  }
}
```

</div>
