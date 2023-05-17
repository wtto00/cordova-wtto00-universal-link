# cordova-wtto00-universal-link

参考 [e-imaxina/cordova-plugin-deeplinks](https://github.com/e-imaxina/cordova-plugin-deeplinks)

- 🌟 支持 Android 13

## 官方文档:

- [Universal Links on iOS](https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html)
- [Deep Linking on Android](https://developer.android.com/training/app-indexing/deep-linking.html)

如果 APP 没有安装的话，这个链接应该能够展示你的网站

## 用法

1. 安装插件 (查看 [安装](#安装)).
2. 在 `config.xml` 中配置链接 (查看 [Cordova 配置](#Cordova配置)).
3. 监听链接启动事件 (查看 [监听链接启动事件](#监听链接启动事件)).
4. web 集成 (查看 [安卓 web 集成](#安卓web集成) 和 [iOS-web 集成](#iOS-web集成)).
5. 启动项目
6. 本地测试 (查看 [安卓本地测试](#安卓本地测试) 和 [iOS 本地测试](#iOS本地测试)).

## 支持平台

- Android 4.0.0 or above.
- iOS 9.0 or above. Xcode 7 is required. To build plugin with Xcode 6

### 安装

```sh
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
- `event` - 链接启动 APP 后，在 APP 中监听的事件名称。默认事件名称是：`didLaunchAppFromLink`

示例：

```xml
<universal-links>
    <host name="example.com">
        <path url="/some/path" />
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

As described in `Step 2` of [Configure apple-app-site-association file for website](#configure-apple-app-site-association-file-for-website) section: when application is build from the CLI - plugin generates `apple-app-site-association` files for each host, defined in `config.xml`. In them there's an `appID` property that holds your iOS Team ID and Bundle ID:

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

- `<BUNDLE_ID>` is replaced with the id, that is defined in the `widget` of your `config.xml`. For example:

  ```xml
  <widget id="com.example.ul" version="0.0.1" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">
  ```

- `<TEAM_ID_FROM_MEMBER_CENTER>` - that property is defined in the member center of your iOS account. So, you can either put it in the generated `apple-app-site-association` file manually, or use `<ios-team-id>` preference in `config.xml` like so:

  ```xml
  <universal-links>
      <ios-team-id value="<TEAM_ID_FROM_MEMBER_CENTER>" />
  </universal-links>
  ```

For example, following `config.xml`

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

will result into

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

This is iOS-only preference, Android doesn't need it.

#### Prevent Android from creating multiple app instances

When clicking on a universal link from another App (typically from an email client), Android will likely create a new instance of your app, even if it is already loaded in memory. It may even create a new instance with each click, resulting in many instances of your app in the task switcher. See details in [issue #37](https://github.com/nordnet/cordova-universal-links-plugin/issues/37).

To force Android opening always the same app instance, a known workaround is to change the [activity launch mode](https://developer.android.com/guide/topics/manifest/activity-element.html#lmode) to `singleInstance`. To do so, you can use the following preference in Cordova `config.xml` file:

```xml
<preference name="AndroidLaunchMode" value="singleInstance" />
```

### Application launch handling

As mentioned - it is not enough just to redirect a user into your app, you will also need to display the correct content. In order to solve that - plugin provides JavaScript module: `universalLinks`. To get notified on application launch do the following:

```js
universalLinks.subscribe("eventName", function (eventData) {
  // do some work
  console.log("Did launch application from the link: " + eventData.url);
});
```

If you didn't specify event name for path and host in `config.xml` - just pass `null` as a first parameter:

```js
universalLinks.subscribe(null, function (eventData) {
  // do some work
  console.log("Did launch application from the link: " + eventData.url);
});
```

`eventData` holds information about the launching url. For example, for `http://myhost.com/news/ul-plugin-released.html?foo=bar#cordova-news` it will be:

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

- `url` - original launch url;
- `scheme` - url scheme;
- `host` - hostname from the url;
- `path` - path component of the url;
- `params` - dictionary with query parameters; the ones that after `?` character;
- `hash` - content after `#` character.

If you want - you can also unsubscribe from the events later on:

```js
universalLinks.unsubscribe("eventName");
```

Now it's time for some examples. In here we are gonna use Android, because it is easier to test (see [testing for Android](#testing-ul-for-android-locally) section). JavaScript side is platform independent, so all the example code below will also work for iOS.

1. Create new Cordova application and add Android platform.

```sh
cordova create TestAndroidApp com.example.ul TestAndroidApp
cd ./TestAndroidApp
cordova platform add android
```

2. Add UL plugin:

```sh
cordova plugin add cordova-universal-links-plugin
```

3. Add `<universal-links>` preference into `config.xml`:

```xml
<!-- some other data from config.xml -->
<universal-links>
 <host name="myhost.com">
   <path url="/news/" event="openNewsListPage" />
   <path url="/news/*" event="openNewsDetailedPage" />
 </host>
</universal-links>
```

As you can see - we want our application to be launched, when user goes to the `news` section of our website. And for that - we are gonna dispatch different events to understand, what has happened.

4. Subscribe to `openNewsListPage` and `openNewsDetailedPage` events. For that - open `www/js/index.js` and make it look like that:

```js
var app = {
  // Application Constructor
  initialize: function () {
    this.bindEvents();
  },

  // Bind Event Listeners
  bindEvents: function () {
    document.addEventListener("deviceready", this.onDeviceReady, false);
  },

  // deviceready Event Handler
  onDeviceReady: function () {
    console.log("Device is ready for work");
    universalLinks.subscribe("openNewsListPage", app.onNewsListPageRequested);
    universalLinks.subscribe(
      "openNewsDetailedPage",
      app.onNewsDetailedPageRequested
    );
  },

  // openNewsListPage Event Handler
  onNewsListPageRequested: function (eventData) {
    console.log("Showing list of awesome news.");

    // do some work to show list of news
  },

  // openNewsDetailedPage Event Handler
  onNewsDetailedPageRequested: function (eventData) {
    console.log("Showing to user details page: " + eventData.path);

    // do some work to show detailed page
  },
};

app.initialize();
```

Now, if the user clicks on `http://myhost.com/news/` link - method `onNewsListPageRequested` will be called, and for every link like `http://myhost.com/news/*` - `onNewsDetailedPageRequested`. Basically, we created a mapping between the links and JavaScript methods.

5. Build and run your application:

```sh
cordova run android
```

6. Close your app.

7. Execute in the terminal:

```sh
adb shell am start -W -a android.intent.action.VIEW -d "http://myhost.com/news/" com.example.ul
```

As a result, your application will be launched, and in JavaScript console you will see message:

```
Showing to user list of awesome news.
```

Repeat operation, but this time with the command:

```sh
adb shell am start -W -a android.intent.action.VIEW -d "http://myhost.com/news/ul-plugin-released.html" com.example.ul
```

Application will be launched and you will see in JS console:

```
Showing to user details page: /news/ul-plugin-released.html
```

Now, let's say, you want your app to handle all links from `myhost.com`, but you need to keep the mapping for the paths. For that you just need to modify your `config.xml` and add default event handler on JavaScript side:

1. Open `config.xml` and change `<universal-links>` block like so:

```xml
<universal-links>
 <host name="myhost.com">
   <path url="/news/" event="openNewsListPage" />
   <path url="/news/*" event="openNewsDetailedPage" />
   <path url="*" event="launchedAppFromLink" />
 </host>
</universal-links>
```

As you can see - we added `*` as `path`. This way we declared, that application should be launched from any `http://myhost.com` link.

2. Add handling for default UL event in the `www/js/index.js`:

```js
var app = {
  // Application Constructor
  initialize: function () {
    this.bindEvents();
  },

  // Bind Event Listeners
  bindEvents: function () {
    document.addEventListener("deviceready", this.onDeviceReady, false);
  },

  // deviceready Event Handler
  onDeviceReady: function () {
    console.log("Handle deviceready event if you need");
    universalLinks.subscribe("openNewsListPage", app.onNewsListPageRequested);
    universalLinks.subscribe(
      "openNewsDetailedPage",
      app.onNewsDetailedPageRequested
    );
    universalLinks.subscribe(
      "launchedAppFromLink",
      app.onApplicationDidLaunchFromLink
    );
  },

  // openNewsListPage Event Handler
  onNewsListPageRequested: function (eventData) {
    console.log("Showing to user list of awesome news");

    // do some work to show list of news
  },

  // openNewsDetailedPage Event Handler
  onNewsDetailedPageRequested: function (eventData) {
    console.log("Showing to user details page for some news");

    // do some work to show detailed page
  },

  // launchedAppFromLink Event Handler
  onApplicationDidLaunchFromLink: function (eventData) {
    console.log("Did launch app from the link: " + eventData.url);
  },
};

app.initialize();
```

That's it! Now, by default for `myhost.com` links `onApplicationDidLaunchFromLink` method will be called, but for `news` section - `onNewsListPageRequested` and `onNewsDetailedPageRequested`.

### Android web integration

If you have already tried to use `adb` to simulate application launch from the link - you probably saw chooser dialog with at least two applications in it: browser and your app. This happens because web content can be handled by multiple things. To prevent this from happening you need to activate app indexing. App indexing is the second part of deep linking, where you link that URI/URL between Google and your app.

Integration process consists of three steps:

1. Modify your web pages by adding special `<link />` tags in the `<head />` section.
2. Verify your website on Webmaster Tools.
3. Connect your app in the Google Play console.

#### Modify web pages

To create a link between your mobile content and the page on the website you need to include proper `<link />` tags in the `<head />` section of your website.

Link tag is constructed like so:

```html
<link
  rel="alternate"
  href="android-app://<package_name>/<scheme>/<host><path>"
/>
```

where:

- `<package_name>` - your application's package name;
- `<scheme>` - url scheme;
- `<host>` - hostname;
- `<path>` - path component.

For example, if your `config.xml` file looks like this:

```xml
<universal-links>
 <host name="myhost.com">
   <path url="/news/" />
   <path url="/profile/" />
 </host>
</universal-links>
```

and a package name is `com.example.ul`, then `<head />` section on your website will be:

```html
<head>
  <link
    rel="alternate"
    href="android-app://com.example.ul/http/myhost.com/news/"
  />
  <link
    rel="alternate"
    href="android-app://com.example.ul/http/myhost.com/profile/"
  />

  <!-- Your other stuff from the head tag -->
</head>
```

Good news is that **plugin generates those tags for you**. When you run `cordova build` (or `cordova run`) - they are placed in `ul_web_hooks/android/android_web_hook.html` file inside your Cordova project root directory.

So, instead of manually writing them down - you can take them from that file and put on the website.

#### Verify your website on Webmaster Tools

If your website is brand new, you’ll want to verify it through [Webmaster Tools](https://www.google.com/webmasters/tools/). That’s how the Google crawler knows that it’s there and can index it to do everything it needs to do. In order to do that - just add your website in the console and follow the instructions to versify, that you own the site. Most likely, they will ask you to add something on your page.

#### Connect your app in the Google Play console

Next, you’ll want to connect your app using the Google Play Console so the app indexing starts working. If you go to your app, there’s a menu that says `Services and API` in which you can click `Verify Website`, and provide the URL to check that it has the appropriate tags in the HTML. Once that’s all set up, it will start showing in search results.

#### Digital Asset Links support

For Android version 6.0 (Marshmallow) or greater [Digital Asset Links](https://developers.google.com/digital-asset-links/v1/getting-started) can be used.

Here's a very simplified example of how the website `www.example.com` could use Digital Asset Links to specify that any links to URLs in that site should open in a designated app rather than the browser:

1. The website `www.example.com` publishes a statement list at `https://www.example.com/.well-known/assetlinks.json`. This is the official name and location for a statement list on a site. Statement lists in any other location, or with any other name, are not valid for this site. In our example, the statement list consists of one statement, granting its Android app the permission to open links on its site:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.app",
      "sha256_cert_fingerprints": ["hash_of_app_certificate"]
    }
  }
]
```

A statement list supports an array of statements within the [ ] marks, but our example file contains only one statement.

2. The Android app listed in the statement above has an intent filter that specifies the scheme, host, and path pattern of URLs that it wants to handle: in this case, `https://www.example.com`. The intent filter includes a special attribute `android:autoVerify`, new to Android M, which indicates that Android should verify the statement on the website, described in the intent filter when the app is installed.

3. A user installs the app. Android sees the intent filter with the `autoVerify` attribute and checks for the presence of the statement list at the specified site. If present, Android checks whether that file includes a statement granting link handling to the app, and verifies the app against the statement by certificate hash. If everything checks out, Android will then forward any `https://www.example.com` intents to the `example.com` app.

4. The user clicks a link to `https://www.example.com/puppies` on the device. This link could be anywhere: in a browser, in a Google Search Appliance suggestion, or anywhere else. Android forwards the intent to the `example.com` app.

5. The `example.com` app receives the intent and chooses to handle it, opening the puppies page in the app. If for some reason the app had declined to handle the link, or if the app were not on the device, then the link will be send to the next default intent handler, matching that intent pattern (i.e. browser).

### Testing UL for Android locally

To test Android application for Deep Linking support you just need to execute the following command in the console:

```sh
adb shell am start
        -W -a android.intent.action.VIEW
        -d <URI> <PACKAGE>
```

where

- `<URI>` - url that you want to test;
- `<PACKAGE>` - your application's package name.

**Note:** if you didn't configure your website for UL support - then most likely after executing the `adb` command you will see a chooser dialog with multiple applications (at least browser and your test app). This happens because you are trying to view web content, and this can be handled by several applications. Just choose your app and proceed. If you configured your website as [described above](#android-web-integration) - then no dialog is shown and your application will be launched directly.

Let's create new application to play with:

1. Create new Cordova project and add Android platform to it:

```sh
cordova create TestAndroidApp com.example.ul TestAndroidApp
cd ./TestAndroidApp
cordova platform add android
```

2. Add UL plugin:

```sh
cordova plugin add cordova-universal-links-plugin
```

3. Add `<universal-links>` preference into `config.xml` (`TestAndroidApp/config.xml`):

```xml
<!-- some other data from config.xml -->
<universal-links>
 <host name="myhost.com" />
</universal-links>
```

4. Build and run the app:

```sh
cordova run android
```

5. Close your application and return to console.
6. Enter in console:

```sh
adb shell am start -W -a android.intent.action.VIEW -d "http://myhost.com/any/path" com.example.ul
```

As a result, your application will be launched and you will see in console:

```
Starting: Intent { act=android.intent.action.VIEW dat=http://myhost.com/any/path pkg=com.example.ul }
Status: ok
Activity: com.example.ul/.MainActivity
ThisTime: 52
TotalTime: 52
Complete
```

If you'll try to use host (or path), that is not defined in `config.xml` - you'll get a following error:

```
Starting: Intent { act=android.intent.action.VIEW dat=http://anotherhost.com/path pkg=com.example.ul }
Error: Activity not started, unable to resolve Intent { act=android.intent.action.VIEW dat=http://anotherhost.com/path flg=0x10000000 pkg=com.example.ul }
```

This way you can experiment with your Android application and check how it corresponds to different links.

### iOS web integration

In the case of iOS integration of the Universal Links is a little harder. It consist of two steps:

1. Register your application on [developer console](https://developer.apple.com) and enable `Associated Domains` feature. Make sure your website is SSL ready.
2. Generate, and upload `apple-app-site-association` file on your website (if you don't have it yet).

First one you will have to do manually, but plugin will help you with the second step.

#### Activate UL support in member center

1. Go to your [developer console](https://developer.apple.com). Click on `Certificate, Identifiers & Profiles` and then on `Identifiers`.

![Developer console](docs/images/developer-console.jpg?raw=true)

2. If you already have a registered App Identifier - just skip this and go to `3`. If not - create it by clicking on `+` sign, fill out `name` and `bundle ID`. `name` can be whatever you want, but `bundle ID` should be the one you defined in your Cordova's `config.xml`.

![App ID](docs/images/app-id.jpg?raw=true)

3. In the `Application Services` section of your App Identifier activate `Associated Domains` and save the changes.

![App ID](docs/images/app-associated-domains.jpg?raw=true)

Now your App ID is registered and has `Associated Domains` feature.

#### Configure apple-app-site-association file for website

In order for Universal Links to work - you need to associate your application with the certain domain. For that you need to:

1. Make your site to work over `https`.
2. Create `apple-app-site-association` file, containing your App ID and paths you want to handle.
3. Upload `apple-app-site-association` file in the root of your website.

##### Step 1

We are not gonna describe stuff regarding certificate acquiring and making your website to work over `https`. You can find lots of information about that on the Internet.

##### Step 2

When you run `cordova build` (or `cordova run`) - plugin takes data from `config.xml` and generates `apple-app-site-association` files for each host you defined. Files are placed in the `ul_web_hooks/ios/` folder of your Cordova project. File names are:

```
<hostname>#apple-app-site-association
```

For example, let's say your application's bundle ID is `com.example.ul`, and `config.xml` has several hosts:

```xml
<universal-links>
  <host name="firsthost.com">
    <path url="/some/path/*" />
  </host>
  <host name="secondhost.com" />
</universal-links>
```

Run `cordova build`, and then go to `ul_web_hooks/ios/` folder in your Cordova project. You will see there two files:

```
firsthost.com#apple-app-site-association
secondhost.com#apple-app-site-association
```

Content of the first one is:

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "<YOUR_TEAM_ID_FROM_MEMBER_CENTER>.com.example.ul",
        "paths": ["/some/path/*"]
      }
    ]
  }
}
```

And the second one:

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "<YOUR_TEAM_ID_FROM_MEMBER_CENTER>.com.example.ul",
        "paths": ["*", "/"]
      }
    ]
  }
}
```

**Note:** in the second case plugin will add `/` to the paths, so the app would be opened with `https://secondhost.com.com/` links, and not only with `https://secondhost.com/some/random.html`.

Before uploading them on your servers - you need to replace `<YOUR_TEAM_ID_FROM_MEMBER_CENTER>` with your actual team ID from the member center. You can find it in `Developer Account Summary` section on the [developer.apple.com](https://developer.apple.com/membercenter/index.action#accountSummary).

Also, it is a `Prefix` preference in the App ID description.

![App ID team prefix](docs/images/app-id-team-prefix.jpg?raw=true)

If you already have `apple-app-site-association` file - then you need to add `applinks` block to it from the generated file.

##### Step 3

Upload `apple-app-site-association` file in the root of your domain.

**It should be downloadable from the direct link.** For example, `https://firsthost.com/apple-app-site-association`.

**No redirects are allowed!** When application is launched - it downloads it from that link, so if it can't find it - Universal Links are not gonna work.

That's it, you have finished configuring iOS for UL support.

### Testing iOS application

Unlike Android, Apple doesn't provide any tools to test Universal Links. So you have to do all the [integration stuff](#ios-web-integration) before any real testing. So please, do that.

But if you don't want to... well, there is one way to skip it. You can use [branch.io](https://branch.io) to handle all the SSL/apple-app-site-association stuff for you. How to do that - described in their [documentation](https://dev.branch.io/recipes/branch_universal_links/#enable-universal-links-on-the-branch-dashboard). From there you can skip Xcode and SDK integration stuff, because you don't need that.

Step-by-step guide:

1. Go to developer console and register your App ID, as described in [Activating UL support in member center](#activate-ul-support-in-member-center).

2. Register account on [branch.io](https://dashboard.branch.io/), if you don't have it yet.

3. Login into [branch dashboard](https://dashboard.branch.io/). Go to `Settings` -> `Link Settings`, activate `Enable Universal Links`, fill in `Bundle identifier` and `Team ID`.

![App ID](docs/images/branch-io.jpg?raw=true)

4. It will take some time to update their servers, so be patient. To check if it is ready - just open [https://bnc.lt/apple-app-site-association](https://bnc.lt/apple-app-site-association) and search for your `Bundle identifier`.

Pay attention for `paths` - if there is any for your app, then write it down.

For example:

```json
...,"9F38WJR2U8.com.example.ul":{"paths":["/a2Be/*"]},...
```

5. Create new Cordova iOS application and add UL plugin:

```sh
cordova create TestProject com.example.ul TestProject
cd ./TestProject
cordova platform add ios
cordova plugin add cordova-universal-links-plugin
```

6. Add `bnc.lt` and your other hosts into `config.xml`:

```xml
<universal-links>
  <host name="bnc.lt" />
  <host name="yourdomain.com" />
</universal-links>
```

For test purpose you can leave only `bnc.lt` in there. But if you specifying your hosts - you need to [white label](https://dev.branch.io/recipes/branch_universal_links/#white-label-domains) them.

7. Attach your real device to the computer and run application on it:

```sh
cordova run ios
```

Emulator will not work.

8. Email yourself a link that need's to be tested.

For example, `https://bnc.lt/a2Be/somepage.html`. As you can see, link constructed from hostname and path component, specified in `apple-app-site-association` file. This link may not even lead to the real page, it doesn't matter. It's only purpose is to open the app.

Now click on your link. Application should be launched. If not - check all the steps above. Also, check your provisioning profiles in Xcode.

### Useful notes on Universal Links for iOS

#### They don't work everywhere

First of all - you need to accept the fact, that Universal Links **doesn't work everywhere**. Some applications doesn't respect them. You can read more in [that article](https://blog.branch.io/ios-9.2-deep-linking-guide-transitioning-to-universal-links), section `Universal Links Still Don’t Work Everywhere`.

#### How links handled in Safari

When user clicks on the link - Safari checks, if any of the installed apps can handle it. If app is found - Safari starts it, if not - link opened as usually in the browser.

Now, let's assume you have a following setup in `config.xml`:

```xml
<universal-links>
  <host name="mywebsite.com">
    <path url="/some/page.html" />
  </host>
</universal-links>
```

By this we state, that our app should handle `http://mywebsite.com/some/page.html` link. So, if user clicks on `http://mywebsite.com` - application would not launch. And this is totally as you want it to be. Now comes the interesting part: if user opens `http://mywebsite.com` in the Safari and then presses on `http://mywebsite.com/some/page.html` link - application is not gonna start, he will stay in the browser. And at the top of that page he will see a Smart Banner. To launch the application user will have to click on that banner. And this is a normal behaviour from iOS. If user already viewing your website in the browser - he doesn't want to leave it, when he clicks on some link, that leads on the page inside your site. But if he clicks on the `http://mywebsite.com/some/page.html` link from some other source - then it will start your application.

Another thing that every developer should be aware of:

When a user is in an app, opened by Universal Links - a return to browser option will persist at the top of the screen (i.e. `mywebsite.com`). Users who have clicked the `mywebsite.com` option will be taken to their Safari browser, and Smart Banner is persistently rendered on the top of the window. This banner has an `OPEN` call to action. For all future clicks of URLs, associated with this app via Universal Links, the app will never be launched again for the user, and the user will continue being redirected to the Safari page with the banner. If the user clicks `OPEN` - then the app will be launched, and all future clicks of the URL will deep linking the user to the app.

### Additional documentation links

**Android:**

- [Video tutorial on Android App Indexing](https://realm.io/news/juan-gomez-android-app-indexing/)
- [Enable Deep Linking on Android](https://developer.android.com/training/app-indexing/deep-linking.html)
- [Specifying App Content for Indexing](https://developer.android.com/training/app-indexing/enabling-app-indexing.html)
- [Documentation on enabling App Indexing on the website](https://developers.google.com/app-indexing/android/publish#host-your-links)

**iOS:**

- [Apple documentation on Universal Links](https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html)
- [Apple documentation on apple-app-site-association file](https://developer.apple.com/library/ios/documentation/Security/Reference/SharedWebCredentialsRef/index.html)
- [How to setup universal links on iOS 9](https://blog.branch.io/how-to-setup-universal-links-to-deep-link-on-apple-ios-9)
- [Branch.io documentation on universal links](https://dev.branch.io/recipes/branch_universal_links/#enable-universal-links-on-the-branch-dashboard)
- [Where universal links don't work](https://blog.branch.io/ios-9.2-deep-linking-guide-transitioning-to-universal-links)
