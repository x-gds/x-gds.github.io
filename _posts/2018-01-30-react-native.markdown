---
layout: post
title:  "移动应用混合开发之React Native"
date:   2018-01-30 14:24:16 +0800
categories: Android
---

### Learn once, write anywhere

1. 使用JavaScript + React
2. 构建出来的是Native APP
3. 开发时省去了重新编译的时间
4. 可以使用native代码

---

# 搭建环境

对国内开发者来说，React Native中文网的入门环境搭建更有参考价值。

要跨平台开发，需要搭建对应平台的开发环境，iOS就Xcode，Android需要Java + AndroidStudio。建议开发使用iOS模拟器，Android模拟器可以忽略，最后再适配Android。

开发工具官方推荐Atom+Nuclide，但是我个人更推荐Webstorm。

此外最重要的就是安装Node.js和react-native-cli，yarn可装可不装:

```shell
brew install node
npm install -g yarn react-native-cli
```

# 创建一个React native项目

搭建好开发环境后，就可以使用react-native-cli新建一个项目：

```shell
react-native init AwesomeProject
```

可以先看一下项目文件夹的结构：

```shell
➜  rn ls AwesomeProject
__tests__        app.json         index.ios.js     node_modules
android          index.android.js ios              package.json
```

`index.android.js`和`index.ios.js`文件分别是iOS平台和Android平台的入口，`android`和`ios`文件夹分别是Android工程和iOS工程的目录，`node_modules`文件夹下存放的是引用的库。

运行：

```shell 
cd AwesomeProject
react-native run-ios
```

# JavaScript及React

借助于babel，开发时可以使用ES6的语法，比如extends、import，还有Promise等。

## React

React是一个用来构建用户界面的JS库，特点是基于组件、响应式布局。

关键知识点有JSX、Component、state、props等，布局方式就是flexbox。

# JSX

先看一下AwesomeProject的`index.ios.js`中UI渲染的部分：

```javascript
export default class AwesomeProject extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
        <Text style={styles.instructions}>
          To get started, edit index.ios.js
        </Text>
        <Text style={styles.instructions}>
          Press Cmd+R to reload,{'\n'}
          Cmd+D or shake for dev menu
        </Text>
      </View>
    );
  }
}
```
JSX代码在构建时会被编译成纯JS代码，`React.createElement(...)`

# Component

React是基于组件的，能看到的所有东西都是组件，页面、文本、图像等，例子中的`View`和`Text`都是`Component`，当然`AwesomeProject`这个页面也是`Component`。

组件的属性就叫`props`，Android中叫`attributes`，比如例子中`View`和`Text`都有的`style`。

React是响应式布局的，有一个比较重要的点就在于组件的状态：`state`，当`state`发生变化后，会自动更新UI。

> 更新UI时，React会先拿内存中的虚拟DOM做个diff，找到真正发生改变的DOM，然后只更新这部分视图

# 样式及Flexbox

每个组件都有一个属性叫`style`，可以为当前组件设置宽、高等，像`Text`还可以设置字号、字色等。

Flexbox是一种规则，其中`flex`很像Android中`LinearLayout`的`weight`，此外还可以控制布局方向`flexDirection`、次轴的排列方式`alignItems`以及主轴的排列方式`justifyContent`。

`justifyContent`的值有`flex-start`、`center`、`flex-end`、`space-around`以及`space-between`。

`alignItems`的值有`flex-start`、`center`、`flex-end`以及`stretch`。

布局这块对Android开发工程师来说应该都不陌生，比如`margin`、`padding`等。

# 销探的开发

React Native上手还是很快的，销探从我开始学习到开发了个差不多，差不多是一周时间，真正用在开发销探的时间压缩后可能在3-4个工作日。

除了前面的知识，销探还用到了4个第三方库：启动屏`react-native-splash-screen`、导航`react-navigation`、loading动画`react-native-spinkit`和html标签的解析库`react-native-htmlview`。

引入第三方库，用`npm`安装就行，比如要引入`react-navigation`：

`npm install react-navigation --save`

---

# 遇到的坑

1.  Unable to resolve module `react/lib/React ComponentWithPureRenderMixin`
	解决办法：打开`package.json`，修改`react-navigation`的版本为`git+https://github.com/react-community/react-navigation.git#7edd9a7`
2. React Native升级到0.43后`react@^16.0.0-alpha.6`找不到
	解决办法：`npm install --save react@^16.0.0-alpha.6`
3. iOS的启动页和icon的各种尺寸

# 开发的套路

1. 根组件：每个React Native应用都有一个根组件，要使用`AppRegistry`注册：
	`AppRegistry.registerComponent('salesgo', () => salesgo);`
2. 如果iOS和Android两个平台的根组件使用同一个，可以使用`import`命令直接导入：将`index.ios.js`和`index.android.js`的内容都替换为：
	`import './core/pages/MainPage';`
3. 几乎每个组件都导入了
	```javascript
    import React, { Component } form 'react';
    import {
        StyleSheet,
        View,
        ...
    } from 'react-native';
    ```

4. 样式一般都定义在一个`StyleSheet`常量中，虽然完全可以在组件的组件的`style`属性中直接指定，但是定义在`StyleSheet`中可以复用，而且好管理。
5. 可以直接在`style`中指定背景边框、颜色、角度，甚至可以分上下左右单独设置，比Android中使用`xml-shape`方便很多。
6. 如果想要处理某个组件的点击事件，就用`Touchable`开头的几个组件将它包裹，这类组件有一个`onPress`属性，可以设置一个处理点击事件的函数:
	1. `TouchableHighlight`点击后背景会变暗
	2. `TouchableOpacity`点击后背景不变，但是按钮会变透明
	3. Android平台使用`TouchableNativeFeedback`可以有涟漪效果
7. 启动页可以使用第三方库`react-native-splash-screen`，也可以跟这个库学习一下JS中使用native代码。
8. 页面的跳转、导航等，推荐`react-navigation`，这个库比较新，用的是Flux架构，页面的跳转、传值、回传、页面堆栈管理等都很方便，还支持Tab类型的页面。
9. loading没有官方实现，自己实现可以使用`Modal`自定义一个，第三方库我使用的是`react-native-spinkit`。
10. html标签的解析，Android中非常方便，但是React Native不支持，自己写了一个，但是最终用了第三方库`react-native-htmlview`。
11. 悬浮窗、对话框等使用`Modal`。
12. 如果想要控制某个组件显示/隐藏，可以搭配`state`在渲染时这样写：
	` { this.state.areaVisible && (<View>...</View>)`
13. 持久化：简单的数据存储可以用`AsyncStorage`，它是一个简单的异步K-V存储系统，对App来说是全局性的。
	也有一些第三方库提供了数据库的支持，甚至Reaml也有React Native版本。
14. 网络请求：React Native提供和Web标准一致的`fetch` API，从销探的开发来看，是完全够用的，`fetch`在Android平台上是用`okhttp`实现的，iOS平台用的是`NSMutableURLRequest`
15.`FlatList`：新版本中提供了一个替代`ListView`的高性能列表组件，叫`FlatList`，自带上拉下拉事件，可以自定义视图：
	首先`import {Animated,  FlatList,} from 'react-native`，
    然后创建一个带动画的组件`const AnimatedFlatList = Animated.createAnimatedComponent(FlatList);`，
    最后渲染
	```javascript
	<AnimatedFlatList
    	ItemSeparatorComponent={SeparatorComponent} //分隔组件
    	keyExtractor={(item) => item...} // 数据的key，比如id
    	data={...} 			     // 数据源
    	refreshing={false}		     
    	onRefresh={() => {...}}	     // 下拉事件
    	onEndReached={() => {...}}	     // 上拉事件
    	onEndReachedThreshold={1}	     // 上拉到剩最后几条时调用
    				     // onEndReached，至少为1
    	renderItem={({item}) => (...)}   // 渲染Item
	/>
	```
	亲测`onEndReachedThreshold`设置为`0`的话，上拉事件有时候不响应

16. Android的适配：开发时，就算用真机也很卡，这点要有个心理准备。
	iOS版开发完成后要针对Android版进行一次适配，重点在UI上，比如Android上的输入框会有个很蛋疼的下划线，要用`underlineColorAndroid="transparent"`去掉。
    可以使用`Platform.OS == 'android'`的方式来根据不同平台写一些适配代码。
    阅读文档时可以看到一些专门用于适配的`props`，甚至专门用于适配平台的组件。
    Android平台，CPU架构可以去掉x86，然后开启混淆

# Flux

Flux是一种架构思想，不同于MVC，在Flux中数据的流动是单向的，目前Flux总共有16中实现，包括大名鼎鼎的Reflux和Redux。

![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016011503.png)

分发器`Dispatcher`用来处理动作`Action`，接到`Action`后，`Dispatcher`会更新数据层`Store`，数据更新后通知视图层`View`更新，`View`的一些动作（点击等）又会交给`Dispatcher`来进行分发。

# Promise

用来传递异步消息的对象，就叫`Promise`，ES6提供。

最大的好处是解决了异步编程中嵌套回调的问题，另外，链式调用的写法也让代码可读性提升很多。`Promise`不能取消。

前面介绍的异步K-V存储系统`AsyncStorage`和`fetch`都用到了`Promise`。`Promise`的状态有三种：`Pending`、`Resolved`和`Rejected`，状态只能由`Pending`变为另两种，`Resolved`和`Rejected`不能互转。`Resolved`一般表示执行成功，`Rejected`表示失败：

```javascript
getItem: function(...: Promise {
    return new Promise((resolve, reject) => {
      ...
        if (errs) { reject(errs[0]);
        } else {
          resolve(value);
        }
...
```

https://github.com/facebook/react-native

http://reactnative.cn/docs/0.43/getting-started.html#content

https://facebook.github.io/react/

http://redux.js.org/

https://reactnavigation.org/docs/intro/

http://es6.ruanyifeng.com/#README

http://www.reactnativeexpress.com/modern_javascript

盗来的一张图

![](http://upload-images.jianshu.io/upload_images/2979409-1d50d6b25bd280bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如前图，开发React Native最简单的方式是只写JS代码。

不说启动流程，后续的视图渲染、网络等都是由JS代码通过中间的C++ Bridge来调用本地代码执行的。本地代码执行的结果再通过C++ Bridge回调到JS代码。

打包后的JS bundle，Android放在assets文件夹下，图片资源放在了drawable-mdpi-v4下。

iOS平台原生提供`JavaScriptCore`，Android平台需要借助于Webkit的`android-jsc`库才能提供一个JavaScript的运行环境。

---

# Android启动流程

1. `MainApplication extends Application implements ReactApplication`：初始化了一个`ReactNativeHost`实例，用来管理一个`ReactInstanceManager`实例，另外，`ReactNativeHost`的`getPackages()`方法中返回了当前App中用到的所有`ReactPackage`，包括基本的`NativeModules`和`UIManager`。
2. `MainActivity extends ReactActivity`：其中声明了入口JS Module的名字，另外，`ReactAcitivty`的所有行为都是由`ReactActivityDelegate`代理的。
3. `loadApp(mMainComponentName)`：这个方法是在`ReactActivityDelegate`定义的，在`onCreate`方法中被调用。
4. `ReactRootView`：在`loadApp`方法中，就做了一件事：创建了一个`ReactRootView`，然后传入`ReactInstanceManager`实例、`mMainComponentName`以及启动选项（为null）调用`startReactApplication`方法，最后调用`Activity`的`setContentView`传入刚才创建的`ReactRootView`实例。
5. `createReactInstanceManager`：`startReactApplication`时调用了`ReactNativeHost#createReactInstanceManager`，其中有一段很有意思：
```javascript
    String jsBundleFile = getJSBundleFile();
    if (jsBundleFile != null) {
      builder.setJSBundleFile(jsBundleFile);
    } else {
      builder.setBundleAssetName(Assertions.assertNotNull(getBundleAssetName()));
    }
```
从这里可以看出，实际上React Native原生支持代码的热修复。
6. `createReactContext`：`ReactRootView#startReactApplication`调用了`ReactInstanceManager#createReactContextInBackground`方法，期间会调用`native`方法初始化`JSCJavaScriptExecutor`。
7. 注册表：`createReactContext`方法中会创建两张注册表：`NativeModuleRegistry`和`JavaScriptModuleRegistry`。最后会构建一个`CatalystInstanceImpl`实例，初始化时会调用native方法`initializeBridge`。从`CatalystInstanceImpl.cpp`中可以看到，这个过程主要做了两件事，一是创建了一个消息线程，一是将注册表在C++层也初始化了一份。
8. `runJSBundle`：运行JS Bundle之前，在`CatalystInstanceImpl`中创建了三条消息线程：`mUiMessageQueueThread`运行在主线程中、`mNativeModulesMessageQueueThread`以及`mJSMessageQueueThread`。
9. `loadScript`：最终调用的是`CatalystInstanceImpl#loadScriptFromAssets`，它又调用了`native`方法`jniLoadScriptFromAssets`，后面一连串调用到了`nativeToJsBridge`中的`loadApplication`函数，这个函数中调用了`JSCExecutor.cpp`中的`loadApplicationScript`，这个函数中会加载这个bundle。最后，在`flush`--`bindBridge`中还顺便初始化了JS世界中的`MessageQueue.js`的几个消息队列。
10. `setupReactContext`：最重要的有两件事：一是初始化`NativeModuleRegistry`中所有`NativeModules`；二是执行Js中`AppRegistry`，开始渲染。
11. 开始渲染：`ReactInstanceManager#attachMeasuredRootViewToInstance`中调用了JS世界的`AppRegistry.js`的`runApplication`函数。

# Native调用JS流程

`catalystInstance.getJSModule(AppRegistry.class).runApplication(jsAppModuleName, appParams);`

`JavaScriptModule`在Java世界中只有一个接口描述，在`JavaScriptModuleRegistry`的`getJavaScriptModule`方法中，返回的其实是一个动态代理类的实例，上面调用`AppRegistry#runApplication`实际上调用的是`CatalystInstanceImpl#callFunction`方法，然后转入C++层`jniCallJSFunction`函数。最后调用了`NativeToJsBridge.cpp`的`callFunction`函数，到`JSCExecutor.cpp`中的`callFunction`，再到JS世界的`m_callFunctionReturnFlushedQueueJS`的`callAsFunction`。

# 渲染流程

Java世界调用JS中`AppRegistry.js`的`runApplication`函数后，会根据传入的组件名调用`renderApplication.js`中定义的`renderApplication`函数：

```javascript
ReactNative.render(
    <AppContainer rootTag={rootTag}>
      <RootComponent
        {...initialProps}
        rootTag={rootTag}
      />
    </AppContainer>,
    rootTag
);
```

在跟踪`ReactNative`的实现时发现并不确定具体实现，于是看`package.json`中`main`字段的值，原因参考[这里](http://javascript.ruanyifeng.com/nodejs/packagejson.html)：

`"main": "Libraries/react-native/react-native-implementation.js",`

`react-native-implementation.js`：

```javascript
const ReactNative = {
	get Button() { return require('Button'); },
    ...
}
const ReactNativeInternal = require('ReactNative');
function applyForwarding(key) {
  ReactNative[key] = ReactNativeInternal[key];
}
for (const key in ReactNativeInternal) {
  applyForwarding(key);
}
module.exports = ReactNative;
```

JS中关系比较乱，这个文件中先是定义了一堆属性，然后导入了一个名为`ReactNative`的`module`，把它的能力复制了一份，自己最后又导出为`ReactNative`。那上面导入的`ReactNative`又是哪一个呢？

# @providesModule ———— Alias in React Native

详细的解释可以看[这里](https://reactnatve.wordpress.com/2016/06/16/alias-in-react-native/#more-550)。

简单说，就是起个别名，打包时通过手动指定的这个别名来判断导入哪个文件。

根据这个知识点，可以找到`ReactNative`来自`ReactNative.js`，继而追查到`ReactNativeStack.js`。

其中`render`函数的实际实现是

`ReactNativeMount.renderComponent(element, mountInto, callback);`

`element`就是传入的组件（这里是`AppContainer`），`mountInfo`是组件的ID，`callback`这里是`null`。

# `ReactNativeMount.renderComponent`

这个函数中主要有两点：

1. 如果顶部组件已存在，就判断是否要更新组件，如果要更新组件，就往`ReactUpdatesQueue`中放一条消息，否则就从当前节点卸载组件。
2. 如果顶部组件不存在，就调用`instantiateReactComponent`函数创建一个复合组件`ReactCompositeComponent`，然后调用`mountComponentIntoNode`函数挂载组件，此时会调用`render`函数。

在`renderComponent`流程的最后，会调用`_mountImageIntoNode`函数，里面调用了`UIManager.setChildren`方法，这是Java层的方法，再后面就又是一个大坑——前端布局引擎Yoga。

# 复合视图的生命周期

- constructor: Initialization of state. The instance is now retained.
  - componentWillMount
  - render
  - [children's constructors]
    - [children's componentWillMount and render]
    - [children's componentDidMount]
    - componentDidMount

      Update Phases:
      - componentWillReceiveProps (only called if parent updated)
      - shouldComponentUpdate
        - componentWillUpdate
          - render
          - [children's constructors or receive props phases]
        - componentDidUpdate

    - componentWillUnmount
    - [children's componentWillUnmount]
  - [children destroyed]
  - (destroyed): The instance is now blank, released by React and ready for GC.

# JS调用Native流程

在启动渲染的最后，JS层调用了Java层的`UIManager.setChildren`方法。

`UIManager`来自`NativeModules`，而从`NativeModules.js`中可以看到

```javascript
if (global.nativeModuleProxy) {
  NativeModules = global.nativeModuleProxy;
}
```

```c++
JSCExecutor::JSCExecutor(...) throw(JSException) :
    ...
    installGlobalProxy(m_context, "nativeModuleProxy", exceptionWrapMethod<&JSCExecutor::getNativeModule>());
}
```

可以看到`NativeModules`实际上是个C++函数`JSCExecutor::getNativeModule`。

# HybridClass

React Native自己实现了`JNI_OnLoad`函数：

```c++
extern "C" JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved) {
  return initialize(vm, [] {
    ...
    JSCJavaScriptExecutorHolder::registerNatives();
    ...
  });
}
```

`JSCJavaScriptExecutorHolder`就是一个`HybridClass`，它是Java层`JSCJavaScriptExecutor`类在C++层的实现。不过初始化`CatalystInstanceImpl`的时候，传入的是一个`JSCJavaScriptExecutor`的对象，实际上在C++层它持有的是一个`JSExecutorFactory`，在初始化`NativeToJsBridge`时，调用了它的`createJSExecutor`函数创建了`JSCExecutor`。

# C++层代理JS层的NativeModules

在初始化`JSCExecutor`时（有两构造器，跟踪代码可以发现用的是4个参数的那个），可以看到有这么一句：

```c++
installGlobalProxy(m_context, "nativeModuleProxy",
                       exceptionWrapMethod<&JSCExecutor::getNativeModule>());
```

结合`NativeModules.js`的源码，可以发现实际上`NativeModules`是被一个C++函数代理的。`exceptionWrapMethod`函数返回的是一个`JSValueRef`，即JS值的引用，关键代码如下：

```c++
auto executor = Object::getGlobalObject(ctx).getPrivate<JSCExecutor>();
        return (executor->*method)(object, propertyName);
```

So，当我们在JS层使用`NativeModules.UIManager`时，实际上调用的是`JSCExecutor::getNativeModule`函数。

# 创建Module

初始化`NativeToJsBridge`时，也顺带初始化了`JsToNativeBridge`，它持有一个`ModuleRegistry`，也就是Native模块的注册表。然后拿这个`ModuleRegistry`初始化`JSCExecutor`的`m_nativeModules`，`getNativeModule`实际调用的是`m_nativeModules.getModule`。

转入`JSCNativeModules.cpp`，如果该`Module`还没有被创建，就调用`createModule`创建一个。创建过程就是执行JS世界的`__fbGenNativeModule`，其实就是`NativeModules.js`中的`genModule`函数，根据传入的config和Module名来动态创建一个Module。

`ModuleRegistry::getConfig`中通过`JavaNativeModule`的`wrapper`(`JavaModuleWrapper`)的同名方法，调用到Java层的`JavaModuleWrapper`类，也就是说JNI层从Java层取数据，然后交给JS层创建Module。

看`NativeModules.js`中的`genMethod`函数，调用创建的Module的方法，除`sync`方法外，实际上调用的都是：

`BatchedBridge.enqueueNativeCall(moduleID, methodID, args, onFail, onSuccess);`

如果是`promise`函数，`onFail`就是`reject`，`onSuccess`就是`resolve`；如果是普通函数，`onFail`是倒数第二个参数（如果这个参数是`function`的话），`onSuccess`是倒数第一个参数（如果这个参数是`function`的话）。

# callNativeModule

`BatchedBridge.enqueueNativeCall`的逻辑和Native调JS差不多，将消息放入消息队列中，然后调用`global.nativeFlushQueueImmediate(this._queue);`，然后通过`JsToNativeBridge`的`callNativeModules`，继续调用`ModuleRegistry::callNativeMethod`，调用到`modules_[moduleId]->invoke(token, methodId, std::move(params));`

`modules_[moduleId]`是一个`JavaNativeModule`，它的`invoke`函数是往消息队列中扔一个调用Java层`JavaModuleWrapper.invoke`方法的消息，执行`JavaModuleWrapper.invoke`时，用反射取得真正要调用的方法，然后执行。