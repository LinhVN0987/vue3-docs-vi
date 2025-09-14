# Creating a Vue Application {#creating-a-vue-application}

## The application instance {#the-application-instance}

Mỗi ứng dụng Vue bắt đầu bằng việc tạo một **application instance** mới với hàm [`createApp`](/api/application#createapp):

```js
import { createApp } from 'vue'

const app = createApp({
  /* root component options */
})
```

## The Root Component {#the-root-component}

Object mà chúng ta truyền vào `createApp` thực chất là một component. Mỗi ứng dụng đều cần một "root component" có thể chứa các component khác làm con.

Nếu bạn dùng Single‑File Components, thông thường chúng ta import root component từ một file khác:

```js
import { createApp } from 'vue'
// import the root component App from a single-file component.
import App from './App.vue'

const app = createApp(App)
```

Mặc dù nhiều ví dụ trong hướng dẫn này chỉ cần một component, hầu hết ứng dụng thực tế được tổ chức thành cây các component lồng nhau và có thể tái sử dụng. Ví dụ, cây component của ứng dụng Todo có thể như sau:

```
App (root component)
├─ TodoList
│  └─ TodoItem
│     ├─ TodoDeleteButton
│     └─ TodoEditButton
└─ TodoFooter
   ├─ TodoClearButton
   └─ TodoStatistics
```

Ở các phần sau của hướng dẫn, chúng ta sẽ bàn về cách định nghĩa và kết hợp nhiều component với nhau. Trước hết, ta sẽ tập trung vào những gì diễn ra bên trong một component đơn lẻ.

## Mounting the App {#mounting-the-app}

Một application instance sẽ không render gì cho đến khi phương thức `.mount()` được gọi. Nó nhận một tham số "container", có thể là một DOM element thực tế hoặc một selector string:

```html
<div id="app"></div>
```

```js
app.mount('#app')
```

Nội dung của root component sẽ được render bên trong phần tử container. Bản thân phần tử container không được xem là một phần của ứng dụng.

Phương thức `.mount()` luôn nên được gọi sau khi đã hoàn tất mọi cấu hình của app và đăng ký asset. Cũng lưu ý rằng giá trị trả về của nó, khác với các phương thức đăng ký asset, là instance của root component chứ không phải application instance.

### In-DOM Root Component Template {#in-dom-root-component-template}

Template cho root component thường nằm trong chính component đó, nhưng cũng có thể cung cấp template riêng bằng cách viết trực tiếp bên trong container dùng để mount:

```html
<div id="app">
  <button @click="count++">{{ count }}</button>
</div>
```

```js
import { createApp } from 'vue'

const app = createApp({
  data() {
    return {
      count: 0
    }
  }
})

app.mount('#app')
```

Nếu root component không có tùy chọn `template`, Vue sẽ tự động dùng `innerHTML` của container làm template.

In‑DOM template thường được dùng trong các ứng dụng [dùng Vue mà không có build step](/guide/quick-start.html#using-vue-from-cdn). Chúng cũng có thể được dùng cùng các server‑side framework, nơi root template có thể được server tạo ra một cách động.

## App Configurations {#app-configurations}

Application instance cung cấp object `.config` cho phép cấu hình một số tùy chọn ở cấp ứng dụng, ví dụ định nghĩa error handler ở cấp app để bắt lỗi từ mọi component con:

```js
app.config.errorHandler = (err) => {
  /* handle error */
}
```

Application instance cũng cung cấp một số phương thức để đăng ký asset có phạm vi trong app. Ví dụ, đăng ký một component:

```js
app.component('TodoDeleteButton', TodoDeleteButton)
```

Điều này giúp `TodoDeleteButton` có thể được dùng ở bất cứ đâu trong ứng dụng. Chúng ta sẽ bàn về việc đăng ký component và các loại asset khác ở những phần sau. Bạn cũng có thể xem đầy đủ các API của application instance trong [API reference](/api/application).

Hãy đảm bảo áp dụng xong mọi cấu hình của app trước khi mount!

## Multiple application instances {#multiple-application-instances}

Bạn không bị giới hạn chỉ một application instance trên cùng một trang. API `createApp` cho phép nhiều ứng dụng Vue cùng tồn tại trên một trang, mỗi ứng dụng có phạm vi cấu hình và global assets riêng:

```js
const app1 = createApp({
  /* ... */
})
app1.mount('#container-1')

const app2 = createApp({
  /* ... */
})
app2.mount('#container-2')
```

Nếu bạn dùng Vue để tăng cường HTML render từ server và chỉ cần Vue điều khiển một số phần cụ thể của trang lớn, hãy tránh mount một application instance duy nhất cho toàn bộ trang. Thay vào đó, tạo nhiều application instance nhỏ và mount chúng vào các phần tử mà chúng phụ trách.
