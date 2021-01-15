# 如何快速打造云开发应用

## 了解云开发应用

云开发应用可以理解为运行在云开发环境的应用，例如一个包含前后端、数据库等能力等服务，可以通过一键部署，直接部署在云开发环境中，使用云开发底层的各项 Serverless 资源，享受弹性免运维的优势。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmodg5rp6qj310009agn2.jpg)

一个云开发应用可以拆解为三个部分，包括代码、声明式配置和环境变量信息。

![](https://tva1.sinaimg.cn/large/008eGmZEly1gmnjhhepcwj31400gtwi2.jpg)

下面我们会演示如何开发一个实时展板的云开发应用，后端能力基于云开发，同时使用了小程序的客服消息能力发送消息，可以实时在 WEB 展板页面中展示消息。

这个应用可以实现**一键部署**，部署在用户的云开发环境中，无需关心服务器管理和运维，可以开箱即用地用于现场互动等多种玩法场景。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gml4oklkvoj31gw0r9jyu.jpg)

### 项目演示地址：

[f.cloudbase.vip](https://f.cloudbase.vip/)

## 体验一键部署

点击下方的部署按钮来一键部署网页、小程序以及后端服务，需要填写小程序 AppId 和 Base64 格式的小程序部署私钥（可以使用 [小程序部署密钥转换小工具](https://framework-1258016615.tcloudbaseapp.com/mp-key-tool/) 来转换为 Base64)

[![](https://main.qcloudimg.com/raw/67f5a389f1ac6f3b4d04c7256438e44f.svg)](https://console.cloud.tencent.com/tcb/env/index?action=CreateAndDeployCloudBaseProject&appUrl=https%3A%2F%2Fgithub.com%2FTCloudBase%2FWXAPP-WEB-ShowMess&branch=master)

注意：一键部署之后需要下载代码并在小程序云开发控制台中【设置-全局设置-添加消息推送】选择 text 类型对接到 contact 云函数

## 如何开发一个云开发应用

那么我们如何打造这样一个可以**一键部署**的云开发应用呢？

整体的步骤分为 3 步，主要分为开发、配置和部署验证三个环节。这篇文档会主要介绍配置和部署验证两个环节。

![](https://tva1.sinaimg.cn/large/008eGmZEly1gmnk7i8b8fj30mo0loabf.jpg)

### 开发

开发环节的部分不重点介绍，可以通过获取源码来了解具体的实现，下面会介绍一下整体的代码结构。

#### 项目源码

本项目项目的源码可以在 Github 中查看和获取：

<https://github.com/TCloudBase/WXAPP-WEB-ShowMess>

#### 项目结构

项目整体结构如下：

```
WXAPP-WEB-ShowMess
├── README.md // 项目介绍文档
├── cloudbaserc.json // 云开发的配置文件
├── cloudfunctions // 云函数目录
├── miniprogram //小程序目录
├── project.config.json // 小程序配置文件
└── webview // Web 网站目录
```

#### 使用的云开发资源

- 静态网站
- 云函数
- 云数据库（以及实时数据推送能力）
- 云开发匿名登录

### 配置

有了项目的代码之后，如何把这个应用打造成为可以**一键部署**的云开发应用呢？

下面会分步骤介绍如何通过配置来打造云开发应用。

#### 配置应用基础信息

首先创建一个 `cloudbaserc.json` 配置文件，文件的内容如下。

```json
{
  "envId": "{{env.ENV_ID}}",
  "version": "2.0",
  "$schema": "https://framework-1258016615.tcloudbaseapp.com/schema/latest.json",
  "framework": {
    "name": "techo-show",
    "plugins": {}
  }
}
```

##### 要点

1. `envId` 指定应用部署在哪个环境下，这里我们用模板变量<code v-pre>env.ENV_ID</code>表示读取 `ENV_ID` 环境变量
2. `framework.name` 是应用的英文名，只支持 A-Z a-z 0-9 - 和 \_，长度 1-32 位，这个例子我们使用 `techo-show` 作为应用名
3. `framework.plugins` 是应用用到的插件信息，这里先留空，下面我们根据资源和应用类型来填写
4. 如果需要了解更多项目信息的配置，请参考[应用项目信息说明文档](https://docs.cloudbase.net/framework/config.html#xiang-mu-xin-xi)

#### 使用云函数插件

这一步我们需要使用云函数插件，来自动化地批量部署和配置云函数。

在 `framework.plugins` 下增加一个字段 `fn`，字段的值是一个 JSON 对象。

```json
{
  "fn": {
    "use": "@cloudbase/framework-plugin-function",
    "inputs": {
      "functionRootPath": "./cloudfunctions",
      "functions": [
        {
          "name": "contact",
          "installDependency": true
        },
        {
          "name": "delete",
          "installDependency": true
        },
        {
          "name": "inituser",
          "installDependency": true
        }
      ]
    }
  }
}
```

##### 要点

1. 字段 `fn`的名字可以自定义，只起到别名的作用，下同
2. `"use": "@cloudbase/framework-plugin-function”` 表示这里使用了 `@cloudbase/framework-plugin-function` 这个云函数插件，用来部署和管理云函数
3. `inputs` 用来设置插件接收的参数，这里我们指定了函数的根目录是 `./cloudfunctions`，同时有 3 个云函数，分别为 `contact`、`delete` 和 `inituser`，都需要云端安装依赖
4. 云函数插件还支持配置 http 访问地址、rutime 语言、定时触发器、超时时间、安全规则等，详细配置说明可以参考[云函数插件配置说明](https://docs.cloudbase.net/framework/plugins/framework-plugin-function.html#pei-zhi-can-shu-shuo-ming)

#### 使用云数据库插件

这一步我们需要使用云数据库插件，来自动化创建云数据库集合和设置安全规则。

在 `framework.plugins` 下增加一个字段 `db`，字段的值是一个 JSON 对象。

```json
{
  "use": "@cloudbase/framework-plugin-database",
  "inputs": {
    "collections": [
      {
        "collectionName": "mess",
        "aclTag": "READONLY"
      },
      {
        "collectionName": "user",
        "aclTag": "ADMINONLY"
      }
    ]
  }
}
```

##### 要点

1. 这里使用了 `@cloudbase/framework-plugin-database` 这个云数据库插件
2. 在 `inputs` 里面配置了两个数据库集合
3. `mess` 集合的权限是所有用户可读，仅创建者和管理员可写
4. `user` 集合的权限是所有用户可读，仅管理员可写
5. 云数据库插件还支持配置索引、自定义安全规则等，详细配置说明可以参考[云数据库插件配置说明](https://docs.cloudbase.net/framework/plugins/framework-plugin-database.html#pei-zhi-can-shu-shuo-ming)

#### 使用静态网站插件

这一步我们需要使用静态网站插件，自动化发布应用中的静态网页。

在 `framework.plugins` 下增加一个字段 `web`，字段的值是一个 JSON 对象。

```json
{
  "use": "@cloudbase/framework-plugin-website",
  "inputs": {
    "outputPath": "webview",
    "cloudPath": "/mess",
    "envVariables": {
      "ENV_ID": "{{env.ENV_ID}}"
    }
  }
}
```

##### 要点

1. 这里我们使用了 `@cloudbase/framework-plugin-website` 这个静态网站插件
2. 在 `inputs` 里面我们指定了静态网站本地的源码目录（ `outputPath` ）是 `webview` 目录，需要部署在云端的静态网站的 `/mess` 路径下
3. 同时我们往静态网站的配置信息中注入了应用的环境 id `ENV_ID`，这样在页面中可以通过请求静态网站根目录下的 `/cloudbaseenv.json` 来拿到当前部署环境的环境 id。这样一套代码不用修改就可以部署在其他环境。如果使用 webpack 构建，也可以把环境变量打包到页面代码中。
4. 静态网站插件还支持配置 npm 依赖安装、自定义构建命令、需要忽略的文件等，详细配置说明可以参考 [静态网站插件配置说明](https://docs.cloudbase.net/framework/plugins/framework-plugin-website.html#pei-zhi-can-shu-shuo-ming)

#### 使用登录鉴权插件

这一步我们需要使用登录鉴权插件，来自动化为应用设置登录方式。

在 `framework.plugins` 下增加一个字段 `auth`，字段的值是一个 JSON 对象。

```json
{
  "use": "@cloudbase/framework-plugin-auth",
  "inputs": {
    "configs": [
      {
        "platform": "ANONYMOUS",
        "status": "ENABLE"
      }
    ]
  }
}
```

##### 要点

1. 这里我们使用了 `@cloudbase/framework-plugin-auth` 这个登录鉴权插件
2. 在 `inputs` 中我们配置了启用匿名登录，会自动帮用户开启这个登录方式。
3. 登录鉴权插件还支持配置未登录等其他登录方式，详细配置说明可以参考 [登录鉴权插件配置说明](https://docs.cloudbase.net/framework/plugins/framework-plugin-auth.html#pei-zhi-shi-li)

#### 使用小程序插件

这一步我们需要使用小程序插件，来自动化构建和发布小程序。

在 `framework.plugins` 下增加一个字段 `mp`，字段的值是一个 JSON 对象。

```json
{
    "use": "@cloudbase/framework-plugin-mp",
    "inputs": {
      "appid": "{{env.WX_APPID}}",
      "privateKey": "{{env.WX_CI_KEY}}",
      "localPath": "./",
      "ignores": ["node_modules/**/*"],
      "deployMode": "preview",
      "previewOptions": {
        "desc": "一键预览",
        "setting": {
          "es6": false
        },
        "qrcodeOutputPath": "./qrcode.jpg",
        "pagePath": "pages/index/index",
        "searchQuery": "",
        "scene": 1011
      }
    }
  }
}
```

##### 要点

1. 这里我们使用了 `@cloudbase/framework-plugin-mp` 这个小程序插件
2. 在 `inputs` 中我们配置了小程序的 appid、部署需要使用的私钥，这里填写的是模板变量，这是需要用户来输入的信息，接下来我们需要在应用参数中获取。
3. `inputs` 配置里我们还指定了部署模式为预览模式，以及预览时的选项。
4. 小程序插件还支持预览和上传时的其他配置，详细配置说明可以参考 [微信小程序插件配置说明](https://docs.cloudbase.net/framework/plugins/framework-plugin-mp.html#pei-zhi)

#### 配置应用参数和依赖

在部署应用时，还可能需要用户来输入一些业务参数，比如用户自己的小程序 APPID 和 小程序部署密钥，我们接下来需要声明应用需要的参数。

首先我们在 `framework.requirement` 中添加如下 JSON 配置。

```json
{
  "environment": {
    "WX_APPID": {
      "description": "请填写微信小程序APPID",
      "required": true,
      "default": "",
      "validation": {
        "rule": {
          "type": "RegExp",
          "pattern": "^wx.*",
          "flag": "g"
        },
        "errorMessage": "必须是小程序的APPID"
      }
    },
    "WX_CI_KEY": {
      "description": "请填写微信小程序上传密钥BASE64",
      "required": true,
      "default": "",
      "validation": {
        "rule": {
          "type": "RegExp",
          "pattern": "^(?:[A-Za-z0-9+/]{4})*(?:[A-Za-z0-9+/]{2}==|[A-Za-z0-9+/]{3}=)?$",
          "flag": "g"
        },
        "errorMessage": "必须是BASE64格式密钥"
      }
    }
  }
}
```

##### 要点

1. `WX_APPID` 和 `WX_CI_KEY` 是我们需要接收的环境变量的 Key，我们可以在模板里面通过 <code v-pre>env.WX_APPID</code> 来使用，也可以在云函数和云托管里面通过环境变量来拿到这些值

2. 每个环境变量我们都用 JSON 描述来说明了这个字段的描述信息、是否必填、默认值以及校验规则用于检查用户输入

3. 除了声明环境变量的依赖之外，还可以声明外部资源的依赖，如应用部署过程中用到的外部云上资源，包括持久文件存储、MySQL 数据库等，具体请参考 [应用依赖](https://docs.cloudbase.net/framework/config.html#ying-yong-yi-lai) 文档。

   ![](https://tva1.sinaimg.cn/large/008eGmZEly1gmniat5k7lj30fr0e20tl.jpg)

### 本地部署验证

在配置完插件信息之后，我们已经可以自动化构建并部署应用依赖所有的云开发资源。

接下来我们需要在本地创建一个 .env 文件

```bash
ENV_ID=环境id
WX_APPID=小程序appid
WX_CI_KEY=小程序部署私钥（需要转换为base64）
```

然后在命令行运行

```bash
tcb framework deploy
```

即可部署应用中包含的静态网站、小程序、数据库、云函数、登录鉴权等等所有的资源。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gml78z3ut0j30s50dk429.jpg)

可以看到命令行在部署成功后显示了网页入口和小程序的预览二维码。

![image-20210112220441078](https://tva1.sinaimg.cn/large/008eGmZEgy1gml9h5l3adj30yl08r78j.jpg)

### 生成部署按钮

本地部署验证之后，我们就可以上传代码到 Git，然后来生成一个一键部署按钮了。

打开一键部署按钮生成地址：

<https://docs.cloudbase.net/framework/deploy-button.html>

在页面当中输入项目的 Git 地址，配置文件所在目录以及分支信息就可以生成下面的部署按钮代码片段了。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmo774ez7xj30mx0kygo4.jpg)

会生成部署按钮的几种代码片段，可以在不同的场景下嵌入部署按钮来让用户部署你的应用

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmo77q1zhsj30mf0nggoo.jpg)

- Markdown 代码适合用在 README、Mardkown 编写的博客文章等场景

- HTML 代码适合用在公众号、HTML 编写的网站/博客文章等场景

- URL 链接可以在任何支持超链接的页面使用，您可以复制下方链接，粘贴到对应的页面中

## 总结

在这篇文章中，我们了解了什么是云开发应用，基于一个实时展板的全栈程序（包含前端页面，后端服务，数据库以及小程序）为例，介绍了如何快速打造一个可以一键部署的云开发应用。

通过实战，我们也了解了云开发以及 CloudBase Framework 的使用。只需要完成开发、配置以及部署验证，就可以快速将应用变为可以快速分发的程序，用户无需手动搭建环境和配置，即可自动化部署应用。

## 参考文档

- 云开发官网：<https://www.cloudbase.net/>
- 云开发应用中心：<https://www.cloudbase.net/marketplace.html>
- CloudBase Framework：<https://github.com/Tencent/cloudbase-framework>
- CloudBase Framework 文档：<https://docs.cloudbase.net/framework/>
- 一键部署按钮生成器：<https://docs.cloudbase.net/framework/deploy-button.html>
- 云开发应用模板：<https://github.com/TencentCloudBase-Marketplace/app-template>
- 微信小程序密钥工具：<https://framework-1258016615.tcloudbaseapp.com/mp-key-tool/>
