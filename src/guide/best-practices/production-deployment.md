# Production Deployment {#production-deployment}

## Development vs. Production {#development-vs-production}

Trong giai đoạn phát triển, Vue cung cấp nhiều tính năng để cải thiện trải nghiệm:

- Warning for common errors and pitfalls
- Props / events validation
- [Reactivity debugging hooks](/guide/extras/reactivity-in-depth#reactivity-debugging)
- Devtools integration

Tuy nhiên, các tính năng này không còn cần thiết ở môi trường production. Một số kiểm tra cảnh báo cũng gây một chút chi phí hiệu năng. Khi triển khai production, ta nên loại bỏ các nhánh code chỉ dùng cho phát triển để giảm kích thước payload và tăng hiệu năng.

## Without Build Tools {#without-build-tools}

Nếu bạn dùng Vue không có build tool (từ CDN hoặc script tự host), hãy đảm bảo dùng production build (các file dist kết thúc bằng `.prod.js`) khi triển khai production. Production build đã được minify sẵn và loại bỏ code chỉ dùng cho phát triển.

- If using global build (accessing via the `Vue` global): use `vue.global.prod.js`.
- If using ESM build (accessing via native ESM imports): use `vue.esm-browser.prod.js`.

Consult the [dist file guide](https://github.com/vuejs/core/tree/main/packages/vue#which-dist-file-to-use) for more details.

## With Build Tools {#with-build-tools}

Các dự án scaffold qua `create-vue` (dựa trên Vite) hoặc Vue CLI (dựa trên webpack) đã được cấu hình sẵn cho build production.

Nếu dùng thiết lập tùy chỉnh, đảm bảo rằng:

1. `vue` resolves to `vue.runtime.esm-bundler.js`.
2. The [compile time feature flags](/api/compile-time-flags) are properly configured.
3. <code>process.env<wbr>.NODE_ENV</code> is replaced with `"production"` during build.

Tài liệu tham khảo thêm:

- [Vite production build guide](https://vitejs.dev/guide/build.html)
- [Vite deployment guide](https://vitejs.dev/guide/static-deploy.html)
- [Vue CLI deployment guide](https://cli.vuejs.org/guide/deployment.html)

## Tracking Runtime Errors {#tracking-runtime-errors}

[App-level error handler](/api/application#app-config-errorhandler) có thể dùng để báo lỗi tới các dịch vụ theo dõi:

```js
import { createApp } from 'vue'

const app = createApp(...)

app.config.errorHandler = (err, instance, info) => {
  // report error to tracking services
}
```

Các dịch vụ như [Sentry](https://docs.sentry.io/platforms/javascript/guides/vue/) và [Bugsnag](https://docs.bugsnag.com/platforms/javascript/vue/) cũng cung cấp tích hợp chính thức cho Vue.
