# Routing {#routing}

## Client-Side vs. Server-Side Routing {#client-side-vs-server-side-routing}

Routing phía server nghĩa là server gửi phản hồi dựa trên URL người dùng truy cập. Khi nhấp vào liên kết trong ứng dụng render từ server truyền thống, trình duyệt nhận HTML từ server và tải lại toàn bộ trang với HTML mới.

Trong [Single‑Page Application](https://developer.mozilla.org/en-US/docs/Glossary/SPA) (SPA), JavaScript phía client có thể chặn điều hướng, tải dữ liệu mới động, và cập nhật trang hiện tại mà không cần reload toàn trang. Điều này thường mang lại trải nghiệm nhạy hơn, đặc biệt cho trường hợp giống “ứng dụng” thực thụ, nơi người dùng tương tác nhiều trong thời gian dài.

Trong SPA, “routing” được thực hiện ở phía client, trong trình duyệt. Client‑side router chịu trách nhiệm quản lý view được render của ứng dụng bằng các API trình duyệt như [History API](https://developer.mozilla.org/en-US/docs/Web/API/History) hoặc [`hashchange` event](https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event).

## Official Router {#official-router}

<!-- TODO update links -->
<div>
  <VueSchoolLink href="https://vueschool.io/courses/vue-router-4-for-everyone" title="Free Vue Router Course">
    Watch a Free Video Course on Vue School
  </VueSchoolLink>
</div>

Vue phù hợp để xây SPA. Với đa số SPA, khuyến nghị dùng [Vue Router](https://github.com/vuejs/router) chính thức. Xem chi tiết tại [tài liệu Vue Router](https://router.vuejs.org/).

## Simple Routing from Scratch {#simple-routing-from-scratch}

Nếu bạn chỉ cần routing rất đơn giản và không muốn dùng thư viện router đầy đủ, có thể kết hợp [Dynamic Components](/guide/essentials/component-basics#dynamic-components) và cập nhật state hiện tại bằng cách lắng nghe [`hashchange` event](https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event) của trình duyệt hoặc dùng [History API](https://developer.mozilla.org/en-US/docs/Web/API/History).

Ví dụ tối giản:

<div class="composition-api">

```vue
<script setup>
import { ref, computed } from 'vue'
import Home from './Home.vue'
import About from './About.vue'
import NotFound from './NotFound.vue'

const routes = {
  '/': Home,
  '/about': About
}

const currentPath = ref(window.location.hash)

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.hash
})

const currentView = computed(() => {
  return routes[currentPath.value.slice(1) || '/'] || NotFound
})
</script>

<template>
  <a href="#/">Home</a> |
  <a href="#/about">About</a> |
  <a href="#/non-existent-path">Broken Link</a>
  <component :is="currentView" />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNptUk1vgkAQ/SsTegAThZp4MmhikzY9mKanXkoPWxjLRpgly6JN1P/eWb5Eywlm572ZN2/m5GyKwj9U6CydsIy1LAyUaKpiHZHMC6UNnEDjbgqxyovKYAIX2GmVg8sktwe9qhzbdz+wga15TW++VWX6fB3dAt6UeVEVJT2me2hhEcWKSgOamVjCCk4RAbiBu6xbT5tI2ML8VDeI6HLlxZXWSOZdmJTJPJB3lJSoo5+pWBipyE9FmU4soU2IJHk+MGUrS4OE2nMtIk4F/aA7BW8Cq3WjYlDbP4isQu4wVp0F1Q1uFH1IPDK+c9cb1NW8B03tyJ//uvhlJmP05hM4n60TX/bb2db0CoNmpbxMDgzmRSYMcgQQCkjZhlXkPASRs7YmhoFYw/k+WXvKiNrTcQgpmuFv7ZOZFSyQ4U9a7ZFgK2lvSTXFDqmIQbCUJTMHFkQOBAwKg16kM3W6O7K3eSs+nbeK+eee1V/XKK0dY4Q3vLhR6uJxMUK8/AFKaB6k)

</div>

<div class="options-api">

```vue
<script>
import Home from './Home.vue'
import About from './About.vue'
import NotFound from './NotFound.vue'

const routes = {
  '/': Home,
  '/about': About
}

export default {
  data() {
    return {
      currentPath: window.location.hash
    }
  },
  computed: {
    currentView() {
      return routes[this.currentPath.slice(1) || '/'] || NotFound
    }
  },
  mounted() {
    window.addEventListener('hashchange', () => {
		  this.currentPath = window.location.hash
		})
  }
}
</script>

<template>
  <a href="#/">Home</a> |
  <a href="#/about">About</a> |
  <a href="#/non-existent-path">Broken Link</a>
  <component :is="currentView" />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNptUstO6zAQ/ZVR7iKtVJKLxCpKK3Gli1ggxIoNZmGSKbFoxpEzoUi0/87YeVBKNonHPmfOmcdndN00yXuHURblbeFMwxtFpm6sY7i1NcLW2RriJPWBB8bT8/WL7Xh6D9FPwL3lG9tROWHGiwGmqLDUMjhhYgtr+FQEEKdxFqRXfaR9YrkKAoqOnocfQaDEre523PNKzXqx7M8ADrlzNEYAReccEj9orjLYGyrtPtnZQrOxlFS6rXqgZJdPUC5s3YivMhuTDCkeDe6/dSalvognrkybnIgl7c4UuLhcwuHgS3v2/7EPvzRruRXJ7/SDU12W/98l451pGQndIvaWi0rTK8YrEPx64ymKFQOce5DOzlfs4cdlkA+NzdNpBSRgrJudZpQIINdQOdyuVfQnVdHGzydP9QYO549hXIII45qHkKUL/Ail8EUjBgX+z9k3JLgz9OZJgeInYElAkJlWmCcDUBGkAsrTyWS0isYV9bv803x1OTiWwzlrWtxZ2lDGDO90mWepV3+vZojHL3QQKQE=)

</div>
