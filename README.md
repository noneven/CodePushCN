## 提供RN代码升级的RN插件-CodePush

>这个插件提供了一个客户端集成的Codepush服务，它让你很容易向你的RN应用增加一个动态的更新

* [CodePush的使用步骤](/Useage.md)
* [它是怎样运作的?](#它是怎样运作的?)
* [支持的RN平台](#支持的RN平台)
* [支持的组件](#支持的组件设置)
* [入门指南](#入门指南)
* [IOS安装](#IOS配置)
	* [插件安装](#插件安装（IOS）)
		* [npm全局安装插件](#IOS插件安装（RNPM）)
		* [CocoaPods](#IOS插件安装iOS - CocoaPods)
		* [手动操作](#IOS插件安装（IOS 手工）)
	* [插件配置](#插件配置（IOS）)
* 安卓安装
	* 插件安装
	* 插件配置（RN<0.18.0）
	* 插件配置（RN>=0.18.0）
* [插件用法](#使用插件)
* [发布更新（只有JS代码时的更新方法）](#发布仅仅有JavaScript变更的更新)
* [发布更新（有JS+IMG时的更新方法）](#发布有JavaScript变更和图片引用的更新)
* [API参考](#API参考)
	* [javascript API](#JavaScript API参考)
	* [OC API （IOS）](#OC API)
	* java API （Android）
* 参考应用
* 调试和故障解决

### <a name="它是怎样运作的?">CodePush是怎么运作的?</a>

一个RN应用一般由一些JS文件和附带的一些图片组成，他们通过打包管理器打包在一起并且他们被构建在两个具体的应用包上（例如：安卓.apk和IOS.ipa），一旦APP发布，更新JS代码（处理bug或者添加新的功能）和图片就需要你去重新编译和打包整个应用包，当然，那还会包括你要发布到的商店的重新审查时间。

CodePush插件通过保持你的JS代码和图片和你提交到CodePush服务器上的代码同步的方式来帮助我们的产品改善立即展现给你的终端用户。通过这种方式，你的APP拥有了离线可移动的体验，并且他们一有更新就会像web的更新方式一样去获取更新，这真是一大亮点啊！

### <a name="支持的RN平台">支持的RN平台</a>

* 安卓
* IOS

我们尽最大努力去让我们的插件保持在RN以前版本的可用，但是由于平台的特性，并且在发布的版本之间存在间隙性的改变，为了支持你使用的RN版本你还有可能需要使用一个具体的Codepush插件版本。下面的图片展示了RN版本需要使用对应的Codepush版本。

| React Native 版本(s)     | 支持的CodePush 版本(s)                           |
|-------------------------|------------------------------------------------|
| <0.14.0                 | **不支持**                                      |
| v0.14.0                 | v1.3.0 *(introduced Android support)*          |
| v0.15.0-v0.18.0         | v1.4.0-v1.6.0 *(introduced iOS asset support)* |
| v0.19.0-v0.22.0         | v1.7.0+ *(introduced Android asset support)*   |
| v0.23.0+                | TBD :) 我们努力去支持RN发布的所有版本，但是有可能会有一些中断，到时每一个RN版本发布我们都会更新这个表格，方便让用户清晰地看到我们“官方”是否支持。|

### <a name="支持的组件设置">支持的组件设置</a>

当使用RN的资源系统（例如使用`require("./foo.png")`语法），下面的列表代表 支持它们所引用的图片更新 的组件的设置（或者说是props）


| Component                                       | Prop(s)                                  | 
|-------------------------------------------------|------------------------------------------|
| `Image`                                         | `source`                                 |
| `ProgressViewIOS`                               | `progressImage`, `trackImage`            |
| `TabBarIOS.Item`                                | `icon`, `selectedIcon`                   |
| `ToolbarAndroid` <br />*(React Native 0.21.0+)* | `actions[].icon`, `logo`, `overflowIcon` |

下面的图表展示了 不支持他们所引用的资源更新，由于它们对静态资源图片的依赖性（例如使用`{ uri: "foo"}`语法）。

| Component   | Prop(s)                                                              |
|-------------|----------------------------------------------------------------------|
| `SliderIOS` | `maximumTrackImage`, `minimumTrackImage`, `thumbImage`, `trackImage` |

由于支持引用资源的新的核心组件还在不断发布，所以为了确保使用者知道他们能使用CodePush更新什么。

### <a name="入门指南">入门指南</a>

一旦你循序渐进看到了入门指南，你就应该知道它的普遍用途，这里它主要介绍的是关于CodePush账户的注册使用。你可以通过在你的APP更目录执行下面的命令开启你的RN APP的CodePush使用之路

```javascript
npm install --save react-native-code-push
```

对于所有的RN插件，对IOS和安卓的整合方式都是不同的，因此下面的设置步骤依赖于你是什么平台，如果你想看其他项目是怎样集成CodePush的话，你可以去查看社区提供的这个例子 [example app](https://github.com/Microsoft/react-native-code-push/blob/master/README.md#example-apps)

### <a name="IOS配置">IOS配置</a>


如果你已经安装了CodePush插件，你需要把它集成到你的RN应用的Xcode项目中去并且需要正确的配置它。需要做的是依次进行下面的几个步骤：

#### <a name="插件安装（IOS）">插件安装（IOS）</a>

为了尽可能的去适应一些开发者的开发习惯，对于IOS，CodePush支持以下三种方式进行安装。

1、RNPM - [React Native Package Manager (RNPM)](http://github.com/rnpm/rnpm)是一个提供了非常好的可以使用很简单的步骤去安装Rn插件的工具。如果你准备使用它，或者你想要使用它，我们都很推荐用这种方式去安装

2、[CocoaPods](https://github.com/Microsoft/react-native-code-push/blob/master/README.md#plugin-installation-ios---cocoapods)-如果你是在你现有的Native APP中集成RN，或者你更喜欢使用CocoaPods，我们推荐你使用我们插件里面的Podspec文件。

3、[Manual](https://github.com/Microsoft/react-native-code-push/blob/master/README.md#plugin-installation-ios---manual)如果你不想依赖其他额外的工具或者很在乎其他额外的安装步骤带来的开销（毕竟这些工作只会做一次），那么采用下面的这种手动添加的方式：

#### <a name="IOS插件安装（RNPM）">IOS插件安装（RNPM）</a>

1、运行命令 `rnpm link react-native-code-push`

>Note：如果你还没有安装RNPM，你应该先简单的执行`[sudo] npm i -g rnpm` 然后再执行以上命令。

2、打开你需要集成CodePush的APP的Xcode 项目，

3、选择最顶层的项目节点，并且在你的工程配置界面中选择 "Build Phases" 这个tab

4、点击下面的"Link Binary With Libraries"列，然后点击这列左下角的"+",选择和你当前目标版本相匹配的“libz.tbd”文件。

![xx](https://cloud.githubusercontent.com/assets/116461/11605042/6f786e64-9aaa-11e5-8ca7-14b852f808b1.png)
>Note: Alternatively, if you prefer, you can add the -lz flag to the Other Linker Flags field in the Linking section of the Build Settings.

我们希望最终可以移除2-4步骤，但是RNPM暂时还不支持将2-4步骤自动化执行。

#### <a name="IOS插件安装iOS - CocoaPods">IOS插件安装(iOS - CocoaPods)</a>

1、向你的Podfile中添加CodePush插件的依赖，指向NPM安装它的路径

```javascript
pod 'CodePush', :path => './node_modules/react-native-code-push`
```

2、执行命令`pod install`

#### <a name="IOS插件安装（IOS 手工）">IOS插件安装（IOS 手工）</a>

1、打开你的Xcode项目

2、找到`node_modules/react-native-code-push/ios`目录下的`CodePush.xcodeproj`文件（如果CodePush版本<=1.7.3-beta就是`node_modules/react-native-code-push`目录下）。并且把它拖到你的项目的`Libraries`节点里。

![2](https://cloud.githubusercontent.com/assets/8598682/13368613/c5c21422-dca0-11e5-8594-c0ec5bde9d81.png)

3、选择项目根目录，并且选中右边工程配置区的"Build Phases" tab

4、把`libCodePush.a`拖到"Link Binary With Libraries" 区域里面去

![4](https://cloud.githubusercontent.com/assets/516559/10322221/a75ea066-6c31-11e5-9d88-ff6f6a4d6968.png)

5、点击"Link Binary With Libraries"下面的“+”，并且选择`IOS 9.1`下面的`libz.tbd`文件。

![5](https://cloud.githubusercontent.com/assets/116461/11605042/6f786e64-9aaa-11e5-8ca7-14b852f808b1.png)

>Note: Alternatively, if you prefer, you can add the -lz flag to the Other Linker Flags field in the Linking section of the Build Settings.

6、点击你的工程配置区的`Build Settings`tab，找到“Header Search Paths”并且双击它编辑值，点击弹出区域的左下角“+”天机一个新值：`$(SRCROOT)/../node_modules/react-native-code-push`，并且选择右边的“recursive”。

![6](https://cloud.githubusercontent.com/assets/516559/10322038/b8157962-6c30-11e5-9264-494d65fd2626.png)

### <a name="插件配置（IOS）">插件配置（IOS）</a>

当你的Xcode项目已经配置好了CodePush插件，你还需要配置你的APP在你的JS Bundle中运用CodePush。为了是JS Bundle和你发布到CodePush服务器上的文件保持同步更新，你需要做以下几个步骤：
1、打开`AppDelegate.m`文件并且增加一个CodePush Header import。

```javascript
#import "CodePush.h"
```

2、注释掉下面这句

```javascript
jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
```

3、用下面这句代替

```javascript
jsCodeLocation = [CodePush bundleURL];
```

你的APP里面做的这些配置（改变）是为了让你的APP可以加载最新版本的JS bundle文件。当第一次启动APP的时候APP将会和编译到APP里面的JS Bundle文件通信，然而，在通过CodePush提交过更新后，CodePush将会返回最近安装更新的下载地址。

>NOTE: `bundleURL`方法假定你的APP的`JS bundle`文件被命名为为`main.jsbundle`。如果你想配置不同的名字，简单的调用`bundleURLForResource`方法(假定你是用的是jsbundle后缀名) 或者`bundleURLForResource:withExtension:`方法, 去重写默认的名字

你仅仅想使用CodePush在发布工程中解决JS Bundle文件的路径问题，因此我们建议你使用DEBUG宏命令在包管理服务器和CodePush服务器之间切换，这依赖于你是否正在DEBUG，这将确保你在生产环境中得到准确的行为。你仍然可以使用Chrome Dev Tools ReLoad等等。

```javascript
NSURL *jsCodeLocation;

#ifdef DEBUG
    jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle?platform=ios&dev=true"];
#else
    jsCodeLocation = [CodePush bundleURL];
#endif
```

为了让CodePush在运行期间知道它应该去哪个部署查找更新。需要做以下几个步骤：

1、打开你的APP的`Info.plist`文件，并且增加一个新Key`CodePushDeploymentKey`,它的值是你想要再次配置你的APP的`deployment Key`(e.g. the key for the Staging deployment for the FooBar app),你可以运行`code-push deployment ls <appName> -k `
![xx](https://cloud.githubusercontent.com/assets/116461/11601733/13011d5e-9a8a-11e5-9ce2-b100498ffb34.png)
复制`Deployment Key`你想要去使用的部署环境那列。

>NOTE:使用部署名字可能不会工作（例如Staging），最好的名字是使用CLI指定的部署名字。

2、在你的APP的`Info.plist`确保`Bundle versions string, short`的值是一个有效的语义化版本号(一般会有一个补丁版本号)

>NOTE: 如果他没有一个补丁版本号，那么CodePush服务器将会假定它为0，例如1.0会被认为是1.0.0。

![xx](https://cloud.githubusercontent.com/assets/116461/12307416/f9b82688-b9f3-11e5-839a-f1c6b4acd093.png)

### <a name="使用插件">使用插件</a>

当你把插件下载并且链接进入你的项目里面后，你的APP会询问CodePush服务器应该从什么地方获取JS Bundle文件，你必须在你的APP 代码里面添加一些必要的代码。

#### <a name="实现更新策略">实现更新策略</a>

a、多久多频繁去check更新，例如app启动时，还是在setting 页面点击更新，或定时去更新
b、当一个更新出现，如何向用户展现
最简单的方式是在你的APP根组件里面操作以下步骤：

1、引入CodePush模块

```javascript
import codePush from "react-native-code-push";
```
2、在组件生命周期的componentDidMount方法里面调用CodePush的sync方法，在每次app启动的时候初始化一个静默更新。

```javascript
codePush.sync();
```
如果有更新，将会被悄悄下载，在用户或系统重启app后，就会安装为新的，为用户提供了一个可扩展的体验。如果设置为强制更新，那么app启动检查到有更新将立刻下载安装更新，让用户尽可能的去获得最近的更新，另外，如果你想要去展示一个确认弹框（一个“可用安装包”），或者通过其他方式改造一些更新体验，sync方法的API里面介绍了怎样去调整默认的一些表现。

>NOTE:苹果开发者协议允许静默更新js文件和一些资源(CodePush就是这样做的)，但是他是反对给APP展示一个更新提示，所以我们建议在AppStore上架是禁用掉更新弹框（当调用sync方法时设置updateDialog为false），然而在谷歌商店和一些其他的商店上架时可以给出更新提示框

#### <a name="发布仅仅有JavaScript变更的更新">发布仅仅有JavaScript变更的更新</a>

你的APP已经配置好了并且已经发布给用户了，现在你需要对立面的js做一些变更，到了需要立即发布更新的时候。如果你没有使用RN去打包图片（通过`require('./foo.png')`），那么执行以下几个步骤，否则跳过这一块：

1、执行`react-native bundle`(需要加一些参数，具体看下面的例子)去打包你需要为你的APP更新的JS Bundle文件。你能通过`--bundle-output`参数设置你需要输出的Bundle文件的位置，这个文件可以是任何路径甚至和CodePush的路径毫无关联。另一个比较重要的是去设置`--dev false`,他可以去压缩你的代码并且去掉一些黄色的警告提示。

2、执行`code-push release <appName> <jsBundleFilePath> <targetBinaryVersion> --deploymentName <deploymentName>`命令去发布生成的JS Bundle文件到CodePush服务器。这个`<jsBundleFilePath>`参数应该和你`--bundle-output`参数输出的地址保持一致，另外，`<targetBinaryVersion>`参数应该等于你想要CodePush去更新的APP在APPStore里的版本[exact app store version](http://codepush.tools/docs/cli.html#app-store-version-parameter)。更多关于`release`命令和它的参数，请参考[CLI documentation.](http://codepush.tools/docs/cli.html#releasing-app-updates)。

##### <a name="使用例子">使用例子</a>

```javascript
react-native bundle --platform ios --entry-file index.ios.js --bundle-output codepush.js --dev false
code-push release MyApp codepush.js 1.0.0
```
更多关于`CLI`命令和发布命令的详细使用请参考这个文档（[documentation](http://microsoft.github.io/code-push/docs/cli.html)）,另外，如果在使用过程中有什么问题，你可以在Reactiflux上的CodePush栏目联系我们或者给我们发邮件（codepushfeed@microsoft.com），或者在最后写下你遇到的问题详情。
>NOTE:不是必须采用`react-native bundle`，你可以运行`xcodebuild`或者`gradlew assemble`去生成JS Bundle文件。但是生成Bundle文件和发布Bundle文件不需要重新构建一遍Native。

#### <a name="发布有JavaScript变更和图片引用的更新">发布有JavaScript变更和图片引用的更新</a>

>Note: 安装在RN V0.19.0支持了assets资源更新，并且需要你使用CodePush V1.7.0，如果你是用的是比较老的RN版本，你将不能通过CodePush更新你的assets资源，你需要做的是升级或者使用你以前的工作方式去更新JSBundle文件。

如果你是用的是新的RN 资源系统（[assets system](https://facebook.github.io/react-native/docs/images.html#content)）,它不支持你通过网路或者具体的平台机制加载图片（例如 IOS的资源目录iOS asset catalogs），你将不能简单的向上面写到的那样简单的提交JS Bundle文件到CodePush服务器。你必须提供你的图片资源，要做到这些，可以简单的使用下面的工作方式。

1、创建一个文件夹去组织你的APP需要发布的资源（比如JSBundle文件和图片资源）。我们建议使用`release`作为文件夹的名字，当然你也可以使用其他名字。如果你在你的工程中创建了这个文件夹，确认增加一个`.gitignore`文件，因为你不需要对这个文件夹做控制（比如在使用github做代码管理时）。

2、当你调用`react-native bundle`的时候，你的文件和图片等静态资源就会生成到这个文件夹下。并且对各自平台和入口文件 你不用开发构建。假定你把文件夹命名为`release`,你应该运行下面的命令：

```javascript
react-native bundle \
--platform ios \
--entry-file index.ios.js \
--bundle-output ./release/main.jsbundle \
--assets-dest ./release \
--dev false
```

>NOTE:你通过`--bundle-output`输出的文件名必须有一个扩展名(.js/.jsbundle/.bundle),例如（main.jsbundle, ios.js），名字没有关系,但是扩展名在更新内容查找的时候需要使用，因此使用其他扩展名将会导致更新失败。

3、执行`code-push release`命令，后面跟第一个参数是你要发布更新的APP名称，第二个是你刚才打包生成Bundle的文件夹名称，第三个是你需要把这个jsBundle推送给你的APP版本，CodePush命令会自动帮你处理你的内容的压缩，你不必自己处理这些。更多介绍参考CLI文档（[CLI文档](http://codepush.tools/docs/cli.html#releasing-app-updates)）

CodePush支持增量升级（也叫差别升级），即使你每一次更新都发布了JSBundle和assets资源，但是你的用户只会下载他们各自需要的部分，你主要精力集中在构建你的APP的业务，我们尽最大可能去减少用户端的下载量。

如果在使用过程中有什么问题，你可以在Reactiflux上的CodePush栏目联系我们或者给我们发邮件（codepushfeed@microsoft.com），或者在最后写下你遇到的问题详情。

>NOTE:发布assets更新需要你使用RN V0.15.0+并且CodePush V1.4.0+，对于安卓你需要使用RN v0.19.0+和CodePush V1.7.0+。如果你使用	的事老版本的CodePush插件你将不能通过CodePush发布asserts更新。因为那将会是APP在加载图片资源的时候发生中断。请注意测试好了再发布。

### <a name="API参考">API参考</a>

CodePush插件有两部分构成，

1、一个需要被引入的JavaScript模块，允许APP在运行过程中和服务器交互。（例如检查更新，检查当前运行APP更新数据）。

2、一个让RN APP定位正确JSBundle位置的Native API（java/OC）。

下面部分详细的描述了这些API具体情况。

#### <a name="JavaScript API参考">JavaScript API参考</a>

当你require `react-native-code-pus`的时候，这个模块提供一些顶级方法：

* checkForUpdate: 询问你CodePush服务器是否有当前APP版本的更新可用。(返回一个Promise)，回调函数传回的update对象就是下面讲到的LocalPackage对象，你可以通过这个对象实现更新。

```javascript
codePush.checkForUpdate()
.then((update) => {
    if (!update) {
        console.log("The app is up to date!"); 
    } else {
        console.log("An update is available! Should we download it?");
    }
});
```

* getCurrentPackage:去除当前需要更新的元数据（比如描述、安装时间、和大小）。（返回Promise），回调函数传回的update对象就是下面讲到的RemotePackage对象，你可以通过这个对象下载更新。


```javascript
codePush.getCurrentPackage()
.then((update) => {
    // If the current app "session" represents the first time
    // this update has run, and it had a description provided
    // with it upon release, let's show it to the end user
    if (update.isFirstRun && update.description) {
        // Display a "what's new?" modal
    }
});
```
* notifyApplicationReady: 通知CodePush运行环境一个可安装的更新成功拿到。如果你手动检查和安装更新（不适用同步方法来处理），这个方法必须被调用。否则CodePush将把这个更新当做失败处理，并且当你app重启的时候回滚到前一个版本。(没有返回值)

* restartApp:立即重启APP,如果有更新阻塞了，它将马上呈现给用户，因此，调用这个方法和用户主动杀死程序重启APP有相同的效果。

```javascript
codePush.restartApp(onlyIfUpdateIsPending: Boolean = false): void; 
```
* sync:通过简单的调用这个方法可以实现检查更新，下载并且安装它，除非你有一些自定义的UI或者交互，我们建议开发者在集成CodePush到他们的APP的时候使用这个方法。

```javascript
codePush.sync(options: Object, syncStatusChangeCallback: function(syncStatus: Number), downloadProgressCallback: function(progress: DownloadProgress)): Promise<Number>;
```
这个方法支持两种不同的模式更新：
1、静默升级（自动下载可用更新，重启APP是安装更新，用户完全没有感知）
2、主动升级（用户主动选择更新）

使用例子：

```javascript
// Fully silent update which keeps the app in
// sync with the server, without ever 
// interrupting the end user
codePush.sync();

// Active update, which lets the end user know
// about each update, and displays it to them
// immediately after downloading it
codePush.sync({ updateDialog: true, installMode: codePush.InstallMode.IMMEDIATE });
```

sync方法接受的第一个参数是`options`对象，可以自定义一些默认行为的提示：

* installMode:安装模式(立即安装、下次重启时安装，下次进入应用时安装等等)【可以这样理解restart和resume，一个是杀死进程重启，一个是下次在进入（不一定杀死应用）】
	codePush.InstallMode.IMMEDIATE
	codePush.InstallMode.ON_NEXT_RESTART
	codePush.InstallMode.ON_NEXT_RESUME
* minimumBackgroundDuration:RESUME更新的倒计时秒数
* updateDialog:对象可以有以下key
	
| 属性|中文解释|默认值|
|-----------|---------------|---------------------|
| `appendReleaseDescription`|`是否展示发布描述`|false|
| `descriptionPrefix` | `发布的描述`|Description|
| `mandatoryContinueButtonLabel`   | `使用强制更新时提示用户的按钮文案`   |Continue|
| `mandatoryUpdateMessage` | `强制更新时的提示文案` |An update is available that must be installed.|
| `optionalIgnoreButtonLabel`    | `忽略按钮文案` |Ignore|
| `optionalInstallButtonLabel`   | `立即安装文案` |Install|
| `optionalUpdateMessage`     | `更新提示文案` |An update is available. Would you like to install it?|
| `title`         | `更新提示弹框标题` |Update available|
	
使用例子

```javascript
// Use a different deployment key for this
// specific call, instead of the one configured
// in the Info.plist file
codePush.sync({ deploymentKey: "KEY" });

// Download the update silently, but install it on
// the next resume, as long as at least 5 minutes
// has passed since the app was put into the background.
codePush.sync({ installMode: codePush.InstallMode.ON_NEXT_RESUME, minimumBackgroundDuration: 60 * 5 });

// Download the update silently, and install optional updates
// on the next restart, but install mandatory updates on the next resume.
codePush.sync({ mandatoryInstallMode: codePush.InstallMode.ON_NEXT_RESUME });

// Changing the title displayed in the
// confirmation dialog of an "active" update
codePush.sync({ updateDialog: { title: "An update is available!" } });

// Displaying an update prompt which includes the
// description associated with the CodePush release
codePush.sync({
   updateDialog: {
    appendReleaseDescription: true,
    descriptionPrefix: "\n\nChange log:\n"   
   },
   installMode: codePush.InstallMode.IMMEDIATE
});
```

sync第二个参数`syncStatusChangedCallback`同步状态变化回调函数：
回调函数传过来的参数是一个字符串，表示当前进行到的状态。

sync第三个参数`downloadProgressCallback`下载进度回调函数：
回调函数传过来的参数是一个对象包括了totalBytes和receivedBytes总字节数和已经接受的字节数

使用例子：

```javascript
// Prompt the user when an update is available
// and then display a "downloading" modal 
codePush.sync({ updateDialog: true }, (status) => {
    switch (status) {
        case codePush.SyncStatus.DOWNLOADING_PACKAGE:
            // Show "downloading" modal
            break;
        case codePush.SyncStatus.INSTALLING_UPDATE:
            // Hide "downloading" modal
            break;
    }
});
```
status有下面几种状态

| 状态| 中文解释| 
|-----------------|----------------|
| `codePush.SyncStatus.CHECKING_FOR_UPDATE`    | `正在检查更新`|
| `codePush.SyncStatus.AWAITING_USER_ACTION` | `询问用户操作`|
| `codePush.SyncStatus.DOWNLOADING_PACKAGE`   | `下载安装包`   |
| `codePush.SyncStatus.INSTALLING_UPDATE` | `安装更新包` |
| `codePush.SyncStatus.UP_TO_DATE`    | `当前版本没有更新` |
| `codePush.SyncStatus.UPDATE_IGNORED`   | `更新被忽略` |
| `codePush.SyncStatus.UPDATE_INSTALLED`     | `更新安装完成` |
| `codePush.SyncStatus.SYNC_IN_PROGRESS`         | `更新下载` |
| `codePush.SyncStatus.UNKNOWN_ERROR`   | `出错了` |

```javascript
CodePush.sync({ 
 updateDialog: {
   title:"更新提示",
   optionalIgnoreButtonLabel: "忽略",
   optionalInstallButtonLabel: "安装",
   optionalUpdateMessage:"有一个提示可用，是否安装？",
 },
 installMode: CodePush.InstallMode.ON_NEXT_RESUME,
}, (syncStatus) => {
 switch(syncStatus) {
   case CodePush.SyncStatus.CHECKING_FOR_UPDATE: 
     console.log("Checking for update..")
     break;
   case CodePush.SyncStatus.DOWNLOADING_PACKAGE:
     console.log("Downloading package..")
     break;
   case CodePush.SyncStatus.AWAITING_USER_ACTION:
     console.log("Awaiting user action.0")
     break;
   case CodePush.SyncStatus.INSTALLING_UPDATE:
     console.log("Installing update..")
     break;
   case CodePush.SyncStatus.UP_TO_DATE:
     console.log("App up to date..")
     break;
   case CodePush.SyncStatus.UPDATE_IGNORED:
     console.log("Update cancelled by user..")
     break;
   case CodePush.SyncStatus.UPDATE_INSTALLED:
     console.log("Update installed and will be run when the app next resumes..")
     break;
   case CodePush.SyncStatus.UNKNOWN_ERROR:
     console.log("An unknown error occurred..");
     break;
 }
})
```

效果图：
![](/images/CodePush测试.png)

### Package对象

#### LocalPackage

>checkForUpdate如果检查到更新将返回的一个LocalPackage对象，代表这个包被下载到本地了但是还未安装，

* 属性：

| 英文属性| 中文解释| 
|-----------------|----------------|
| `appVersion`    | `获得这个更新的AppStore的版本`|
| `deploymentKey` | `下载更新是需要的部署key`|
| `description`   | `更新包描述`   |
| `failedInstall` | `安装失败` |
| `isFirstRun`    | `是否第一次启动` |
| `isMandatory`   | `是否是强制更新方式` |
| `isPending`     | `是否被阻塞` |
| `label`         | `CodePush服务器返回的唯一key` |
| `packageHash`   | `更新的SHA hash` |
| `packageSize`   | `更新包大小(字节数)` |



* 方法：

可以传安装方式和到计时时间

```javascript
install(installMode: codePush.InstallMode = codePush.InstallMode.ON_NEXT_RESTART, minimumBackgroundDuration = 0): Promise<void>: 
```

#### RemotePackage对象

>getCurrentPackage返回的对象

* 属性：

downloadUrl

* 方法：

```javascript
download(downloadProgressCallback?: Function): Promise<LocalPackage>:
```

>callback返回update = { totalBytes: Number, receivedBytes: Number }对象，可以拿到进度信息。


### <a name="OC API">OC API</a>
#### CodePush

方法：

1、`(NSURL *)bundleURL`

返回JSBundle文件路径，这个方法假定JSBundle文件的名字为main.jsbundle

2、`(NSURL *)bundleURLForResource:(NSString *)resourceName`

和bundleURL方法一样，但是可以通过参数设置JSBundle文件的名字

3、`(NSURL *)bundleURLForResource:(NSString *)resourceName withExtension:(NSString *)resourceExtension: `

和bundleURLForResource方法一样，但是这个方法不但允许你设置你的JSBundle文件名还可以设置文件扩展名(js/jsbundle/bundle)。

4、`(void)setDeploymentKey:(NSString *)deploymentKey`

在查询更新的时候设置部署key


| 方法| 解释| 
|-----------------|--------------------------|
| `(NSURL *)bundleURL`| `返回JSBundle文件路径，这个方法假定JSBundle文件的名字为main.jsbundle`|
| `(NSURL *)bundleURLForResource:(NSString *)resourceName` | `和bundleURL方法一样，但是可以通过参数设置JSBundle文件的名字`|
| `(NSURL *)bundleURLForResource:(NSString *)resourceName withExtension:(NSString *)resourceExtension:`   | `和bundleURLForResource方法一样，但是这个方法不但允许你设置你的JSBundle文件名还可以设置文件扩展名(js/jsbundle/bundle)。`   |
| `(void)setDeploymentKey:(NSString *)deploymentKey` | `在查询更新的时候设置部署key` |

