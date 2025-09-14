# Template Syntax {#template-syntax}

<ScrimbaLink href="https://scrimba.com/links/vue-template-syntax" title="Free Vue.js Template Syntax Lesson" type="scrimba">
  Watch an interactive video lesson on Scrimba
</ScrimbaLink>

Vue dùng template syntax dựa trên HTML cho phép bạn bind DOM render ra với dữ liệu của component một cách khai báo. Mọi template của Vue đều là HTML hợp lệ về mặt cú pháp và có thể được parse bởi các trình duyệt và HTML parser tuân thủ tiêu chuẩn.

Bên dưới, Vue biên dịch template thành mã JavaScript được tối ưu cao. Kết hợp với hệ thống reactivity, Vue có thể xác định thông minh số lượng component tối thiểu cần re‑render và thực hiện tối thiểu các thao tác DOM khi state của ứng dụng thay đổi.

Nếu bạn quen với khái niệm Virtual DOM và thích sức mạnh thuần của JavaScript, bạn cũng có thể [viết render function trực tiếp](/guide/extras/render-function) thay cho template, với hỗ trợ JSX tùy chọn. Tuy nhiên, lưu ý rằng chúng không được tối ưu ở compile‑time như template.

## Text Interpolation {#text-interpolation}

Hình thức binding dữ liệu cơ bản nhất là text interpolation dùng cú pháp "Mustache" (hai dấu ngoặc nhọn):

```vue-html
<span>Message: {{ msg }}</span>
```

Thẻ mustache sẽ được thay thế bằng giá trị của thuộc tính `msg` [từ component instance tương ứng](/guide/essentials/reactivity-fundamentals#declaring-reactive-state). Nó cũng sẽ được cập nhật mỗi khi thuộc tính `msg` thay đổi.

## Raw HTML {#raw-html}

Hai dấu ngoặc nhọn hiểu dữ liệu là plain text, không phải HTML. Để xuất ra HTML thực sự, bạn cần dùng [`v-html` directive](/api/built-in-directives#v-html):

```vue-html
<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

<script setup>
  const rawHtml = '<span style="color: red">This should be red.</span>'
</script>

<div class="demo">
  <p>Using text interpolation: {{ rawHtml }}</p>
  <p>Using v-html directive: <span v-html="rawHtml"></span></p>
</div>

Ở đây chúng ta gặp khái niệm mới. Thuộc tính `v-html` bạn thấy được gọi là **directive**. Các directive có tiền tố `v-` để chỉ rằng đây là các thuộc tính đặc biệt do Vue cung cấp, và như bạn đoán, chúng áp dụng hành vi reactive đặc biệt lên DOM được render. Ở đây, ta đang nói rằng "giữ inner HTML của phần tử này luôn đồng bộ với thuộc tính `rawHtml` trên current active instance."

Nội dung của `span` sẽ được thay bằng giá trị của thuộc tính `rawHtml`, được hiểu là HTML thuần — mọi data binding sẽ bị bỏ qua. Lưu ý bạn không thể dùng `v-html` để ghép các template partial, vì Vue không phải templating engine dựa trên chuỗi. Thay vào đó, component là đơn vị cơ bản để tái sử dụng và cấu thành UI.

:::warning Security Warning
Render động HTML tùy ý trên website có thể rất nguy hiểm vì dễ dẫn đến [lỗ hổng XSS](https://en.wikipedia.org/wiki/Cross-site_scripting). Chỉ dùng `v-html` với nội dung đáng tin cậy và **không bao giờ** dùng với nội dung do người dùng cung cấp.
:::

## Attribute Bindings {#attribute-bindings}

Mustache không thể dùng bên trong HTML attributes. Thay vào đó, dùng [`v-bind` directive](/api/built-in-directives#v-bind):

```vue-html
<div v-bind:id="dynamicId"></div>
```

`v-bind` hướng dẫn Vue giữ cho thuộc tính `id` của phần tử luôn đồng bộ với thuộc tính `dynamicId` của component. Nếu giá trị bind là `null` hoặc `undefined`, thuộc tính sẽ bị loại bỏ khỏi phần tử render ra.

### Shorthand {#shorthand}

Vì `v-bind` được dùng rất thường xuyên, nó có shorthand syntax riêng:

```vue-html
<div :id="dynamicId"></div>
```

Các attribute bắt đầu bằng `:` có thể trông hơi khác HTML thông thường, nhưng thực ra đây là ký tự hợp lệ cho tên attribute và tất cả trình duyệt được Vue hỗ trợ đều parse đúng. Ngoài ra, chúng không xuất hiện trong markup render cuối cùng. Shorthand syntax là tùy chọn, nhưng có lẽ bạn sẽ thấy nó tiện lợi hơn khi dùng nhiều sau này.

> Với phần còn lại của hướng dẫn, chúng tôi sẽ dùng shorthand syntax trong ví dụ code vì đó là cách dùng phổ biến nhất với lập trình viên Vue.

### Same-name Shorthand {#same-name-shorthand}

- Chỉ hỗ trợ từ 3.4+

Nếu attribute có cùng tên với biến JavaScript đang được bind, cú pháp có thể rút gọn hơn nữa bằng cách lược bỏ giá trị của attribute:

```vue-html
<!-- same as :id="id" -->
<div :id></div>

<!-- this also works -->
<div v-bind:id></div>
```

Điều này tương tự property shorthand khi khai báo object trong JavaScript. Lưu ý đây là tính năng chỉ có từ Vue 3.4 trở lên.

### Boolean Attributes {#boolean-attributes}

[Boolean attributes](https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#boolean-attributes) là các attribute thể hiện giá trị true/false thông qua sự hiện diện trên phần tử. Ví dụ, [`disabled`](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/disabled) là một boolean attribute được dùng rất phổ biến.

Trong trường hợp này, `v-bind` hoạt động hơi khác:

```vue-html
<button :disabled="isButtonDisabled">Button</button>
```

Attribute `disabled` sẽ được thêm nếu `isButtonDisabled` có [giá trị truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy). Nó cũng sẽ được thêm nếu giá trị là chuỗi rỗng, giữ nhất quán với `<button disabled="">`. Với các [giá trị falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) khác, attribute sẽ bị bỏ qua.

### Dynamically Binding Multiple Attributes {#dynamically-binding-multiple-attributes}

Nếu bạn có một object JavaScript đại diện cho nhiều attribute như sau:

<div class="composition-api">

```js
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper',
  style: 'background-color:green'
}
```

</div>
<div class="options-api">

```js
data() {
  return {
    objectOfAttrs: {
      id: 'container',
      class: 'wrapper'
    }
  }
}
```

</div>

Bạn có thể bind chúng vào một phần tử duy nhất bằng cách dùng `v-bind` không có tham số:

```vue-html
<div v-bind="objectOfAttrs"></div>
```

## Using JavaScript Expressions {#using-javascript-expressions}

Từ nãy đến giờ, chúng ta chỉ bind các property key đơn giản trong template. Nhưng thực tế, Vue hỗ trợ đầy đủ sức mạnh của JavaScript expression trong mọi kiểu data binding:

```vue-html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div :id="`list-${id}`"></div>
```

Các expression này sẽ được đánh giá như JavaScript trong phạm vi dữ liệu của component instance hiện tại.

Trong template của Vue, JavaScript expression có thể dùng ở các vị trí sau:

- Bên trong text interpolation (mustache)
- Trong giá trị attribute của bất kỳ Vue directive nào (các thuộc tính đặc biệt bắt đầu với `v-`)

### Expressions Only {#expressions-only}

Mỗi binding chỉ có thể chứa **một expression duy nhất**. Expression là một đoạn code có thể được đánh giá thành một giá trị. Cách kiểm tra đơn giản là nó có thể được dùng sau `return` hay không.

Vì vậy, các ví dụ sau **KHÔNG** hoạt động:

```vue-html
<!-- this is a statement, not an expression: -->
{{ var a = 1 }}

<!-- flow control won't work either, use ternary expressions -->
{{ if (ok) { return message } }}
```

### Calling Functions {#calling-functions}

Có thể gọi method được component lộ ra bên trong một binding expression:

```vue-html
<time :title="toTitleDate(date)" :datetime="date">
  {{ formatDate(date) }}
</time>
```

:::tip
Các hàm được gọi trong binding expression sẽ được gọi mỗi lần component cập nhật, nên chúng **không** nên có side effect, như thay đổi dữ liệu hoặc kích hoạt thao tác bất đồng bộ.
:::

### Restricted Globals Access {#restricted-globals-access}

Template expression được sandbox và chỉ có thể truy cập [danh sách globals bị giới hạn](https://github.com/vuejs/core/blob/main/packages/shared/src/globalsAllowList.ts#L3). Danh sách này cung cấp các global built‑in thường dùng như `Math` và `Date`.

Các global không nằm trong danh sách, ví dụ thuộc tính người dùng gắn vào `window`, sẽ không truy cập được trong template expression. Tuy nhiên, bạn có thể định nghĩa rõ ràng các global bổ sung cho mọi Vue expression bằng cách thêm chúng vào [`app.config.globalProperties`](/api/application#app-config-globalproperties).

## Directives {#directives}

Directive là các thuộc tính đặc biệt có tiền tố `v-`. Vue cung cấp một số [built‑in directives](/api/built-in-directives), bao gồm `v-html` và `v-bind` như đã giới thiệu ở trên.

Giá trị của attribute trên directive được kỳ vọng là một JavaScript expression duy nhất (ngoại trừ `v-for`, `v-on` và `v-slot`, sẽ được bàn trong các phần riêng). Nhiệm vụ của directive là áp dụng cập nhật lên DOM một cách reactive khi giá trị expression của nó thay đổi. Ví dụ [`v-if`](/api/built-in-directives#v-if):

```vue-html
<p v-if="seen">Now you see me</p>
```

Ở đây, directive `v-if` sẽ xóa hoặc chèn phần tử `<p>` dựa trên truthiness của giá trị expression `seen`.

### Arguments {#arguments}

Một số directive có thể nhận "argument", ký hiệu bằng dấu hai chấm sau tên directive. Ví dụ, directive `v-bind` dùng để cập nhật một HTML attribute một cách reactive:

```vue-html
<a v-bind:href="url"> ... </a>

<!-- shorthand -->
<a :href="url"> ... </a>
```

Ở đây, `href` là argument, cho `v-bind` biết cần bind attribute `href` của phần tử với giá trị expression `url`. Ở dạng viết tắt, toàn bộ phần trước argument (tức `v-bind:`) được rút gọn thành một ký tự `:`.

Một ví dụ khác là directive `v-on`, lắng nghe các DOM event:

```vue-html
<a v-on:click="doSomething"> ... </a>

<!-- shorthand -->
<a @click="doSomething"> ... </a>
```

Ở đây, argument là tên event cần lắng nghe: `click`. `v-on` có shorthand tương ứng là ký tự `@`. Chúng ta cũng sẽ bàn về xử lý sự kiện chi tiết hơn.

### Dynamic Arguments {#dynamic-arguments}

Cũng có thể dùng JavaScript expression trong argument của directive bằng cách bao trong dấu ngoặc vuông:

```vue-html
<!--
Note that there are some constraints to the argument expression,
as explained in the "Dynamic Argument Value Constraints" and "Dynamic Argument Syntax Constraints" sections below.
-->
<a v-bind:[attributeName]="url"> ... </a>

<!-- shorthand -->
<a :[attributeName]="url"> ... </a>
```

Ở đây, `attributeName` sẽ được đánh giá động như một JavaScript expression, và giá trị sau khi đánh giá sẽ được dùng làm giá trị cuối cùng cho argument. Ví dụ, nếu component instance của bạn có data property `attributeName` với giá trị `"href"`, thì binding này tương đương `v-bind:href`.

Tương tự, bạn có thể dùng dynamic argument để bind một handler cho một tên event động:

```vue-html
<a v-on:[eventName]="doSomething"> ... </a>

<!-- shorthand -->
<a @[eventName]="doSomething"> ... </a>
```

Trong ví dụ này, khi giá trị của `eventName` là `"focus"`, `v-on:[eventName]` tương đương `v-on:focus`.

#### Dynamic Argument Value Constraints {#dynamic-argument-value-constraints}

Dynamic argument được kỳ vọng đánh giá ra một chuỗi, ngoại trừ `null`. Giá trị đặc biệt `null` có thể dùng để gỡ bỏ binding một cách tường minh. Bất kỳ giá trị không phải chuỗi nào khác sẽ gây cảnh báo.

#### Dynamic Argument Syntax Constraints {#dynamic-argument-syntax-constraints}

Dynamic argument expression có một số ràng buộc cú pháp vì một số ký tự như khoảng trắng và dấu nháy không hợp lệ trong tên HTML attribute. Ví dụ sau là không hợp lệ:

```vue-html
<!-- This will trigger a compiler warning. -->
<a :['foo' + bar]="value"> ... </a>
```

Nếu bạn cần truyền một dynamic argument phức tạp, tốt hơn nên dùng [computed property](./computed), chúng ta sẽ tìm hiểu ngay sau đây.

Khi dùng in‑DOM template (template viết trực tiếp trong file HTML), bạn cũng nên tránh đặt key có ký tự viết hoa, vì trình duyệt sẽ chuyển attribute name thành chữ thường:

```vue-html
<a :[someAttr]="value"> ... </a>
```

Đoạn trên sẽ bị chuyển thành `:[someattr]` trong in‑DOM template. Nếu component của bạn có property `someAttr` thay vì `someattr`, code sẽ không hoạt động. Template bên trong Single‑File Components **không** bị ràng buộc này.

### Modifiers {#modifiers}

Modifier là hậu tố đặc biệt được ký hiệu bằng dấu chấm, cho biết directive nên được bind theo một cách đặc biệt. Ví dụ, modifier `.prevent` yêu cầu directive `v-on` gọi `event.preventDefault()` trên event được kích hoạt:

```vue-html
<form @submit.prevent="onSubmit">...</form>
```

Bạn sẽ thấy các ví dụ khác về modifier sau này, [cho `v-on`](./event-handling#event-modifiers) và [cho `v-model`](./forms#modifiers), khi chúng ta khám phá các tính năng đó.

Và cuối cùng, đây là hình minh họa đầy đủ về cú pháp directive:

![directive syntax graph](./images/directive.png)

<!-- https://www.figma.com/file/BGWUknIrtY9HOmbmad0vFr/Directive -->
