# react-recaptcha-net

React 组件，用于 [Google reCAPTCHA v2][reCAPTCHA]。

基于(dozoisch/react-google-recaptcha)[https://github.com/dozoisch/react-google-recaptcha]

强制替换了验证服务地址为`recatptcha.net`用来规避GFW下使用`react-google-recaptcha`遇到的一些奇奇怪怪的问题，同时添加了TypeScript类型支持。

> 被一条龙服务过.jpg


## 安装

```shell
npm install --save react-recaptcha-net
```

## 使用

首先，你需要 [注册一个 API 密钥对][signup]，获取客户端密钥后，便可以使用 `<ReCAPTCHA />`。

默认使用方式会导入一个封装的组件，异步加载 Google reCAPTCHA 脚本，然后初始化一个 `reCAPTCHA`，用户可以与之交互。

示例代码：
```jsx
import ReCAPTCHA from "react-google-recaptcha";

function onChange(value) {
  console.log("Captcha 值:", value);
}

ReactDOM.render(
  <ReCAPTCHA
    sitekey="你的客户端站点密钥"
    onChange={onChange}
  />,
  document.body
);
```

### 组件属性

用于自定义渲染的属性：

| 名称              | 类型    | 描述                                                         |
|------------------|---------|------------------------------------------------------------|
| asyncScriptOnLoad | func    | *可选* 在 Google reCAPTCHA 脚本加载完成后的回调函数               |
| badge            | enum    | *可选* `bottomright`、`bottomleft` 或 `inline`，设置 reCAPTCHA 的徽标位置。*仅适用于不可见的 reCAPTCHA* |
| hl               | string  | *可选* 设置 `hl` 参数，使 reCAPTCHA 支持多种语言。请参见 [reCAPTCHA hl] |
| isolated         | bool    | *可选* 插件开发者用来避免与页面上已有的 reCAPTCHA 实例冲突。如果为 `true`，此 reCAPTCHA 实例将处于一个独立的 ID 空间。*（默认值：`false`）* |
| onChange         | func    | 用户成功完成验证码后调用的函数                                 |
| onErrored        | func    | *可选* 在挑战过程中出错时的回调，通常是由于网络问题。               |
| onExpired        | func    | *可选* 在挑战过期后调用的回调，用户需要重新完成验证码。默认会调用 `onChange`，并传入 `null` 来表示过期回调。 |
| sitekey          | string  | API 客户端密钥                                               |
| size             | enum    | *可选* `compact`、`normal` 或 `invisible`，用来设置验证码的大小或使用不可见验证码 |
| stoken           | string  | *可选* 设置 `stoken` 参数，使 reCAPTCHA 支持从不同域名使用。参见 [reCAPTCHA secure-token] |
| tabindex         | number  | *可选* 元素的 tabindex *（默认值：`0`）*                       |
| type             | enum    | *可选* `image` 或 `audio`，设置初始验证码的类型 *（默认值：`image`）* |
| theme            | enum    | *可选* `light` 或 `dark`，设置小部件的主题 *（默认值：`light`）*。参见 [example][docs_theme] |

### 组件实例 API

该组件实例还提供了一些可以调用的实用函数，可以通过 `ref` 访问。

- `getValue()`：返回验证码字段的值
- `getWidgetId()`：返回 reCAPTCHA 小部件的 ID
- `reset()`：强制重置。参见 [JavaScript API 文档][js_api]
- `execute()`：程序化调用挑战
  - 在使用 `"invisible"` reCAPTCHA 时需要调用 [示例](#invisible-recaptcha)
- `executeAsync()`：程序化调用挑战，并返回一个 Promise，成功时返回令牌，出错时返回错误。
  - 可与 `onChange()` 属性结合使用的另一种方法 [示例](#invisible-recaptcha)

示例：
```javascript
const recaptchaRef = React.createRef();
...
onSubmit = () => {
  const recaptchaValue = recaptchaRef.current.getValue();
  this.props.onSubmit(recaptchaValue);
}
render() {
  return (
    <form onSubmit={this.onSubmit} >
      <ReCAPTCHA
        ref={recaptchaRef}
        sitekey="你的客户端站点密钥"
        onChange={onChange}
      />
    </form>
  )
}
```

### 不可见 reCAPTCHA

参见 [reCAPTCHA 文档](https://developers.google.com/recaptcha/docs/invisible) 了解如何配置它。

使用不可见选项时，你需要以不同的方式处理。需要手动调用 `execute` 方法。

```jsx
import ReCAPTCHA from "react-google-recaptcha";

const recaptchaRef = React.createRef();

ReactDOM.render(
  <form onSubmit={() => { recaptchaRef.current.execute(); }}>
    <ReCAPTCHA
      ref={recaptchaRef}
      size="invisible"
      sitekey="你的客户端站点密钥"
      onChange={onChange}
    />
  </form>,
  document.body
);
```

此外，你还可以使用 `executeAsync` 方法，以基于 Promise 的方式调用挑战。

```jsx
import ReCAPTCHA from "react-google-recaptcha";

const ReCAPTCHAForm = (props) => {
  const recaptchaRef = React.useRef();

  const onSubmitWithReCAPTCHA = async () => {
    const token = await recaptchaRef.current.executeAsync();

    // 将令牌应用到表单数据
  }

  return (
    <form onSubmit={onSubmitWithReCAPTCHA}>
      <ReCAPTCHA
        ref={recaptchaRef}
        size="invisible"
        sitekey="你的客户端站点密钥"
      />
    </form>
  )
}

ReactDOM.render(
  <ReCAPTCHAForm />,
  document.body
);
```

### 高级用法

### CSP Nonce 支持
```js
window.recaptchaOptions = {
  nonce: document.querySelector('meta[name=\'csp-nonce\']').getAttribute('content'),
};
```

#### 手动加载 Google reCAPTCHA 脚本

你也可以使用裸组件来手动加载脚本。使用该组件时，你需要自己管理 grecaptcha 依赖并手动加载脚本。

```jsx
import { ReCAPTCHA } from "react-google-recaptcha";

const grecaptchaObject = window.grecaptcha; // 必须提供对 Google grecaptcha 对象的访问。

render(
  <ReCAPTCHA
    ref={(r) => this.recaptcha = r}
    sitekey="你的客户端站点密钥"
    grecaptcha={grecaptchaObject}
  />,
  document.body
);
```

#### 隐藏 reCAPTCHA

根据 [Google 文档](https://developers.google.com/recaptcha/docs/faq#id-like-to-hide-the-recaptcha-badge.-what-is-allowed)，如果你在用户流程中明确显示 reCAPTCHA 品牌，允许隐藏徽标。

请添加以下文本：

```html
此站点受 reCAPTCHA 保护，适用 Google
    <a href="https://policies.google.com/privacy">隐私政策</a> 和
    <a href="https://policies.google.com/terms">服务条款</a>。
```

如果你希望隐藏logo，可以在 CSS 中添加以下内容：

```css
.grecaptcha-badge { visibility: hidden; }
```
