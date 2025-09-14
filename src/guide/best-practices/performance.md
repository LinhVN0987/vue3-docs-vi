---
outline: deep
---

# Performance {#performance}

## Overview {#overview}

Vue được thiết kế hiệu năng cho hầu hết trường hợp mà không cần tối ưu thủ công. Tuy nhiên, luôn có các kịch bản khó cần tinh chỉnh thêm. Phần này bàn về những điểm bạn nên chú ý liên quan hiệu năng trong ứng dụng Vue.

Trước hết, có hai khía cạnh chính của hiệu năng web:

- **Page Load Performance**: how fast the application shows content and becomes interactive on the initial visit. This is usually measured using web vital metrics like [Largest Contentful Paint (LCP)](https://web.dev/lcp/) and [Interaction to Next Paint](https://web.dev/articles/inp).

- **Update Performance**: how fast the application updates in response to user input. For example, how fast a list updates when the user types in a search box, or how fast the page switches when the user clicks a navigation link in a Single-Page Application (SPA).

Lý tưởng là tối đa cả hai, nhưng kiến trúc frontend khác nhau ảnh hưởng mức độ dễ đạt hiệu năng mong muốn. Ngoài ra, loại ứng dụng bạn xây cũng quyết định ưu tiên. Vì vậy, bước đầu để có hiệu năng tối ưu là chọn kiến trúc đúng cho loại ứng dụng bạn đang xây:

- Consult [Ways of Using Vue](/guide/extras/ways-of-using-vue) to see how you can leverage Vue in different ways.

- Jason Miller discusses the types of web applications and their respective ideal implementation / delivery in [Application Holotypes](https://jasonformat.com/application-holotypes/).

## Profiling Options {#profiling-options}

Để cải thiện hiệu năng, trước tiên cần biết cách đo. Có nhiều công cụ hữu ích:

For profiling load performance of production deployments:

- [PageSpeed Insights](https://pagespeed.web.dev/)
- [WebPageTest](https://www.webpagetest.org/)

For profiling performance during local development:

- [Chrome DevTools Performance Panel](https://developer.chrome.com/docs/devtools/evaluate-performance/)
  - [`app.config.performance`](/api/application#app-config-performance) enables Vue-specific performance markers in Chrome DevTools' performance timeline.
- [Vue DevTools Extension](/guide/scaling-up/tooling#browser-devtools) also provides a performance profiling feature.

## Page Load Optimizations {#page-load-optimizations}

Có nhiều khía cạnh tối ưu hiệu năng tải trang không phụ thuộc framework — xem [hướng dẫn trên web.dev](https://web.dev/fast/). Ở đây, ta tập trung vào kỹ thuật đặc thù cho Vue.

### Choosing the Right Architecture {#choosing-the-right-architecture}

Nếu use case nhạy với hiệu năng tải trang, tránh phát hành dưới dạng SPA thuần client. Bạn muốn server gửi trực tiếp HTML chứa nội dung người dùng cần. Client‑side rendering thuần bị chậm time‑to‑content. Có thể khắc phục bằng [SSR](/guide/extras/ways-of-using-vue#fullstack-ssr) hoặc [SSG](/guide/extras/ways-of-using-vue#jamstack-ssg). Xem [SSR Guide](/guide/scaling-up/ssr). Nếu app không đòi hỏi tương tác phong phú, cũng có thể render HTML bằng backend truyền thống và tăng cường bằng Vue trên client.

Nếu ứng dụng chính buộc là SPA nhưng có trang marketing (landing, about, blog), hãy tách riêng! Các trang marketing nên được triển khai dưới dạng HTML tĩnh với JS tối thiểu, dùng SSG.

### Bundle Size and Tree-shaking {#bundle-size-and-tree-shaking}

Một cách hiệu quả để cải thiện tải trang là giảm kích thước bundle JavaScript. Một số cách giảm khi dùng Vue:

- Use a build step if possible.

  - Many of Vue's APIs are ["tree-shakable"](https://developer.mozilla.org/en-US/docs/Glossary/Tree_shaking) if bundled via a modern build tool. For example, if you don't use the built-in `<Transition>` component, it won't be included in the final production bundle. Tree-shaking can also remove other unused modules in your source code.

  - Khi dùng build step, template được tiền biên dịch nên không cần mang compiler đến trình duyệt. Tiết kiệm **14kb** (min+gzipped) và tránh chi phí biên dịch lúc chạy.

- Be cautious of size when introducing new dependencies! In real-world applications, bloated bundles are most often a result of introducing heavy dependencies without realizing it.

  - If using a build step, prefer dependencies that offer ES module formats and are tree-shaking friendly. For example, prefer `lodash-es` over `lodash`.

  - Check a dependency's size and evaluate whether it is worth the functionality it provides. Note if the dependency is tree-shaking friendly, the actual size increase will depend on the APIs you actually import from it. Tools like [bundlejs.com](https://bundlejs.com/) can be used for quick checks, but measuring with your actual build setup will always be the most accurate.

- If you are using Vue primarily for progressive enhancement and prefer to avoid a build step, consider using [petite-vue](https://github.com/vuejs/petite-vue) (only **6kb**) instead.

### Code Splitting {#code-splitting}

Code splitting là việc build tool chia bundle thành nhiều chunk nhỏ, có thể tải theo nhu cầu hoặc song song. Với code splitting hợp lý, phần cần cho tải trang được tải ngay, còn các chunk bổ sung chỉ lazy load khi cần, giúp cải thiện hiệu năng.

Bundlers like Rollup (which Vite is based upon) or webpack can automatically create split chunks by detecting the ESM dynamic import syntax:

```js
// lazy.js and its dependencies will be split into a separate chunk
// and only loaded when `loadLazy()` is called.
function loadLazy() {
  return import('./lazy.js')
}
```

Lazy loading is best used on features that are not immediately needed after initial page load. In Vue applications, this can be used in combination with Vue's [Async Component](/guide/components/async) feature to create split chunks for component trees:

```js
import { defineAsyncComponent } from 'vue'

// a separate chunk is created for Foo.vue and its dependencies.
// it is only fetched on demand when the async component is
// rendered on the page.
const Foo = defineAsyncComponent(() => import('./Foo.vue'))
```

Với ứng dụng dùng Vue Router, rất khuyến nghị lazy load cho route component. Vue Router hỗ trợ lazy load riêng, khác với `defineAsyncComponent`. Xem [Lazy Loading Routes](https://router.vuejs.org/guide/advanced/lazy-loading.html).

## Update Optimizations {#update-optimizations}

### Props Stability {#props-stability}

Trong Vue, child component chỉ cập nhật khi có ít nhất một prop nhận vào thay đổi. Xem ví dụ:

```vue-html
<ListItem
  v-for="item in list"
  :id="item.id"
  :active-id="activeId" />
```

Inside the `<ListItem>` component, it uses its `id` and `activeId` props to determine whether it is the currently active item. While this works, the problem is that whenever `activeId` changes, **every** `<ListItem>` in the list has to update!

Lý tưởng là chỉ item có trạng thái active đổi mới cập nhật. Ta đạt được bằng cách chuyển tính toán trạng thái active lên parent, và để `<ListItem>` nhận trực tiếp prop `active`:

```vue-html
<ListItem
  v-for="item in list"
  :id="item.id"
  :active="item.id === activeId" />
```

Now, for most components the `active` prop will remain the same when `activeId` changes, so they no longer need to update. In general, the idea is keeping the props passed to child components as stable as possible.

### `v-once` {#v-once}

`v-once` là directive built‑in để render nội dung phụ thuộc dữ liệu lúc chạy nhưng không cần cập nhật. Toàn bộ nhánh con sẽ bị bỏ qua ở các lần cập nhật sau. Xem [API reference](/api/built-in-directives#v-once).

### `v-memo` {#v-memo}

`v-memo` là directive built‑in để có thể bỏ qua cập nhật có điều kiện cho các nhánh lớn hoặc danh sách `v-for`. Xem [API reference](/api/built-in-directives#v-memo).

### Computed Stability {#computed-stability}

In Vue 3.4 and above, a computed property will only trigger effects when its computed value has changed from the previous one. For example, the following `isEven` computed only triggers effects if the returned value has changed from `true` to `false`, or vice-versa:

```js
const count = ref(0)
const isEven = computed(() => count.value % 2 === 0)

watchEffect(() => console.log(isEven.value)) // true

// will not trigger new logs because the computed value stays `true`
count.value = 2
count.value = 4
```

Điều này giảm các lần kích hoạt effect không cần thiết, nhưng không hiệu quả nếu computed tạo object mới mỗi lần tính:

```js
const computedObj = computed(() => {
  return {
    isEven: count.value % 2 === 0
  }
})
```

Vì tạo object mới mỗi lần, giá trị mới luôn khác giá trị cũ. Dù `isEven` giữ nguyên, Vue không biết được trừ khi so sánh sâu giữa giá trị cũ và mới — việc này tốn kém và thường không đáng.

Thay vào đó, ta có thể tối ưu bằng cách tự so sánh giá trị mới với cũ và có điều kiện trả về giá trị cũ nếu biết không có gì đổi:

```js
const computedObj = computed((oldValue) => {
  const newValue = {
    isEven: count.value % 2 === 0
  }
  if (oldValue && oldValue.isEven === newValue.isEven) {
    return oldValue
  }
  return newValue
})
```

[Try it in the playground](https://play.vuejs.org/#eNqVVMtu2zAQ/JUFgSZK4UpuczMkow/40AJ9IC3aQ9mDIlG2EokUyKVt1PC/d0lKtoEminMQQC1nZ4c7S+7Yu66L11awGUtNoesOwQi03ZzLuu2URtiBFtUECtV2FkU5gU2OxWpRVaJA2EOlVQuXxHDJJZeFkgYJayVC5hKj6dUxLnzSjZXmV40rZfFrh3Vb/82xVrLH//5DCQNNKPkweNiNVFP+zBsrIJvDjksgGrRahjVAbRZrIWdBVLz2yBfwBrIsg6mD7LncPyryfIVnywupUmz68HOEEqqCI+XFBQzrOKR79MDdx66GCn1jhpQDZx8f0oZ+nBgdRVcH/aMuBt1xZ80qGvGvh/X6nlXwnGpPl6qsLLxTtitzFFTNl0oSN/79AKOCHHQuS5pw4XorbXsr9ImHZN7nHFdx1SilI78MeOJ7Ca+nbvgd+GgomQOv6CNjSQqXaRJuHd03+kHRdg3JoT+A3a7XsfcmpbcWkQS/LZq6uM84C8o5m4fFuOg0CemeOXXX2w2E6ylsgj2gTgeYio/f1l5UEqj+Z3yC7lGuNDlpApswNNTrql7Gd0ZJeqW8TZw5t+tGaMdDXnA2G4acs7xp1OaTj6G2YjLEi5Uo7h+I35mti3H2TQsj9Jp6etjDXC8Fhu3F9y9iS+vDZqtK2xB6ZPNGGNVYpzHA3ltZkuwTnFf70b+1tVz+MIstCmmGQzmh/p56PGf00H4YOfpR7nV8PTxubP8P2GAP9Q==)

Lưu ý nên luôn thực hiện tính toán đầy đủ trước khi so sánh và trả về giá trị cũ, để cùng một tập dependency được thu thập ở mỗi lần chạy.

## General Optimizations {#general-optimizations}

> The following tips affect both page load and update performance.

### Virtualize Large Lists {#virtualize-large-lists}

Một vấn đề hiệu năng phổ biến là render danh sách lớn. Dù framework hiệu năng ra sao, render danh sách hàng nghìn item **sẽ** chậm vì số lượng DOM node lớn trình duyệt phải xử lý.

Tuy nhiên, không nhất thiết render tất cả node ngay lập tức. Thường thì màn hình chỉ hiển thị được một phần nhỏ danh sách. Ta có thể cải thiện mạnh bằng **list virtualization** — kỹ thuật chỉ render các item đang ở trong hoặc gần viewport.

Việc hiện thực virtualization không dễ, may mắn là có thư viện cộng đồng bạn có thể dùng trực tiếp:

- [vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller)
- [vue-virtual-scroll-grid](https://github.com/rocwang/vue-virtual-scroll-grid)
- [vueuc/VVirtualList](https://github.com/07akioni/vueuc)

### Reduce Reactivity Overhead for Large Immutable Structures {#reduce-reactivity-overhead-for-large-immutable-structures}

Reactivity của Vue mặc định là deep. Điều này trực quan cho quản lý state, nhưng tạo overhead khi dữ liệu lớn, vì mỗi truy cập thuộc tính kích hoạt proxy trap để theo dõi phụ thuộc. Điều này thường thấy khi xử lý mảng lớn các object lồng sâu, nơi một lần render cần truy cập 100,000+ thuộc tính — chỉ ảnh hưởng các use case rất đặc thù.

Vue cung cấp lối thoát khỏi deep reactivity bằng [`shallowRef()`](/api/reactivity-advanced#shallowref) và [`shallowReactive()`](/api/reactivity-advanced#shallowreactive). Các API shallow tạo state chỉ reactive ở cấp gốc, và để nguyên các object lồng bên trong. Điều này giúp truy cập thuộc tính lồng nhanh hơn, đổi lại cần coi các object lồng là immutable và chỉ cập nhật bằng cách thay root state:

```js
const shallowArray = shallowRef([
  /* big list of deep objects */
])

// this won't trigger updates...
shallowArray.value.push(newObject)
// this does:
shallowArray.value = [...shallowArray.value, newObject]

// this won't trigger updates...
shallowArray.value[0].foo = 1
// this does:
shallowArray.value = [
  {
    ...shallowArray.value[0],
    foo: 1
  },
  ...shallowArray.value.slice(1)
]
```

### Avoid Unnecessary Component Abstractions {#avoid-unnecessary-component-abstractions}

Đôi khi ta tạo [renderless component](/guide/components/slots#renderless-components) hoặc higher‑order component (component render component khác với prop bổ sung) để trừu tượng/ tổ chức code. Điều này không sai, nhưng nhớ rằng instance component tốn kém hơn DOM node thuần, và tạo quá nhiều do mô hình trừu tượng sẽ tăng chi phí hiệu năng.

Giảm chỉ vài instance sẽ không thấy rõ, nên đừng lo nếu component chỉ render vài lần. Nên cân nhắc tối ưu này ở danh sách lớn: danh sách 100 item, mỗi item chứa nhiều component con — loại bỏ một lớp trừu tượng không cần thiết có thể giảm hàng trăm instance component.
