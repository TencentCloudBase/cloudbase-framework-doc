# 如何将容器应用快速打造为云开发应用

## 了解云开发应用

云开发应用可以理解为运行在云开发环境的应用，例如一个包含前后端、数据库等能力等服务，可以通过一键部署，直接部署在云开发环境中，使用云开发底层的各项 Serverless 资源，享受弹性免运维的优势。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmodg5rp6qj310009agn2.jpg)

一个云开发应用可以拆解为三个部分，包括代码、声明式配置和环境变量信息。

![](https://tva1.sinaimg.cn/large/008eGmZEly1gmnjhhepcwj31400gtwi2.jpg)

下面我们会演示如何将一个开源的容器化的项目快速打造为可以一键部署的云开发应用。

**Nextcloud** 是一套个人云存储解决方案，内置了图片相册、日历联系人、文件管理、RSS 阅读等丰富的应用，这个应用本身是开源的，同时提供了部署的 Docker 镜像，需要搭配 MySQL 数据库，下面会演示如何实现这个应用的一键部署。

![img](https://github.com/TencentCloudBase-Marketplace/nextcloud/raw/master/02.jpg)


通过云开发**一键部署**，可以部署在用户的云开发环境中，无需关心服务器管理和运维。

![img](https://github.com/TencentCloudBase-Marketplace/nextcloud/raw/master/01.jpg)



### 项目演示地址：

<https://fx-1259727701.ap-shanghai.service.tcloudbase.com/>

## 体验一键部署

点击下方的部署按钮来一键部署 NextCloud 应用

[![](https://main.qcloudimg.com/raw/67f5a389f1ac6f3b4d04c7256438e44f.svg)](https://console.cloud.tencent.com/tcb/env/index?action=CreateAndDeployCloudBaseProject&tdl_anchor=github&tdl_site=0&appUrl=https://github.com/TencentCloudBase-Marketplace/nextcloud)

点击上方按钮之后会跳转腾讯云开发控制台进入快速安装部署流程。

第一步我们可以选择云开发环境（注意：这个应用我们需要选择一个 HTTP 访问路径的根路径未被占用的环境）。



![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmrp378pscj30lt0boq3o.jpg)

第二步，可以进行网络配置、标签配置，以及关联或者创建云上的资源，比如这个应用依赖了 CFS 来实现容器的文件存储，使用了  CynosDB  for MySQL（Serverless版本）来作为数据库依赖。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmrp3t4pmsj30lv0krtal.jpg)

点击 **完成** ，等待安装完成之后即可在控制台打开应用的访问地址来进行访问。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gms0uftry5j30zw0pqtuk.jpg)

## 如何开发一个云开发应用

那么我们如何打造这样一个可以**一键部署**的云开发应用呢？

整体的步骤分为 3 步，主要分为**开发、配置和部署验证**三个环节。这篇文档会主要介绍配置和部署验证两个环节。

### 开发

开发环节的部分不重点介绍，可以通过获取源码来了解具体的实现。

#### 项目源码

本项目项目的源码可以在 Github 中查看和获取：



<https://github.com/TencentCloudBase-Marketplace/nextcloud>

Nextcloud 官方开源仓库

<https://github.com/nextcloud/server>

#### 使用的云开发和云上其他资源

- 云开发的云托管服务：使用云托管来部署应用的后端服务

- CynosDB：使用 CynosDB 数据库存储数据
- CFS：使用 CFS 持久化存储数据

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
    "name": "nextcloud",
    "plugins": {}
  }
}
```

##### 要点

1. `envId` 指定应用部署在哪个环境下，这里我们用模板变量<code v-pre>env.ENV_ID</code>表示读取 `ENV_ID` 环境变量
2. `framework.name` 是应用的英文名，只支持 A-Z a-z 0-9 - 和 \_，长度 1-32 位
3. `framework.plugins` 是应用用到的插件信息，这里先留空，下面我们根据资源和应用类型来填写
4. 如果需要了解更多项目信息的配置，请参考[应用项目信息说明文档](https://docs.cloudbase.net/framework/config.html#xiang-mu-xin-xi)

#### 使用云托管插件

这一步我们需要使用云托管插件，来自动化地部署容器服务到云开发的云托管上。

在 `framework.plugins` 下增加一个字段 `server`，字段的值是一个 JSON 对象。

```json
{
  "server": {
    "use": "@cloudbase/framework-plugin-container",
    "inputs": {
      "cpu": 0.5,
      "mem": 1,
      "serviceName": "nextcloud",
      "servicePath": "/",
      "uploadType": "image",
      "containerPort": 80,
      "imageInfo": {
        "imageUrl": "nextcloud:20"
      },
      "envVariables": {
        "MYSQL_HOST": "{{env.DB_IP}}:{{env.DB_PORT}}",
        "MYSQL_DATABASE": "nextcloud"
      },
      "volumeMounts": {
        "/var/www/html": "nextcloud-cfs"
      }
    }
  }
}
```

##### 要点

1. 字段 `server`的名字可以自定义，只起到别名的作用，下同
2. `"use": "@cloudbase/framework-plugin-container”` 表示这里使用了 `@cloudbase/framework-plugin-contaienr` 这个云托管插件，用来部署镜像和服务到云托管上。
3. `inputs` 用来设置插件接收的参数，上面的配置部分做了如下的事情：
   1. 指定了云托管服务的 CPU 和内存的规格，
   2. 指定了服务名 "serviceName": "nextcloud"
   3. 指定了服务部署的HTTP访问路径 "servicePath": “/“，相当于根目录
   4. 指定了使用镜像来部署，端口为80端口，镜像的地址和版本为 `nextcloud:20`
   5. 环境变量`envVariables` 部分，我们指定了要为容器运行时注入的环境变量，`MYSQL_HOST` 是 NextCloud 这个程序支持的一个环境变量，可以用来配置应用的数据库连接信息，我们使用<code v-pre>{{env.DB_IP}}:{{env.DB_PORT}}</code>作为模板变量来生成连接地址，具体的IP和端口会在构建时根据关联的数据库真实的IP和端口来填充。
   6. 挂载目录设置 `volumeMounts` 部分，我们声明了将在容器内的 "/var/www/html”  路径上挂载一个名称为 "nextcloud-cfs” 的 CFS 持久化存储的实例。
4. 云托管插件还可以配置代码来源、自动扩缩容配置等，详细配置说明可以参考 [云托管容器配置文档](https://docs.cloudbase.net/framework/plugins/framework-plugin-container.html)

#### 配置应用参数和依赖

在部署应用时，还可能需要用户来输入一些自定义的参数，或者配置像上文提到的云上外部资源。

这些都可以在`framework.requirement` 中进行配置

| 字段          | 描述                                                         | 类型                |
| ------------- | ------------------------------------------------------------ | ------------------- |
| `addons`      | 应用部署过程中用到的外部云上资源，包括 cfs、cynosdb、redis 等 | `AddonsConfig`      |
| `environment` | 应用在构建时和运行时的环境变量配置声明，默认注入计算环境中(云函数、云应用)，也会在云端构建时作为构建部署的环境变量，可以在 `cloudbaserc.json` 中通过 <code v-pre>env.ENV_NAME</code> 引用 | `EnvironmentConfig` |

接下来我们在 `framework.requirement` 中添加如下 JSON 配置。

```json
{
  "requirement": {
    "addons": [
      {
        "type": "CFS",
        "name": "nextcloud-cfs"
      },
      {
        "type": "CynosDB",
        "name": "nextcloud",
        "envMap": {
          "IP": "DB_IP",
          "PORT": "DB_PORT",
          "USERNAME": "MYSQL_USER",
          "PASSWORD": "MYSQL_PASSWORD"
        }
      }
    ]
  }
}
```

##### 要点

1. 在 `addons`里面我们可以声明依赖的云上资源。
2. 首先我们指定了依赖 `CFS` ，名称为 `nextcloud-cfs`，这个名称和上面云托管插件里面挂载的需要对应起来。
3. 接着我们指定了依赖 `CynosDB`，这里只需要关注 `envMap` 的配置，在数据库实例创建成功之后，会把数据库的 IP、PORT、USERNAME和 PASSWORD 写入应用的全局环境变量中，这里我们可以把这些连接信息映射为我们需要的环境变量名，比如密码信息我们就配置了可以映射为 `MYSQL_PASSWORD`， 在容器中可以直接获取到这个环境变量。
4. 其他高级的使用技巧，可以参考 [应用依赖配置说明](https://docs.cloudbase.net/framework/config.html#ying-yong-yi-lai)

### 生成部署按钮

接下里我们就可以上传代码到 Git，来生成一个一键部署按钮了。首先打开一键部署按钮生成地址：

<https://docs.cloudbase.net/framework/deploy-button.html>

在页面当中输入项目的 Git 地址，配置文件所在目录以及分支信息，就可以自动生成下面的部署按钮代码片段。

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmo774ez7xj30mx0kygo4.jpg)

这里会生成部署按钮的几种格式的代码片段，可以在不同的场景下嵌入部署按钮来让用户部署你的应用

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gmo77q1zhsj30mf0nggoo.jpg)

- Markdown 代码适合用在 README、Mardkown 编写的博客文章等场景
- HTML 代码适合用在公众号、HTML 编写的网站/博客文章等场景
- URL 链接可以在任何支持超链接的页面使用，您可以复制下方链接，粘贴到对应的页面中

生成完部署按钮之后，可以按照云开发应用模板来编写 README 文档，提交应用到云开发应用中心。

**应用模板地址**

<https://github.com/TencentCloudBase-Marketplace/app-template>

## 总结

在这篇文章中，我们了解了什么是云开发应用，以开源项目 Nextcloud 为例，介绍了如何将开源的容器化的项目，快速打造为可以一键部署的云开发应用。

通过实战，我们也了解了云开发以及 CloudBase Framework 的使用。只需要完成开发、配置以及部署验证，就可以快速将应用变为可以快速分发的程序，用户无需手动搭建环境和配置，即可自动化部署应用。

## 参考文档

- 云开发官网：<https://www.cloudbase.net/>
- 云开发应用中心：<https://www.cloudbase.net/marketplace.html>
- CloudBase Framework：<https://github.com/Tencent/cloudbase-framework>
- CloudBase Framework 文档：<https://docs.cloudbase.net/framework/>
- 一键部署按钮生成器：<https://docs.cloudbase.net/framework/deploy-button.html>
- 云开发应用模板：<https://github.com/TencentCloudBase-Marketplace/app-template>
- 云开发应用源码列表：<https://github.com/TencentCloudBase-Marketplace>
