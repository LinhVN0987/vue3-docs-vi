# Ways of Using Vue {#ways-of-using-vue}

Chúng tôi tin rằng không có “một cỡ cho tất cả” trên web. Vì thế Vue được thiết kế linh hoạt và có thể áp dụng dần. Tùy use case, Vue có thể được dùng theo các cách khác nhau để đạt cân bằng tối ưu giữa độ phức tạp stack, trải nghiệm phát triển và hiệu năng cuối cùng.

## Standalone Script {#standalone-script}

Vue có thể dùng như một file script độc lập — không cần build step! Nếu backend đã render phần lớn HTML, hoặc logic frontend chưa đủ phức tạp để cần build, đây là cách dễ nhất tích hợp Vue vào stack. Bạn có thể xem Vue như một thay thế mang tính khai báo cho jQuery trong các trường hợp này.

Trước đây chúng tôi cung cấp bản phân phối [petite-vue](https://github.com/vuejs/petite-vue) tối ưu cho progressive enhancement, nhưng hiện không còn được duy trì tích cực (phiên bản cuối ở Vue 3.2.27).

## Embedded Web Components {#embedded-web-components}

Bạn có thể dùng Vue để [xây Web Components tiêu chuẩn](/guide/extras/web-components) và nhúng vào bất kỳ trang HTML nào, bất kể cách render. Tùy chọn này cho phép tận dụng Vue theo cách hoàn toàn trung lập: web component kết quả có thể nhúng vào ứng dụng cũ, HTML tĩnh, thậm chí ứng dụng dùng framework khác.

## Single-Page Application (SPA) {#single-page-application-spa}

Một số ứng dụng cần tương tác phong phú, phiên làm việc dài và stateful logic phức tạp ở frontend. Cách tốt nhất là kiến trúc nơi Vue không chỉ kiểm soát toàn bộ trang mà còn xử lý cập nhật dữ liệu và điều hướng không cần tải lại trang — thường gọi là SPA.

Vue cung cấp thư viện lõi và [hệ sinh thái tooling](/guide/scaling-up/tooling) toàn diện với trải nghiệm phát triển tuyệt vời cho SPA hiện đại, bao gồm:

- Client-side router
- Blazing fast build tool chain
- IDE support
- Browser devtools
- TypeScript integrations
- Testing utilities

SPA thường yêu cầu backend cung cấp API endpoint — nhưng bạn cũng có thể ghép Vue với [Inertia.js](https://inertiajs.com) để có lợi ích SPA trong khi vẫn giữ mô hình phát triển tập trung ở server.

## Fullstack / SSR {#fullstack-ssr}

SPA thuần client gặp vấn đề khi ứng dụng nhạy với SEO và time‑to‑content, vì trình duyệt nhận HTML gần như rỗng và phải chờ JavaScript tải xong mới render.

Vue cung cấp API hạng nhất để “render” ứng dụng trên server thành chuỗi HTML. Server có thể gửi HTML đã render để người dùng thấy nội dung ngay trong khi JavaScript đang tải; sau đó Vue sẽ “hydrate” ở client để tương tác. Đây là [Server‑Side Rendering (SSR)](/guide/scaling-up/ssr) và cải thiện đáng kể chỉ số Core Web Vitals như [LCP](https://web.dev/lcp/).

Có các framework cấp cao dựa trên Vue như [Nuxt](https://nuxt.com/) cho phép bạn phát triển fullstack bằng Vue và JavaScript.

## JAMStack / SSG {#jamstack-ssg}

Server‑side rendering có thể thực hiện trước nếu dữ liệu là tĩnh. Ta có thể pre‑render toàn bộ ứng dụng thành HTML và phục vụ như file tĩnh. Điều này cải thiện hiệu năng và đơn giản hóa triển khai vì không cần render động mỗi request. Vue vẫn có thể hydrate để cung cấp tương tác phong phú. Kỹ thuật này gọi là Static‑Site Generation (SSG), còn được biết đến là [JAMStack](https://jamstack.org/what-is-jamstack/).

Có hai dạng SSG: single‑page và multi‑page. Cả hai đều pre‑render thành HTML tĩnh, khác biệt là:

- After the initial page load, a single-page SSG "hydrates" the page into an SPA. This requires more upfront JS payload and hydration cost, but subsequent navigations will be faster, since it only needs to partially update the page content instead of reloading the entire page.

- A multi-page SSG loads a new page on every navigation. The upside is that it can ship minimal JS - or no JS at all if the page requires no interaction! Some multi-page SSG frameworks such as [Astro](https://astro.build/) also support "partial hydration" - which allows you to use Vue components to create interactive "islands" inside static HTML.

Single‑page SSG phù hợp nếu bạn cần tương tác đáng kể, phiên làm việc dài, hoặc phần tử/state được giữ qua lần điều hướng. Ngược lại, multi‑page SSG sẽ phù hợp hơn.

Team Vue cũng duy trì static‑site generator [VitePress](https://vitepress.dev/), công cụ đang vận hành chính website này! VitePress hỗ trợ cả hai dạng SSG. [Nuxt](https://nuxt.com/) cũng hỗ trợ SSG. Bạn thậm chí có thể trộn SSR và SSG cho các route khác nhau trong cùng ứng dụng Nuxt.

## Beyond the Web {#beyond-the-web}

Dù Vue chủ yếu thiết kế cho web, nó không chỉ giới hạn trong trình duyệt. Bạn có thể:

- Build desktop apps with [Electron](https://www.electronjs.org/) or [Wails](https://wails.io)
- Build mobile apps with [Ionic Vue](https://ionicframework.com/docs/vue/overview)
- Build desktop and mobile apps from the same codebase with [Quasar](https://quasar.dev/) or [Tauri](https://tauri.app)
- Build 3D WebGL experiences with [TresJS](https://tresjs.org/)
- Use Vue's [Custom Renderer API](/api/custom-renderer) to build custom renderers, like those for [the terminal](https://github.com/vue-terminal/vue-termui)!
