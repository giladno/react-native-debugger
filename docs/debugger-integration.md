# Debugger integration

The Debugger worker is referenced from [react-native](https://github.com/facebook/react-native/blob/master/local-cli/server/util/) debugger-ui.

## Multiple React Native packager (custom port) support

We can use [`react-native-debugger-open`](../npm-package) package to detect RN packager port, it will open an another window automatically if another debugger workers are running.

If you don't use [the npm package](../npm-package) and want to change port, click `Debugger` -> `New Window` (`Command⌘ + T` for macOS, `Ctrl + T` for Linux / Windows) in application menu, you need to type an another RN packager port. The default port is use [`Expo`](https://github.com/expo/expo) (and [`create-react-native-app`](https://github.com/react-community/create-react-native-app)) default port.

For macOS (10.12+), it used native tabs feature, see [the support page](https://support.apple.com/en-us/HT206998) for known how to use and setting.

## Developer menu integration

We have [developer menu](https://facebook.github.io/react-native/docs/debugging.html#accessing-the-in-app-developer-menu) of React Native integration:

![Dev menu](https://cloud.githubusercontent.com/assets/3001525/25920996/5c488966-3606-11e7-8d0c-cb564671067b.gif)

Just includes two developer menu features for iOS, these would be useful for real device, instead of open developer menu in iOS device manually:

* Reload JS
* Toogle Elements Inspector (RN ^0.43 support)
* Show Developer Menu

Other features (cross-platform):

* Clear AsyncStorage
* Network Inspect (Enable / Disable [this tip](#inpsect-network-requests-by-network-tab-of-chrome-devtools-see-also-15))

#### macOS Touch Bar support

<img alt="touch-bar" src="https://user-images.githubusercontent.com/3001525/27730359-8565810a-5dbb-11e7-9052-9fd4feb72181.png">

The `Redux Slider` will shown on right if you're using [`Redux API`](redux-devtools-integration.md),

If your Mac haven't TouchBar support, you can use [`touch-bar-simulator`](https://github.com/sindresorhus/touch-bar-simulator), the features are still very useful.

## Debugging tips

#### Get global variables of React Native runtime in the console

You need to switch worker thread for console, open `Console` tab on Chrome DevTools, tap `top` context and change to `RNDebuggerWorker.js` context:

![2016-11-05 6 56 45](https://cloud.githubusercontent.com/assets/3001525/20025024/7edce770-a325-11e6-9e77-618c7ba04123.png)

#### Use `require('<providesModule>')` in the console

In the console, you can use `require` for module of specified [`@providesModule`](https://github.com/facebook/react-native/search?l=JavaScript&q=providesModule&type=&utf8=✓) words in React Native, this is example for use `AsyncStorage`:

<img width="519" alt="t" src="https://cloud.githubusercontent.com/assets/3001525/25587896/a1253c9e-2ed8-11e7-9d70-6368cfd5e016.png">

Make sure you're changed to `RNDebuggerWorker.js` context, the same as the previous tip.

#### Inspect network requests by `Network` tab of Chrome DevTools (See also [#15](https://github.com/jhen0409/react-native-debugger/issues/15))

We can do:

```js
// const bakXHR = global.XMLHttpRequest;
// const bakFormData = global.FormData;
global.XMLHttpRequest = global.originalXMLHttpRequest ?
  global.originalXMLHttpRequest :
  global.XMLHttpRequest;
global.FormData = global.originalFormData ?
  global.originalFormData :
  global.FormData;
```

See also - [the comments of `react-native/Libraries/Core/InitializeCore.js#L43-L53`](https://github.com/facebook/react-native/blob/0.45-stable/Libraries/Core/InitializeCore.js#L43-L53).

Warning:

* It will break `NSExceptionDomains` for iOS, because `originalXMLHttpRequest` is from debugger worker (it will replace native request), so we should be clear about the difference in debug mode.
* It have some limitations from Chrome (like you cannot set some headers), you need pay attention to this.
* It can't inspect request like `Image` load, so if your Image source have set session, the session can't apply to `fetch` and `XMLHttpRequest`.

Also, if you want to inspect deeper network requests (Like request of `Image`), use tool like [Stetho](https://facebook.github.io/stetho) will be better.

#### [iOS only] Force your app on debug mode

For enable `Debug Remotely` in real device, you may fatigued to shake device to show developer menu, you can use the built-in `DevSettings` native module on iOS:

```js
import { NativeModules } from 'react-native'

if (__DEV__) {
  NativeModules.DevSettings.setIsDebuggingRemotely(true)
}

// For RN < 0.43
if (__DEV__) {
  NativeModules.DevMenu.debugRemotely(true)
}
```
