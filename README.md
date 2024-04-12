# cordova-wtto00-universal-link

参考 [e-imaxina/cordova-plugin-deeplinks](https://github.com/e-imaxina/cordova-plugin-deeplinks)

- 🌟 支持 Android 13
- 🌟 添加 Types 定义
- 🐛 修复了一些在最新版本 SDK 上面出现的问题

## 官方文档

- [Universal Links on iOS](https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html)
- [Deep Linking on Android](https://developer.android.com/training/app-indexing/deep-linking.html)

## 用法

1. 安装插件 (查看 [安装](#安装)).
2. 在 `config.xml` 中配置链接 (查看 [Cordova 配置](#cordova配置)).
3. 监听链接启动事件 (查看 [监听链接启动事件](#监听链接启动事件)).
4. web 集成 (查看 [安卓 web 集成](#安卓web集成) 和 [iOS-web 集成](#ios-web集成)).
5. 启动项目，本地测试 (查看 [安卓本地测试](#安卓本地测试) 和 [iOS 本地测试](#ios本地测试)).

## 支持平台

- Android 4.0.0 或以上。
- iOS 9.0 或以上。

### 安装

```shell
cordova plugin add cordova-wtto00-universal-link
```

### Cordova 配置

项目根目录下的`config.xml`。

示例：

```xml
<universal-links>
    <host name="example.com">
        <path url="/some/path/*" />
    </host>
</universal-links>
```

#### host

`<host />` 标签表示 universal link 的主机信息

- `name` - 域名，**必填**。
- `scheme` - 协议 `http` 或者 `https`. 默认是 `http`。
- `event` - 链接启动 APP 后，在 APP 中监听的事件名称。默认事件名称是：`didLaunchAppFromLink`

示例：

```xml
<universal-links>
    <host name="example.com" scheme="https" event="my_launch_event_name" />
</universal-links>
```

这个示例表示，用户浏览器进入 `https://example.com` 后，将会打开 APP，然后在 APP 中触发监听的 `ul_myExampleEvent` 事件。详情查看 [监听链接启动事件](#监听链接启动事件).

你还可以使用域名通配符：

```xml
<universal-links>
    <host name="*.users.example.com" scheme="https" event="my_launch_event_name_from_user" />
    <host name="*.example.com" scheme="https" event="my_launch_event_name" />
</universal-links>
```

#### path

`<path />` 标签定义主机域名下的哪些路径将被响应打开 APP。如果没有 `path` 标签，则响应所有的路径。

- `url` - 路径地址 **This is a required attribute.**
- `event` - 链接启动 APP 后，在 APP 中监听的事件名称。空缺的话，将会触发父级 `host` 标签上定义的事件名称。

多个 `path` 标签，可以对与不同的链接路径，监听不同的处理事件。

示例：

```xml
<universal-links>
    <host name="example.com">
        <path url="/some/path" event="my_launch_event_for_this_path" />
    </host>
</universal-links>
```

当浏览器访问 `http://example.com/some/path` 时，将打开 APP。该域名下的所有其他链接都将被忽略，不会响应。

注意：url 的 query 参数不会计入匹配规则中。即 `http://example.com/some/path?foo=bar#some_tag` 一样可以正常响应。

如果想要支持 `/some/path/` 后面的所有子路径，你可以使用 `*` 通配符。  
比如：

```xml
<universal-links>
    <host name="example.com">
        <path url="/some/path/*" />
    </host>
</universal-links>
```

`*` 通配符用在 path 标签的 url 路径时，可以放在路径的任意位置。  
比如：

```xml
<universal-links>
    <host name="example.com">
        <path url="*mypath*" />
    </host>
</universal-links>
```

这个配置将会响应以下示例：

- `http://example.com/some/long/mypath/test.html`
- `http://example.com/testmypath.html`

等等...

#### ios-team-id

`ios-team-id` 标签用于生成 iOS 配置文件 `apple-app-site-association`。

当使用 CLI 构建项目时，本插件会自动在 `ul_web_hooks` 目录中生成用于上传到服务器的配置文件 `apple-app-site-association`

示例：

`config.xml` :

```xml
<widget id="com.example.ul" version="0.0.1" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">

  <!-- some other cordova preferences -->

  <universal-links>
      <ios-team-id value="1Q2WER3TY" />
      <host name="mysite.com" >
        <path url="/some/path/*" />
      </host>
  </universal-links>
</widget>
```

将会生成配置文件 `apple-app-site-association` :

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "1Q2WER3TY.com.example.ul",
        "paths": ["/some/path/*"]
      }
    ]
  }
}
```

该标签作用仅仅是生成这个 iOS 的配置文件。不影响打包运行。Android 会忽略此配置。

#### 阻止安卓生成多个 APP 实例

当通过 universal link 打开安卓 APP 时，安卓默认会重新生成一个新实例，即使 APP 已经在运行中。

为了解决此问题，需要在 `config.xml` 文件中配置单例模式：

```xml
<preference name="AndroidLaunchMode" value="singleInstance" />
```

### 监听链接启动事件

仅仅是打开 APP 是不够的，你需要获得打开 APP 的链接地址，然后根据链接地址以及参数，做各种不同的逻辑处理。`universalLinks` 变量已挂载在全局变量 `window` 下，可以直接使用，并且在 `Typescript` 中也定义了类型。

**注意：** 监听事件，必须在 `cordova` 生命周期的 `deviceready` 事件后执行，即 `document.addEventListener("deviceready",callback, false);` 中的回调 `callback` 中注册监听事件。

示例：

```js
universalLinks.subscribe('eventName', function (eventData) {
  // do some work
  console.log('Did launch application from the link: ' + eventData.url)
})
```

如果你没有在 `config.xml` 中配置 `eventName`, 则可以使用默认的事件名称 `didLaunchAppFromLink`。

```js
universalLinks.subscribe('didLaunchAppFromLink', function (eventData) {
  // do some work
  console.log('Did launch application from the link: ' + eventData.url)
})
```

`eventData` 参数是打开 APP 的链接地址。

比如： 对于 `http://myhost.com/news/ul-plugin-released.html?foo=bar#cordova-news`

`eventData` 将会是如下的数据：

```json
{
  "url": "http://myhost.com/news/ul-plugin-released.html?foo=bar#cordova-news",
  "scheme": "http",
  "host": "myhost.com",
  "path": "/news/ul-plugin-released.html",
  "params": {
    "foo": "bar"
  },
  "hash": "cordova-news"
}
```

如果你想，你还可以取消监听该事件，如下所示：

```js
universalLinks.unsubscribe('eventName')
```

### web-集成

前端方面的已基本完成了，接下来你需要在后端服务器上面做一些配置。

假如你在 `config.xml` 中是这样配置的：

```xml
<universal-links>
    <ios-team-id value="1Q2WER3TY" />
    <host name="mysite.com" scheme="https" >
      <path url="/some/path/*" />
    </host>
</universal-links>
```

#### 安卓 web 集成

1. 在服务器上传配置文件

   按照上面的 `config.xml` 示例，你需要在 `mysite.com` 域名所解析的服务器上的**根目录**中，创建 `.well-known` 目录，并在该目录下，创建文件 `assetlinks.json` ，内容如下:

   ```json
   [
     {
       "relation": ["delegate_permission/common.handle_all_urls"],
       "target": {
         "namespace": "android_app",
         "package_name": "<PACKAGE_ID>",
         "sha256_cert_fingerprints": ["<SHA256_CERT_FINGERPRINTS>"]
       }
     }
   ]
   ```

   详情可查看 [官方说明](https://developer.android.com/training/app-links/verify-android-applinks?hl=zh-cn#web-assoc)

   - `<PACKAGE_ID>` : 你的包名。
   - `<SHA256_CERT_FINGERPRINTS>` : 应用的签名证书的 SHA256 指纹。可以通过以下命令获得。

     ```shell
     keytool -list -v -keystore my-release-key.keystore
     # 14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5
     ```

1. 确认配置文件有效性  
   按照上面的配置示例：

   - `https://mysite.com/.well-known/assetlinks.json` 该链接应该能够正常访问或可以下载。
   - `https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://mysite.com&relation=delegate_permission/common.handle_all_urls` 该链接应该能返回配置的正确信息。

   > 注意：测试链接，请更换为自己的配置信息。

1. 在链接 `html` 页面上添加信息

   ```html
   <link rel="alternate" href="android-app://<PACKAGE_ID>/<SCHEME>/<HOST><PATH>" />
   ```

   - `<PACKAGE_ID>` : APP 包名。
   - `<SCHEME>`: `config.xml` 中配置的 `host` 标签协议。
   - `<HOST>`: `config.xml` 中配置的 `host` 标签域名。
   - `<PATH>`: `config.xml` 中配置的 `host` 中子元素 `path` 标签地址。假如配置多个 `path` 标签，则需要有多个对应的 `link` 标签。

   > `config.xml` 中配置后，通过 CLI 启动命令 `cordova prepare android` 会自动在项目根目录下的 `ul_web_hooks/android` 目录中生成一个 `android_web_hook.html` 文件。可以拷贝其中的 `link` 标签，粘贴到自己网站页面的 `html` 中。

1. 在其他页面手动触发打开 APP

   如果你想在任意的其他页面，手动触发打开 APP，

   - 可以使用 `a` 标签即： `<a href="android-app://<PACKAGE_ID>/<SCHEME>/<HOST><PATH>">打开APP</a>`。
   - 或者使用重定向：`window.location.href = 'android-app://<PACKAGE_ID>/<SCHEME>/<HOST><PATH>'`

#### iOS-web 集成

1. 在服务器上上传配置文件

   按照上面的 `config.xml` 示例，你需要在 `mysite.com` 域名所解析的服务器上的**根目录**中，创建 `.well-known` 目录，并在该目录下，创建文件 `apple-app-site-association` ，内容如下:

   ```json
   {
     "applinks": {
       "apps": [],
       "details": [
         {
           "appID": "<TEAM_ID_FROM_MEMBER_CENTER>.<BUNDLE_ID>",
           "paths": ["/some/path/*"]
         }
       ]
     }
   }
   ```

   也可以直接在根目录中创建该文件，`.well-known` 目录可省略。

   - `<TEAM_ID_FROM_MEMBER_CENTER>` : 开发者 team_id。
   - `<BUNDLE_ID>` : APP 包名。

   > `config.xml` 中配置后，通过 CLI 启动命令 `cordova prepare ios` 会自动在项目根目录下的 `ul_web_hooks` 目录中生成文件 `<hostname>#apple-app-site-association`。可改名后直接上传到自己的域名服务器中。

1. 确认配置文件有效性  
   按照上面的配置示例：

   - `https://mysite.com/.well-known/apple-app-site-association` 或者 `https://mysite.com/apple-app-site-association` 该链接应该能够正常访问或可以下载。
   - `https://app-site-association.cdn-apple.com/a/v1/mysite.com` 该链接应该能返回配置的正确信息。

   > 注意：测试链接，请更换为自己的配置信息。

1. Associated Domains

   **注意：** 在 `apple` 开发者中心创建，在 `xcode` 中打包，所使用的证书，应该包含此功能选项。

### 启动项目

现在万事具备了，启动项目，开始测试吧。

#### 安卓本地测试

1. 启动项目

   ```shell
   cordova run android
   ```

1. 在设备上，关闭刚刚安装打开的 APP
1. 在终端中执行：

   ```shell
   adb shell am start -W -a android.intent.action.VIEW -d "<URL>" <PACKAGE_ID>
   ```

   - `<URL>` : 响应的链接地址。按照上面的示例，应该是：`http://mysite.com/some/path/`。
   - `<PACKAGE_ID>` : APP 包名。
     **注意：** 更换为自己的配置信息。

没有出错的话，设备上应该直接打开了你正在开发的 APP。

#### ios 本地测试

1. 启动项目

   ```shell
   cordova run ios
   ```

1. 在设备上，关闭刚刚安装打开的 APP
1. 在设备上，打开 `safari` ，输入地址 `http://mysite.com/some/path/` 并访问。  
   **注意：** 更换为自己的配置信息。

没有出错的话，设备上应该直接打开了你正在开发的 APP。
