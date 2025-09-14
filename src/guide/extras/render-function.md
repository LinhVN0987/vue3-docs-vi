---
outline: deep
---

# Render Functions & JSX {#render-functions-jsx}

Vue khuyến nghị dùng template trong đa số trường hợp. Tuy nhiên, có tình huống cần toàn bộ sức mạnh lập trình của JavaScript — đó là lúc dùng **render function**.

> Nếu bạn mới với khái niệm virtual DOM và render function, hãy đọc trước [Rendering Mechanism](/guide/extras/rendering-mechanism).

## Basic Usage {#basic-usage}

### Creating Vnodes {#creating-vnodes}

Vue cung cấp hàm `h()` để tạo vnode:

```js
import { h } from 'vue'

const vnode = h(
  'div', // type
  { id: 'foo', class: 'bar' }, // props
  [
    /* children */
  ]
)
```

`h()` là viết tắt của **hyperscript** — “JavaScript tạo ra HTML”. Tên này theo quy ước chung của nhiều hiện thực virtual DOM. Tên mô tả hơn có thể là `createVNode()`, nhưng tên ngắn hữu ích khi gọi nhiều lần trong render function.

Hàm `h()` được thiết kế rất linh hoạt:

```js
// all arguments except the type are optional
h('div')
h('div', { id: 'foo' })

// both attributes and properties can be used in props
// Vue automatically picks the right way to assign it
h('div', { class: 'bar', innerHTML: 'hello' })

// props modifiers such as `.prop` and `.attr` can be added
// with `.` and `^` prefixes respectively
h('div', { '.name': 'some-name', '^width': '100' })

// class and style have the same object / array
// value support that they have in templates
h('div', { class: [foo, { bar }], style: { color: 'red' } })

// event listeners should be passed as onXxx
h('div', { onClick: () => {} })

// children can be a string
h('div', { id: 'foo' }, 'hello')

// props can be omitted when there are no props
h('div', 'hello')
h('div', [h('span', 'hello')])

// children array can contain mixed vnodes and strings
h('div', ['hello', h('span', 'hello')])
```

Vnode tạo ra có dạng:

```js
const vnode = h('div', { id: 'foo' }, [])

vnode.type // 'div'
vnode.props // { id: 'foo' }
vnode.children // []
vnode.key // null
```

:::warning Note
`VNode` đầy đủ có nhiều thuộc tính nội bộ khác, nhưng khuyến nghị không dựa vào thuộc tính ngoài những cái liệt kê ở đây để tránh hỏng khi thuộc tính nội bộ thay đổi.
:::

### Declaring Render Functions {#declaring-render-functions}

<div class="composition-api">

Khi dùng template với Composition API, giá trị trả về của `setup()` dùng để expose dữ liệu cho template. Với render function, ta trả về chính render function:

```js
import { ref, h } from 'vue'

export default {
  props: {
    /* ... */
  },
  setup(props) {
    const count = ref(1)

    // return the render function
    return () => h('div', props.msg + count.value)
  }
}
```

Render function được khai báo trong `setup()` nên tự nhiên truy cập được props và state reactive cùng scope.

Ngoài trả về một vnode, bạn có thể trả về chuỗi hoặc mảng:

```js
export default {
  setup() {
    return () => 'hello world!'
  }
}
```

```js
import { h } from 'vue'

export default {
  setup() {
    // use an array to return multiple root nodes
    return () => [
      h('div'),
      h('div'),
      h('div')
    ]
  }
}
```

:::tip
Hãy trả về một hàm chứ không trả về trực tiếp giá trị! `setup()` chỉ được gọi một lần mỗi component, trong khi render function được gọi nhiều lần.
:::

</div>
<div class="options-api">

Ta có thể khai báo render function bằng tùy chọn `render`:

```js
import { h } from 'vue'

export default {
  data() {
    return {
      msg: 'hello'
    }
  },
  render() {
    return h('div', this.msg)
  }
}
```

Hàm `render()` có thể truy cập instance component qua `this`.

Ngoài trả về một vnode, bạn cũng có thể trả về chuỗi hoặc mảng:

```js
export default {
  render() {
    return 'hello world!'
  }
}
```

```js
import { h } from 'vue'

export default {
  render() {
    // use an array to return multiple root nodes
    return [
      h('div'),
      h('div'),
      h('div')
    ]
  }
}
```

</div>

Nếu một component render function không cần state instance, có thể khai báo trực tiếp dưới dạng hàm cho gọn:

```js
function Hello() {
  return 'hello world!'
}
```

Đúng vậy, đây là một Vue component hợp lệ! Xem [Functional Components](#functional-components) để biết chi tiết về cú pháp này.
### Vnodes Must Be Unique {#vnodes-must-be-unique}

Tất cả vnode trong cây component phải duy nhất. Nghĩa là render function sau là không hợp lệ:

```js
function render() {
  const p = h('p', 'hi')
  return h('div', [
    // Yikes - duplicate vnodes!
    p,
    p
  ])
}
```

Nếu bạn thực sự muốn lặp cùng phần tử/component nhiều lần, hãy dùng một hàm factory. Ví dụ, render function sau hợp lệ để render 20 đoạn văn giống hệt:

```js
function render() {
  return h(
    'div',
    Array.from({ length: 20 }).map(() => {
      return h('p', 'hi')
    })
  )
}
```

## JSX / TSX {#jsx-tsx}

[JSX](https://facebook.github.io/jsx/) là phần mở rộng giống XML cho JavaScript cho phép viết code như sau:

```jsx
const vnode = <div>hello</div>
```

Bên trong biểu thức JSX, dùng ngoặc nhọn để chèn giá trị động:

```jsx
const vnode = <div id={dynamicId}>hello, {userName}</div>
```

`create-vue` và Vue CLI đều có tuỳ chọn scaffold dự án với hỗ trợ JSX cấu hình sẵn. Nếu cấu hình JSX thủ công, tham khảo tài liệu [`@vue/babel-plugin-jsx`](https://github.com/vuejs/jsx-next).

Mặc dù được giới thiệu bởi React, JSX thực chất không có ngữ nghĩa runtime xác định và có thể được biên dịch thành nhiều đầu ra khác nhau. Nếu bạn từng dùng JSX, lưu ý **JSX transform của Vue khác với React**, nên không thể dùng transform của React trong ứng dụng Vue. Một số khác biệt đáng chú ý so với React JSX:

- Có thể dùng thuộc tính HTML như `class` và `for` làm prop — không cần `className` hay `htmlFor`.
- Truyền children cho component (tức slot) [hoạt động khác](#passing-slots).

Định nghĩa kiểu của Vue cũng cung cấp suy luận kiểu cho TSX. Khi dùng TSX, đảm bảo đặt `"jsx": "preserve"` trong `tsconfig.json` để TypeScript giữ nguyên cú pháp JSX cho Vue xử lý.

### Suy luận kiểu JSX {#jsx-type-inference}

Tương tự transform, JSX của Vue cũng cần định nghĩa kiểu khác.

Bắt đầu từ Vue 3.4, Vue không còn đăng ký ngầm namespace toàn cục `JSX`. Để TypeScript dùng định nghĩa kiểu JSX của Vue, thêm vào `tsconfig.json`:

```json
{
  "compilerOptions": {
    "jsx": "preserve",
    "jsxImportSource": "vue"
    // ...
  }
}
```

Bạn cũng có thể bật theo từng file bằng cách thêm dòng `/* @jsxImportSource vue */` ở đầu file.

Nếu có code phụ thuộc vào namespace toàn cục `JSX`, bạn có thể giữ nguyên hành vi trước 3.4 bằng cách import hoặc reference `vue/jsx` trong dự án để đăng ký namespace `JSX` toàn cục.

## Render Function Recipes {#render-function-recipes}

Bên dưới là một số công thức phổ biến để hiện thực tính năng template bằng render function / JSX tương đương.

### `v-if` {#v-if}

Template:

```vue-html
<div>
  <div v-if="ok">yes</div>
  <span v-else>no</span>
</div>
```

Equivalent render function / JSX:

<div class="composition-api">

```js
h('div', [ok.value ? h('div', 'yes') : h('span', 'no')])
```

```jsx
<div>{ok.value ? <div>yes</div> : <span>no</span>}</div>
```

</div>
<div class="options-api">

```js
h('div', [this.ok ? h('div', 'yes') : h('span', 'no')])
```

```jsx
<div>{this.ok ? <div>yes</div> : <span>no</span>}</div>
```

</div>

### `v-for` {#v-for}

Template:

```vue-html
<ul>
  <li v-for="{ id, text } in items" :key="id">
    {{ text }}
  </li>
</ul>
```

Equivalent render function / JSX:

<div class="composition-api">

```js
h(
  'ul',
  // assuming `items` is a ref with array value
  items.value.map(({ id, text }) => {
    return h('li', { key: id }, text)
  })
)
```

```jsx
<ul>
  {items.value.map(({ id, text }) => {
    return <li key={id}>{text}</li>
  })}
</ul>
```

</div>
<div class="options-api">

```js
h(
  'ul',
  this.items.map(({ id, text }) => {
    return h('li', { key: id }, text)
  })
)
```

```jsx
<ul>
  {this.items.map(({ id, text }) => {
    return <li key={id}>{text}</li>
  })}
</ul>
```

</div>

### `v-on` {#v-on}

Prop có tên bắt đầu bằng `on` theo sau là chữ hoa sẽ được coi là listener sự kiện. Ví dụ, `onClick` tương đương `@click` trong template.

```js
h(
  'button',
  {
    onClick(event) {
      /* ... */
    }
  },
  'Click Me'
)
```

```jsx
<button
  onClick={(event) => {
    /* ... */
  }}
>
  Click Me
</button>
```

#### Event Modifiers {#event-modifiers}

Với các modifier `.passive`, `.capture`, và `.once`, bạn có thể nối sau tên sự kiện bằng camelCase.

For example:

```js
h('input', {
  onClickCapture() {
    /* listener in capture mode */
  },
  onKeyupOnce() {
    /* triggers only once */
  },
  onMouseoverOnceCapture() {
    /* once + capture */
  }
})
```

```jsx
<input
  onClickCapture={() => {}}
  onKeyupOnce={() => {}}
  onMouseoverOnceCapture={() => {}}
/>
```

Với các event modifier và key modifier khác, có thể dùng helper [`withModifiers`](/api/render-function#withmodifiers):

```js
import { withModifiers } from 'vue'

h('div', {
  onClick: withModifiers(() => {}, ['self'])
})
```

```jsx
<div onClick={withModifiers(() => {}, ['self'])} />
```

### Components {#components}

Để tạo vnode cho component, tham số đầu tiên truyền cho `h()` phải là định nghĩa component. Nghĩa là khi dùng render function, không cần đăng ký component — bạn có thể dùng trực tiếp các component đã import:

```js
import Foo from './Foo.vue'
import Bar from './Bar.jsx'

function render() {
  return h('div', [h(Foo), h(Bar)])
}
```

```jsx
function render() {
  return (
    <div>
      <Foo />
      <Bar />
    </div>
  )
}
```

Như bạn thấy, `h` hoạt động với component import từ bất kỳ định dạng file nào miễn là Vue component hợp lệ.

Dynamic component rất trực quan với render function:

```js
import Foo from './Foo.vue'
import Bar from './Bar.jsx'

function render() {
  return ok.value ? h(Foo) : h(Bar)
}
```

```jsx
function render() {
  return ok.value ? <Foo /> : <Bar />
}
```

Nếu một component chỉ được đăng ký theo tên và không thể import trực tiếp (ví dụ đăng ký toàn cục bởi thư viện), có thể resolve bằng helper [`resolveComponent()`](/api/render-function#resolvecomponent).

### Rendering Slots {#rendering-slots}

<div class="composition-api">

Trong render function, slot có thể truy cập từ context `setup()`. Mỗi slot trên đối tượng `slots` là một **hàm trả về mảng vnode**:

```js
export default {
  props: ['message'],
  setup(props, { slots }) {
    return () => [
      // default slot:
      // <div><slot /></div>
      h('div', slots.default()),

      // named slot:
      // <div><slot name="footer" :text="message" /></div>
      h(
        'div',
        slots.footer({
          text: props.message
        })
      )
    ]
  }
}
```

Tương đương JSX:

```jsx
// default
<div>{slots.default()}</div>

// named
<div>{slots.footer({ text: props.message })}</div>
```

</div>
<div class="options-api">

In render functions, slots can be accessed from [`this.$slots`](/api/component-instance#slots):

```js
export default {
  props: ['message'],
  render() {
    return [
      // <div><slot /></div>
      h('div', this.$slots.default()),

      // <div><slot name="footer" :text="message" /></div>
      h(
        'div',
        this.$slots.footer({
          text: this.message
        })
      )
    ]
  }
}
```

JSX equivalent:

```jsx
// <div><slot /></div>
<div>{this.$slots.default()}</div>

// <div><slot name="footer" :text="message" /></div>
<div>{this.$slots.footer({ text: this.message })}</div>
```

</div>

### Passing Slots {#passing-slots}

Passing children to components works a bit differently from passing children to elements. Instead of an array, we need to pass either a slot function, or an object of slot functions. Slot functions can return anything a normal render function can return - which will always be normalized to arrays of vnodes when accessed in the child component.

```js
// single default slot
h(MyComponent, () => 'hello')

// named slots
// notice the `null` is required to avoid
// the slots object being treated as props
h(MyComponent, null, {
  default: () => 'default slot',
  foo: () => h('div', 'foo'),
  bar: () => [h('span', 'one'), h('span', 'two')]
})
```

JSX equivalent:

```jsx
// default
<MyComponent>{() => 'hello'}</MyComponent>

// named
<MyComponent>{{
  default: () => 'default slot',
  foo: () => <div>foo</div>,
  bar: () => [<span>one</span>, <span>two</span>]
}}</MyComponent>
```

Passing slots as functions allows them to be invoked lazily by the child component. This leads to the slot's dependencies being tracked by the child instead of the parent, which results in more accurate and efficient updates.

### Scoped Slots {#scoped-slots}

To render a scoped slot in the parent component, a slot is passed to the child. Notice how the slot now has a parameter `text`. The slot will be called in the child component and the data from the child component will be passed up to the parent component.

```js
// parent component
export default {
  setup() {
    return () => h(MyComp, null, {
      default: ({ text }) => h('p', text)
    })
  }
}
```

Remember to pass `null` so the slots will not be treated as props.

```js
// child component
export default {
  setup(props, { slots }) {
    const text = ref('hi')
    return () => h('div', null, slots.default({ text: text.value }))
  }
}
```

JSX equivalent:

```jsx
<MyComponent>{{
  default: ({ text }) => <p>{ text }</p>  
}}</MyComponent>
```

### Built-in Components {#built-in-components}

[Built-in components](/api/built-in-components) such as `<KeepAlive>`, `<Transition>`, `<TransitionGroup>`, `<Teleport>` and `<Suspense>` must be imported for use in render functions:

<div class="composition-api">

```js
import { h, KeepAlive, Teleport, Transition, TransitionGroup } from 'vue'

export default {
  setup () {
    return () => h(Transition, { mode: 'out-in' }, /* ... */)
  }
}
```

</div>
<div class="options-api">

```js
import { h, KeepAlive, Teleport, Transition, TransitionGroup } from 'vue'

export default {
  render () {
    return h(Transition, { mode: 'out-in' }, /* ... */)
  }
}
```

</div>

### `v-model` {#v-model}

The `v-model` directive is expanded to `modelValue` and `onUpdate:modelValue` props during template compilation—we will have to provide these props ourselves:

<div class="composition-api">

```js
export default {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    return () =>
      h(SomeComponent, {
        modelValue: props.modelValue,
        'onUpdate:modelValue': (value) => emit('update:modelValue', value)
      })
  }
}
```

</div>
<div class="options-api">

```js
export default {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  render() {
    return h(SomeComponent, {
      modelValue: this.modelValue,
      'onUpdate:modelValue': (value) => this.$emit('update:modelValue', value)
    })
  }
}
```

</div>

### Custom Directives {#custom-directives}

Custom directives can be applied to a vnode using [`withDirectives`](/api/render-function#withdirectives):

```js
import { h, withDirectives } from 'vue'

// a custom directive
const pin = {
  mounted() { /* ... */ },
  updated() { /* ... */ }
}

// <div v-pin:top.animate="200"></div>
const vnode = withDirectives(h('div'), [
  [pin, 200, 'top', { animate: true }]
])
```

If the directive is registered by name and cannot be imported directly, it can be resolved using the [`resolveDirective`](/api/render-function#resolvedirective) helper.

### Template Refs {#template-refs}

<div class="composition-api">

With the Composition API, when using [`useTemplateRef()`](/api/composition-api-helpers#usetemplateref) <sup class="vt-badge" data-text="3.5+" />  template refs are created by passing the string value as prop to the vnode:

```js
import { h, useTemplateRef } from 'vue'

export default {
  setup() {
    const divEl = useTemplateRef('my-div')

    // <div ref="my-div">
    return () => h('div', { ref: 'my-div' })
  }
}
```

<details>
<summary>Usage before 3.5</summary>

In versions before 3.5 where useTemplateRef() was not introduced, template refs are created by passing the ref() itself as a prop to the vnode:

```js
import { h, ref } from 'vue'

export default {
  setup() {
    const divEl = ref()

    // <div ref="divEl">
    return () => h('div', { ref: divEl })
  }
}
```
</details>
</div>
<div class="options-api">

With the Options API, template refs are created by passing the ref name as a string in the vnode props:

```js
export default {
  render() {
    // <div ref="divEl">
    return h('div', { ref: 'divEl' })
  }
}
```

</div>

## Functional Components {#functional-components}

Functional components are an alternative form of component that don't have any state of their own. They act like pure functions: props in, vnodes out. They are rendered without creating a component instance (i.e. no `this`), and without the usual component lifecycle hooks.

To create a functional component we use a plain function, rather than an options object. The function is effectively the `render` function for the component.

<div class="composition-api">

The signature of a functional component is the same as the `setup()` hook:

```js
function MyComponent(props, { slots, emit, attrs }) {
  // ...
}
```

</div>
<div class="options-api">

As there is no `this` reference for a functional component, Vue will pass in the `props` as the first argument:

```js
function MyComponent(props, context) {
  // ...
}
```

The second argument, `context`, contains three properties: `attrs`, `emit`, and `slots`. These are equivalent to the instance properties [`$attrs`](/api/component-instance#attrs), [`$emit`](/api/component-instance#emit), and [`$slots`](/api/component-instance#slots) respectively.

</div>

Most of the usual configuration options for components are not available for functional components. However, it is possible to define [`props`](/api/options-state#props) and [`emits`](/api/options-state#emits) by adding them as properties:

```js
MyComponent.props = ['value']
MyComponent.emits = ['click']
```

If the `props` option is not specified, then the `props` object passed to the function will contain all attributes, the same as `attrs`. The prop names will not be normalized to camelCase unless the `props` option is specified.

For functional components with explicit `props`, [attribute fallthrough](/guide/components/attrs) works much the same as with normal components. However, for functional components that don't explicitly specify their `props`, only the `class`, `style`, and `onXxx` event listeners will be inherited from the `attrs` by default. In either case, `inheritAttrs` can be set to `false` to disable attribute inheritance:

```js
MyComponent.inheritAttrs = false
```

Functional components can be registered and consumed just like normal components. If you pass a function as the first argument to `h()`, it will be treated as a functional component.

### Typing Functional Components<sup class="vt-badge ts" /> {#typing-functional-components}

Functional Components can be typed based on whether they are named or anonymous. [Vue - Official extension](https://github.com/vuejs/language-tools) also supports type checking properly typed functional components when consuming them in SFC templates.

**Named Functional Component**

```tsx
import type { SetupContext } from 'vue'
type FComponentProps = {
  message: string
}

type Events = {
  sendMessage(message: string): void
}

function FComponent(
  props: FComponentProps,
  context: SetupContext<Events>
) {
  return (
    <button onClick={() => context.emit('sendMessage', props.message)}>
        {props.message} {' '}
    </button>
  )
}

FComponent.props = {
  message: {
    type: String,
    required: true
  }
}

FComponent.emits = {
  sendMessage: (value: unknown) => typeof value === 'string'
}
```

**Anonymous Functional Component**

```tsx
import type { FunctionalComponent } from 'vue'

type FComponentProps = {
  message: string
}

type Events = {
  sendMessage(message: string): void
}

const FComponent: FunctionalComponent<FComponentProps, Events> = (
  props,
  context
) => {
  return (
    <button onClick={() => context.emit('sendMessage', props.message)}>
        {props.message} {' '}
    </button>
  )
}

FComponent.props = {
  message: {
    type: String,
    required: true
  }
}

FComponent.emits = {
  sendMessage: (value) => typeof value === 'string'
}
```
