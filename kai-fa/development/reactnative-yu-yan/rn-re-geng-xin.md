---
description: >-
  此文使用当前最新版本的`RN`与`Code-Push`进行演示，其中的参数不会过多进行详细解释，更多参数解释可参考其它文章，这里只保证APP能正常进行热更新操作，方便快速入门
---

# RN热更新

创建`React-Native`项目

```bash
react-native init dounineApp
```

安装`code-push-cli`

```bash
npm install -g code-push-cli
```

注册`code-push`帐号

```bash
$ code-push register
Please login to Mobile Center in the browser window we've just opened.
Enter your token from the browser: 
#会弹出一个浏览器,让你注册,可以使用github帐号对其进行授权,授权成功会给一串Token,点击复制，在控制进行粘贴回车(或者使用code-push login命令)。
```
```
Enter your token from the browser:  b0c9ba1f91dd232xxxxxxxxxxxxxxxxx
#成功提示如下方
Successfully logged-in. Your session file was written to /Users/huanghuanlai/.code-push.config. You can run the code-push logout command at any time to delete this file and terminate your session.
```

![](../../../.gitbook/assets/image%20%2813%29.png)

在`code-push`添加一个ios的app

```bash
$ code-push app add dounineApp-ios ios react-native
#成功提示如下方
Successfully added the "dounineApp-ios" app, along with the following default deployments:
┌────────────┬──────────────────────────────────────────────────────────────────┐
│ Name       │ Deployment Key                                                   │
├────────────┼──────────────────────────────────────────────────────────────────┤
│ Production │ yMAPMAjXpfXoTfxCd0Su9c4-U4lU6dec4087-57cf-4c9d-b0dc-ad38ce431e1d │
├────────────┼──────────────────────────────────────────────────────────────────┤
│ Staging    │ IjC3_iRGEZE8-9ikmBZ4ITJTz9wn6dec4087-57cf-4c9d-b0dc-ad38ce431e1d │
└────────────┴──────────────────────────────────────────────────────────────────┘
```

继续在`code-push`添加一个android的app

```bash
$ code-push app add dounineApp-android android react-native
#成功提示如下方
Successfully added the "dounineApp-android" app, along with the following default deployments:
┌────────────┬──────────────────────────────────────────────────────────────────┐
│ Name       │ Deployment Key                                                   │
├────────────┼──────────────────────────────────────────────────────────────────┤
│ Production │ PZVCGLlVW-0FtdoCF-3ZDWLcX58L6dec4087-57cf-4c9d-b0dc-ad38ce431e1d │
├────────────┼──────────────────────────────────────────────────────────────────┤
│ Staging    │ T0NshYi9X8nRkIe_cIRZGbAut90a6dec4087-57cf-4c9d-b0dc-ad38ce431e1d │
└────────────┴──────────────────────────────────────────────────────────────────┘
```

在项目根目录添加`react-native-code-push`

```bash
npm install react-native-code-push --save
#或者
yarn add react-native-code-push
```

链接

```bash
$ react-native link
Scanning folders for symlinks in /Users/huanghuanlai/dounine/oschina/dounineApp/node_modules (8ms)
? What is your CodePush deployment key for Android (hit <ENTER> to ignore) T0NshYi9X8nRkIe_cIRZGbAut90a6dec4087-57cf-4c9d-b0dc-ad38ce431e1d

#将刚才添加的Android App的Deployment Key复制粘贴到这里,复制名为Staging测试Deployment Key。

rnpm-install info Linking react-native-code-push android dependency 
rnpm-install info Android module react-native-code-push has been successfully linked 
rnpm-install info Linking react-native-code-push ios dependency 
rnpm-install WARN ERRGROUP Group 'Frameworks' does not exist in your Xcode project. We have created it automatically for you.
rnpm-install info iOS module react-native-code-push has been successfully linked 
Running ios postlink script
? What is your CodePush deployment key for iOS (hit <ENTER> to ignore) IjC3_iRGEZE8-9ikmBZ4ITJTz9wn6dec4087-57cf-4c9d-b0dc-ad38ce431e1d

#继续复制Ios的Deployment Key

Running android postlink script
```

在`react-native`的`App.js`文件添加自动更新代码

```javascript
import codePush from "react-native-code-push";
const codePushOptions = { checkFrequency: codePush.CheckFrequency.MANUAL };
export default class App extends Component<{}> {
  
  componentDidMount(){
    codePush.sync({
      updateDialog: true,
      installMode: codePush.InstallMode.IMMEDIATE,
      mandatoryInstallMode:codePush.InstallMode.IMMEDIATE,
      //deploymentKey为刚才生成的,打包哪个平台的App就使用哪个Key,这里用IOS的打包测试
      deploymentKey: 'IjC3_iRGEZE8-9ikmBZ4ITJTz9wn6dec4087-57cf-4c9d-b0dc-ad38ce431e1d',
      });
  }
  ...
```

运行

```javascript
react-native run-ios
```

### IOS发布

![&#x8C03;&#x8BD5;](../../../.gitbook/assets/image%20%287%29.png)

发布一个ios新版本

```javascript
$ code-push release-react dounineApp-ios ios
Detecting ios app version:

Using the target binary version value "1.0" from "ios/dounineApp/Info.plist".

Running "react-native bundle" command:

node node_modules/react-native/local-cli/cli.js bundle --assets-dest /var/folders/m_/xcdff0xd62j4l2xbn_nfz00w0000gn/T/CodePush --bundle-output /var/folders/m_/xcdff0xd62j4l2xbn_nfz00w0000gn/T/CodePush/main.jsbundle --dev false --entry-file index.js --platform ios
Scanning folders for symlinks in /Users/huanghuanlai/dounine/oschina/dounineApp/node_modules (10ms)
Scanning folders for symlinks in /Users/huanghuanlai/dounine/oschina/dounineApp/node_modules (10ms)
Loading dependency graph, done.

bundle: start
bundle: finish
bundle: Writing bundle output to: /var/folders/m_/xcdff0xd62j4l2xbn_nfz00w0000gn/T/CodePush/main.jsbundle
bundle: Done writing bundle output

Releasing update contents to CodePush:

Upload progress:[==================================================] 100% 0.0s
Successfully released an update containing the "/var/folders/m_/xcdff0xd62j4l2xbn_nfz00w0000gn/T/CodePush" directory to the "Staging" deployment of the "dounineApp-ios" app.
```

重新Load刷新应用

![](../../../.gitbook/assets/image%20%2810%29.png)

### 安卓发布

与上面9~11步骤是一样的，命令改成Android对应的,以下命令结果简化

修改App.js的deploymentKey为安卓的

```javascript
deploymentKey:'T0NshYi9X8nRkIe_cIRZGbAut90a6dec4087-57cf-4c9d-b0dc-ad38ce431e1d'
```

运行

```javascript
react-native run-android
```

发布

```javascript
code-push release-react dounineApp-android android
```

刷新应用

![](../../../.gitbook/assets/image%20%283%29.png)

