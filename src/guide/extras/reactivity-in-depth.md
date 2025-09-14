---
outline: deep
---

<script setup>
import SpreadSheet from './demos/SpreadSheet.vue'
</script>

# Reactivity in Depth {#reactivity-in-depth}

Một trong những đặc trưng nổi bật của Vue là hệ thống reactivity tinh gọn. State của component là các object JavaScript reactive. Khi bạn sửa chúng, view được cập nhật. Điều này khiến quản lý state đơn giản và trực quan, nhưng cũng quan trọng để hiểu cách hoạt động nhằm tránh một số lỗi thường gặp. Phần này đi sâu vào chi tiết tầng thấp của hệ thống reactivity của Vue.

## What is Reactivity? {#what-is-reactivity}

Thuật ngữ này xuất hiện khá nhiều gần đây, nhưng ý nghĩa là gì? Reactivity là một mô hình lập trình cho phép ta phản ứng với thay đổi một cách khai báo. Ví dụ kinh điển là bảng tính Excel:

<SpreadSheet />

Ô A2 được định nghĩa bằng công thức `= A0 + A1` (bạn có thể nhấp vào A2 để xem/sửa), nên bảng cho ra 3. Nhưng nếu bạn cập nhật A0 hoặc A1, A2 cũng tự động cập nhật theo.

JavaScript thường không hoạt động như vậy. Nếu viết một ví dụ tương đương trong JavaScript:

```js
let A0 = 1
let A1 = 2
let A2 = A0 + A1

console.log(A2) // 3

A0 = 2
console.log(A2) // Still 3
```

Khi chúng ta thay đổi `A0`, `A2` sẽ không tự động thay đổi.

Vậy làm sao trong JavaScript? Đầu tiên, để chạy lại đoạn cập nhật `A2`, hãy bọc nó trong một hàm:

```js
let A2

function update() {
  A2 = A0 + A1
}
```

Tiếp theo, ta định nghĩa vài thuật ngữ:

- Hàm `update()` tạo ra một **side effect** (viết ngắn là **effect**), vì nó thay đổi trạng thái của chương trình.

- `A0` và `A1` được coi là **phụ thuộc** (dependencies) của effect, vì giá trị của chúng được dùng để thực thi effect. Nói cách khác, effect được xem là **subscriber** của các phụ thuộc của nó.

Ta cần một hàm “ma thuật” gọi `update()` ( **effect** ) bất cứ khi nào `A0` hoặc `A1` ( **dependencies** ) thay đổi:

```js
whenDepsChange(update)
```

Hàm `whenDepsChange()` có nhiệm vụ:

1. Theo dõi khi một biến được đọc. Ví dụ, khi đánh giá biểu thức `A0 + A1`, cả `A0` và `A1` đều được đọc.

2. Nếu một biến được đọc trong khi đang có effect chạy, hãy thêm effect đó làm subscriber của biến. Ví dụ, vì `A0` và `A1` được đọc khi `update()` đang thực thi, sau lần gọi đầu tiên `update()` trở thành subscriber của cả `A0` và `A1`.

3. Phát hiện khi một biến bị thay đổi. Ví dụ, khi gán giá trị mới cho `A0`, hãy thông báo cho tất cả effect subscriber của nó chạy lại.

## How Reactivity Works in Vue {#how-reactivity-works-in-vue}

Chúng ta không thể theo dõi đọc/ghi biến cục bộ như ví dụ — JavaScript thuần không có cơ chế đó. Nhưng ta **có thể** chặn việc đọc/ghi **thuộc tính của object**.

Có hai cách để can thiệp việc truy cập thuộc tính trong JavaScript: [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get#description)/[setter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set#description) và [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy). Vue 2 chỉ dùng getter/setter do hạn chế hỗ trợ trình duyệt. Ở Vue 3, Proxy được dùng cho các object reactive và getter/setter được dùng cho ref. Dưới đây là pseudo-code minh hoạ cách chúng hoạt động:

```js{4,9,17,22}
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key)
      return target[key]
    },
    set(target, key, value) {
      target[key] = value
      trigger(target, key)
    }
  })
}

function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value')
      return value
    },
    set value(newValue) {
      value = newValue
      trigger(refObject, 'value')
    }
  }
  return refObject
}
```

:::tip
Các đoạn code ở đây nhằm giải thích khái niệm cốt lõi đơn giản nhất nên lược bỏ nhiều chi tiết và không xét edge case.
:::

Điều này giải thích một vài [giới hạn của reactive object](/guide/essentials/reactivity-fundamentals#limitations-of-reactive) đã bàn ở phần cơ bản:

- Khi bạn gán hoặc destructure một thuộc tính của object reactive ra biến cục bộ, việc truy cập/gán qua biến đó là không reactive vì nó không còn kích hoạt bẫy get/set của proxy trên object nguồn. Lưu ý “mất kết nối” này chỉ ảnh hưởng tới binding của biến — nếu biến trỏ tới giá trị không nguyên thuỷ như object, thay đổi object đó vẫn là reactive.

- Proxy trả về từ `reactive()`, dù hành xử giống như bản gốc, vẫn có danh tính khác nếu so sánh với bản gốc bằng toán tử `===`.

Bên trong `track()`, chúng ta kiểm tra xem có effect nào đang chạy hay không. Nếu có, ta tra cứu các effect subscriber (được lưu trong một Set) cho thuộc tính đang được theo dõi, và thêm effect vào Set:

```js
// This will be set right before an effect is about
// to be run. We'll deal with this later.
let activeEffect

function track(target, key) {
  if (activeEffect) {
    const effects = getSubscribersForProperty(target, key)
    effects.add(activeEffect)
  }
}
```

Effect subscriptions được lưu trong một cấu trúc dữ liệu toàn cục `WeakMap<target, Map<key, Set<effect>>>`. Nếu không tìm thấy Set effect đã đăng ký cho một thuộc tính (lần đầu được theo dõi), nó sẽ được tạo. Nói ngắn gọn, đó là những gì hàm `getSubscribersForProperty()` thực hiện. Để đơn giản, ta bỏ qua chi tiết.

Bên trong `trigger()`, chúng ta lại tra cứu các subscriber effect của thuộc tính, nhưng lần này là để gọi chúng:

```js
function trigger(target, key) {
  const effects = getSubscribersForProperty(target, key)
  effects.forEach((effect) => effect())
}
```

Giờ hãy quay lại hàm `whenDepsChange()`:

```js
function whenDepsChange(update) {
  const effect = () => {
    activeEffect = effect
    update()
    activeEffect = null
  }
  effect()
}
```

Hàm này bọc `update` gốc trong một effect đặt chính nó làm effect đang hoạt động trước khi chạy cập nhật thực sự. Nhờ vậy, các lời gọi `track()` trong quá trình cập nhật có thể tìm thấy effect hiện hành.

Tại thời điểm này, chúng ta đã tạo ra một effect tự động theo dõi các phụ thuộc của nó và chạy lại bất cứ khi nào một phụ thuộc thay đổi. Chúng ta gọi nó là **Reactive Effect**.

Vue cung cấp API giúp bạn tạo reactive effect: [`watchEffect()`](/api/reactivity-core#watcheffect). Thực tế, bạn có thể nhận thấy nó hoạt động rất giống với `whenDepsChange()` “ma thuật” trong ví dụ. Giờ ta có thể viết lại ví dụ ban đầu bằng các API thực sự của Vue:

```js
import { ref, watchEffect } from 'vue'

const A0 = ref(0)
const A1 = ref(1)
const A2 = ref()

watchEffect(() => {
  // tracks A0 and A1
  A2.value = A0.value + A1.value
})

// triggers the effect
A0.value = 2
```

Dùng reactive effect để thay đổi một ref không phải trường hợp thú vị nhất — thực tế dùng computed sẽ khai báo hơn:

```js
import { ref, computed } from 'vue'

const A0 = ref(0)
const A1 = ref(1)
const A2 = computed(() => A0.value + A1.value)

A0.value = 2
```

Bên trong, `computed` quản lý việc vô hiệu hoá và tính toán lại thông qua một reactive effect.

Vậy ví dụ phổ biến và hữu ích của reactive effect là gì? Chính là cập nhật DOM! Ta có thể hiện thực “reactive rendering” đơn giản như sau:

```js
import { ref, watchEffect } from 'vue'

const count = ref(0)

watchEffect(() => {
  document.body.innerHTML = `Count is: ${count.value}`
})

// updates the DOM
count.value++
```

Thực tế, đây khá gần với cách một component Vue giữ state và DOM đồng bộ — mỗi instance component tạo một reactive effect để render và cập nhật DOM. Dĩ nhiên, component Vue dùng cách cập nhật DOM hiệu quả hơn nhiều so với `innerHTML`. Vấn đề này được bàn trong [Rendering Mechanism](./rendering-mechanism).

<div class="options-api">

Các API `ref()`, `computed()` và `watchEffect()` đều thuộc Composition API. Nếu trước giờ bạn chỉ dùng Options API với Vue, bạn sẽ thấy Composition API gần với cách hệ reactivity của Vue vận hành bên dưới hơn. Thực tế, ở Vue 3, Options API được hiện thực trên nền Composition API. Mọi truy cập thuộc tính trên instance component (`this`) đều kích hoạt getter/setter để theo dõi reactivity, và các option như `watch` và `computed` sẽ gọi tới phiên bản tương đương của Composition API ở bên trong.

</div>

## Runtime vs. Compile-time Reactivity {#runtime-vs-compile-time-reactivity}

Hệ reactivity của Vue chủ yếu dựa trên runtime: việc theo dõi và kích hoạt đều diễn ra khi mã chạy trực tiếp trong trình duyệt. Ưu điểm của reactivity ở runtime là có thể hoạt động mà không cần build step và có ít edge case hơn. Mặt khác, nó bị giới hạn bởi hạn chế cú pháp của JavaScript, dẫn đến nhu cầu các “hộp chứa giá trị” như ref của Vue.

Một số framework như [Svelte](https://svelte.dev/) chọn vượt qua các hạn chế này bằng cách hiện thực reactivity trong lúc biên dịch. Nó phân tích và biến đổi mã để mô phỏng reactivity. Bước biên dịch cho phép framework thay đổi ngữ nghĩa của chính JavaScript — ví dụ, chèn ngầm code phân tích phụ thuộc và kích hoạt effect quanh việc truy cập biến cục bộ. Nhược điểm là các biến đổi này cần build step, và việc thay đổi ngữ nghĩa JavaScript về bản chất là tạo ra một ngôn ngữ trông giống JavaScript nhưng biên dịch ra thứ khác.

Đội ngũ Vue đã thử hướng này thông qua một tính năng thử nghiệm gọi là [Reactivity Transform](/guide/extras/reactivity-transform), nhưng cuối cùng chúng tôi quyết định nó không phù hợp với dự án vì [the reasoning here](https://github.com/vuejs/rfcs/discussions/369#discussioncomment-5059028).

## Gỡ lỗi Reactivity {#reactivity-debugging}

Thật tuyệt khi hệ reactivity của Vue tự động theo dõi phụ thuộc, nhưng đôi khi chúng ta muốn biết chính xác cái gì đang được theo dõi, hoặc điều gì khiến một component render lại.

### Component Debugging Hooks {#component-debugging-hooks}

Chúng ta có thể debug các phụ thuộc được dùng trong lúc render của component và phụ thuộc nào kích hoạt cập nhật bằng các hook vòng đời <span class="options-api">`renderTracked`</span><span class="composition-api">`onRenderTracked`</span> và <span class="options-api">`renderTriggered`</span><span class="composition-api">`onRenderTriggered`</span>. Cả hai hook sẽ nhận một sự kiện debugger chứa thông tin về phụ thuộc liên quan. Khuyến nghị đặt lệnh `debugger` trong callback để kiểm tra tương tác phụ thuộc:

<div class="composition-api">

```vue
<script setup>
import { onRenderTracked, onRenderTriggered } from 'vue'

onRenderTracked((event) => {
  debugger
})

onRenderTriggered((event) => {
  debugger
})
</script>
```

</div>
<div class="options-api">

```js
export default {
  renderTracked(event) {
    debugger
  },
  renderTriggered(event) {
    debugger
  }
}
```

</div>

:::tip
Các hook debug của component chỉ hoạt động ở chế độ phát triển.
:::

Đối tượng sự kiện debug có kiểu như sau:

<span id="debugger-event"></span>

```ts
type DebuggerEvent = {
  effect: ReactiveEffect
  target: object
  type:
    | TrackOpTypes /* 'get' | 'has' | 'iterate' */
    | TriggerOpTypes /* 'set' | 'add' | 'delete' | 'clear' */
  key: any
  newValue?: any
  oldValue?: any
  oldTarget?: Map<any, any> | Set<any>
}
```

### Gỡ lỗi computed {#computed-debugging}

<!-- TODO options API equivalent -->

Chúng ta có thể debug computed bằng cách truyền cho `computed()` một đối tượng tuỳ chọn thứ hai với các callback `onTrack` và `onTrigger`:

- `onTrack` sẽ được gọi khi một thuộc tính reactive hoặc ref được theo dõi như một phụ thuộc.
- `onTrigger` sẽ được gọi khi callback của watcher được kích hoạt bởi việc thay đổi một phụ thuộc.

Cả hai callback sẽ nhận sự kiện debugger theo [cùng định dạng](#debugger-event) như các hook debug của component:

```js
const plusOne = computed(() => count.value + 1, {
  onTrack(e) {
    // triggered when count.value is tracked as a dependency
    debugger
  },
  onTrigger(e) {
    // triggered when count.value is mutated
    debugger
  }
})

// access plusOne, should trigger onTrack
console.log(plusOne.value)

// mutate count.value, should trigger onTrigger
count.value++
```

:::tip
Tùy chọn `onTrack` và `onTrigger` cho computed chỉ hoạt động ở chế độ phát triển.
:::

### Watcher Debugging {#watcher-debugging}

<!-- TODO options API equivalent -->

Tương tự `computed()`, các watcher cũng hỗ trợ các tuỳ chọn `onTrack` và `onTrigger`:

```js
watch(source, callback, {
  onTrack(e) {
    debugger
  },
  onTrigger(e) {
    debugger
  }
})

watchEffect(callback, {
  onTrack(e) {
    debugger
  },
  onTrigger(e) {
    debugger
  }
})
```

:::tip
Các tuỳ chọn `onTrack` và `onTrigger` cho watcher chỉ hoạt động ở chế độ phát triển.
:::

## Tích hợp với hệ thống state bên ngoài {#integration-with-external-state-systems}

Hệ reactivity của Vue hoạt động bằng cách chuyển đổi sâu (deeply convert) các object JavaScript thuần thành các proxy reactive. Việc chuyển đổi sâu này có thể không cần thiết hoặc đôi khi không mong muốn khi tích hợp với hệ thống quản lý state bên ngoài (ví dụ nếu giải pháp bên ngoài cũng dùng Proxy).

Ý tưởng chung để tích hợp hệ reactivity của Vue với một giải pháp quản lý state bên ngoài là giữ state bên ngoài trong một [`shallowRef`](/api/reactivity-advanced#shallowref). Shallow ref chỉ reactive khi thuộc tính `.value` của nó được truy cập — giá trị bên trong được giữ nguyên. Khi state bên ngoài thay đổi, hãy thay giá trị của ref để kích hoạt cập nhật.

### Immutable Data {#immutable-data}

Nếu bạn đang hiện thực tính năng undo/redo, bạn có thể muốn chụp trạng thái ứng dụng ở mỗi lần người dùng chỉnh sửa. Tuy nhiên, hệ reactivity mutable của Vue không tối ưu cho việc này nếu cây state lớn, vì tuần tự hoá toàn bộ object state ở mỗi lần cập nhật có thể tốn kém cả CPU lẫn bộ nhớ.

[Immutable data structures (Cấu trúc dữ liệu bất biến)](https://en.wikipedia.org/wiki/Persistent_data_structure) giải quyết vấn đề bằng cách không bao giờ thay đổi các object state — thay vào đó, tạo ra các object mới chia sẻ phần không đổi với object cũ. Có nhiều cách dùng dữ liệu bất biến trong JavaScript, nhưng chúng tôi khuyến nghị dùng [Immer](https://immerjs.github.io/immer/) với Vue vì nó cho phép dùng dữ liệu bất biến trong khi vẫn giữ cú pháp dễ chịu, có thể thay đổi.

Ta có thể tích hợp Immer với Vue thông qua một composable đơn giản:

```js
import { produce } from 'immer'
import { shallowRef } from 'vue'

export function useImmer(baseState) {
  const state = shallowRef(baseState)
  const update = (updater) => {
    state.value = produce(state.value, updater)
  }

  return [state, update]
}
```

[Try it in the Playground](https://play.vuejs.org/#eNp9VMFu2zAM/RXNl6ZAYnfoTlnSdRt66DBsQ7vtEuXg2YyjRpYEUU5TBPn3UZLtuE1RH2KLfCIfycfsk8/GpNsGkmkyw8IK4xiCa8wVV6I22jq2Zw3CbV2DZQe2srpmZ2km/PmMK8a4KrRCxxbCQY1j1pgyd3DrD0s27++OFh689z/0OOEkTBlPvkNuFfvbAE/Gra/UilzOko0Mh2A+ufcHwd9ij8KtWUjwMsAqlxgjcLU854qrVaMKJ7RiTleVDBRHQpWwO4/xB8xHoRg2v+oyh/MioJepT0ClvTsxhnSUi1LOsthN6iMdCGgkBacTY7NGhjd9ScG2k5W2c56M9rG6ceBPdbOWm1AxO0/a+uiZFjJHpFv7Fj10XhdSFBtyntTJkzaxf/ZtQnYguoFNJkUkmAWGs2xAm47onqT/jPWHxjjYuUkJhba57+yUSaFg4tZWN9X6Y9eIcC8ZJ1FQkzo36QNqRZILQXjroAqnXb+9LQzVD3vtnMFpljXKbKq00HWU3/X7i/QivcxKgS5aUglVXjxNAGvK8KnWZSNJWa0KDoGChzmk3L28jSVcQX1o1d1puwfgOpdSP97BqsfQxhCCK9gFTC+tXu7/coR7R71rxRWXBL2FpHOMOAAeYVGJhBvFL3s+kGKIkW5zSfKfd+RHA2u3gzZEpML9y9JS06YtAq5DLFmOMWXsjkM6rET1YjzUcSMk2J/G1/h8TKGOb8HmV7bdQbqzhmLziv0Bd3Govywg2O1x8Umvua3ARffN/Q/S1sDZDfMN5x2glo3nGGFfGlUS7QEusL0NcxWq+o03OwcKu6Ke/+fwhIb89Y3Sj3Qv0w+9xg7/AWfvyMs=)

### State Machines {#state-machines}

[State Machine](https://en.wikipedia.org/wiki/Finite-state_machine) là mô hình mô tả tất cả trạng thái mà một ứng dụng có thể có, và mọi cách nó có thể chuyển đổi giữa các trạng thái đó. Dù có thể là “quá tay” cho component đơn giản, nó giúp luồng state phức tạp trở nên chắc chắn và dễ quản lý hơn.

Một trong những hiện thực state machine phổ biến trong JavaScript là [XState](https://xstate.js.org/). Dưới đây là một composable tích hợp với nó:

```js
import { createMachine, interpret } from 'xstate'
import { shallowRef } from 'vue'

export function useMachine(options) {
  const machine = createMachine(options)
  const state = shallowRef(machine.initialState)
  const service = interpret(machine)
    .onTransition((newState) => (state.value = newState))
    .start()
  const send = (event) => service.send(event)

  return [state, send]
}
```

[Try it in the Playground](https://play.vuejs.org/#eNp1U81unDAQfpWRL7DSFqqqUiXEJumhyqVVpDa3ugcKZtcJjC1syEqId8/YBu/uIRcEM9/P/DGz71pn0yhYwUpTD1JbMMKO+o6j7LUaLMwwGvGrqk8SBSzQDqqHJMv7EMleTMIRgGOt0Fj4a2xlxZ5EsPkHhytuOjucbApIrDoeO5HsfQCllVVHUYlVbeW0xr2OKcCzHCwkKQAK3fP56fHx5w/irSyqbfFMgA+h0cKBHZYey45jmYfeqWv6sKLXHbnTF0D5f7RWITzUnaxfD5y5ztIkSCY7zjwKYJ5DyVlf2fokTMrZ5sbZDu6Bs6e25QwK94b0svgKyjwYkEyZR2e2Z2H8n/pK04wV0oL8KEjWJwxncTicnb23C3F2slabIs9H1K/HrFZ9HrIPX7Mv37LPuTC5xEacSfa+V83YEW+bBfleFkuW8QbqQZDEuso9rcOKQQ/CxosIHnQLkWJOVdept9+ijSA6NEJwFGePaUekAdFwr65EaRcxu9BbOKq1JDqnmzIi9oL0RRDu4p1u/ayH9schrhlimGTtOLGnjeJRAJnC56FCQ3SFaYriLWjA4Q7SsPOp6kYnEXMbldKDTW/ssCFgKiaB1kusBWT+rkLYjQiAKhkHvP2j3IqWd5iMQ+M=)

### RxJS {#rxjs}

[RxJS](https://rxjs.dev/) là một thư viện làm việc với luồng sự kiện bất đồng bộ. Thư viện [VueUse](https://vueuse.org/) cung cấp add-on [`@vueuse/rxjs`](https://vueuse.org/rxjs/readme.html) để kết nối các stream RxJS với hệ reactivity của Vue.

## Connection to Signals {#connection-to-signals}

Không ít framework khác đã giới thiệu các primitive reactivity tương tự ref trong Composition API của Vue, dưới thuật ngữ “signals”:

- [Solid Signals](https://www.solidjs.com/docs/latest/api#createsignal)
- [Angular Signals](https://angular.dev/guide/signals)
- [Preact Signals](https://preactjs.com/guide/v10/signals/)
- [Qwik Signals](https://qwik.builder.io/docs/components/state/#usesignal)

Về cơ bản, signals là cùng kiểu primitive reactivity như Vue ref. Nó là “hộp chứa giá trị” cung cấp theo dõi phụ thuộc khi truy cập và kích hoạt side-effect khi thay đổi. Mô hình dựa trên primitive reactivity không phải khái niệm mới trong thế giới frontend: có từ thời [Knockout observables](https://knockoutjs.com/documentation/observables.html) và [Meteor Tracker](https://docs.meteor.com/api/tracker.html) hơn một thập kỷ trước. Vue Options API và thư viện quản lý state React [MobX](https://mobx.js.org/) cũng dựa trên nguyên tắc tương tự, nhưng ẩn các primitive đằng sau thuộc tính object.

Mặc dù không phải đặc tính bắt buộc để được coi là signals, ngày nay khái niệm này thường được bàn cùng mô hình render nơi cập nhật được thực hiện thông qua các fine-grained subscriptions. Do sử dụng Virtual DOM, Vue hiện [dựa vào compiler để đạt các tối ưu tương tự](/guide/extras/rendering-mechanism#compiler-informed-virtual-dom). Tuy vậy, chúng tôi cũng đang khám phá chiến lược biên dịch mới lấy cảm hứng từ Solid, gọi là [Vapor Mode](https://github.com/vuejs/core-vapor), không phụ thuộc vào Virtual DOM và tận dụng tốt hơn hệ reactivity tích hợp sẵn của Vue.

### Đánh đổi trong thiết kế API {#api-design-trade-offs}

Thiết kế signals của Preact và Qwik rất giống với [shallowRef](/api/reactivity-advanced#shallowref) của Vue: cả ba đều cung cấp giao diện mutable qua thuộc tính `.value`. Chúng ta sẽ tập trung thảo luận về signals của Solid và Angular.

#### Solid Signals {#solid-signals}

API `createSignal()` của Solid nhấn mạnh việc tách biệt đọc/ghi. Signal được expose dưới dạng một getter chỉ‑đọc và một setter tách biệt:

```js
const [count, setCount] = createSignal(0)

count() // access the value
setCount(1) // update the value
```

Lưu ý `count` có thể được truyền xuống mà không kèm setter. Điều này đảm bảo state không thể bị thay đổi trừ khi setter được expose một cách tường minh. Việc đánh đổi an toàn này có đáng với cú pháp dài hơn hay không tùy yêu cầu dự án và sở thích cá nhân — nhưng nếu bạn thích phong cách API này, bạn có thể bắt chước dễ dàng trong Vue:

```js
import { shallowRef, triggerRef } from 'vue'

export function createSignal(value, options) {
  const r = shallowRef(value)
  const get = () => r.value
  const set = (v) => {
    r.value = typeof v === 'function' ? v(r.value) : v
    if (options?.equals === false) triggerRef(r)
  }
  return [get, set]
}
```

[Try it in the Playground](https://play.vuejs.org/#eNpdUk1TgzAQ/Ss7uQAjgr12oNXxH+ix9IAYaDQkMV/qMPx3N6G0Uy9Msu/tvn2PTORJqcI7SrakMp1myoKh1qldI9iopLYwQadpa+krG0TLYYZeyxGSojSSs/d7E8vFh0ka0YhOCmPh0EknbB4mPYfTEeqbIelD1oiqXPRQCS+WjoojAW8A1Wmzm1A39KYZzHNVYiUib85aKeCx46z7rBuySqQe6h14uINN1pDIBWACVUcqbGwtl17EqvIiR3LyzwcmcXFuTi3n8vuF9jlYzYaBajxfMsDcomv6E/m9E51luN2NV99yR3OQKkAmgykss+SkMZerxMLEZFZ4oBYJGAA600VEryAaD6CPaJwJKwnr9ldR2WMedV1Dsi6WwB58emZlsAV/zqmH9LzfvqBfruUmNvZ4QN7VearjenP4aHwmWsABt4x/+tiImcx/z27Jqw==)

#### Angular Signals {#angular-signals}

Angular is undergoing some fundamental changes by foregoing dirty-checking and introducing its own implementation of a reactivity primitive. The Angular Signal API looks like this:

```js
const count = signal(0)

count() // access the value
count.set(1) // set new value
count.update((v) => v + 1) // update based on previous value
```

Tương tự, chúng ta có thể bắt chước API này trong Vue một cách dễ dàng:

```js
import { shallowRef } from 'vue'

export function signal(initialValue) {
  const r = shallowRef(initialValue)
  const s = () => r.value
  s.set = (value) => {
    r.value = value
  }
  s.update = (updater) => {
    r.value = updater(r.value)
  }
  return s
}
```

[Try it in the Playground](https://play.vuejs.org/#eNp9Ul1v0zAU/SuWX9ZCSRh7m9IKGHuAB0AD8WQJZclt6s2xLX+ESlH+O9d2krbr1Df7nnPu17k9/aR11nmgt7SwleHaEQvO6w2TvNXKONITyxtZihWpVKu9g5oMZGtUS66yvJSNF6V5lyjZk71ikslKSeuQ7qUj61G+eL+cgFr5RwGITAkXiyVZb5IAn2/IB+QWeeoHO8GPg1aL0gH+CCl215u7mJ3bW9L3s3IYihyxifMlFRpJqewL1qN3TknysRK8el4zGjNlXtdYa9GFrjryllwvGY18QrisDLQgXZTnSX8pF64zzD7pDWDghbbI5/Hoip7tFL05eLErhVD/HmB75Edpyd8zc9DUaAbso3TrZeU4tjfawSV3vBR/SuFhSfrQUXLHBMvmKqe8A8siK7lmsi5gAbJhWARiIGD9hM7BIfHSgjGaHljzlDyGF2MEPQs6g5dpcAIm8Xs+2XxODTgUn0xVYdJ5RxPhKOd4gdMsA/rgLEq3vEEHlEQPYrbgaqu5APNDh6KWUTyuZC2jcWvfYswZD6spXu2gen4l/mT3Icboz3AWpgNGZ8yVBttM8P2v77DH9wy2qvYC2RfAB7BK+NBjon32ssa2j3ix26/xsrhsftv7vQNpp6FCo4E5RD6jeE93F0Y/tHuT3URd2OLwHyXleRY=)

So với Vue ref, phong cách API dựa trên getter của Solid và Angular mang tới một số đánh đổi thú vị khi dùng trong component Vue:

- `()` ít rườm rà hơn một chút so với `.value`, nhưng thao tác cập nhật giá trị lại dài dòng hơn.
- Không có “unwrap” ref: truy cập giá trị luôn cần `()`. Điều này giúp truy cập giá trị nhất quán ở mọi nơi. Nó cũng có nghĩa bạn có thể truyền signals thô xuống dưới dạng props của component.

Các phong cách API này có phù hợp với bạn hay không phụ thuộc phần nào vào chủ quan. Mục tiêu của chúng tôi là cho thấy sự tương đồng nền tảng và các đánh đổi giữa những thiết kế API khác nhau. Chúng tôi cũng muốn cho thấy Vue linh hoạt: bạn không bị khoá cứng vào các API hiện có. Nếu cần, bạn có thể tạo primitive reactivity của riêng mình để phù hợp nhu cầu cụ thể hơn.
