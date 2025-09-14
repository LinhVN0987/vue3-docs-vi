# Plugins {#plugins}

## Introduction {#introduction}

Plugin là đoạn mã tự chứa thường bổ sung chức năng cấp ứng dụng cho Vue. Cách cài plugin:

```js
import { createApp } from 'vue'

const app = createApp({})

app.use(myPlugin, {
  /* optional options */
})
```

Plugin được định nghĩa hoặc là object có phương thức `install()`, hoặc đơn giản là một hàm đóng vai trò hàm cài đặt. Hàm install nhận [app instance](/api/application) và các tùy chọn truyền cho `app.use()` (nếu có):

```js
const myPlugin = {
  install(app, options) {
    // configure the app
  }
}
```

Không có phạm vi bắt buộc cho plugin, nhưng các tình huống phổ biến hữu ích gồm:

1. Register one or more global components or custom directives with [`app.component()`](/api/application#app-component) and [`app.directive()`](/api/application#app-directive).

2. Make a resource [injectable](/guide/components/provide-inject) throughout the app by calling [`app.provide()`](/api/application#app-provide).

3. Add some global instance properties or methods by attaching them to [`app.config.globalProperties`](/api/application#app-config-globalproperties).

4. A library that needs to perform some combination of the above (e.g. [vue-router](https://github.com/vuejs/vue-router-next)).

## Writing a Plugin {#writing-a-plugin}

Để hiểu cách tạo plugin Vue.js của riêng bạn, ta sẽ tạo phiên bản đơn giản của plugin hiển thị chuỗi `i18n` (viết tắt của [Internationalization](https://en.wikipedia.org/wiki/Internationalization_and_localization)).

Bắt đầu bằng việc tạo object plugin. Nên đặt trong file riêng và export như dưới để tách bạch logic.

```js [plugins/i18n.js]
export default {
  install: (app, options) => {
    // Plugin code goes here
  }
}
```

Ta muốn tạo hàm dịch. Hàm này nhận chuỗi `key` phân tách bằng dấu chấm, dùng để tra chuỗi dịch trong options do người dùng cung cấp. Cách dùng trong template:

```vue-html
<h1>{{ $translate('greetings.hello') }}</h1>
```

Vì hàm này cần khả dụng toàn cục trong mọi template, ta gắn nó vào `app.config.globalProperties` trong plugin:

```js{3-10} [plugins/i18n.js]
export default {
  install: (app, options) => {
    // inject a globally available $translate() method
    app.config.globalProperties.$translate = (key) => {
      // retrieve a nested property in `options`
      // using `key` as the path
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, options)
    }
  }
}
```

Hàm `$translate` nhận chuỗi như `greetings.hello`, tra trong cấu hình người dùng cung cấp và trả về giá trị dịch.

Object chứa các khóa dịch nên được truyền vào plugin lúc cài đặt qua tham số bổ sung cho `app.use()`:

```js
import i18nPlugin from './plugins/i18n'

app.use(i18nPlugin, {
  greetings: {
    hello: 'Bonjour!'
  }
})
```

Now, our initial expression `$translate('greetings.hello')` will be replaced by `Bonjour!` at runtime.

See also: [Augmenting Global Properties](/guide/typescript/options-api#augmenting-global-properties) <sup class="vt-badge ts" />

:::tip
Use global properties scarcely, since it can quickly become confusing if too many global properties injected by different plugins are used throughout an app.
:::

### Provide / Inject with Plugins {#provide-inject-with-plugins}

Plugins also allow us to use `provide` to give plugin users access to a function or attribute. For example, we can allow the application to have access to the `options` parameter to be able to use the translations object.

```js{3} [plugins/i18n.js]
export default {
  install: (app, options) => {
    app.provide('i18n', options)
  }
}
```

Plugin users will now be able to inject the plugin options into their components using the `i18n` key:

<div class="composition-api">

```vue{4}
<script setup>
import { inject } from 'vue'

const i18n = inject('i18n')

console.log(i18n.greetings.hello)
</script>
```

</div>
<div class="options-api">

```js{2}
export default {
  inject: ['i18n'],
  created() {
    console.log(this.i18n.greetings.hello)
  }
}
```

</div>

### Bundle for NPM

If you further want to build and publish your plugin for others to use, see [Vite's section on Library Mode](https://vitejs.dev/guide/build.html#library-mode).
