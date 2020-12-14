# 在持续集成（CI）环境中使用

持续集成在现代软件研发流程中，扮演了十分重要的角色。通过对每次提交的代码进行自动化的单元测试、代码检查、编译构建、契约测试，甚至自动部署，能够大大降低了开发人员的工作负担，减少了许多不必要的重复劳动，持续提升代码质量和开发效率。

主流的持续集成平台有 Jenkins、Github Actions、GitLab CI、Coding 等

您可以在这些平台中集成云开发 CloudBase Framework 来实现应用的自动化的 CI/CD。

## 1. 准备构建环境

### 方式一：

您需要在构建环境中准备以下环境

- 准备 Node.js 环境
- 全局安装 CloudBase CLI

```bash
npm install -g @cloudbase/cli@latest
```

### 方式二：使用云开发基础构建镜像

如果使用 Docker 镜像来作为持续集成的运行环境，建议使用云开发官方提供的 Docker 镜像 `tencentcloudbase/cloudbase-framework-runner:latest` 作为基础镜像

云开发基础构建镜像内置所有必需的依赖，不用在每次构建时进行安装，提升构建部署速度

- 内置 Node.js/ npm 等环境
- 内置 最新版本 CloudBase CLI 和 CloudBase Framework
- 内置 CloudBase Framework 所有的官方插件

镜像地址

<https://hub.docker.com/r/tencentcloudbase/cloudbase-framework-runner/tags>

## 2. 设置环境变量

### 构建环境变量

建议将构建上下文依赖的信息通过环境变量或者 Secrets 来注入，主流的 CI 平台都支持在触发构建时设置注入环境变量

例如：

- 云开发 环境 id `ENV_ID`
- 腾讯云密钥
- 其他 `.env` 文件中用到的环境变量

### CI 环境变量

在 CI 环境中应设置`CI`环境变量为`true`

通过设置 CI 环境变量可以让 CloudBase Framework 识别出当前为 CI 环境，避免出现用户交互来阻塞构建。

例如，在 Linux 环境可以按照下面的命令来设置环境变量

```bash
export CI=true
```

## 3. 登录 CloudBase CLI

在使用 CloudBase Framework 部署之前，您需要登录 CloudBase CLI

在 CI 持续集成构建中，您可以使用下面的方式通过腾讯云的 API 秘钥直接登录，避免交互式输入

```bash
tcb login --apiKeyId xxx --apiKey xxx
```

也可以使用腾讯云临时秘钥登录

```bash
tcb login --apiKeyId xxx --apiKey xxx --token xxx
```

## 4. 调用 CloudBase Framework 来一键部署应用

在登录完成之后，即可调用以下命令来一键部署，`--verbose` 参数可以打印更多的构建部署信息，方便排查问题

```
tcb framework deploy --verbose
```
