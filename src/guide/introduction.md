---
footer: false
---

# Introduction {#introduction}

:::info Bạn đang đọc tài liệu cho Vue 3!

- Hỗ trợ cho Vue 2 đã kết thúc vào **31/12/2023**. Tìm hiểu thêm về [Vue 2 EOL](https://v2.vuejs.org/eol/).
- Đang nâng cấp từ Vue 2? Xem [Migration Guide](https://v3-migration.vuejs.org/).
  :::

<style src="@theme/styles/vue-mastery.css"></style>
<div class="vue-mastery-link">
  <a href="https://www.vuemastery.com/courses/" target="_blank">
    <div class="banner-wrapper">
      <img class="banner" alt="Vue Mastery banner" width="96px" height="56px" src="https://storage.googleapis.com/vue-mastery.appspot.com/flamelink/media/vuemastery-graphical-link-96x56.png" />
    </div>
    <p class="description">Học Vue qua các video hướng dẫn trên <span>VueMastery.com</span></p>
    <div class="logo-wrapper">
        <img alt="Vue Mastery Logo" width="25px" src="https://storage.googleapis.com/vue-mastery.appspot.com/flamelink/media/vue-mastery-logo.png" />
    </div>
  </a>
</div>

## What is Vue? {#what-is-vue}

Vue (phát âm /vjuː/, như **view**) là một JavaScript framework để xây dựng giao diện người dùng (UI). Nó được xây dựng trên các chuẩn HTML, CSS và JavaScript, cung cấp mô hình lập trình khai báo dựa trên component giúp bạn phát triển UI hiệu quả cho mọi mức độ phức tạp.

Dưới đây là một ví dụ tối giản:

<div class="options-api">

```js
import { createApp } from 'vue'

createApp({
  data() {
    return {
      count: 0
    }
  }
}).mount('#app')
```

</div>
<div class="composition-api">

```js
import { createApp, ref } from 'vue'

createApp({
  setup() {
    return {
      count: ref(0)
    }
  }
}).mount('#app')
```

</div>

```vue-html
<div id="app">
  <button @click="count++">
    Count is: {{ count }}
  </button>
</div>
```

**Result**

<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<div class="demo">
  <button @click="count++">
    Count is: {{ count }}
  </button>
</div>

Ví dụ trên minh họa hai đặc trưng cốt lõi của Vue:

- **Declarative Rendering**: Vue mở rộng HTML tiêu chuẩn với template syntax cho phép mô tả đầu ra HTML theo cách khai báo dựa trên JavaScript state.

- **Reactivity**: Vue tự động theo dõi thay đổi của JavaScript state và cập nhật DOM hiệu quả khi có thay đổi.

Bạn có thể đã có thắc mắc — đừng lo. Phần còn lại của tài liệu sẽ trình bày chi tiết từng khía cạnh. Trước mắt, hãy tiếp tục đọc để nắm cái nhìn tổng quan về những gì Vue mang lại.

:::tip Prerequisites
Phần tài liệu còn lại giả định bạn đã quen thuộc ở mức cơ bản với HTML, CSS và JavaScript. Nếu bạn hoàn toàn mới với frontend development, có lẽ chưa nên bắt đầu ngay với một framework — hãy nắm vững các nền tảng rồi quay lại! Bạn có thể tự kiểm tra mức độ hiểu biết qua các tổng quan về [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript), [HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML) và [CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps) nếu cần. Kinh nghiệm với framework khác là hữu ích nhưng không bắt buộc.
:::

## The Progressive Framework {#the-progressive-framework}

Vue là một framework và ecosystem bao phủ hầu hết các tính năng thường gặp trong frontend development. Tuy nhiên, thế giới web vô cùng đa dạng — những gì chúng ta xây dựng có thể khác nhau rất nhiều về hình thức và quy mô. Vì thế, Vue được thiết kế linh hoạt và có thể áp dụng dần dần. Tùy trường hợp sử dụng, Vue có thể được dùng theo nhiều cách:

- Tăng cường HTML tĩnh mà không cần build step
- Nhúng dưới dạng Web Components trên bất kỳ trang nào
- Single-Page Application (SPA)
- Fullstack / Server-Side Rendering (SSR)
- Jamstack / Static Site Generation (SSG)
- Nhắm đến desktop, mobile, WebGL, thậm chí cả terminal

Nếu bạn thấy các khái niệm này khó nhằn, đừng lo! Tutorial và guide chỉ yêu cầu kiến thức HTML và JavaScript cơ bản; bạn vẫn có thể theo kịp mà không cần là chuyên gia về chúng.

Nếu bạn là developer giàu kinh nghiệm muốn tích hợp Vue vào stack của mình, hoặc tò mò các thuật ngữ này nghĩa là gì, chúng tôi bàn thêm chi tiết trong [Ways of Using Vue](/guide/extras/ways-of-using-vue).

Mặc dù rất linh hoạt, kiến thức cốt lõi về cách Vue hoạt động được dùng chung cho tất cả các cách dùng trên. Ngay cả khi bạn chỉ mới bắt đầu, kiến thức tích lũy trên đường đi sẽ vẫn hữu ích khi bạn hướng đến các mục tiêu tham vọng hơn. Nếu bạn là người dùng dày dạn, bạn có thể chọn cách tận dụng Vue tối ưu dựa trên vấn đề cần giải quyết, đồng thời vẫn giữ năng suất. Đó là lý do chúng tôi gọi Vue là "The Progressive Framework": một framework có thể lớn lên cùng bạn và thích ứng với nhu cầu của bạn.

## Single-File Components {#single-file-components}

Trong hầu hết các dự án Vue dùng build tools, chúng ta viết component bằng định dạng giống HTML gọi là **Single-File Component** (còn gọi là `*.vue`, viết tắt **SFC**). Đúng như tên gọi, một SFC đóng gói logic (JavaScript), template (HTML) và styles (CSS) của component trong một file duy nhất. Dưới đây là ví dụ trước, viết theo định dạng SFC:

<div class="options-api">

```vue
<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

</div>
<div class="composition-api">

```vue
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

</div>

SFC là một đặc trưng quan trọng của Vue và là cách được khuyến nghị để viết component **khi** use case của bạn cần build setup. Bạn có thể tìm hiểu thêm về [cách làm và lý do dùng SFC](/guide/scaling-up/sfc) ở phần chuyên biệt — còn hiện tại, chỉ cần biết rằng Vue sẽ xử lý toàn bộ phần thiết lập build tools cho bạn.

## API Styles {#api-styles}

Component trong Vue có thể được viết theo hai phong cách API: **Options API** và **Composition API**.

### Options API {#options-api}

Với Options API, ta định nghĩa logic của component bằng một object các tùy chọn như `data`, `methods` và `mounted`. Các thuộc tính được định nghĩa bởi các tùy chọn sẽ xuất hiện trên `this` bên trong các hàm, trỏ đến instance của component:

```vue
<script>
export default {
  // Properties returned from data() become reactive state
  // and will be exposed on `this`.
  data() {
    return {
      count: 0
    }
  },

  // Methods are functions that mutate state and trigger updates.
  // They can be bound as event handlers in templates.
  methods: {
    increment() {
      this.count++
    }
  },

  // Lifecycle hooks are called at different stages
  // of a component's lifecycle.
  // This function will be called when the component is mounted.
  mounted() {
    console.log(`The initial count is ${this.count}.`)
  }
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNptkMFqxCAQhl9lkB522ZL0HNKlpa/Qo4e1ZpLIGhUdl5bgu9es2eSyIMio833zO7NP56pbRNawNkivHJ25wV9nPUGHvYiaYOYGoK7Bo5CkbgiBBOFy2AkSh2N5APmeojePCkDaaKiBt1KnZUuv3Ky0PppMsyYAjYJgigu0oEGYDsirYUAP0WULhqVrQhptF5qHQhnpcUJD+wyQaSpUd/Xp9NysVY/yT2qE0dprIS/vsds5Mg9mNVbaDofL94jZpUgJXUKBCvAy76ZUXY53CTd5tfX2k7kgnJzOCXIF0P5EImvgQ2olr++cbRE4O3+t6JxvXj0ptXVpye1tvbFY+ge/NJZt)

### Composition API {#composition-api}

Với Composition API, ta định nghĩa logic của component bằng các API function được import. Trong SFC, Composition API thường được dùng cùng [`<script setup>`](/api/sfc-script-setup). Thuộc tính `setup` là một gợi ý để Vue thực hiện các biến đổi ở compile-time, cho phép dùng Composition API với ít boilerplate hơn. Ví dụ, các import và biến/hàm top‑level được khai báo trong `<script setup>` có thể dùng trực tiếp trong template.

Dưới đây là cùng một component, dùng đúng template đó, nhưng viết bằng Composition API và `<script setup>`:

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

[Try it in the Playground](https://play.vuejs.org/#eNpNkMFqwzAQRH9lMYU4pNg9Bye09NxbjzrEVda2iLwS0spQjP69a+yYHnRYad7MaOfiw/tqSliciybqYDxDRE7+qsiM3gWGGQJ2r+DoyyVivEOGLrgRDkIdFCmqa1G0ms2EELllVKQdRQa9AHBZ+PLtuEm7RCKVd+ChZRjTQqwctHQHDqbvMUDyd7mKip4AGNIBRyQujzArgtW/mlqb8HRSlLcEazrUv9oiDM49xGGvXgp5uT5his5iZV1f3r4HFHvDprVbaxPhZf4XkKub/CDLaep1T7IhGRhHb6WoTADNT2KWpu/aGv24qGKvrIrr5+Z7hnneQnJu6hURvKl3ryL/ARrVkuI=)

### Which to Choose? {#which-to-choose}

Cả hai phong cách API đều đáp ứng tốt các trường hợp sử dụng phổ biến. Chúng là hai giao diện khác nhau chạy trên cùng một hệ thống nền tảng. Thực tế, Options API được triển khai trên nền Composition API! Các khái niệm nền tảng và kiến thức về Vue được chia sẻ giữa hai phong cách.

Options API xoay quanh khái niệm "component instance" (`this` như trong ví dụ), thường phù hợp hơn với tư duy dựa trên lớp của người dùng đến từ ngôn ngữ OOP. Nó cũng thân thiện với người mới nhờ trừu tượng hóa chi tiết về reactivity và áp đặt tổ chức code theo nhóm tùy chọn.

Composition API tập trung vào việc khai báo các biến state reactive trực tiếp trong phạm vi hàm và tổng hợp state từ nhiều hàm để xử lý độ phức tạp. Cách này tự do hơn và cần hiểu reactivity trong Vue để dùng hiệu quả. Đổi lại, tính linh hoạt cho phép các pattern mạnh mẽ để tổ chức và tái sử dụng logic.

Bạn có thể tìm hiểu thêm về so sánh giữa hai phong cách và lợi ích tiềm năng của Composition API trong [Composition API FAQ](/guide/extras/composition-api-faq).

Nếu bạn mới với Vue, khuyến nghị chung của chúng tôi:

- Với mục đích học tập, hãy chọn phong cách mà bạn thấy dễ hiểu hơn. Nhắc lại, phần lớn khái niệm cốt lõi là dùng chung giữa hai phong cách. Bạn luôn có thể học phong cách còn lại sau.

- Với môi trường sản xuất:

  - Chọn Options API nếu bạn không dùng build tools, hoặc dự định dùng Vue chủ yếu cho tình huống ít phức tạp, ví dụ progressive enhancement.

  - Chọn Composition API + Single-File Components nếu bạn định xây dựng ứng dụng hoàn chỉnh với Vue.

Bạn không cần ràng buộc vào một phong cách duy nhất trong giai đoạn học. Phần còn lại của tài liệu sẽ có ví dụ code ở cả hai phong cách khi phù hợp, và bạn có thể chuyển đổi giữa chúng bất cứ lúc nào bằng **API Preference switches** ở đầu sidebar bên trái.

## Still Got Questions? {#still-got-questions}

Xem [FAQ](/about/faq).

## Pick Your Learning Path {#pick-your-learning-path}

Mỗi developer có một cách học khác nhau. Hãy chọn lộ trình học phù hợp với bạn — dù vậy, nếu có thể, chúng tôi khuyến nghị bạn nên đọc qua toàn bộ nội dung.

<div class="vt-box-container next-steps">
  <a class="vt-box" href="/tutorial/">
    <p class="next-steps-link">Try the Tutorial</p>
    <p class="next-steps-caption">Dành cho những ai thích học bằng thực hành.</p>
  </a>
  <a class="vt-box" href="/guide/quick-start.html">
    <p class="next-steps-link">Read the Guide</p>
    <p class="next-steps-caption">Hướng dẫn này đưa bạn qua mọi khía cạnh của framework một cách chi tiết.</p>
  </a>
  <a class="vt-box" href="/examples/">
    <p class="next-steps-link">Check out the Examples</p>
    <p class="next-steps-caption">Khám phá các ví dụ về tính năng cốt lõi và các tác vụ UI phổ biến.</p>
  </a>
</div>
