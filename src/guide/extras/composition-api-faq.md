---
outline: deep
---

# Composition API FAQ {#composition-api-faq}

:::tip
FAQ này giả định bạn đã có kinh nghiệm với Vue — đặc biệt là đã dùng Vue 2 với Options API là chính.
:::

## What is Composition API? {#what-is-composition-api}

<VueSchoolLink href="https://vueschool.io/lessons/introduction-to-the-vue-js-3-composition-api" title="Free Composition API Lesson"/>

Composition API là tập các API cho phép viết component Vue bằng hàm import thay vì khai báo options. Thuật ngữ này bao trùm các API sau:

- [Reactivity API](/api/reactivity-core), e.g. `ref()` and `reactive()`, that allows us to directly create reactive state, computed state, and watchers.

- [Lifecycle Hooks](/api/composition-api-lifecycle), e.g. `onMounted()` and `onUnmounted()`, that allow us to programmatically hook into the component lifecycle.

- [Dependency Injection](/api/composition-api-dependency-injection), i.e. `provide()` and `inject()`, that allow us to leverage Vue's dependency injection system while using Reactivity APIs.

Composition API là tính năng dựng sẵn của Vue 3 và [Vue 2.7](https://blog.vuejs.org/posts/vue-2-7-naruto.html). Với các bản Vue 2 cũ hơn, hãy dùng plugin chính thức [`@vue/composition-api`](https://github.com/vuejs/composition-api). Trong Vue 3, nó chủ yếu đi kèm cú pháp [`<script setup>`](/api/sfc-script-setup) trong SFC. Dưới đây là ví dụ cơ bản:

```vue
<script setup>
import { ref, onMounted } from 'vue'

// reactive state
const count = ref(0)

// functions that mutate state and trigger updates
function increment() {
  count.value++
}

// lifecycle hooks
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

Mặc dù có phong cách API dựa trên hợp thành hàm, **Composition API KHÔNG phải lập trình hàm**. Composition API dựa trên mô hình reactivity mutable, tinh‑grain của Vue, trong khi lập trình hàm nhấn mạnh tính bất biến.

Nếu bạn muốn học dùng Vue với Composition API, hãy bật tuỳ chọn ưu tiên Composition API ở đầu sidebar trái và đi qua phần hướng dẫn từ đầu.

## Why Composition API? {#why-composition-api}

### Better Logic Reuse {#better-logic-reuse}

Ưu điểm lớn nhất của Composition API là cho phép tái sử dụng logic gọn gàng, hiệu quả dưới dạng [Composable function](/guide/reusability/composables), giải quyết [mọi nhược điểm của mixin](/guide/reusability/composables#vs-mixins), cơ chế tái sử dụng chính của Options API.

Khả năng tái sử dụng logic của Composition API thúc đẩy các dự án cộng đồng như [VueUse](https://vueuse.org/). Nó cũng là cơ chế sạch để tích hợp dịch vụ / thư viện bên thứ ba có state vào hệ reactivity của Vue, ví dụ [immutable data](/guide/extras/reactivity-in-depth#immutable-data), [state machines](/guide/extras/reactivity-in-depth#state-machines) và [RxJS](/guide/extras/reactivity-in-depth#rxjs).

### Tổ chức mã linh hoạt hơn {#more-flexible-code-organization}

Nhiều người thích việc Options API giúp mã có tổ chức theo từng mục. Tuy nhiên, khi logic của một component vượt quá ngưỡng phức tạp nhất định, hạn chế của Options API bộc lộ rõ, đặc biệt với component có nhiều **mối quan tâm logic** — điều chúng tôi từng thấy ở nhiều ứng dụng Vue 2 thực tế.

Take the folder explorer component from Vue CLI's GUI as an example: this component is responsible for the following logical concerns:

- Tracking current folder state and displaying its content
- Handling folder navigation (opening, closing, refreshing...)
- Handling new folder creation
- Toggling show favorite folders only
- Toggling show hidden folders
- Handling current working directory changes

The [original version](https://github.com/vuejs/vue-cli/blob/a09407dd5b9f18ace7501ddb603b95e31d6d93c0/packages/@vue/cli-ui/src/components/folder/FolderExplorer.vue#L198-L404) of the component was written in Options API. If we give each line of code a color based on the logical concern it is dealing with, this is how it looks:

<img alt="folder component before" src="./images/options-api.png" width="129" height="500" style="margin: 1.2em auto">

Hãy chú ý cách code cho cùng một mối quan tâm bị buộc chia dưới các option khác nhau, nằm rải rác trong file. Ở component vài trăm dòng, hiểu và theo dõi một mối quan tâm yêu cầu cuộn lên xuống liên tục, khó hơn nhiều so với mong đợi. Thêm vào đó, nếu muốn trích xuất một mối quan tâm thành tiện ích tái sử dụng, sẽ mất nhiều công sức để tìm và tách đúng phần code ở các vị trí khác nhau.

Here's the same component, before and after the [refactor into Composition API](https://gist.github.com/yyx990803/8854f8f6a97631576c14b63c8acd8f2e):

![folder component after](./images/composition-api-after.png)

Lưu ý phần code liên quan cùng một mối quan tâm giờ có thể gom lại với nhau, không cần nhảy giữa nhiều block option. Ta cũng dễ trích xuất nhóm mã thành file ngoài. Ma sát thấp khi refactor là chìa khóa cho khả năng bảo trì lâu dài.

### Better Type Inference {#better-type-inference}

Những năm gần đây, ngày càng nhiều lập trình viên frontend áp dụng [TypeScript](https://www.typescriptlang.org/) vì nó giúp chúng ta viết mã vững chắc hơn, thay đổi với sự tự tin cao hơn và mang lại trải nghiệm phát triển tuyệt vời nhờ hỗ trợ IDE. Tuy nhiên, Options API vốn được hình thành từ năm 2013, được thiết kế mà không tính đến suy luận kiểu. Chúng tôi đã phải triển khai một số [“nhào lộn kiểu” cực kỳ phức tạp](https://github.com/vuejs/core/blob/44b95276f5c086e1d88fa3c686a5f39eb5bb7821/packages/runtime-core/src/componentPublicInstance.ts#L132-L165) để khiến suy luận kiểu hoạt động với Options API. Dù vậy, suy luận kiểu cho Options API vẫn có thể bị phá vỡ khi dùng mixin và dependency injection.

Điều này từng khiến nhiều người nghiêng về Class API (qua `vue-class-component`). Tuy nhiên, API dựa trên class phụ thuộc nặng vào decorator, khi 2019 mới ở stage 2 — quá rủi ro để làm API chính thức. Đến 2022 decorator mới lên stage 3. Hơn nữa, class-based API cũng gặp hạn chế tương tự Options API về tái sử dụng và tổ chức logic.

So với đó, Composition API chủ yếu dùng biến và hàm thuần, thân thiện kiểu. Mã viết với Composition API được suy luận kiểu đầy đủ với rất ít gợi ý thủ công, và thường trông gần như giống nhau giữa TS và JS.

### Smaller Production Bundle and Less Overhead {#smaller-production-bundle-and-less-overhead}

Mã viết bằng Composition API và `<script setup>` cũng hiệu quả hơn và thân thiện với minification so với phần tương đương trong Options API. Lý do là template trong component `<script setup>` được biên dịch thành một hàm và được inline trong cùng phạm vi với mã của `<script setup>`. Khác với việc truy cập thuộc tính qua `this`, mã template đã biên dịch có thể truy cập trực tiếp các biến được khai báo bên trong `<script setup>` mà không cần lớp proxy của instance ở giữa. Điều này cũng giúp minification tốt hơn vì tất cả tên biến có thể được rút gọn một cách an toàn.

## Relationship with Options API {#relationship-with-options-api}

### Trade-offs {#trade-offs}

Some users moving from Options API found their Composition API code less organized, and concluded that Composition API is "worse" in terms of code organization. We recommend users with such opinions to look at that problem from a different perspective.

Đúng là Composition API không còn “lan can” hướng bạn đặt mã vào từng mục như Options API. Đổi lại, bạn viết code như JS thuần. **Hãy áp dụng các best practice tổ chức code như khi viết JS bình thường**.

Options API giúp “đỡ phải nghĩ” khi viết code, nhưng cũng khoá bạn vào một khuôn tổ chức nhất định, gây khó refactor ở dự án lớn. Về khía cạnh này, Composition API mở rộng tốt hơn về lâu dài.

### Does Composition API cover all use cases? {#does-composition-api-cover-all-use-cases}

Có, xét về logic có state. Khi dùng Composition API, chỉ còn vài option có thể cần: `props`, `emits`, `name`, và `inheritAttrs`.

:::tip

Since 3.3 you can directly use `defineOptions` in `<script setup>` to set the component name or `inheritAttrs` property

:::

Nếu bạn dùng độc quyền Composition API (kèm các option trên), bạn có thể giảm vài KB khỏi bundle production bằng [cờ compile-time](/api/compile-time-flags) để loại bỏ mã liên quan Options API khỏi Vue. Lưu ý điều này cũng ảnh hưởng đến component Vue trong dependencies.

### Can I use both APIs in the same component? {#can-i-use-both-apis-in-the-same-component}

Có. Bạn có thể dùng Composition API qua tùy chọn [`setup()`](/api/composition-api-setup) trong một component Options API.

Tuy nhiên, chỉ nên làm vậy nếu bạn có codebase Options API hiện có cần tích hợp tính năng mới / thư viện bên ngoài viết với Composition API.

### Will Options API be deprecated? {#will-options-api-be-deprecated}

Không. Chúng tôi không có kế hoạch như vậy. Options API là phần không thể thiếu của Vue và là lý do nhiều lập trình viên yêu thích nó. Chúng tôi cũng nhận thấy lợi ích của Composition API chủ yếu thể hiện ở dự án quy mô lớn, và Options API vẫn là lựa chọn vững chắc cho nhiều tình huống độ phức tạp thấp‑trung bình.

## Relationship with Class API {#relationship-with-class-api}

Chúng tôi không còn khuyến nghị dùng Class API với Vue 3, vì Composition API đã cung cấp tích hợp TS tốt cùng lợi ích tái sử dụng và tổ chức logic.

## Comparison with React Hooks {#comparison-with-react-hooks}

Composition API cung cấp khả năng tổ hợp logic tương đương React Hooks, nhưng có khác biệt quan trọng.

Hooks trong React được gọi lặp lại mỗi lần component cập nhật, kéo theo nhiều lưu ý dễ gây nhầm lẫn và vấn đề tối ưu hoá. Ví dụ:

- Hooks phụ thuộc vào thứ tự gọi và không thể đặt điều kiện.

- Các biến khai báo trong component React có thể bị “bắt” bởi closure của hook và trở nên “cũ” (stale) nếu lập trình viên không truyền đúng mảng dependencies. Điều này khiến lập trình viên React phải dựa vào các rule ESLint để đảm bảo truyền đúng phụ thuộc. Tuy nhiên, các rule này thường chưa đủ “thông minh” và có xu hướng “quá tay” vì độ chính xác, dẫn đến vô hiệu hoá không cần thiết và gây đau đầu khi gặp edge case.

- Các phép tính tốn kém cần dùng `useMemo`, và lại phải truyền thủ công mảng dependencies chính xác.

- Trình xử lý sự kiện truyền xuống component con mặc định sẽ gây cập nhật con không cần thiết và cần dùng `useCallback` một cách tường minh để tối ưu. Điều này hầu như luôn cần, và lại yêu cầu mảng dependencies chính xác. Bỏ qua sẽ khiến ứng dụng render quá mức theo mặc định và có thể gây vấn đề hiệu năng mà không nhận ra.

- Vấn đề stale closure, kết hợp với các tính năng Concurrent, khiến khó suy luận thời điểm một đoạn code hook được chạy và làm cho việc xử lý state mutable cần tồn tại qua nhiều lần render (qua `useRef`) trở nên rườm rà.

> Lưu ý: một số vấn đề liên quan đến memoization ở trên có thể được giải quyết bởi [React Compiler](https://react.dev/learn/react-compiler) sắp ra mắt.

So với đó, Vue Composition API:

- Gọi code trong `setup()` hoặc `<script setup>` chỉ một lần. Điều này phù hợp hơn với trực giác khi viết JavaScript thuần vì không có stale closure phải lo. Các lời gọi Composition API cũng không nhạy với thứ tự gọi và có thể đặt điều kiện.

- Hệ reactivity lúc chạy của Vue tự động thu thập các phụ thuộc reactive dùng trong computed và watcher, nên không cần khai báo dependencies thủ công.

- Không cần cache thủ công các hàm callback để tránh cập nhật con không cần thiết. Nhìn chung, hệ reactivity tinh‑granular của Vue đảm bảo component con chỉ cập nhật khi cần. Việc tối ưu cập nhật con thủ công hiếm khi là mối bận tâm của lập trình viên Vue.

Chúng tôi ghi nhận tính sáng tạo của React Hooks và nó là nguồn cảm hứng lớn cho Composition API. Tuy nhiên, các vấn đề nêu trên có tồn tại và mô hình reactivity của Vue cung cấp cách tránh chúng.
