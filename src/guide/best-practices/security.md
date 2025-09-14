# Security {#security}

## Reporting Vulnerabilities {#reporting-vulnerabilities}

Khi một lỗ hổng được báo cáo, đó lập tức là ưu tiên hàng đầu của chúng tôi, với một người đóng góp toàn thời gian tập trung xử lý. Để báo cáo lỗ hổng, vui lòng email [security@vuejs.org](mailto:security@vuejs.org).

Dù hiếm khi phát hiện lỗ hổng mới, chúng tôi khuyến nghị luôn dùng phiên bản mới nhất của Vue và các thư viện chính thức đi kèm để đảm bảo ứng dụng an toàn tối đa.

## Rule No.1: Never Use Non-trusted Templates {#rule-no-1-never-use-non-trusted-templates}

Quy tắc bảo mật cơ bản nhất khi dùng Vue là **không bao giờ dùng nội dung không tin cậy làm template của component**. Làm vậy tương đương cho phép thực thi JavaScript tùy ý trong ứng dụng — tệ hơn, có thể dẫn tới xâm nhập máy chủ nếu code chạy trong server‑side rendering. Ví dụ:

```js
Vue.createApp({
  template: `<div>` + userProvidedString + `</div>` // NEVER DO THIS
}).mount('#app')
```

Template của Vue được biên dịch thành JavaScript, và các biểu thức trong template sẽ được thực thi như một phần quá trình render. Dù các biểu thức được đánh giá trong ngữ cảnh render cụ thể, do sự phức tạp của môi trường thực thi toàn cục, không thực tế để framework như Vue có thể bảo vệ bạn hoàn toàn khỏi mã độc mà không phải trả giá hiệu năng lớn. Cách đơn giản nhất để tránh hoàn toàn vấn đề này là đảm bảo nội dung template của bạn luôn đáng tin cậy và do bạn kiểm soát.

## What Vue Does to Protect You {#what-vue-does-to-protect-you}

### HTML content {#html-content}

Dù dùng template hay render function, nội dung đều được escape tự động. Tức là trong template:

```vue-html
<h1>{{ userProvidedString }}</h1>
```

if `userProvidedString` contained:

```js
'<script>alert("hi")</script>'
```

then it would be escaped to the following HTML:

```vue-html
&lt;script&gt;alert(&quot;hi&quot;)&lt;/script&gt;
```

nhờ đó ngăn tiêm script. Việc escape dùng API gốc của trình duyệt như `textContent`, nên chỉ có lỗ hổng nếu chính trình duyệt có lỗ hổng.

### Attribute bindings {#attribute-bindings}

Tương tự, binding attribute động cũng được escape tự động. Trong template:

```vue-html
<h1 :title="userProvidedString">
  hello
</h1>
```

if `userProvidedString` contained:

```js
'" onclick="alert(\'hi\')'
```

then it would be escaped to the following HTML:

```vue-html
&quot; onclick=&quot;alert('hi')
```

nhờ đó ngăn việc đóng thuộc tính `title` để tiêm HTML tùy ý. Việc escape dùng API gốc như `setAttribute`, nên chỉ có lỗ hổng nếu trình duyệt có lỗ hổng.

## Potential Dangers {#potential-dangers}

Trong mọi ứng dụng web, cho phép nội dung người dùng cung cấp (chưa được làm sạch) được thực thi như HTML/CSS/JS là nguy hiểm, nên tránh khi có thể. Tuy nhiên đôi khi có thể chấp nhận rủi ro.

Ví dụ, CodePen và JSFiddle cho phép thực thi nội dung do người dùng cung cấp, nhưng trong ngữ cảnh được kỳ vọng và sandbox một phần trong iframe. Khi một tính năng quan trọng vốn cần chấp nhận rủi ro, đội ngũ của bạn cần cân nhắc giữa giá trị tính năng và kịch bản xấu nhất mà lỗ hổng có thể gây ra.

### HTML Injection {#html-injection}

Như đã đề cập, Vue tự động escape nội dung HTML, ngăn việc vô tình tiêm HTML có thể thực thi vào ứng dụng. Tuy nhiên, **khi bạn chắc chắn HTML là an toàn**, bạn có thể render HTML một cách tường minh:

- Using a template:

  ```vue-html
  <div v-html="userProvidedHtml"></div>
  ```

- Using a render function:

  ```js
  h('div', {
    innerHTML: this.userProvidedHtml
  })
  ```

- Using a render function with JSX:

  ```jsx
  <div innerHTML={this.userProvidedHtml}></div>
  ```

:::warning
HTML do người dùng cung cấp không bao giờ 100% an toàn trừ khi ở trong iframe sandbox hoặc ở khu vực chỉ chính người đó nhìn thấy. Ngoài ra, cho phép người dùng viết template Vue của riêng họ mang lại rủi ro tương tự.
:::

### URL Injection {#url-injection}

In a URL like this:

```vue-html
<a :href="userProvidedUrl">
  click me
</a>
```

Có rủi ro bảo mật nếu URL chưa được “làm sạch” để ngăn thực thi JavaScript qua `javascript:`. Có thư viện như [sanitize-url](https://www.npmjs.com/package/@braintree/sanitize-url) hỗ trợ, nhưng lưu ý: nếu bạn làm sạch URL ở frontend, tức đã có vấn đề bảo mật. **URL do người dùng cung cấp nên luôn được backend làm sạch trước khi lưu vào database.** Khi đó mọi client đều tránh được vấn đề, kể cả ứng dụng di động native. Cũng lưu ý ngay cả với URL đã làm sạch, Vue không thể đảm bảo chúng dẫn đến điểm đến an toàn.

### Style Injection {#style-injection}

Looking at this example:

```vue-html
<a
  :href="sanitizedUrl"
  :style="userProvidedStyles"
>
  click me
</a>
```

Giả sử `sanitizedUrl` đã được làm sạch, chắc chắn là URL thật chứ không phải JavaScript. Với `userProvidedStyles`, kẻ xấu vẫn có thể cung cấp CSS để “clickjacking”, ví dụ biến liên kết thành hộp trong suốt đè lên nút “Log in”. Nếu `https://user-controlled-website.com/` được dựng giống trang đăng nhập của bạn, họ có thể lấy được thông tin đăng nhập thật.

Cho phép nội dung người dùng cho phần tử `<style>` còn nguy hiểm hơn, trao quyền kiểm soát toàn bộ trang. Vì vậy Vue ngăn render thẻ style trong template, như:

```vue-html
<style>{{ userProvidedStyles }}</style>
```

Để bảo vệ người dùng khỏi clickjacking, nên chỉ cho phép kiểm soát CSS đầy đủ trong iframe sandbox. Hoặc khi cho phép điều khiển qua style binding, dùng [object syntax](/guide/essentials/class-and-style#binding-to-objects-1) và chỉ cho phép giá trị cho các thuộc tính an toàn, như sau:

```vue-html
<a
  :href="sanitizedUrl"
  :style="{
    color: userProvidedColor,
    background: userProvidedBackground
  }"
>
  click me
</a>
```

### JavaScript Injection {#javascript-injection}

Chúng tôi khuyến cáo không bao giờ render phần tử `<script>` với Vue, vì template và render function không nên có side effect. Tuy nhiên, đây không phải cách duy nhất để đưa chuỗi sẽ được đánh giá như JavaScript khi chạy.

Mọi phần tử HTML đều có thuộc tính nhận chuỗi JavaScript như `onclick`, `onfocus`, `onmouseenter`. Bind JavaScript do người dùng cung cấp vào các thuộc tính này là rủi ro bảo mật, nên tránh.

:::warning
JavaScript do người dùng cung cấp không bao giờ 100% an toàn trừ khi ở iframe sandbox hoặc khu vực chỉ chính họ nhìn thấy.
:::

Sometimes we receive vulnerability reports on how it's possible to do cross-site scripting (XSS) in Vue templates. In general, we do not consider such cases to be actual vulnerabilities because there's no practical way to protect developers from the two scenarios that would allow XSS:

1. The developer is explicitly asking Vue to render user-provided, unsanitized content as Vue templates. This is inherently unsafe, and there's no way for Vue to know the origin.

2. The developer is mounting Vue to an entire HTML page which happens to contain server-rendered and user-provided content. This is fundamentally the same problem as \#1, but sometimes devs may do it without realizing it. This can lead to possible vulnerabilities where the attacker provides HTML which is safe as plain HTML but unsafe as a Vue template. The best practice is to **never mount Vue on nodes that may contain server-rendered and user-provided content**.

## Best Practices {#best-practices}

The general rule is that if you allow unsanitized, user-provided content to be executed (as either HTML, JavaScript, or even CSS), you might open yourself up to attacks. This advice actually holds true whether using Vue, another framework, or even no framework.

Beyond the recommendations made above for [Potential Dangers](#potential-dangers), we also recommend familiarizing yourself with these resources:

- [HTML5 Security Cheat Sheet](https://html5sec.org/)
- [OWASP's Cross Site Scripting (XSS) Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

Then use what you learn to also review the source code of your dependencies for potentially dangerous patterns, if any of them include 3rd-party components or otherwise influence what's rendered to the DOM.

## Backend Coordination {#backend-coordination}

HTTP security vulnerabilities, such as cross-site request forgery (CSRF/XSRF) and cross-site script inclusion (XSSI), are primarily addressed on the backend, so they aren't a concern of Vue's. However, it's still a good idea to communicate with your backend team to learn how to best interact with their API, e.g., by submitting CSRF tokens with form submissions.

## Server-Side Rendering (SSR) {#server-side-rendering-ssr}

There are some additional security concerns when using SSR, so make sure to follow the best practices outlined throughout [our SSR documentation](/guide/scaling-up/ssr) to avoid vulnerabilities.
