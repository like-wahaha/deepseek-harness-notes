- [第 2 章：Harness 不是一个大 Agent 类](#第-2-章harness-不是一个大-agent-类)
  - [2.1 Profile、Bundle、Patch、Plugin：四个概念到底有什么区别](#21-profilebundlepatchplugin四个概念到底有什么区别)
    - [2.1.1 先理解 Plugin：真正提供运行时能力的单元](#211-先理解-plugin真正提供运行时能力的单元)
    - [2.1.2 Bundle 到底是什么：从 dsh-base 理解](#212-bundle-到底是什么从-dsh-base-理解)
      - [继续看 dsh-base 的 package.json](#继续看-dsh-base-的-packagejson)
      - [查看 dsh-base 的 cordis.patch.yml](#查看-dsh-base-的-cordispatchyml)
    - [2.1.3 再看 dsh-web-app：为什么一个项目需要多个 Bundle？](#213-再看-dsh-web-app为什么一个项目需要多个-bundle)
      - [一个 Bundle 可以包含多个 Patch](#一个-bundle-可以包含多个-patch)
    - [2.1.4 Patch 是什么：它负责修改插件配置树](#214-patch-是什么它负责修改插件配置树)
      - [为什么需要 Patch？](#为什么需要-patch)
    - [2.1.5 Profile 是什么：决定最终运行形态](#215-profile-是什么决定最终运行形态)
    - [2.1.6 把四个概念放在一起：完整装配关系](#216-把四个概念放在一起完整装配关系)
    - [2.1.7 源码陪读：沿着启动器追踪 Bundle 和 Patch](#217-源码陪读沿着启动器追踪-bundle-和-patch)
      - [Profile、Bundle、Patch 的实际加载流程](#profilebundlepatch-的实际加载流程)
    - [2.1.8 实验：### 通过 Patch 扩展功能，而不是直接修改核心](#218-实验-通过-patch-扩展功能而不是直接修改核心)
      - [第一阶段：找到一个已经存在的插件](#第一阶段找到一个已经存在的插件)
      - [第二阶段：从插件配置追踪到插件源码](#第二阶段从插件配置追踪到插件源码)
      - [第三阶段：从零编写一个最简单的插件](#第三阶段从零编写一个最简单的插件)
        - [3.1 创建插件目录](#31-创建插件目录)
        - [3.2 编写 package.json](#32-编写-packagejson)
        - [3.3 编写插件入口文件](#33-编写插件入口文件)
        - [3.4 将插件加入 Web Bundle](#34-将插件加入-web-bundle)
        - [3.5 安装依赖并启动](#35-安装依赖并启动)
        - [3.6 回顾：一个插件是如何进入 Harness 的？](#36-回顾一个插件是如何进入-harness-的)
    - [2.1.9 Agent 生命周期事件：认识 12 个核心事件](#219-agent-生命周期事件认识-12-个核心事件)
      - [1. 在哪里查找 Agent 事件？](#1-在哪里查找-agent-事件)
      - [2. Agent 的 12 个核心事件](#2-agent-的-12-个核心事件)
      - [3. 三种事件处理模式](#3-三种事件处理模式)
      - [4. 12 个事件应该如何学习？](#4-12-个事件应该如何学习)
      - [5. 阅读其他插件时，如何追踪 ctx.on()？](#5-阅读其他插件时如何追踪-ctxon)
      - [6. 本节小结](#6-本节小结)
    - [2.1.10 本节必须掌握的结论](#2110-本节必须掌握的结论)
  - [2.2 从 CLI 追到 Loader：一次启动究竟发生了什么](#22-从-cli-追到-loader一次启动究竟发生了什么)
    - [2.2.1 先明确本节要追踪的启动链路](#221-先明确本节要追踪的启动链路)
    - [2.2.2 第一步：从 package.json 找到 CLI 入口](#222-第一步从-packagejson-找到-cli-入口)
    - [2.2.3 第二步：阅读 bin.ts，理解 CLI 如何分派命令](#223-第二步阅读-bints理解-cli-如何分派命令)
    - [2.2.4 第三步：理解 parseDshArgs() 如何选择 Profile](#224-第三步理解-parsedshargs-如何选择-profile)
    - [2.2.5 第四步：进入 runProfile()，找到真正的启动入口](#225-第四步进入-runprofile找到真正的启动入口)
    - [2.2.6 第五步：追踪 composeProfile()，理解 Bundle 和 Patch 如何组合](#226-第五步追踪-composeprofile理解-bundle-和-patch-如何组合)
    - [2.2.7 第六步：理解为什么还需要 Runtime Resolution](#227-第六步理解为什么还需要-runtime-resolution)
    - [2.2.8 第七步：进入 boot()，找到 Cordis Loader](#228-第七步进入-boot找到-cordis-loader)
    - [2.2.9 第八步：插件加载之后，启动流程还会做什么](#229-第八步插件加载之后启动流程还会做什么)
    - [2.2.10 实践一：找到服务启动后，在哪里配置大模型 API Key](#2210-实践一找到服务启动后在哪里配置大模型-api-key)
      - [第一步：找到 Models 页面](#第一步找到-models-页面)
      - [第二步：找到 credentials.set() 的实现](#第二步找到-credentialsset-的实现)
      - [第三步：找到真正保存密钥的插件](#第三步找到真正保存密钥的插件)
      - [第四步：找到模型插件在哪里声明 API Key 引用](#第四步找到模型插件在哪里声明-api-key-引用)
      - [第五步：追踪模型请求如何使用 API Key](#第五步追踪模型请求如何使用-api-key)
    - [2.2.11 实践二：如果要给项目添加一个默认 API Key，应该怎么办？](#2211-实践二如果要给项目添加一个默认-api-key应该怎么办)
      - [第一步：理解当前凭据解析优先级](#第一步理解当前凭据解析优先级)
      - [第二步：尝试通过项目 .env 提供默认 API Key](#第二步尝试通过项目-env-提供默认-api-key)
      - [第三步：启动服务，验证默认 API Key 是否生效](#第三步启动服务验证默认-api-key-是否生效)
      - [第四步：思考为什么不需要修改模型插件](#第四步思考为什么不需要修改模型插件)
    - [2.2.12 实践三：如果希望修改默认凭据引用，应该怎么办？](#2212-实践三如果希望修改默认凭据引用应该怎么办)
    - [2.2.13 本节小结](#2213-本节小结)
  - [2.3 从“目录结构”转向“运行时装配”](#23-从目录结构转向运行时装配)
    - [2.3.1 从静态结构到运行时](#231-从静态结构到运行时)
    - [2.3.2 启动过程分为两个阶段](#232-启动过程分为两个阶段)
    - [2.3.3 把两个阶段连起来](#233-把两个阶段连起来)
    - [2.3.4 后续源码阅读应该追踪什么？](#234-后续源码阅读应该追踪什么)
    - [2.3.5 本节小结](#235-本节小结)


# 第 2 章：Harness 不是一个大 Agent 类

Harness 的关键设计是可组合运行时：能力通过插件挂载到共享
Context。模型适配器、工具注册、Session、Agent Loop 等都属于可替换能力。

因此分析功能时，不只问"哪个类实现了它"，还要问：

-   哪个插件提供能力？
-   服务注册在哪个 Context？
-   哪个消费者注入它？
-   插件何时加载？
-   卸载时撤销哪些副作用？
-   能否通过另一个 Provider 替换实现？

先读：

1.  `AGENTS.md`
2.  `docs/architecture.md`
3.  `docs/subsystems/README.md`
4.  `docs/subsystems/core.md`

架构文档说明，运行中的 `dsh` 是由 Profile 和有序 Bundle
层组合成的插件树。Patch 可以替换已有 id 的配置行，也可以插入新行。

## 2.1 Profile、Bundle、Patch、Plugin：四个概念到底有什么区别

这一节需要先解决一个问题：

**DeepSeek Harness 启动一个运行环境时，究竟是如何决定加载哪些插件的？**

例如，我们启动 Web 版本的 Harness：

```bash
dsh --profile web
```

Harness 最终需要知道：

- 当前运行的是哪一种 Profile？
- 这个 Profile 需要组合哪些 Bundle？
- 每个 Bundle 声明了哪些 Patch？
- Patch 要增加、修改或调整哪些 Plugin 配置？
- 最终哪些 Plugin 会被加载？
- Plugin 加载后又会提供哪些服务和行为？

这几个问题分别涉及 Profile、Bundle、Patch 和 Plugin。

它们不是四个可以互相替换的概念，而是处于不同层次的四个对象。

先记住这四句话：

| 概念 | 可以把它理解成 | 主要回答的问题 |
|---|---|---|
| Profile | 一种运行形态的装配方案 | 我要运行哪一种 Harness？ |
| Bundle | 一组可复用的功能装配包 | 这套运行形态需要哪些功能组合？ |
| Patch | 对插件配置树进行修改的配置文件 | 具体增加、修改或调整哪些插件配置？ |
| Plugin | 运行时实际提供能力的插件 | 加载之后究竟能做什么？ |

接下来不只讲定义，而是直接阅读仓库中的实际代码。

本节源码基线：

```text
仓库：https://github.com/DeepSeek-ai/deepseek-harness
Commit：2e39e70496aa02986ca6a820c5d497e816b69d7e
```

### 2.1.1 先理解 Plugin：真正提供运行时能力的单元

先从最容易理解的 Plugin 开始。

Plugin 是运行时实际提供能力的单元。

例如：

- LLM Plugin：提供模型访问能力。
- Session Plugin：提供会话相关能力。
- Agent Plugin：提供 Agent 运行相关能力。
- Bash Tool Plugin：提供 Bash 工具。
- Filesystem Tool Plugin：提供文件系统工具。
- Authorization Plugin：提供授权能力。

一个 Plugin 通常不只是一个 TypeScript 文件。

它可能包含：

```text
一个 Plugin 包
    │
    ├── package.json
    ├── src/
    │   ├── index.ts
    │   ├── service.ts
    │   ├── types.ts
    │   └── ...
    ├── tests/
    └── 编译后的运行时代码
```

Plugin 的核心工作是：

1. 注册服务；
2. 监听或触发事件；
3. 提供 API；
4. 执行具体业务逻辑；
5. 与其他 Plugin 协作。

例如，一个简化的 Plugin 可以写成：

```ts
import { Context } from 'cordis'

export default function examplePlugin(ctx: Context) {
  ctx.on('ready', () => {
    console.log('example plugin ready')
  })

  ctx.set('example', {
    hello() {
      return 'hello harness'
    },
  })
}
```

这是一个教学示例，不代表仓库中某个具体插件的原始实现。

它表达的是：

- `ctx.on()`：监听运行时事件；
- `ctx.set()`：向运行环境注册服务；
- `example.hello()`：其他插件可以通过服务访问这个能力。

因此，Plugin 是真正提供功能的地方。

但是，仅有 Plugin 源码还不够。

Harness 还需要知道：

- 应该加载哪个 Plugin？
- 应该给它什么配置？
- 应该在哪种运行形态下加载它？
- 它需要哪些依赖？

这些问题就需要 Bundle、Patch 和 Profile 来解决。

### 2.1.2 Bundle 到底是什么：从 dsh-base 理解

Bundle 是这一节最容易混淆的概念。

初学者经常把 Bundle 简单理解为：

> Bundle 就是多个 Plugin 放在一起。

这个理解不完全错，但不够准确。

更准确地说：

**Bundle 是一份可复用的功能装配包，它通过 Patch 文件组织多个 Plugin 及其配置，并通过 package.json 声明相关依赖和 Patch 文件。**

它不是一个具体的运行时功能。

它负责组织一批功能，让 Harness 能够按照这批配置构建运行环境。

我们直接看仓库中的真实例子。

打开：

```text
packages/bundle/base/
```

重点关注：

```text
packages/bundle/base/package.json
packages/bundle/base/cordis.patch.yml
packages/bundle/base/src/
```

这里的 `@deepseek-ai/dsh-base` 就是一个真实的 Bundle。

先看它的 README：

```text
packages/bundle/base/README.zh.md
```

它的定位是提供共享核心能力。

基于它构建的 Web、Headless、SDK、ACP 等运行形态，可以复用其中的基础能力。

为什么需要这个 Bundle？

因为这些运行形态虽然不同，但都需要一批相同的基础功能。

例如：

- 模型访问；
- Agent；
- Session；
- 文件工具；
- Shell 工具；
- 权限控制；
- 沙箱；
- 子 Agent。

如果每一种运行形态都自己声明这些插件，就会出现大量重复配置。

例如：

```text
Web Profile
    ├── LLM
    ├── Session
    ├── Agent
    ├── File Tool
    ├── Bash Tool
    └── Authorization

Headless Profile
    ├── LLM
    ├── Session
    ├── Agent
    ├── File Tool
    ├── Bash Tool
    └── Authorization

SDK Profile
    ├── LLM
    ├── Session
    ├── Agent
    ├── File Tool
    ├── Bash Tool
    └── Authorization
```

这些配置明显存在重复。

Harness 把公共部分抽取出来，形成 `dsh-base`：

```text
                 dsh-base
                     │
          ┌──────────┼──────────┐
          │          │          │
         Web      Headless      SDK
          │          │          │
       Web 层      CLI 层      SDK 层
```

这样做有几个好处：

1. 公共插件只需要维护一份装配配置。
2. 不同 Profile 可以复用相同的基础能力。
3. 公共插件的配置调整不需要在多个 Profile 中重复修改。
4. 不同运行形态可以继续增加自己的专属功能。

这就是 Bundle 存在的主要原因。

#### 继续看 dsh-base 的 package.json

打开：

```text
packages/bundle/base/package.json
```

重点找：

```json
{
  "name": "@deepseek-ai/dsh-base",
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  },
  "dependencies": {
    "..."
  }
}
```

上面是字段结构示意，实际内容以仓库文件为准。

这里需要理解两个关键字段。

**第一：dependencies**

例如：

```json
{
  "dependencies": {
    "@deepseek-ai/dsh-agent": "workspace:*",
    "@deepseek-ai/dsh-llm": "workspace:*",
    "@deepseek-ai/dsh-session": "workspace:*"
  }
}
```

它说明 Bundle 依赖哪些软件包。

但一定要注意：

**dependencies 不等于插件加载清单。**

一个包出现在 dependencies 中，只能说明当前软件包声明了对它的依赖，不能直接推断它一定会以插件行的形式进入最终运行配置。

要判断它是否被装配，还需要继续检查 Patch。

**第二：dsh.bundle.patch**

```json
{
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  }
}
```

这个字段告诉 Harness：

当前 Bundle 的 Patch 文件在哪里。

Harness 可以通过这个声明找到对应的插件装配配置。

所以 Bundle 不只是一个目录，也不只是一个 package.json。

它是通过软件包元数据、依赖关系和 Patch 文件共同组织起来的功能装配单元。

#### 查看 dsh-base 的 cordis.patch.yml

打开：

```text
packages/bundle/base/cordis.patch.yml
```

重点关注插件配置行。

你会看到类似这样的结构：

```yaml
- insert:
    - id: session
      name: '@deepseek-ai/dsh-session'

    - id: agent
      name: '@deepseek-ai/dsh-agent'

    - id: tool-fs
      name: '@deepseek-ai/dsh-tool-fs'
```

这是简化后的教学示意，实际 Patch 内容以仓库文件为准。

这段配置可以这样理解：

```text
Bundle
   │
   ▼
cordis.patch.yml
   │
   ├── 声明 session 插件行
   ├── 声明 agent 插件行
   └── 声明 tool-fs 插件行
```

每个插件配置行至少需要区分两个字段：

```yaml
- id: tool-fs
  name: '@deepseek-ai/dsh-tool-fs'
```

- `id`：配置树中这条插件行的标识。
- `name`：实际插件包的名称。

不要把它们混为一谈。

`id` 用于配置层定位插件行，`name` 用于指向实际的软件包。

至此，你应该理解：

**Bundle 通过 Patch 把一组 Plugin 及其配置组织起来，使这组功能能够被不同的运行形态复用。**

### 2.1.3 再看 dsh-web-app：为什么一个项目需要多个 Bundle？

只看 `dsh-base` 还不够。

我们再看另一个真实 Bundle：

```text
packages/bundle/web-app/
```

对应的软件包是：

```text
@deepseek-ai/dsh-web-app
```

打开：

```text
packages/bundle/web-app/README.zh.md
```

再打开：

```text
packages/bundle/web-app/package.json
```

你会发现，它与 `dsh-base` 的职责不同。

`dsh-base` 负责共享核心能力。

`dsh-web-app` 负责 Web 应用运行形态所需的功能。

例如 Web 运行环境可能需要：

- Web Server；
- Web Runtime；
- 浏览器端插件模块；
- 浏览器与 Host 之间的连接；
- Web 应用相关 API；
- 浏览器界面所需的运行时能力。

这些功能不是所有运行形态都需要的。

例如 Headless 模式不需要启动 Web Server，也不需要加载浏览器界面相关组件。

因此，把 Web 专属能力全部塞进 Base Bundle 并不合理。

可以将它们分开：

```text
dsh-base
    │
    ├── LLM
    ├── Agent
    ├── Session
    ├── File Tool
    ├── Bash Tool
    ├── Authorization
    └── Sandbox

dsh-web-app
    │
    ├── Web Server
    ├── Web Runtime
    ├── Browser Modules
    └── Client Connection
```

这两个 Bundle 负责不同层次的功能。

一个负责共享核心，一个负责 Web 应用层。

当 Web Profile 需要使用它们时，就可以将它们组合起来。

```text
                 Web Profile
                      │
             ┌────────┴────────┐
             │                 │
          dsh-base         dsh-web-app
             │                 │
        共享核心能力         Web 专属能力
             │                 │
             └────────┬────────┘
                      │
                最终运行环境
```

这里有一个非常重要的认识：

**Bundle 是 Plugin 的逻辑组合与复用单元。它通过一个或多个 Patch 组织插件配置，使一组相关 Plugin 可以被不同 Profile 复用。**


#### 一个 Bundle 可以包含多个 Patch

继续看：

```text
packages/bundle/web-app/package.json
```

其中的 `dsh.bundle.patch` 声明了多个 Patch 文件：

```json
{
  "dsh": {
    "bundle": {
      "patch": [
        "./cordis.patch.yml",
        "./presets/standard.patch.yml",
        "./presets/ptc.patch.yml",
        "./presets/minimal.patch.yml",
        "./presets/cordis.patch.yml"
      ]
    }
  }
}
```

这说明一个 Bundle 不一定只有一个 Patch。

它可以通过多个 Patch 文件组织不同层次的插件配置。

这里需要注意：

- `cordis.patch.yml` 是一个 Patch 文件；
- `presets/standard.patch.yml` 是另一个 Patch 文件；
- `presets/ptc.patch.yml` 又是另一个 Patch 文件；
- 它们都属于这个 Bundle 声明的配置资源。

不能仅凭文件名就认为所有 preset 都会在每次启动时同时生效。

具体哪些配置被应用，以及它们之间如何叠加，需要继续检查启动器读取 Bundle 和处理 Patch 的源码。

### 2.1.4 Patch 是什么：它负责修改插件配置树

现在已经知道：

- Plugin 是实际提供能力的单元；
- Bundle 是组织多个 Plugin 及其配置的装配包。

那么 Patch 又是什么？

Patch 是对插件配置进行增加、修改或调整的配置描述。

它本身通常不是一个运行时插件。

它描述的是：

> 我希望当前插件配置树发生什么变化。

例如，某个已有插件的配置是：

```yaml
- id: session
  name: '@deepseek-ai/dsh-session'
  config:
    persistence: true
```

现在我们希望调整它的配置：

```yaml
- id: session
  config:
    persistence: false
```

这只是一个概念示例。

它表示希望修改 `session` 这一配置行，而不是重新实现 Session Plugin。

实际能否这样覆盖、配置是递归合并还是整体替换、Patch 的具体语法如何执行，都必须以当前仓库的 Patch 处理实现为准。

#### 为什么需要 Patch？

因为 Plugin 的实现和 Plugin 的配置是两个不同的问题。

假设一个 Plugin 提供文件访问能力。

它的 TypeScript 代码可能实现：

- 读取文件；
- 写入文件；
- 列出目录；
- 删除文件。

但是，不同运行形态可能需要不同的配置：

```yaml
# 运行形态 A
config:
  read: true
  write: true
  delete: false
```

```yaml
# 运行形态 B
config:
  read: true
  write: false
  delete: false
```

Plugin 的代码可以保持不变，只通过配置改变它的行为。

这就是 Patch 的价值：

**尽可能通过配置调整已有能力，而不是一遇到需求就修改 Plugin 源码。**

在二次开发中，这一点非常重要。

如果只是要：

- 调整某个插件的参数；
- 禁用某个可选功能；
- 增加一个已有插件；
- 替换一个插件实现；

应该先研究 Profile 和 Patch 的配置机制。

只有当现有插件本身没有所需能力时，才考虑修改 Plugin 源码或开发新 Plugin。

**Patch 是描述插件配置树如何变化的配置文件，其中可以包含 Plugin 的配置，也可以包含插件的增加、调整等操作。**

### 2.1.5 Profile 是什么：决定最终运行形态

现在来看 Profile。

Profile 可以理解成一份运行形态的装配方案。

例如：

```bash
dsh --profile web
```

表示选择 Web 运行形态。

而：

```bash
dsh --profile headless
```

表示选择 Headless 运行形态。

不同 Profile 可以组合不同的 Bundle 和 Patch。

例如：

```text
Web Profile
    │
    ├── 共享核心 Bundle
    ├── Web 应用 Bundle
    └── Profile 自己的配置调整

Headless Profile
    │
    ├── 共享核心 Bundle
    ├── Headless 相关功能
    └── Profile 自己的配置调整
```

这里需要区分：

Profile 不是 Plugin。

Profile 也不是 Bundle。

它更接近一个最终运行形态的选择入口。

当用户指定 Profile 时，Harness 需要根据该 Profile 的定义找到对应的装配资源，再经过配置处理，形成最终的插件配置。

例如：

```bash
dsh --profile web --dump-config
```

这条命令的价值就在于：

它可以帮助我们观察 Web Profile 最终形成的配置。

你可以从结果中检查：

- 哪些 Plugin 配置行存在；
- 每条配置行的 id 是什么；
- 对应的 name 是什么；
- config 中有哪些参数；
- 哪些配置行被调整或禁用。

这比只看某一个 Bundle 的 Patch 更接近最终运行结果。

### 2.1.6 把四个概念放在一起：完整装配关系

现在可以把整个关系串起来：

```text
用户执行启动命令
       │
       ▼
选择 Profile
       │
       ▼
确定运行形态
       │
       ▼
找到该 Profile 使用的 Bundle
       │
       ▼
读取 Bundle 的 package.json
       │
       ▼
找到 Bundle 声明的 Patch 文件
       │
       ▼
处理 Patch
       │
       ▼
形成最终插件配置
       │
       ▼
根据配置加载 Plugin
       │
       ▼
Plugin 注册服务、监听事件、提供功能
       │
       ▼
Harness 开始运行
```

用一句话总结：

- Profile 决定运行形态；
- Bundle 组织可复用的功能组合；
- Patch 描述插件配置如何变化；
- Plugin 在运行时真正提供功能。

再用一个例子理解：

假设我们要启动 Web 版本的 Harness。

```text
Profile：web
    │
    ▼
选择 Web 所需的功能组合
    │
    ├── dsh-base
    │     ├── Session Plugin
    │     ├── Agent Plugin
    │     ├── LLM Plugin
    │     └── Tool Plugin
    │
    └── dsh-web-app
          ├── Web Server Plugin
          ├── Web Runtime Plugin
          └── Connection Plugin
    │
    ▼
处理各层 Patch
    │
    ▼
生成最终配置
    │
    ▼
加载 Plugin
    │
    ▼
Web Harness 启动
```

这就是四个概念之间的关系。

### 2.1.7 源码陪读：沿着启动器追踪 Bundle 和 Patch

概念理解之后，必须看真实代码。

本节优先阅读以下文件：

| 文件 | 阅读重点 |
|---|---|
| `packages/bundle/base/package.json` | Base Bundle 如何声明 Patch 和依赖 |
| `packages/bundle/base/cordis.patch.yml` | Base Bundle 实际装配哪些插件 |
| `packages/bundle/web-app/package.json` | Web Bundle 如何声明多个 Patch |
| `packages/bundle/web-app/cordis.patch.yml` | Web Bundle 如何组织 Web 专属插件 |
| `packages/boot/app-boot/src/index.ts` | 启动器如何加载 Profile、准备 Patch |
| `packages/boot/app-boot/src/bundle-platform.ts` | Bundle 相关的平台处理逻辑 |
| `packages/boot/app-boot/src/profile-context.ts` | Profile 上下文相关逻辑 |
| `apps/cli/composition.md` | 查看生成的插件组合信息 |

首先打开：

```text
packages/boot/app-boot/src/index.ts
```

重点搜索以下函数或标识符：

```ts
loadProfileDirectory
prepareProfilePatches
bundlePatchFiles
```

这些位置是理解 Profile 加载与 Patch 准备过程的重要入口。

阅读时不要只看函数名。

需要回答：

1. Profile 从哪里读取？
2. Profile 目录如何确定？
3. Bundle 的 Patch 文件从哪里找到？
4. 多个 Patch 文件如何组织？
5. 最终配置如何传递到后续启动流程？

建议使用：

```bash
rg -n "loadProfileDirectory|prepareProfilePatches|bundlePatchFiles" packages/boot/app-boot/src
```

再查看命中位置的上下文：

```bash
rg -n -C 12 "prepareProfilePatches" packages/boot/app-boot/src/index.ts
```

然后继续搜索：

```bash
rg -n "bundle|patch|profile" packages/boot/app-boot/src
```

注意：这里先建立调用关系，不要仅凭函数名称推断 Patch 的优先级、合并策略或禁用语义。

这些行为必须继续追踪具体实现。

#### Profile、Bundle、Patch 的实际加载流程

在 Harness 中，Profile 并不是直接加载 Plugin，而是先读取配置，再由 Loader 根据最终配置加载插件。核心流程如下：

1. **确定 Profile 目录**：`resolveProfileDir()` 根据 Harness Home 和 Profile 名称，定位 `$DSH_HOME/profiles/<profile-name>`。
2. **读取 Profile**：`loadProfileDirectory()` 读取 Profile 的 `package.json`，通过 `dsh.profile.bundles` 获取需要使用的 Bundle。
3. **查找 Bundle 的 Patch**：遍历 Bundle，使用 `resolveBundleDir()` 定位软件包目录，读取 Bundle 自己的 `package.json`，再通过 `bundlePatchPaths()` 获取 `dsh.bundle.patch` 声明的 Patch 文件。
4. **汇总 Patch**：`readProfilePatches()` 按顺序汇总 Bundle Patch、Profile 自身的 Patch、Home Patch 和命令行 Overlay。
5. **生成最终插件配置**：`prepareProfilePatches()` 调用 `applyEntryPatches()` 将 Patch 应用到配置树，再通过 `prepareProfileEntries()` 进行插件兼容性预检，最后将结果包装成 `insert` Patch。
6. **交给 Loader**：启动代码将处理后的 Patch 放入 `Include.Config.patches`，通过 `ctx.loader.create(rootInclude)` 交给 Include Loader，继续加载插件。

关键代码：

```ts
// profile.ts：确定 Profile 目录
const dir = resolveProfileDir(name, home)

// 读取 Profile 和 Bundle 配置
const profile = loadProfileDirectory(
  binName,
  dir,
  installAnchor,
  options,
)

// profile-context.ts：汇总各层 Patch
const patches = readProfilePatches(
  binName,
  context,
  initialProfile,
)

// compatibility-preflight.ts：合成配置并进行兼容性检查
const entries = applyEntryPatches(
  [],
  structuredClone(patches),
  patchWarning(ctx),
).data

const rows = prepareProfileEntries(
  ctx,
  entries,
  parentURL,
  binName,
)

// index.ts：交给 Include Loader
const prepared = prepareProfilePatches(
  ctx,
  [...patches],
  parentURL,
  binName,
)

const includeConfig: Include.Config = {
  path: pathToFileURL(absoluteConfigPath).href,
  ...(prepared.length > 0 ? { patches: prepared } : {}),
}

const rootInclude: EntryOptions = {
  id: 'include',
  name: 'cordis:include',
  config: includeConfig,
}

const includeId = await ctx.loader.create(rootInclude)
```

**理解这条调用链的关键：**

```text
Profile
  → Bundle
  → Patch
  → 合成最终插件配置
  → 兼容性预检
  → Include Loader
  → Plugin
```

Bundle 负责组织可复用的插件配置，Patch 描述配置如何变化，Profile 决定使用哪些功能组合。它们本身不直接执行插件，真正的插件加载发生在后续 Loader 阶段。

源码位置（固定 Commit：`2e39e70496aa02986ca6a820c5d497e816b69d7e`）：

- `packages/boot/app-boot/src/profile.ts`
- `packages/boot/app-boot/src/profile-context.ts`
- `packages/boot/app-boot/src/compatibility-preflight.ts`
- `packages/boot/app-boot/src/index.ts`



### 2.1.8 实验：### 通过 Patch 扩展功能，而不是直接修改核心

这一节先不要求你开发插件。

我们先从 Harness 仓库中已经存在的插件入手，弄清楚三个问题：

1. 一个插件在项目中是如何组织的？
2. Harness 是如何通过 Patch 配置插件的？
3. 插件配置进入最终配置树之后，又是如何被加载的？

整个实践分成三个阶段。

#### 第一阶段：找到一个已经存在的插件

先不要自己编写插件，而是从现有 Bundle 中寻找一个已经使用的插件。

**步骤 1：找到 Web Profile 使用的 Bundle**

阅读：

```text
packages/util/official-bundles/src/bundles.ts
```

找到 `PROFILE_TEMPLATES`，确认 Web Profile 使用哪些 Bundle。

本仓库中 Web Profile 使用：

```ts
web: {
  bundles: ['@deepseek-ai/dsh-base', '@deepseek-ai/dsh-web-app'],
},
```

**步骤 2：找到 Bundle 声明的 Patch 文件**

阅读：

```text
packages/bundle/base/package.json
packages/bundle/web-app/package.json
```

找到 `dsh.bundle.patch` 字段，确认 Bundle 声明了哪些 Patch 文件。

例如，Web App Bundle 声明了多个 Patch 文件：

```text
packages/bundle/web-app/cordis.patch.yml
packages/bundle/web-app/presets/standard.patch.yml
packages/bundle/web-app/presets/ptc.patch.yml
packages/bundle/web-app/presets/minimal.patch.yml
packages/bundle/web-app/presets/cordis.patch.yml
```

注意：这些文件不一定同时全部生效，具体使用哪些 Patch，还要结合 Profile 和启动逻辑分析。

**步骤 3：打开 Patch 文件，找到已有插件的配置**

先阅读：

```text
packages/bundle/web-app/cordis.patch.yml
```

如果还没有看懂，可以继续阅读：

```text
packages/bundle/web-app/presets/standard.patch.yml
```

重点观察：

- 插件配置在 YAML 中如何表示？
- `id`、`name`、`config` 分别表示什么？
- 一个 Patch 如何插入插件配置？
- 不同插件之间是否存在依赖关系？

这里不要急着修改，先找到一个真实的插件配置作为参考。

#### 第二阶段：从插件配置追踪到插件源码

**步骤 4：根据 Patch 中的插件名称找到对应的 npm 包**

在 Patch 中找到一个插件的 `name`，然后在仓库中搜索这个包名。

找到对应的插件目录后，阅读它的：

```text
package.json
```

以及源码入口文件。

重点检查：

- `name`：插件包名。
- `exports`、`main`：包对外暴露的入口。
- `dependencies`：插件依赖哪些其他包。
- `peerDependencies`：插件要求宿主提供哪些依赖。

不要只看 `package.json`，还要继续阅读源码，确认：

- 插件从哪里导出？
- 插件初始化函数是什么？
- 初始化时注册了哪些服务或事件？
- 插件是否需要其他插件提供的服务？

**步骤 5：追踪插件是如何被加载的**

结合前面找到的插件源码，继续追踪 Harness 的加载过程。

重点阅读：

```text
packages/boot/app-boot/src/index.ts
packages/boot/app-boot/src/compatibility-preflight.ts
```

理解以下过程：

1. Profile 读取 Bundle 和 Patch。
2. Patch 被汇总并进行预处理。
3. 处理后的 Patch 被放入 `Include.Config.patches`。
4. 启动代码创建 `cordis:include` 入口。
5. Include 插件读取并应用 Patch。
6. Cordis Loader 根据最终配置加载插件。

这一阶段的目标是：能够从 YAML 中的插件配置，一直追踪到插件源码的加载过程。



#### 第三阶段：从零编写一个最简单的插件

这一阶段不使用已经存在的插件，也不通过 CLI 安装外部插件，而是从零创建一个最简单的插件，并将它直接集成到 Harness 源码的 Web Bundle 中。

本阶段的目标是：

- 理解一个最简单的 Harness 插件由什么组成；
- 学会编写插件入口文件；
- 理解插件如何监听 Agent 生命周期事件；
- 理解如何通过 Bundle 的依赖声明和 Patch 加载插件；
- 让插件随着 Web Bundle 一起运行，而不是在启动后再手动安装。

本阶段编写的插件只有一个功能：

**每当用户发起一个新的问题时，在交给模型处理的消息列表中追加一条用户消息：`娃哈哈真好喝`。**

注意：这条消息会进入模型的消息上下文，不是在聊天界面中额外显示一条提示。

---

##### 3.1 创建插件目录

本次直接在 Harness 源码仓库中创建插件，不需要另外建立一个独立项目。

在仓库根目录下创建：

```text
packages/
└── experimental/
    └── wahaha-before-question/
        ├── package.json
        └── src/
            └── index.js
```

其中：

| 文件 | 作用 |
|---|---|
| `package.json` | 声明插件包的名称、入口和依赖关系 |
| `src/index.js` | 插件真正执行的代码 |

为什么放在 `packages/experimental/`？

因为这是一个实验性插件。我们先把它作为仓库内部的 workspace package 开发，暂时不需要将它发布到 npm。

仓库的 `pnpm-workspace.yaml` 已经包含能够匹配 `packages/*/*` 的目录规则，因此这个目录可以作为 workspace 包参与依赖管理。

##### 3.2 编写 package.json

创建：

`packages/experimental/wahaha-before-question/package.json`

内容如下：

```json
{
  "name": "@deepseek-ai/dsh-wahaha-before-question",
  "version": "0.2.2-rc.5",
  "private": true,
  "type": "module",
  "main": "./src/index.js",
  "exports": {
    ".": "./src/index.js"
  },
  "peerDependencies": {
    "@deepseek-ai/cordis": "workspace:~",
    "@deepseek-ai/dsh-agent": "workspace:*",
    "@deepseek-ai/dsh-llm": "workspace:*"
  }
}
```

逐项理解：

**1. name**

```json
"name": "@deepseek-ai/dsh-wahaha-before-question"
```

这是插件包的名称。

后续在 Web Bundle 的 `package.json` 和 `cordis.patch.yml` 中，都要使用这个名称。

**2. private**

```json
"private": true
```

表示这是仓库内部使用的包，不允许直接发布到 npm。

**3. type**

```json
"type": "module"
```

表示使用 ES Module 模块规范，可以使用：

```js
import { createUserMessage } from '@deepseek-ai/dsh-llm'
```

**4. main 和 exports**

```json
"main": "./src/index.js",
"exports": {
  ".": "./src/index.js"
}
```

告诉模块解析系统：当其他包导入这个插件时，应该从哪个文件加载。

**5. peerDependencies**

```json
"peerDependencies": {
  "@deepseek-ai/cordis": "workspace:~",
  "@deepseek-ai/dsh-agent": "workspace:*",
  "@deepseek-ai/dsh-llm": "workspace:*"
}
```

表示插件依赖 Harness 提供的运行环境及相关 API。

这里的 `@deepseek-ai/dsh-llm` 用于创建用户消息；Cordis 和 Agent 则属于插件运行时所依赖的框架契约。

注意：`peerDependencies` 是依赖关系声明，不代表插件已经被加载。真正的加载配置还需要在后面通过 Bundle Patch 完成。

##### 3.3 编写插件入口文件

创建：

`packages/experimental/wahaha-before-question/src/index.js`

写入：

```js
import { createUserMessage } from '@deepseek-ai/dsh-llm'

export const name = 'wahaha-before-question'

export function apply(ctx) {
  ctx.on(
    'agent/pre-step',
    async ({ step, signal }, next) => {
      // 先执行后续监听器，取得当前的消息处理结果
      const decision = await next()

      // 如果当前 Step 被拒绝，或者任务已经取消，就不再追加消息
      if (decision.kind === 'reject' || signal.aborted) {
        return decision
      }

      // 每个 Turn 只在第一个 Step 追加一次
      if (step !== 1) {
        return decision
      }

      // 创建一条新的用户消息
      const message = createUserMessage({
        content: [
          {
            type: 'text',
            text: '娃哈哈真好喝'
          }
        ],
        source: {
          kind: 'user'
        }
      })

      // 将新消息追加到原来的消息列表中
      return {
        ...decision,
        messages: [...decision.messages, message]
      }
    },
    { prepend: true }
  )
}
```

下面重点理解这段代码。

**1. 导入消息创建函数**

```js
import { createUserMessage } from '@deepseek-ai/dsh-llm'
```

这里使用 Harness 已经提供的消息创建 API，而不是自己手动拼接一条消息。

`createUserMessage()` 会按照 Harness 的消息结构创建用户消息。

**2. 插件入口**

```js
export const name = 'wahaha-before-question'

export function apply(ctx) {
  // ...
}
```

这里有两个重要概念：

- `name`：插件自身的标识。
- `apply(ctx)`：插件被加载时执行的入口函数。

`ctx` 是 Cordis 提供的插件上下文，可以通过它注册事件监听器。

**3. 监听 Agent 生命周期事件**

```js
ctx.on(
  'agent/pre-step',
  async ({ step, signal }, next) => {
    // ...
  },
  { prepend: true }
)
```

`agent/pre-step` 是 Agent 在执行一个 Step 之前触发的事件。

可以把它理解为：

```text
用户提出问题
    ↓
Agent 准备执行 Step
    ↓
触发 agent/pre-step
    ↓
插件有机会修改消息列表
    ↓
Agent 继续执行
```

其中：

- `step`：当前是第几个 Step。
- `signal`：用于判断任务是否已经取消。
- `next`：继续执行事件处理链中的后续监听器。

`{ prepend: true }` 表示将这个监听器放在事件处理链的前面注册。

**4. 创建用户消息**

```js
const message = createUserMessage({
  content: [
    {
      type: 'text',
      text: '娃哈哈真好喝'
    }
  ],
  source: {
    kind: 'user'
  }
})
```

这里创建的是一条真正进入 Agent 消息列表的用户消息。

`content` 是消息内容，`source` 用来标记消息来源。

本例使用已有的 `user` 来源类型，避免为了一个最简单的插件额外引入 TypeScript 类型扩展。

**5. 将消息追加到原消息列表**

```js
return {
  ...decision,
  messages: [...decision.messages, message]
}
```

这段代码是整个插件的核心。

假设原来的消息列表是：

```js
[
  {
    role: 'user',
    content: '今天天气怎么样？'
  }
]
```

追加后，逻辑上变成：

```js
[
  {
    role: 'user',
    content: '今天天气怎么样？'
  },
  {
    role: 'user',
    content: '娃哈哈真好喝'
  }
]
```

这里使用：

```js
...decision.messages
```

保留原来的所有消息，再把新消息追加到末尾。

注意：上面的消息列表只是为了说明追加操作的示意结构，不是 `createUserMessage()` 返回对象的完整实际输出。

##### 3.4 将插件加入 Web Bundle

插件代码写完后，还不能直接运行。

因为我们希望插件成为 Harness Web Bundle 的一部分，所以需要完成两个动作：

1. 把插件声明为 Web Bundle 的依赖；
2. 通过 Web Bundle 的 Patch 告诉 Harness 加载这个插件。

这两个动作缺一不可。

**第一步：修改 Web Bundle 的 package.json**

打开：

`packages/bundle/web-app/package.json`

找到：

```json
"dependencies": {
```

在这个对象中加入：

```json
"@deepseek-ai/dsh-wahaha-before-question": "workspace:*"
```

例如：

```json
"dependencies": {
  "@deepseek-ai/dsh-wahaha-before-question": "workspace:*"
}
```

注意：

- 这里展示的是需要添加的依赖条目；
- 不要用这段示例覆盖原来的整个 `dependencies`；
- 应当保留原有依赖，只增加这一项。

为什么要加入 `dependencies`？

因为插件现在是 Web Bundle 的运行时依赖。

通过：

```json
"@deepseek-ai/dsh-wahaha-before-question": "workspace:*"
```

告诉 pnpm：Web Bundle 依赖的是当前仓库中的这个 workspace 包，而不是从 npm 下载一个独立版本。

**第二步：修改 Web Bundle 的 cordis.patch.yml**

打开：

`packages/bundle/web-app/cordis.patch.yml`

找到现有的 `insert` 配置。

例如，已有配置可能类似：

```yaml
- insert:
    - id: desktop-product-telemetry
      name: '@deepseek-ai/dsh-host-product-telemetry-otel'
```

在 `insert` 列表中增加：

```yaml
    - id: wahaha-before-question
      name: '@deepseek-ai/dsh-wahaha-before-question'
```

最终效果类似：

```yaml
- insert:
    - id: desktop-product-telemetry
      name: '@deepseek-ai/dsh-host-product-telemetry-otel'

    - id: wahaha-before-question
      name: '@deepseek-ai/dsh-wahaha-before-question'
```

注意：这里只展示结构示意。实际修改时，必须保留文件中原有的所有 Patch 配置，并将新条目放在正确的 `insert` 列表中。

这里有两个字段：

| 字段 | 作用 |
|---|---|
| `id` | Patch 中用于标识这个插件的 ID |
| `name` | 对应插件的 npm 包名称 |

特别注意：

`id` 和 `name` 不是同一个概念。

`id` 是 Patch 配置中的插件标识，而 `name` 必须对应前面 `package.json` 中声明的包名。

##### 3.5 安装依赖并启动

完成以上修改后，回到仓库根目录。

**第一步：安装依赖**  
在执行 pnpm install 之前，确保 Node.js 和 Corepack 可用，并且能够正常执行项目要求版本的 pnpm。

```powershell
pnpm install
```

这一步会让 pnpm 重新计算 workspace 依赖关系，并建立新插件与 Web Bundle 之间的依赖关系。

**第二步：启动 Web Profile**

```powershell
pnpm run start:web
```

这里使用仓库根目录中已经定义的启动脚本。

**第三步：测试插件**

启动后，在聊天界面输入：

```text
你好，请介绍一下自己。
```

插件预期会在 Agent 的消息列表中追加：

```text
娃哈哈真好喝
```

因此，模型接收到的消息上下文中，除了用户原本的问题，还会包含插件追加的这条消息。

这里的验证重点不是聊天界面是否出现一条独立的提示，而是 `agent/pre-step` 事件处理后的消息列表是否包含这条新消息。

如果没有达到预期，不要直接认定插件代码有问题，应依次检查：

1. 插件目录是否符合 workspace 包匹配规则；
2. Web Bundle 的 `package.json` 是否正确添加了 workspace 依赖；
3. `cordis.patch.yml` 是否正确添加了插件加载条目；
4. 插件入口是否正确导出 `name` 和 `apply`；
5. `agent/pre-step` 监听器是否成功注册；
6. 启动日志中是否存在模块解析或插件加载错误。

##### 3.6 回顾：一个插件是如何进入 Harness 的？

到这里，我们从零编写了一个插件，并把它接入了 Web Bundle。

整个过程可以归纳为：

```text
创建插件目录
    │
    ▼
编写 package.json
    │
    ▼
编写 src/index.js
    │
    ▼
Web Bundle package.json
声明 workspace 依赖
    │
    ▼
Web Bundle cordis.patch.yml
声明插件加载配置
    │
    ▼
pnpm install
    │
    ▼
启动 Web Profile
    │
    ▼
Harness 加载插件并执行
```

现在应该能够区分三个不同层次：

| 层次 | 解决的问题 |
|---|---|
| Workspace | 告诉 pnpm：插件源码是仓库中的一个包 |
| Bundle dependencies | 告诉 Web Bundle：运行时需要这个插件 |
| Bundle Patch | 告诉 Harness：需要加载这个插件 |

**特别注意：**

本阶段完成的是 Web Bundle 的源码集成与加载配置，不等于已经证明 Windows/macOS Desktop 最终安装包一定包含该插件。

Desktop 安装包还涉及独立的运行时依赖收集和打包流程，后续需要沿着 Desktop 构建链路继续验证。

本阶段先掌握插件从零编写、加入 Bundle 依赖、通过 Patch 加载的完整过程。

### 2.1.9 Agent 生命周期事件：认识 12 个核心事件

在正式编写插件之前，需要先理解 Harness 提供了哪些 Agent 生命周期事件。

前面编写的 `wahaha-before-question` 插件中，有这样一段代码：

```js
ctx.on('agent/pre-step', async (payload, next) => {
  // 插件逻辑
})
```

其中：

- `ctx.on()`：注册事件监听器；
- `agent/pre-step`：指定要监听的事件；
- 回调函数：事件触发时执行的插件逻辑。

那么问题来了：

**除了 `agent/pre-step`，Harness 还提供了哪些事件？**

这些事件分别在什么时候触发？哪些适合修改消息，哪些适合观察运行状态？

本节先建立完整的事件认知，再通过后续实战逐个学习。

#### 1. 在哪里查找 Agent 事件？

首先打开源码文件：

```text
packages/core/agent/src/runtime-types.ts
```

搜索：

```ts
declare module '@deepseek-ai/cordis' {
  interface Events {
```

你会看到 Harness 对 Cordis 的 `Events` 接口进行了扩展。

这里集中定义了 Agent 相关事件，包括：

- 事件名称；
- 事件参数；
- 事件处理方式；
- 事件触发时机；
- 是否支持异步处理。

因此，当你在某个插件中看到：

```js
ctx.on('agent/xxx', ...)
```

应该首先到这个文件中搜索对应的事件名称，而不是直接猜测它的作用。

#### 2. Agent 的 12 个核心事件

在当前学习使用的源码版本中，`runtime-types.ts` 定义了以下 12 个 `agent/*` 事件。

**第一类：Agent 生命周期事件**

| 事件名称 | 作用 | 常见用途 |
|---|---|---|
| `agent/created` | Agent 创建完成 | 初始化插件数据、注册 Agent 级资源 |
| `agent/disposed` | Agent 从注册表中移除 | 清理资源、释放监听器 |
| `agent/status` | Agent 状态发生变化 | 监控 Agent 是否正在运行 |

这三个事件关注的是 Agent 本身的生命周期。

例如：

```js
ctx.on('agent/created', ({ agent, source }) => {
  console.log('Agent 创建完成：', agent.id)
  console.log('创建来源：', source)
})
```

**第二类：输入队列事件**

| 事件名称 | 作用 | 常见用途 |
|---|---|---|
| `agent/inbox/inserted` | 消息进入输入队列 | 观察新输入 |
| `agent/inbox/claimed` | 消息被 Agent 领取 | 观察输入消费 |
| `agent/inbox/discarded` | 消息从输入队列中丢弃 | 监控消息丢弃情况 |

这里需要区分：

- `inserted`：消息进入队列；
- `claimed`：消息被 Agent 领取；
- `discarded`：消息被丢弃。

它们不是同一个动作，而是输入队列处理过程中的不同阶段。

**第三类：Agent 执行扩展点**

| 事件名称 | 作用 | 常见用途 |
|---|---|---|
| `agent/pre-step` | Agent 准备执行 Step | 检查、替换或拒绝消息 |
| `agent/request` | 修改模型调用配置 | 兼容已有请求配置扩展 |
| `agent/request-error` | 模型请求失败 | 处理重试或恢复逻辑 |
| `agent/turn-stopping` | Turn 即将结束 | 检查是否需要继续处理 |
 
其中：

`agent/pre-step` 是本阶段已经使用的事件。

`agent/request` 虽然仍然存在于事件定义中，但已经标记为 deprecated，不建议新插件继续使用。

新代码应优先使用 `agent.model.select()` 等当前公开 API 管理模型选择。

**第四类：输出与错误事件**

| 事件名称 | 作用 | 常见用途 |
|---|---|---|
| `agent/assistant-stream` | 发布 Assistant 流式输出 | 观察模型输出过程 |
| `agent/error` | Agent 执行过程中发生错误 | 错误记录、监控和通知 |

这两个事件分别关注模型输出和执行错误。

#### 3. 三种事件处理模式

除了事件名称，还需要关注事件定义中的处理模式。

在 `runtime-types.ts` 中，可以看到事件注释包含不同的 `@mode` 标记。

主要有三种：

**① emit：通知型事件**

例如：

```ts
'agent/status'(payload: {
  agent: Agent
  status: AgentStatus
}): void
```

这种事件主要用于通知监听器发生了什么。

插件可以监听它：

```js
ctx.on('agent/status', ({ status }) => {
  console.log('Agent 状态：', status)
})
```

适合观察、记录和通知，不适合用来修改 Agent 的执行决策。

**② serial：顺序执行型事件**

例如：

```ts
'agent/created'(payload: {
  agent: Agent
  source: SessionStartSource
  signal?: AbortSignal
}): undefined | Promise<undefined>
```

监听器按照顺序执行，框架会等待异步监听器完成。

适合需要在后续流程继续之前完成的初始化工作。

**③ waterfall：流水线型事件**

例如：

```ts
'agent/pre-step'(
  payload: {
    agent: Agent
    messages: UserMessage[]
    turn: number
    step: number
    signal: AbortSignal
    modelSelection: Readonly<ModelSelection>
  },
  next: () => Promise<PreStepDecision>
): Promise<PreStepDecision>
```

这种事件允许监听器参与处理结果。

监听器可以：

- 调用 `next()`，继续执行后续处理；
- 获取后续处理返回的结果；
- 根据需要修改结果；
- 按事件契约返回处理决策。

前面编写的 `wahaha-before-question` 插件，就是利用 waterfall 事件修改进入 Step 的消息列表。

注意：不同事件的处理模式决定了监听器能做什么，不能只看事件名称就判断它是否可以修改数据。

#### 4. 12 个事件应该如何学习？

不建议一开始就把 12 个事件全部写成插件。

更合理的方式是按照使用场景逐步学习：

| 学习顺序 | 事件 | 实战目标 |
|---|---|---|
| 1 | `agent/pre-step` | 在模型执行前追加消息 |
| 2 | `agent/created` | Agent 创建时输出初始化日志 |
| 3 | `agent/status` | 监听 Agent 的运行状态 |
| 4 | `agent/inbox/inserted` | 观察消息进入输入队列 |
| 5 | `agent/inbox/claimed` | 观察消息被领取 |
| 6 | `agent/inbox/discarded` | 观察消息被丢弃 |
| 7 | `agent/assistant-stream` | 观察模型流式输出 |
| 8 | `agent/request-error` | 观察模型请求失败并理解重试机制 |
| 9 | `agent/turn-stopping` | 观察 Turn 结束前的处理 |
| 10 | `agent/error` | 记录 Agent 执行错误 |
| 11 | `agent/disposed` | 观察 Agent 销毁过程 |
| 12 | `agent/request` | 阅读旧接口，理解 deprecated 的含义，不再作为新插件实战入口 |

#### 5. 阅读其他插件时，如何追踪 ctx.on()？

以后看到其他插件中出现：

```js
ctx.on('agent/pre-step', handler)
```

不要只看 `handler` 中的代码。

建议按照下面的顺序追踪：

1. **找到事件定义**

   打开：

   ```text
   packages/core/agent/src/runtime-types.ts
   ```

   搜索 `agent/pre-step`。

2. **理解事件参数**

   查看回调参数包含哪些字段，例如：

   ```ts
   agent
   messages
   turn
   step
   signal
   modelSelection
   ```

3. **理解事件处理模式**

   查看注释中的 `@mode`，判断它属于 emit、serial 还是 waterfall。

4. **找到事件触发位置**

   在仓库中搜索：

   ```text
   agent/pre-step
   ```

   找到实际触发事件的代码，观察它在 Agent 执行流程中的位置。

5. **回到插件入口**

   最后再阅读插件的 `ctx.on()` 回调，理解插件为什么选择这个事件，以及它修改了什么。

这样就形成了完整的源码阅读路径：

```text
插件中的 ctx.on()
        │
        ▼
识别事件名称
        │
        ▼
runtime-types.ts
查看事件定义和参数
        │
        ▼
查看事件处理模式
        │
        ▼
搜索事件触发位置
        │
        ▼
理解 Agent 执行流程
        │
        ▼
回到插件理解具体实现
```

#### 6. 本节小结

这一节最重要的不是记住 12 个事件的名称，而是掌握一种阅读插件源码的方法。

以后遇到不熟悉的：

```js
ctx.on('xxx', ...)
```

先找到事件定义，再找到事件触发位置，最后理解插件监听器的具体实现。

**记住：事件定义告诉你这个事件能做什么，事件触发位置告诉你它什么时候发生，插件代码告诉你开发者利用它完成了什么功能。**

### 2.1.10 本节必须掌握的结论

最后用一个表格检查自己是否真正理解：

| 问题 | 正确理解 |
|---|---|
| Profile 是什么？ | 一种运行形态的装配方案 |
| Bundle 是什么？ | 可复用的功能装配包 |
| Patch 是什么？ | 描述插件配置变化的文件 |
| Plugin 是什么？ | 真正提供运行时能力的单元 |
| Bundle 是否等于 Plugin？ | 不等于 |
| dependencies 是否等于插件清单？ | 不等于 |
| 一个 Bundle 是否可以声明多个 Patch？ | 可以 |
| 修改插件配置是否一定需要改 Plugin 源码？ | 不一定 |
| dump-config 是否等于插件已经成功运行？ | 不等于 |

记住整个装配关系：

```text
Profile
   │
   ▼
选择运行形态
   │
   ▼s
Bundle
   │
   ▼
组织 Patch 和插件配置
   │
   ▼
Patch
   │
   ▼
修改最终插件配置树
   │
   ▼
Plugin
   │
   ▼
注册服务、监听事件、提供功能
```

本节最重要的开发原则：

**遇到二次开发需求时，先判断应该修改 Profile、Bundle、Patch 还是 Plugin。不要一开始就修改核心源码。**

如果只是调整已有插件的配置，优先研究 Patch。

如果是增加某个运行形态的功能，优先研究对应 Bundle。

如果是增加或改变插件自身的能力，再考虑修改 Plugin。

如果是改变整体运行形态，才需要进一步研究 Profile。

## 2.2 从 CLI 追到 Loader：一次启动究竟发生了什么

前面已经认识了 Profile、Bundle、Patch 和 Plugin，也已经知道一个 Profile 可以组合多个 Bundle，每个 Bundle 又可以通过 Patch 声明需要加载的插件。

但我们还没有真正回答一个问题：

**执行 `pnpm run start:web` 之后，DeepSeek Harness 究竟是怎样把这些配置变成一个正在运行的插件系统的？**

例如，执行：

```powershell
pnpm run start:web
```

程序不会直接读取某一个插件目录，然后把里面的插件全部加载进来。

它需要经历一系列步骤：

1. CLI 接收启动参数，确定要启动哪个 Profile；
2. 根据 Profile 找到对应的 Bundle；
3. 读取每个 Bundle 的 `package.json` 和 Patch 文件；
4. 按照规定的优先级组合所有 Patch；
5. 建立插件包的运行时模块解析关系；
6. 创建 Cordis Loader；
7. 将最终配置树交给 Loader；
8. Loader 根据配置加载插件；
9. 插件注册服务、监听事件，并完成初始化；
10. 启动流程检查必要插件是否成功激活。

本节不再停留在架构图，而是沿着真实源码，从 CLI 一直追踪到 Loader。

### 2.2.1 先明确本节要追踪的启动链路

先看整个启动过程：

```text
pnpm run start:web
        |
        v
根目录 package.json
        |
        v
apps/cli/src/bin.ts
        |
        v
parseDshArgs()
        |
        v
runProfile()
        |
        v
composeProfile()
        |
        +-----------------------------+
        |                             |
        v                             v
    loadProfile()             createRuntimeResolution()
        |                             |
        v                             v
读取 Profile 和 Bundle          建立运行时包解析关系
        |
        v
读取各层 Patch
        |
        v
组合最终配置
        |
        v
packages/boot/app-boot/src/index.ts
        |
        v
boot()
        |
        +--> 创建 Cordis Context
        |
        +--> 加载 Loader 插件
        |
        +--> 执行 prepare()
        |
        +--> mountRootInclude()
        |
        v
Cordis Loader 加载配置树
        |
        v
插件初始化、服务注入
        |
        v
启动检查
        |
        v
应用就绪
```

这张图先记住，不要求现在就理解每个函数。

接下来我们从入口开始，一步一步追踪。

本节源码以以下仓库版本为准：

```text
Repository:
DeepSeek-ai/deepseek-harness

Commit:
2e39e70496aa02986ca6a820c5d497e816b69d7e
```

### 2.2.2 第一步：从 package.json 找到 CLI 入口

首先打开项目根目录的 `package.json`。

找到 `scripts` 中的两个命令：

```json
{
  "scripts": {
    "dsh": "node --import tsx/esm apps/cli/src/bin.ts",
    "start:web": "node --import tsx/esm apps/cli/src/bin.ts web"
  }
}
```

这里有两个需要理解的地方。

**1. `start:web` 实际执行了什么？**

```powershell
pnpm run start:web
```

等价于执行：

```powershell
node --import tsx/esm apps/cli/src/bin.ts web
```

它做了三件事：

- 使用 Node.js 启动程序；
- 通过 `--import tsx/esm` 让 Node 能够加载 TypeScript ESM 源码；
- 将 `web` 作为命令行参数传给 `apps/cli/src/bin.ts`。

因此，`start:web` 并不是一个独立的 Web 启动器。

它只是给通用 CLI 传入了 `web` 参数。

**2. 为什么还有一个 `dsh` 命令？**

```json
{
  "dsh": "node --import tsx/esm apps/cli/src/bin.ts"
}
```

它启动的是同一个 CLI，只是不预先指定 Profile。

例如：

```powershell
pnpm dsh --profile web
```

和：

```powershell
pnpm run start:web
```

都可以进入 Web Profile 的启动流程。

前者显式指定 Profile，后者通过 `web` 位置参数选择 Profile。

这一点很重要：

**Web、Headless、SDK 等应用模式并不是各自拥有一套完全独立的 CLI 启动入口，而是由同一个 CLI 根据 Profile 选择不同的插件组合。**

源码位置：

- `package.json`
- `apps/cli/src/bin.ts`

### 2.2.3 第二步：阅读 bin.ts，理解 CLI 如何分派命令

打开：

```text
apps/cli/src/bin.ts
```

找到：

```ts
export async function runCli(options: RunCliOptions = {}): Promise<void> {
  const version = getDshRuntimeVersion()
  const { manageDesktopProfile, ...profileOptions } = options
  const invocation = parseDshArgs(process.argv.slice(2), version, manageDesktopProfile)

  switch (invocation.mode) {
    case 'profile': {
      const { runProfile } = await import('./profile-boot.ts')
      try {
        await runProfile({
          environment: loadLayeredEnv('dsh'),
          profile: invocation.profile,
          fromDefaultProfile: invocation.fromDefaultProfile,
          patchFiles: invocation.patches,
          args: invocation.args,
          ...profileOptions,
        })
      } catch (error) {
        // 启动错误处理
      }
      break
    }

    case 'plugin': {
      const { runPlugin } = await import('./plugin.ts')
      process.exit(await runPlugin(invocation.profile, invocation.args, options.packageManager))
      break
    }

    case 'dump-config': {
      const { runDumpConfig } = await import('./dump-config.ts')
      runDumpConfig(
        invocation.profile,
        invocation.defaultOnly,
        invocation.patches,
        invocation.fromDefaultProfile,
      )
      break
    }
  }
}
```

为了方便阅读，省略了部分错误处理和其他分支。

重点看这行：

```ts
const invocation = parseDshArgs(
  process.argv.slice(2),
  version,
  manageDesktopProfile
)
```

它把命令行参数交给 `parseDshArgs()` 解析。

然后根据解析结果的 `invocation.mode` 决定执行哪条流程。

这里至少有三种不同的工作：

| mode | 作用 |
|---|---|
| `profile` | 正常启动一个 Profile |
| `plugin` | 管理 Profile 中安装的插件依赖 |
| `dump-config` | 输出组合后的配置树，但不启动应用 |

注意：

`plugin` 和 `dump-config` 都不是正常的应用启动流程。

本节重点追踪的是：

```text
invocation.mode === 'profile'
```

它会动态导入：

```ts
const { runProfile } = await import('./profile-boot.ts')
```

然后调用：

```ts
await runProfile(...)
```

这意味着我们下一步应该进入：

```text
apps/cli/src/profile-boot.ts
```

**源码阅读练习：**

打开 `apps/cli/src/bin.ts`，自己回答：

1. CLI 从哪里取得命令行参数？
2. 谁负责解析这些参数？
3. 正常启动 Profile 时进入哪个函数？
4. 执行 `--dump-config` 时是否会进入 `runProfile()`？

如果这四个问题都能回答，说明你已经找到了 CLI 的分派入口。

### 2.2.4 第三步：理解 parseDshArgs() 如何选择 Profile

打开：

```text
apps/cli/src/args.ts
```

找到：

```ts
export function parseDshArgs(
  argv: readonly string[],
  version: string,
  manageDesktopProfile = false
): DshInvocation
```

这个函数负责把原始命令行参数转换成一个结构化的调用描述。

例如：

```powershell
pnpm dsh --profile web
```

会选择 `web` Profile。

而：

```powershell
pnpm dsh web
```

也可以选择 `web` Profile。

为什么？

看源码中的：

```ts
const expanded = first !== undefined && !first.startsWith('-') && first !== 'plugin'
  ? ['--profile', ...argv]
  : argv
```

如果第一个参数不是选项，并且不是 `plugin`，CLI 就会把它转换成 `--profile` 的形式。

因此：

```text
dsh web
```

实际上相当于：

```text
dsh --profile web
```

再看：

```ts
.option('--profile <name>', 'the profile under $DSH_HOME/profiles to boot')
```

这说明 Profile 名称由 CLI 参数选择，而不是写死在 `bin.ts` 中。

此外，CLI 还支持：

```text
--patch <path>
--from-default-profile <name>
--dump-config
--dump-config-schema
--dump-default-config
```

其中：

- `--patch` 用来追加配置覆盖层；
- `--from-default-profile` 用于从官方模板初始化或替换自定义 Profile；
- `--dump-config` 输出组合后的配置树；
- `--dump-default-config` 只输出 Bundle 层；
- `--dump-config-schema` 输出配置 Schema。

这里先记住：

**CLI 负责识别启动参数，并决定使用哪个 Profile；真正读取 Profile 和组合 Bundle 的工作由后续启动代码完成。**

源码位置：

```text
apps/cli/src/args.ts
```

### 2.2.5 第四步：进入 runProfile()，找到真正的启动入口

现在打开：

```text
apps/cli/src/profile-boot.ts
```

找到：

```ts
export async function runProfile(
  options: RunProfileOptions
): Promise<{ ctx: Context; shutdown: ProcessShutdown }>
```

这是所有正常 Profile 启动流程的核心入口之一。

函数内部有几个关键步骤。

**第一步：准备启动环境。**

```ts
const disposeProxy = await installProxyFromEnvironment(
  options.environment,
  (message) => {
    process.stderr.write(`${NAME}: ${message}\n`)
  },
)
```

这里处理启动环境中的代理配置。

它发生在插件挂载之前，保证后续插件使用一致的启动环境。

**第二步：组合 Profile。**

```ts
const composed = await composeProfile(
  options.profile,
  options.patchFiles,
  options.fromDefaultProfile,
  options.resolvedProfile,
)
```

这是本节最重要的函数调用之一。

它负责：

- 找到当前 Profile；
- 读取 Profile 选择的 Bundle；
- 读取 Bundle 的 Patch；
- 计算运行时模块解析关系；
- 保存额外的 Patch overlay 路径。

注意：这里还没有正式调用 Cordis Loader 加载整棵插件树。

它主要是在准备启动所需的配置和依赖解析信息。

**第三步：准备启动上下文。**

```ts
const profileContext: ProfileContext = {
  name: options.profile,
  dir: composed.profile.dir,
  patchPath: composed.profile.patchPath,
  installAnchor: INSTALL_ANCHOR,
  startedBundles: composed.profile.layers.map(layer => layer.packageName),
  cwd: process.cwd(),
  home: resolveDshHome(),
  overlayPaths: composed.overlayPaths,
  telemetryDisabledEnv: process.env.DSH_TELEMETRY_DISABLED,
}
```

这里构造了 `profileContext`。

它保存当前启动所需的一些信息，例如：

- Profile 名称；
- Profile 目录；
- Profile 自己的 Patch 文件；
- 安装目录；
- 当前启动的 Bundle；
- 工作目录；
- Harness Home；
- 额外 Patch 文件。

这些信息之后会提供给插件树中的其他插件使用。

**第四步：真正进入 app-boot。**

继续往下看：

```ts
const ctx = await boot(
  NAME,
  rootConfig,
  readProfilePatches(NAME, profileContext, composed.profile),
  async (hostCtx) => {
    app.current = hostCtx

    hostCtx.provide('profileContext', profileContext)

    hostCtx.provide(
      DSH_LAUNCH_ENVIRONMENT_KEY,
      options.environment
    )

    await hostCtx.plugin(PluginPackages, {
      resolution: composed.resolution,
    })

    provideCmdline(hostCtx, {
      args: options.args,
      exit: code => void shutdown.shutdown(code),
      ready: appReady.service,
    })
  }
)
```

这里出现了一个非常重要的函数：

```ts
boot(...)
```

它来自：

```text
@deepseek-ai/dsh-app-boot
```

也就是：

```text
packages/boot/app-boot/
```

到这里，CLI 已经完成了启动参数解析、Profile 选择和配置准备。

接下来，真正的插件树启动工作交给了 app-boot。

### 2.2.6 第五步：追踪 composeProfile()，理解 Bundle 和 Patch 如何组合

回到：

```text
apps/cli/src/profile-boot.ts
```

找到：

```ts
async function composeProfile(...)
```

它的核心代码可以概括为：

```ts
const profile = resolvedProfile?.profile
  ?? prepareProfile(name, true, fromDefaultProfile)

const resolutionOptions = {
  installAnchor: resolvedProfile?.installAnchor ?? INSTALL_ANCHOR,
  profile,
}

const resolution = await createRuntimeResolution(resolutionOptions)

return {
  profile,
  resolution,
  overlayPaths: patchFiles.map(file => resolve(file)),
}
```

这里实际上有两项重要工作：

1. 准备 Profile 和 Bundle 配置；
2. 创建运行时模块解析关系。

先看第一项。

`prepareProfile()` 会调用：

```ts
loadProfile(
  NAME,
  name,
  INSTALL_ANCHOR,
  undefined,
  { userLayer }
)
```

`loadProfile()` 的实现位于：

```text
packages/boot/app-boot/src/profile.ts
```

继续追踪它。

首先找到：

```ts
export function loadProfile(...)
```

这个函数会：

- 根据 Profile 名称找到对应目录；
- 如果 Profile 尚未初始化，则根据官方模板初始化；
- 处理历史链接投影；
- 规范化官方 Profile；
- 调用 `loadProfileDirectory()` 读取 Profile 内容。

这里需要认识一个实际目录：

```text
$DSH_HOME/
    |
    +-- profiles/
          |
          +-- web/
                |
                +-- package.json
                |
                +-- cordis.patch.yml
                |
                +-- node_modules/
```

其中：

- `package.json` 保存 Profile 的 Bundle 声明和树外插件依赖；
- `cordis.patch.yml` 保存用户自己的 Patch；
- `node_modules` 保存 Profile 安装的依赖。

接下来，Profile 会根据自己的 Bundle 列表，逐个解析 Bundle。

Bundle 的 `package.json` 通过 `dsh.bundle.patch` 声明 Patch 文件位置。

例如：

```json
{
  "name": "@deepseek-ai/dsh-web-app",
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  }
}
```

这里的 `patch` 指向 Bundle 自己的 Patch 文件。

源码中对应的辅助函数：

```ts
bundlePatchFiles(bundle)
```

负责将声明的 Patch 路径转换为有序列表。

而：

```ts
bundlePatchPaths(packageDir, bundle)
```

负责把这些相对路径转换成绝对路径。

因此，Profile 并不是直接把所有插件目录扫描一遍。

它是根据 Bundle 的清单，找到每个 Bundle 声明的 Patch，再读取其中的插件配置。

**接下来重点理解 Patch 的叠加顺序。**

在当前源码版本中，配置层按照下面的顺序组合：

```text
空配置列表
    |
    v
Profile 中第一个 Bundle 的 Patch
    |
    v
Profile 中第二个 Bundle 的 Patch
    |
    v
后续 Bundle 的 Patch
    |
    v
Profile 自己的 cordis.patch.yml
    |
    v
Harness Home 下的 cordis.patch.yml
    |
    v
命令行 --patch 指定的 Overlay
    |
    v
最终配置
```

注意：

- Bundle 按照 Profile 中声明的顺序加载；
- 后面的 Patch 可以覆盖前面已经声明的配置；
- Profile 自己的 Patch 优先于 Bundle；
- Home Patch 优先于 Profile 自己的 Patch；
- 命令行 `--patch` Overlay 最后应用。

这一顺序非常重要。

例如，某个 Bundle 声明了插件：

```yaml
- id: example-plugin
  name: '@deepseek-ai/example-plugin'
```

用户可以在更高优先级的 Patch 中，通过相同的 `id` 修改这一行配置。

Patch 不只是简单地追加插件，它也可以修改已有配置。

源码位置：

```text
packages/boot/app-boot/src/profile.ts
apps/cli/src/profile-boot.ts
```

### 2.2.7 第六步：理解为什么还需要 Runtime Resolution

在前面的代码中，除了读取 Profile 和 Patch，还出现了：

```ts
createRuntimeResolution(...)
```

为什么要额外创建这个对象？

因为：

**配置中写了一个插件包名，并不意味着 Node.js 一定能够正确找到这个插件。**

例如，Patch 中存在：

```yaml
- id: example-plugin
  name: '@deepseek-ai/example-plugin'
```

Loader 后续需要导入这个插件。

这时 Node.js 必须知道：

```text
@deepseek-ai/example-plugin
```

究竟应该解析到磁盘上的哪个目录。

Harness 需要处理两类依赖来源：

1. Harness 安装目录中携带的包；
2. 当前 Profile 自己安装的包。

因此，`createRuntimeResolution()` 会根据安装目录、Profile 和依赖关系，建立运行时模块解析表。

可以把它理解为：

```text
插件配置
    |
    v
需要加载的包名
    |
    v
Runtime Resolution
    |
    v
确定包名对应的实际目录
    |
    v
Node.js 能够找到插件模块
```

这里有一个重要区别：

| 对象 | 负责什么 |
|---|---|
| Patch | 声明加载哪个插件，以及使用什么配置 |
| Runtime Resolution | 确定插件包名应该解析到哪个实际目录 |
| Cordis Loader | 根据配置加载插件并管理插件生命周期 |

不要把这三个职责混为一谈。

在 `profile-boot.ts` 中，运行时解析结果最终通过下面的代码提供给启动上下文：

```ts
await hostCtx.plugin(PluginPackages, {
  resolution: composed.resolution,
})
```

`PluginPackages` 是提供包解析能力的插件。

它在配置树中的插件模块需要被导入之前，就已经由 `prepare()` 安装到宿主上下文。

这样后续插件才能按照 Harness 的运行时包解析规则找到依赖。

源码位置：

```text
packages/boot/app-boot/src/profile.ts
packages/boot/app-boot/src/profile-resolution/
apps/cli/src/profile-boot.ts
```

本节只需要理解它的职责，不要求现在就读懂 Node.js ESM 和 CommonJS resolver 的内部实现。

这些内容放到后面的源码解读中深入研究。

### 2.2.8 第七步：进入 boot()，找到 Cordis Loader

现在进入真正的 Loader 启动阶段。

打开：

```text
packages/boot/app-boot/src/index.ts
```

找到：

```ts
export async function boot(
  binName: string,
  absoluteConfigPath: string,
  patches?: PatchOptions[],
  prepare?: (ctx: Context) => Promise<void> | void,
  bareModuleBaseUrl?: string,
): Promise<Context>
```

这个函数是 app-boot 的核心启动入口。

先看它创建上下文的代码：

```ts
const ctx = new Context()
```

这里创建了 Cordis 的根 Context。

接下来：

```ts
ctx.baseUrl = pathToFileURL(
  dirname(absoluteConfigPath)
).href + '/'
```

它设置配置文件的基础 URL。

然后：

```ts
await ctx.plugin(Loader)
```

这里才正式把 Cordis Loader 插件安装到根 Context。

注意：

`Loader` 是 Cordis 的插件加载器，不是 DeepSeek Harness 自己的某个业务插件。

它负责读取配置树、根据配置导入插件，并管理这些插件的生命周期。

接着：

```ts
await prepare?.(ctx)
```

这里执行前面由 `profile-boot.ts` 传入的准备函数。

也就是：

- 提供 `profileContext`；
- 提供启动环境快照；
- 安装 `PluginPackages`；
- 提供命令行参数和应用就绪服务。

为什么这些工作要放在 `mountRootInclude()` 之前？

因为后续配置树中的插件可能依赖这些服务。

所以必须先准备好宿主 Context，再开始挂载配置树。

接下来是最关键的一行：

```ts
await mountRootInclude(
  ctx,
  absoluteConfigPath,
  patches,
  bareModuleBaseUrl,
  binName
)
```

这一步将根配置文件及其 Patch 交给 Loader 的配置挂载流程。

在当前启动流程中，`absoluteConfigPath` 指向 Profile 目录下的：

```text
cordis.yml
```

而 `profile-boot.ts` 中定义的根配置是一个空列表：

```yaml
[]
```

为什么是空列表？

因为真正的插件配置已经通过 Patch 层叠加出来。

根配置只是 Loader 需要的配置树入口，不需要再把所有 Bundle 的插件手动写进去。

最终的插件配置由：

```ts
readProfilePatches(...)
```

提供给 `boot()`。

到这里，整个过程可以总结为：

```text
创建 Cordis Context
        |
        v
安装 Loader
        |
        v
执行 prepare()
        |
        +--> 提供 profileContext
        |
        +--> 提供启动环境
        |
        +--> 安装 PluginPackages
        |
        +--> 提供命令行服务
        |
        v
mountRootInclude()
        |
        v
Loader 读取根配置和 Patch
        |
        v
加载配置树中的插件
```

源码位置：

```text
packages/boot/app-boot/src/index.ts
apps/cli/src/profile-boot.ts
```

### 2.2.9 第八步：插件加载之后，启动流程还会做什么

很多人看到 Loader 开始加载插件，就认为启动过程已经结束。

实际上并不是这样。

继续阅读 `packages/boot/app-boot/src/index.ts` 中的 `boot()`。

在：

```ts
await mountRootInclude(...)
```

之后，还有：

```ts
await ctx.get('loader')?.await()
```

这一步等待 Loader 中的插件加载流程结算。

然后：

```ts
await auditStartupEntries(ctx, binName)
```

它负责检查启动时插件的激活状态。

为什么需要检查？

因为插件加载可能出现不同情况：

- 插件成功导入并初始化；
- 插件模块导入失败；
- 插件配置不符合要求；
- 插件依赖的服务不存在；
- 插件初始化过程中抛出异常。

Harness 不只是把插件交给 Loader 就结束了，还需要根据启动策略判断哪些失败可以容忍，哪些失败必须终止启动。

app-boot 的启动诊断会区分 Optional 和 Required 插件。

可以先这样理解：

| 插件类型 | 启动失败时的处理 |
|---|---|
| Optional | 可以记录警告，并允许其他插件继续运行 |
| Required | 如果必要插件无法激活，则启动失败并释放已经挂载的资源 |

因此，启动成功不只是指 CLI 没有报错。

还需要确认：

1. Loader 已经完成初始加载；
2. 必要插件已经成功激活；
3. 应用所需的服务已经可以使用；
4. 应用向启动器报告就绪。

回到 `profile-boot.ts`，还可以看到：

```ts
if (
  !signalShutdown.signal.aborted
  && ctx.fiber.state === FiberState.ACTIVE
  && ctx.get('loader') !== undefined
) {
  appReady.commit()
}
```

这段代码表明，应用只有在启动树仍然有效、Loader 仍然存在的情况下，才提交就绪信号。

因此，**插件加载完成、必要插件通过启动检查、应用就绪，是三个需要区分的阶段。**

### 2.2.10 实践一：找到服务启动后，在哪里配置大模型 API Key

前面我们已经追踪了 Harness 的启动过程：

```text
CLI → Profile → Bundle → Patch → Loader
```

现在换一个实际问题：

**Web 服务启动之后，如果用户想配置 DeepSeek 的 API Key，究竟是哪个插件负责接收、保存和使用这个密钥的？**

先启动 Web 服务：

```powershell
pnpm run start:web
```

打开 Web 页面，进入：

```text
Settings → Models
```

找到 DeepSeek 提供方的 API Key 输入框，输入自己的 API Key 并保存。

现在先不要急着修改源码。我们要从这个用户操作出发，反向追踪整个调用链。

#### 第一步：找到 Models 页面

打开：

```text
packages/client/ui-settings-models/
```

先阅读：

```text
packages/client/ui-settings-models/README.zh.md
```

找到其中的：

```markdown
### API 密钥
```

重点阅读这一部分，弄清楚：

1. API Key 输入框接收什么内容？
2. 用户输入的密钥会不会进入普通的 `settings.yaml`？
3. 页面通过哪个服务保存密钥？
4. 已保存的密钥会不会重新读取到浏览器输入框？
5. 页面如何显示已保存密钥的状态？

然后在该目录中搜索：

```powershell
rg -n "credentials\.set|ProviderEditor" packages/client/ui-settings-models
```

重点找到实际调用 `credentials.set()` 的客户端代码。

这里需要理解一个设计：

**Models 页面只负责接收密钥，并通过凭据服务写入密钥。它不会把真实密钥作为普通设置字段保存，也不会把已经保存的完整密钥重新发送给浏览器。**

#### 第二步：找到 credentials.set() 的实现

在整个项目中搜索：

```powershell
rg -n "abstract set\(ref|async set\(ref|credentials\.set" packages
```

重点阅读：

```text
packages/credentials/credentials/src/index.ts
```

找到：

```ts
abstract set(ref: CredentialRef, value: string): Promise<void>
```

这是凭据服务定义的写入接口。

它表达了两个参数：

- `ref`：密钥引用，告诉系统要保存哪一个凭据；
- `value`：实际的密钥内容。

例如：

```ts
const ref = credentialRef('DEEPSEEK_API_KEY')

await ctx.credentials.set(ref, apiKey)
```

这段代码只是展示凭据服务的调用方式，不代表 Models 页面完整的实际实现。

继续阅读 `CredentialProvider`，找到：

```ts
abstract resolve(ref: CredentialRef): Promise<ResolvedCredential | undefined>

abstract describe(ref: CredentialRef): Promise<CredentialInfo>

abstract set(ref: CredentialRef, value: string): Promise<void>

abstract unset(ref: CredentialRef): Promise<void>
```

先理解这四个接口：

| 接口 | 作用 |
|---|---|
| `set()` | 保存密钥 |
| `resolve()` | 解析并取得实际密钥 |
| `describe()` | 查询密钥是否已配置、来源和脱敏信息 |
| `unset()` | 删除已保存的密钥 |

特别注意：

`describe()` 不会返回完整密钥。它是给配置界面使用的安全查询接口。

#### 第三步：找到真正保存密钥的插件

现在打开：

```text
packages/bundle/base/cordis.patch.yml
```

搜索：

```yaml
- id: credentials
  name: '@deepseek-ai/dsh-credentials-local'
```

这就是当前基础 Bundle 中负责提供本地凭据存储的插件。

继续打开：

```text
packages/credentials/credentials-local/src/index.ts
```

找到：

```ts
export const CREDENTIALS_FILENAME = '.credentials.yaml'
```

再搜索：

```powershell
rg -n "async set|resolve\(|CREDENTIALS_FILENAME|renderRef" packages/credentials/credentials-local/src/index.ts
```

重点阅读：

- `set()` 如何写入凭据；
- `resolve()` 如何查找凭据；
- `renderRef()` 如何修改 YAML 文档；
- 文件发生变化后如何重新加载。

默认存储位置：

```text
$DSH_HOME/.credentials.yaml
```

如果没有设置 `DSH_HOME`，则使用默认 Harness Home。

文件结构示意：

```yaml
version: 1

refs:
  DEEPSEEK_API_KEY: sk-你的密钥
```

这里要理解三个角色：

| 对象 | 职责 |
|---|---|
| Models 页面 | 接收用户输入 |
| `dsh-credentials` | 定义统一的凭据读写接口 |
| `dsh-credentials-local` | 将凭据持久化到本地文件 |

**不要把 `.credentials.yaml` 当成普通设置文件。它是专门用于保存机密的凭据存储。**

#### 第四步：找到模型插件在哪里声明 API Key 引用

现在打开：

```text
packages/llm/llm-deepseek-api-key/src/config.ts
```

找到：

```ts
export const Config = z.object({
  ...deepSeekConfigFields,
  ...modelListingConfigFields,
  apiKeyEnv: z.string()
    .role('credential-ref')
    .default('DEEPSEEK_API_KEY')
    .volatile(),
})
```

这里最重要的是：

```ts
apiKeyEnv: z.string()
  .role('credential-ref')
  .default('DEEPSEEK_API_KEY')
```

它表示：

- 模型插件有一个 `apiKeyEnv` 配置字段；
- 默认引用名是 `DEEPSEEK_API_KEY`；
- 这个字段保存的是凭据引用，而不是实际的 API Key；
- `volatile()` 表示该字段不会作为普通持久化配置字段处理。

因此，模型插件默认寻找的是：

```text
DEEPSEEK_API_KEY
```

而不是在 `cordis.patch.yml` 中寻找一个叫 `apiKey` 的字段。

**这里一定要区分：**

```text
apiKeyEnv = DEEPSEEK_API_KEY
```

表示密钥引用的名字。

```text
DEEPSEEK_API_KEY = sk-真实密钥
```

才表示实际的密钥内容。

#### 第五步：追踪模型请求如何使用 API Key

继续打开：

```text
packages/llm/llm-deepseek-api-key/src/index.ts
```

找到：

```ts
ctx.effect(() => ctx.deepseekAuth.connect(
  'api-key',
  PROVIDER,
  () => {
    const connection = options()
    return {
      baseURL: connection.baseURL,
      reference: connection.apiKeyEnv,
      owner: settingsNs,
    }
  }
))
```

这里将 DeepSeek 官方模型路由的：

- API 地址；
- API Key 引用；
- 所属设置命名空间；

登记到 `deepseekAuth` 服务。

继续找到：

```ts
const resolveAuth = async (
  connection: ResolvedDeepSeekOptions,
  endpoint: string,
) => {
  return await ctx.deepseekAuth.authorize({
    consumer: settingsNs,
    accepts: ['api-key'],
    billing: 'api-key',
    destination: { url: endpoint },
    connections: {
      'api-key': {
        baseURL: connection.baseURL,
        reference: connection.apiKeyEnv,
      },
    },
    apiKeyHeaders: ['x-api-key'],
  })
}
```

这里是模型请求取得授权信息的关键位置。

继续打开：

```text
packages/account/deepseek-auth/README.zh.md
```

阅读其中的：

```markdown
### 每种凭据及其部署的来源
```

理解 `deepseekAuth` 如何根据模型路由找到对应的凭据，并生成请求所需的认证信息。

整个过程可以概括为：

```text
llm-deepseek-api-key
          |
          v
读取 apiKeyEnv
          |
          v
DEEPSEEK_API_KEY
          |
          v
deepseekAuth.authorize()
          |
          v
解析实际凭据
          |
          v
生成请求认证头
          |
          v
发送 DeepSeek API 请求
```

注意：

- `credentials` 负责凭据存储和解析；
- `deepseekAuth` 负责 DeepSeek 请求鉴权；
- `llm-deepseek-api-key` 负责 DeepSeek 官方模型路由。

这三个插件的职责不能混为一谈。

---

### 2.2.11 实践二：如果要给项目添加一个默认 API Key，应该怎么办？

现在提出一个二次开发问题：

假设我们正在开发自己的 DeepSeek Harness，希望开发者克隆项目后，只需要配置一次 API Key，就可以启动 Web 服务并使用 DeepSeek 模型。

应该怎样实现？

首先要明确：

**我们要增加的是一个默认密钥的配置来源，而不是把真实密钥写进模型插件。**

前面已经找到：

```ts
apiKeyEnv: z.string()
  .role('credential-ref')
  .default('DEEPSEEK_API_KEY')
  .volatile()
```

这意味着模型插件默认已经知道应该寻找哪个密钥引用。

所以，我们不需要先修改这个配置。

接下来要研究的是：如何让 `DEEPSEEK_API_KEY` 有实际的值。

#### 第一步：理解当前凭据解析优先级

打开：

```text
packages/credentials/credentials-local/src/index.ts
```

阅读文件顶部的注释。

当前凭据解析顺序是：

```text
$DSH_HOME/.credentials.yaml
          |
          v
启动进程环境变量
          |
          v
项目 .env
          |
          v
$DSH_HOME/.env
```

**越靠前，优先级越高。**

这意味着：

- 用户在 Models 页面保存过密钥，优先使用保存的密钥；
- 没有已保存密钥时，使用启动环境变量；
- 再没有时，读取项目 `.env`；
- 最后才读取 Harness Home 下的 `.env`。

这里有一个非常重要的设计：

**项目 `.env` 已经是 Harness 支持的凭据来源，我们不需要为了这个需求重新开发一个凭据插件。**

#### 第二步：尝试通过项目 .env 提供默认 API Key

先确认项目根目录是否存在：

```text
.env
```

如果不存在，就创建一个。

写入：

```dotenv
DEEPSEEK_API_KEY=sk-你的真实密钥
```

注意：

- 使用自己的真实 API Key；
- 不要添加多余的引号；
- 不要把真实密钥提交到 Git；
- 确保 `.env` 已经被 `.gitignore` 排除。

如果项目没有忽略 `.env`，就在 `.gitignore` 中添加：

```gitignore
.env
```

同时可以提供一个不包含真实密钥的模板：

```text
.env.example
```

内容：

```dotenv
DEEPSEEK_API_KEY=
```

这样其他开发者克隆项目后，可以复制模板：

```powershell
Copy-Item .env.example .env
```

再在自己的 `.env` 中填写密钥。

#### 第三步：启动服务，验证默认 API Key 是否生效

先检查是否存在更高优先级的已保存密钥。

如果之前在 Models 页面保存过 `DEEPSEEK_API_KEY`，它会覆盖项目 `.env` 中的值。

为了验证项目 `.env` 的效果，可以在测试环境中暂时移除对应的已保存凭据，或者使用一个全新的 `DSH_HOME`。

然后执行：

```powershell
pnpm run start:web
```

进入：

```text
Settings → Models
```

观察 DeepSeek API Key 的状态。

如果凭据已经从环境中检测到，页面会显示相应的环境认证状态。

注意：页面的环境检测只能说明检测到了对应引用的值，并不代表 API Key 一定有效，也不代表模型请求一定成功。

最后实际发送一次模型请求，确认密钥能够通过 DeepSeek API 的认证。

#### 第四步：思考为什么不需要修改模型插件

现在回顾：

```ts
apiKeyEnv: z.string()
  .role('credential-ref')
  .default('DEEPSEEK_API_KEY')
  .volatile()
```

以及：

```text
项目 .env
    |
    v
credentials-local
    |
    v
DEEPSEEK_API_KEY
    |
    v
deepseekAuth
    |
    v
模型请求
```

你会发现：

**模型插件不需要知道 API Key 是从 `.credentials.yaml`、启动环境还是项目 `.env` 中取得的。**

它只需要知道凭据引用。

这就是凭据服务抽象的价值：

以后即使我们改变密钥的存储方式，也不需要让每一个模型插件都跟着修改。

---

### 2.2.12 实践三：如果希望修改默认凭据引用，应该怎么办？

前面我们已经成功通过项目 `.env` 提供默认 API Key。

现在再做一个小练习：

假设我们不希望使用默认的：

```text
DEEPSEEK_API_KEY
```

而希望改成：

```text
MY_DEFAULT_DEEPSEEK_KEY
```

应该修改哪里？

打开：

```text
packages/llm/llm-deepseek-api-key/src/config.ts
```

找到：

```ts
apiKeyEnv: z.string()
  .role('credential-ref')
  .default('DEEPSEEK_API_KEY')
  .volatile()
```

将默认值修改为：

```ts
apiKeyEnv: z.string()
  .role('credential-ref')
  .default('MY_DEFAULT_DEEPSEEK_KEY')
  .volatile()
```

然后把项目 `.env` 修改为：

```dotenv
MY_DEFAULT_DEEPSEEK_KEY=sk-你的真实密钥
```

重新启动：

```powershell
pnpm run start:web
```

观察模型是否能够正常使用。

这里需要理解：

修改 `apiKeyEnv` 只是修改了模型插件默认查找的引用名。

它不会自动创建密钥，也不会自动把原来的密钥复制到新引用。

如果已有用户通过 Models 页面保存了旧引用对应的密钥，还需要考虑配置迁移问题。

因此，这种修改更适合你控制的项目默认配置，不适合在不了解用户已有配置的情况下直接修改已发布产品。

---

### 2.2.13 本节小结

完成本节后，你应该能够回答：

1. Models 页面在哪里接收 API Key？
2. 为什么 API Key 不应该直接写进普通的 `settings.yaml`？
3. `credentials.set()` 负责什么？
4. `.credentials.yaml` 是哪个插件负责管理的？
5. `apiKeyEnv` 保存的是密钥值还是密钥引用？
6. 模型插件通过哪个服务取得请求授权？
7. 当前凭据来源的优先级是什么？

本节最重要的结论：

**模型插件负责声明凭据引用，凭据服务负责保存和解析密钥，DeepSeek 鉴权服务负责为请求提供授权信息。**

理解这三个职责之后，你就不仅知道 API Key 在哪里配置，也知道以后应该从哪里入手扩展 Harness 的模型鉴权功能。

## 2.3 从“目录结构”转向“运行时装配”

刚打开一个大型仓库时，人们通常先看目录树，然后试图给每个目录贴标签。这只能得到静态地图，不能解释程序启动后到底运行了什么。

前面已经认识了 Profile、Bundle、Patch 和 Plugin，也通过实际案例了解了如何添加插件、修改配置。

但还有一个问题没有真正解决：

**这些配置在程序启动时是如何串联起来，最终变成真正运行的插件的？**

这就是本节要建立的核心认识。

### 2.3.1 从静态结构到运行时

可以把 Harness 的启动过程想成餐厅开门前的备餐：

- Profile 决定今天采用哪套菜单；
- Bundle 提供一组经过组织的菜品和服务；
- Patch 对配置进行追加或调整；
- Plugin 是最终需要加载和运行的功能单元。

但菜单本身不是厨房，配置中的一行也不等于插件已经运行。

要让插件真正参与程序运行，还需要经过后续的加载和初始化过程。

因此，不能只通过目录结构或配置文件判断程序最终运行了什么。

### 2.3.2 启动过程分为两个阶段

学习 Harness 启动链，需要把整个过程分成两个阶段。

**第一阶段：静态配置阶段**

这个阶段解决：

> 当前运行环境应该使用哪些配置和插件？

主要过程：

1. 解析启动参数和运行环境。
2. 选择当前使用的 Profile。
3. 根据 Profile 确定 Bundle 组合。
4. 收集并合并相关 Patch。
5. 形成最终插件配置。

这一阶段结束后，我们应该能够回答：

- 当前使用的是哪个 Profile？
- 使用了哪些 Bundle？
- Patch 合并后得到了什么配置？
- 最终配置中包含哪些插件？

**第二阶段：动态激活阶段**

这个阶段解决：

> 如何让最终配置中的插件真正运行起来？

主要过程：

1. Loader 获取最终插件配置。
2. 查找对应的插件模块。
3. 处理插件之间的依赖关系。
4. 加载插件并执行初始化。
5. 注册服务、事件监听器等运行时能力。

这一阶段结束后，插件才真正进入运行时。

### 2.3.3 把两个阶段连起来

将前面的过程放在一起，可以得到 Harness 启动装配的整体流程：

```text
                  程序启动
                     |
                     v
              解析启动参数和环境
                     |
                     v
                选择 Profile
                     |
                     v
              确定 Bundle 组合
                     |
                     v
              收集并合并 Patch
                     |
                     v
              形成最终插件配置
                     |
              静态配置阶段结束
                     |
                     v
                 Loader
                     |
                     v
               查找插件模块
                     |
                     v
               处理插件依赖
                     |
                     v
                加载插件
                     |
                     v
                初始化插件
                     |
                     v
               形成运行时
                     |
              动态激活阶段结束
```

这张图是后续阅读启动源码时的主线。

需要特别注意：

**插件进入最终配置，不代表插件已经成功运行。**

如果某个插件没有生效，问题可能发生在两个不同阶段：

- 静态配置阶段：插件根本没有进入最终配置。
- 动态激活阶段：插件已经进入配置，但加载、依赖处理或初始化失败。

只有区分这两个阶段，才能准确定位问题。

### 2.3.4 后续源码阅读应该追踪什么？

建立整体流程之后，下一步就是回到源码，逐步验证每个环节。

阅读时不要只记住函数名称，而要重点追踪：

1. 启动入口在哪里？
2. 谁负责选择 Profile？
3. Bundle 是在哪里解析的？
4. Patch 是如何收集和合并的？
5. 最终插件配置在哪里形成？
6. Loader 从哪里接收这份配置？
7. 插件如何被找到、加载和初始化？

每个环节都应该进一步追踪：

```text
调用者
   |
   v
当前函数
   |
   v
输入数据
   |
   v
处理过程
   |
   v
输出结果
   |
   v
下一个调用者
```

后续章节将沿着这条链路进入具体源码，逐个分析文件、函数、调用关系、数据流和实际运行效果。

### 2.3.5 本节小结

本节只需要记住两个核心概念：

- **静态配置阶段**：确定当前运行环境应该使用哪些插件和配置。
- **动态激活阶段**：通过 Loader 等运行时机制，将配置中的插件加载并初始化。

前面章节解决的是各个概念是什么、如何使用；从本节开始，我们要研究的是：

**这些概念在程序启动时如何协作，以及它们最终如何形成真正运行的 Harness。**
