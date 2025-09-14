---
outline: deep
---

# Rendering Mechanism {#rendering-mechanism}

Vue biến template thành DOM node thật như thế nào? Vue cập nhật các DOM node đó hiệu quả ra sao? Ở đây chúng ta sẽ làm rõ bằng cách đi sâu vào cơ chế render nội bộ của Vue.

## Virtual DOM {#virtual-dom}

Có lẽ bạn đã nghe thuật ngữ “virtual DOM”, nền tảng của hệ thống render của Vue.

Virtual DOM (VDOM) là khái niệm nơi một biểu diễn “ảo” của UI được giữ trong bộ nhớ và đồng bộ với DOM “thật”. Khái niệm này tiên phong bởi [React](https://reactjs.org/), và được nhiều framework áp dụng (bao gồm Vue) với hiện thực khác nhau.

Virtual DOM là một pattern hơn là công nghệ cụ thể, nên không có hiện thực “chuẩn”. Ta minh họa bằng ví dụ đơn giản:

```js
const vnode = {
  type: 'div',
  props: {
    id: 'hello'
  },
  children: [
    /* more vnodes */
  ]
}
```

Ở đây, `vnode` là object JavaScript thuần (một “virtual node”) đại diện cho phần tử `<div>`. Nó chứa mọi thông tin cần để tạo phần tử thật, và có các vnode con, trở thành gốc của cây virtual DOM.

Renderer lúc chạy có thể duyệt cây virtual DOM và dựng cây DOM thật từ nó — gọi là **mount**.

Nếu có hai bản cây virtual DOM, renderer có thể duyệt và so sánh, tìm khác biệt và áp vào DOM thật — gọi là **patch** (còn gọi “diffing” / “reconciliation”).

Lợi ích chính của virtual DOM là cho phép lập trình viên tạo, kiểm tra và ghép UI mong muốn một cách khai báo, còn thao tác DOM trực tiếp do renderer xử lý.

## Render Pipeline {#render-pipeline}

Ở mức cao, khi một component Vue được mount sẽ diễn ra:

1. **Compile**: Vue templates are compiled into **render functions**: functions that return virtual DOM trees. This step can be done either ahead-of-time via a build step, or on-the-fly by using the runtime compiler.

2. **Mount**: Renderer gọi render function, duyệt cây virtual DOM trả về và tạo DOM node thật. Bước này là một [reactive effect](./reactivity-in-depth), nên theo dõi các phụ thuộc reactive đã dùng.

3. **Patch**: Khi phụ thuộc dùng lúc mount thay đổi, effect chạy lại, tạo cây virtual DOM mới. Renderer duyệt cây mới, so sánh với cây cũ và áp cập nhật cần thiết vào DOM thật.

![render pipeline](./images/render-pipeline.png)

<!-- https://www.figma.com/file/elViLsnxGJ9lsQVsuhwqxM/Rendering-Mechanism -->

## Templates vs. Render Functions {#templates-vs-render-functions}

Template Vue được biên dịch thành render function của virtual DOM. Vue cũng có API để bỏ qua bước biên dịch template và viết render function trực tiếp. Render function linh hoạt hơn khi xử lý logic cực kỳ động, vì bạn thao tác vnode bằng toàn bộ sức mạnh JavaScript.

Vậy tại sao Vue khuyến nghị template theo mặc định? Một vài lý do:

1. Template gần với HTML thực tế. Điều này giúp tái sử dụng đoạn HTML sẵn có, áp dụng best practice accessibility, style bằng CSS, và để designer hiểu, chỉnh sửa dễ hơn.

2. Template dễ phân tích tĩnh hơn nhờ cú pháp mang tính xác định. Điều này cho phép compiler của Vue áp dụng nhiều tối ưu ở thời điểm biên dịch để cải thiện hiệu năng virtual DOM (sẽ bàn bên dưới).

Trong thực tế, template đủ cho đa số use case. Render function thường dùng trong component tái sử dụng cần render rất động. Xem chi tiết ở [Render Functions & JSX](./render-function).

## Compiler-Informed Virtual DOM {#compiler-informed-virtual-dom}

Hiện thực virtual DOM trong React và đa số framework khác là thuần runtime: thuật toán reconciliation không thể giả định gì về cây virtual DOM đầu vào, nên phải duyệt toàn bộ cây và diff props của mọi vnode để đảm bảo đúng đắn. Thêm nữa, ngay cả khi một phần của cây không bao giờ đổi, các vnode mới vẫn luôn được tạo cho chúng ở mỗi lần render lại, gây áp lực bộ nhớ không cần thiết. Đây là một khía cạnh thường bị phê bình của virtual DOM: quá trình reconciliation có phần “dùng lực” hy sinh hiệu quả để đổi lấy tính khai báo và đúng đắn.

Nhưng không nhất thiết phải vậy. Trong Vue, framework kiểm soát cả compiler lẫn runtime. Điều này cho phép thực hiện nhiều tối ưu compile‑time mà chỉ renderer kết hợp chặt mới tận dụng được. Compiler phân tích tĩnh template và để lại gợi ý trong code sinh ra để runtime có thể “đi đường tắt” khi có thể. Đồng thời, người dùng vẫn có thể hạ xuống lớp render function để kiểm soát trực tiếp trong các tình huống biên. Cách tiếp cận lai này gọi là **Compiler‑Informed Virtual DOM**.

Bên dưới, chúng ta sẽ bàn về một số tối ưu chính mà compiler của Vue thực hiện để cải thiện hiệu năng runtime của virtual DOM.

### Cache Static {#cache-static}

Thường có những phần trong template không chứa binding động:

```vue-html{2-3}
<div>
  <div>foo</div> <!-- cached -->
  <div>bar</div> <!-- cached -->
  <div>{{ dynamic }}</div>
</div>
```

[Inspect in Template Explorer](https://template-explorer.vuejs.org/#eyJzcmMiOiI8ZGl2PlxuICA8ZGl2PmZvbzwvZGl2PiA8IS0tIGNhY2hlZCAtLT5cbiAgPGRpdj5iYXI8L2Rpdj4gPCEtLSBjYWNoZWQgLS0+XG4gIDxkaXY+e3sgZHluYW1pYyB9fTwvZGl2PlxuPC9kaXY+XG4iLCJvcHRpb25zIjp7ImhvaXN0U3RhdGljIjp0cnVlfX0=)

Hai div `foo` và `bar` là static — tạo lại vnode và diff ở mỗi lần render lại là không cần thiết. Renderer tạo các vnode này ở lần render đầu, cache lại và tái sử dụng ở các lần sau, đồng thời bỏ qua diff khi phát hiện vnode cũ và mới là cùng một tham chiếu.

In addition, when there are enough consecutive static elements, they will be condensed into a single "static vnode" that contains the plain HTML string for all these nodes ([Example](https://template-explorer.vuejs.org/#eyJzcmMiOiI8ZGl2PlxuICA8ZGl2IGNsYXNzPVwiZm9vXCI+Zm9vPC9kaXY+XG4gIDxkaXYgY2xhc3M9XCJmb29cIj5mb288L2Rpdj5cbiAgPGRpdiBjbGFzcz1cImZvb1wiPmZvbzwvZGl2PlxuICA8ZGl2IGNsYXNzPVwiZm9vXCI+Zm9vPC9kaXY+XG4gIDxkaXYgY2xhc3M9XCJmb29cIj5mb288L2Rpdj5cbiAgPGRpdj57eyBkeW5hbWljIH19PC9kaXY+XG48L2Rpdj4iLCJzc3IiOmZhbHNlLCJvcHRpb25zIjp7ImhvaXN0U3RhdGljIjp0cnVlfX0=)). These static vnodes are mounted by directly setting `innerHTML`.

### Patch Flags {#patch-flags}

Với một phần tử có binding động, chúng ta cũng có thể suy ra nhiều thông tin ngay tại thời điểm biên dịch:

```vue-html
<!-- class binding only -->
<div :class="{ active }"></div>

<!-- id and value bindings only -->
<input :id="id" :value="value">

<!-- text children only -->
<div>{{ dynamic }}</div>
```

[Inspect in Template Explorer](https://template-explorer.vuejs.org/#eyJzcmMiOiI8ZGl2IDpjbGFzcz1cInsgYWN0aXZlIH1cIj48L2Rpdj5cblxuPGlucHV0IDppZD1cImlkXCIgOnZhbHVlPVwidmFsdWVcIj5cblxuPGRpdj57eyBkeW5hbWljIH19PC9kaXY+Iiwib3B0aW9ucyI6e319)

Khi sinh code render function cho các phần tử này, Vue mã hoá kiểu cập nhật mà mỗi phần tử cần ngay trong lời gọi tạo vnode:

```js{3}
createElementVNode("div", {
  class: _normalizeClass({ active: _ctx.active })
}, null, 2 /* CLASS */)
```

Tham số cuối cùng, `2`, là một [patch flag](https://github.com/vuejs/core/blob/main/packages/shared/src/patchFlags.ts). Một phần tử có thể có nhiều patch flag, được gộp thành một số. Renderer lúc chạy sẽ kiểm tra các cờ này bằng [toán tử bit](https://en.wikipedia.org/wiki/Bitwise_operation) để quyết định có cần làm công việc nhất định hay không:

```js
if (vnode.patchFlag & PatchFlags.CLASS /* 2 */) {
  // update the element's class
}
```

Kiểm tra bitwise cực nhanh. Nhờ patch flag, Vue chỉ thực hiện lượng công việc tối thiểu cần thiết khi cập nhật phần tử có binding động.

Vue also encodes the type of children a vnode has. For example, a template that has multiple root nodes is represented as a fragment. In most cases, we know for sure that the order of these root nodes will never change, so this information can also be provided to the runtime as a patch flag:

```js{4}
export function render() {
  return (_openBlock(), _createElementBlock(_Fragment, null, [
    /* children */
  ], 64 /* STABLE_FRAGMENT */))
}
```

Nhờ đó, runtime có thể bỏ qua hoàn toàn việc hoà giải thứ tự con cho fragment gốc.

### Tree Flattening {#tree-flattening}

Nhìn lại code sinh ra ở ví dụ trước, bạn sẽ thấy gốc của cây virtual DOM trả về được tạo bằng lời gọi đặc biệt `createElementBlock()`:

```js{2}
export function render() {
  return (_openBlock(), _createElementBlock(_Fragment, null, [
    /* children */
  ], 64 /* STABLE_FRAGMENT */))
}
```

Về khái niệm, “block” là phần của template có cấu trúc bên trong ổn định. Trường hợp này, toàn bộ template có một block vì không chứa directive cấu trúc như `v-if` và `v-for`.

Mỗi block theo dõi mọi node hậu duệ (không chỉ con trực tiếp) có patch flag. Ví dụ:

```vue-html{3,5}
<div> <!-- root block -->
  <div>...</div>         <!-- not tracked -->
  <div :id="id"></div>   <!-- tracked -->
  <div>                  <!-- not tracked -->
    <div>{{ bar }}</div> <!-- tracked -->
  </div>
</div>
```

Kết quả là một mảng phẳng chỉ chứa các node hậu duệ động:

```
div (block root)
- div with :id binding
- div with {{ bar }} binding
```

Khi component cần render lại, nó chỉ cần duyệt cây đã phẳng hoá thay vì toàn bộ cây. Đây gọi là **Tree Flattening**, và giảm mạnh số node phải duyệt trong quá trình reconciliation của virtual DOM. Phần tĩnh trong template được bỏ qua hiệu quả.

Các directive `v-if` và `v-for` sẽ tạo block node mới:

```vue-html
<div> <!-- root block -->
  <div>
    <div v-if> <!-- if block -->
      ...
    </div>
  </div>
</div>
```

Block con được theo dõi trong mảng hậu duệ động của block cha. Điều này giữ cấu trúc ổn định cho block cha.

### Impact on SSR Hydration {#impact-on-ssr-hydration}

Cả patch flag lẫn tree flattening cũng cải thiện đáng kể hiệu năng [SSR Hydration](/guide/scaling-up/ssr#client-hydration) của Vue:

- Việc hydrate từng phần tử có thể đi “đường tắt” dựa trên patch flag của vnode tương ứng.

- Chỉ cần duyệt các block node và hậu duệ động của chúng trong quá trình hydrate, đạt hiệu ứng partial hydration ở cấp độ template.
