## CodePush使用步骤

#### 1、全局安装code-push CLI

```javascript
npm install -g code-push-cli
```

>查看安装是否成功code-push -v（注意不同版本的CodePush支持的RN版本不同）

#### 2、通过github账号注册CodePush用户

```javascript
code-push register
```

>终端输入 `code-push register`，会弹出网页 "https://codepush.azurewebsites.net/auth/register?hostname=imChenJiandeMacBook-Pro.local"
不出意外最后是你机器的名字。输入了gitHub的帐号，允许了权限之后，CodePush自动给你注册了一个帐号，后面会弹出一个token。这是你输入code-push login将会提示你已经登陆了（[Error]  You are already logged in from this machine.）

#### 3、向CodePush增加一个应用管理

```
code-push app add <APP名字>
```

>更名执行 `code-push app rename` 旧名字 新名字
删除执行 `code-push app rm` 旧名字

#### 4、部署你的RN应用

```javascript
code-push deployment add <APP名字> <部署名字>
```

>部署管理:
上面的部署类型Production Staging，还可以自己加例如`dev  alpha beta`等，
重命名部署名字：`code-push deployment rename app名字 旧部署名字 新部署名字`
删除部署名字：`code-push deployment rm app名字 部署名字`
列出部署名字：`code-push deployment ls app名字`


#### 5、集成到APP中

>回到Xcode里info.plist里添加的CodePushDeploymentKey的值，拷贝Staging的key值添加到那里面去，再确保 Bundle Version String short 这一行的值是1.0.0，而不是 1.0

#### 6、打包你的JSBundle文件(具体参数参考入门指南)

```javascript
react-native bundle --entry-file ./index.ios.js --bundle-output ./ios/bundle/index.ios.jsbundle --platform ios --assets-dest ./ios/bundle --dev false
```

#### 7、发布你的JSBundle文件到CodePush

```javascript
code-push release Post './ios/bundle' 1.0.0
```

>这时打开你的1.0.0版本的APP将会收到提示
