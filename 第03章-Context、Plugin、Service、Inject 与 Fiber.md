- [第 3 章：Context、Plugin、Service、Inject 与 Fiber](#第-3-章contextpluginserviceinject-与-fiber)
  - [3.1 先认识仓库自带的官方 Cordis Tutorial](#31-先认识仓库自带的官方-cordis-tutorial)
    - [3.1.1 官方教程包含哪些内容？](#311-官方教程包含哪些内容)
    - [3.1.2 如何运行官方教程？](#312-如何运行官方教程)
    - [3.1.3 本教材与官方教程如何配合？](#313-本教材与官方教程如何配合)
  - [3.2 Context：插件运行时所处的上下文](#32-context插件运行时所处的上下文)
    - [3.2.1 Context 是插件访问运行环境的入口](#321-context-是插件访问运行环境的入口)
    - [3.2.2 Context 不只是一个普通对象](#322-context-不只是一个普通对象)
    - [3.2.3 Context 可以创建子上下文](#323-context-可以创建子上下文)
  - [3.3 Plugin：插件究竟是什么？](#33-plugin插件究竟是什么)
    - [3.3.1 函数插件](#331-函数插件)
    - [3.3.2 对象插件](#332-对象插件)
    - [3.3.3 类插件](#333-类插件)
    - [3.3.4 三种插件形式有什么区别？](#334-三种插件形式有什么区别)
    - [3.3.5 插件如何被挂载？](#335-插件如何被挂载)
  - [3.4 Service：插件如何向其他插件提供能力？](#34-service插件如何向其他插件提供能力)
    - [3.4.1 Service 是什么？](#341-service-是什么)
    - [3.4.2 使用 Service 类提供服务](#342-使用-service-类提供服务)
    - [3.4.3 为什么还需要 declare module？](#343-为什么还需要-declare-module)
    - [3.4.4 Service 注册会随插件卸载而撤销](#344-service-注册会随插件卸载而撤销)
  - [3.5 Inject：插件如何声明服务依赖？](#35-inject插件如何声明服务依赖)
    - [3.5.1 声明服务依赖](#351-声明服务依赖)
    - [3.5.2 inject 如何影响插件加载？](#352-inject-如何影响插件加载)
    - [3.5.3 什么是 PENDING？](#353-什么是-pending)
    - [3.5.4 Inject 不只是启动时检查一次](#354-inject-不只是启动时检查一次)
    - [3.5.5 可选依赖：ctx.get()](#355-可选依赖ctxget)
  - [3.6 Fiber：插件实例的运行时管理对象](#36-fiber插件实例的运行时管理对象)
    - [3.6.1 Fiber 不等于 Plugin](#361-fiber-不等于-plugin)
    - [3.6.2 Fiber 管理插件生命周期](#362-fiber-管理插件生命周期)
    - [3.6.3 如何获取 Fiber？](#363-如何获取-fiber)
    - [3.6.4 使用 fiber.dispose() 卸载插件](#364-使用-fiberdispose-卸载插件)
  - [3.7 Effect：为什么插件卸载时能够自动清理资源？](#37-effect为什么插件卸载时能够自动清理资源)
    - [3.7.1 使用 ctx.effect() 管理外部资源](#371-使用-ctxeffect-管理外部资源)
    - [3.7.2 哪些操作已经属于 Effect？](#372-哪些操作已经属于-effect)
    - [3.7.3 Effect 的清理顺序](#373-effect-的清理顺序)
  - [3.8 Context 的作用域：extend、isolate 与 intercept](#38-context-的作用域extendisolate-与-intercept)
    - [3.8.1 extend：扩展上下文](#381-extend扩展上下文)
    - [3.8.2 isolate：隔离服务](#382-isolate隔离服务)
    - [3.8.3 intercept：增加服务拦截元数据](#383-intercept增加服务拦截元数据)
    - [3.8.4 三者对比](#384-三者对比)
  - [3.9 动手实验：创建一个 Service，并让另一个插件依赖它](#39-动手实验创建一个-service并让另一个插件依赖它)
    - [3.9.1 准备实验目录](#391-准备实验目录)
    - [3.9.2 创建服务提供方 greeter.ts](#392-创建服务提供方-greeterts)
    - [3.9.3 创建服务消费者 consumer.ts](#393-创建服务消费者-consumerts)
    - [3.9.4 创建 cordis.yml](#394-创建-cordisyml)
    - [3.9.5 运行实验](#395-运行实验)
    - [3.9.6 实验一：交换插件顺序](#396-实验一交换插件顺序)
    - [3.9.7 实验二：移除服务提供方](#397-实验二移除服务提供方)
    - [3.9.8 实验三：观察依赖变化](#398-实验三观察依赖变化)
  - [3.10 本章知识总结：五个概念如何协同工作？](#310-本章知识总结五个概念如何协同工作)
  - [3.11 继续阅读官方 Cordis Tutorial](#311-继续阅读官方-cordis-tutorial)
    - [本章暂时不做的事情](#本章暂时不做的事情)


# 第 3 章：Context、Plugin、Service、Inject 与 Fiber

本章开始学习 DeepSeek Harness 底层使用的插件框架 Cordis。

在前面的章节中，我们已经接触过 Profile、Bundle、Patch 和 Plugin，知道 Harness 会通过插件组合不同的功能。

但还有几个问题需要弄清楚：

- 插件加载时，为什么会收到一个 `ctx` 对象？
- 为什么有些插件可以通过 `ctx.llm`、`ctx.tools` 直接访问其他插件提供的功能？
- 一个插件如何声明自己依赖另一个插件提供的服务？
- 为什么插件配置文件中的排列顺序不一定决定加载顺序？
- 当一个插件被卸载时，它注册的服务、事件监听器和定时器如何处理？
- `Fiber` 又是什么？它和 Plugin 有什么关系？

这些问题涉及 Cordis 的五个核心概念：

| 概念 | 解决的问题 |
|---|---|
| Context | 插件通过什么环境访问其他能力？ |
| Plugin | 如何向应用注册一项功能？ |
| Service | 插件如何向其他插件公开自己的能力？ |
| Inject | 插件如何声明自己需要哪些服务？ |
| Fiber | 如何管理一个插件实例的运行状态和生命周期？ |

这五个概念不是彼此独立的。

它们共同构成了 Cordis 的运行基础：

```text
Context
  │
  ├── Plugin
  │     │
  │     └── Fiber
  │           │
  │           ├── 生命周期
  │           ├── Effect
  │           └── Dispose
  │
  ├── Service
  │     │
  │     └── 通过 Context 向其他插件提供能力
  │
  └── Inject
        │
        └── 声明插件需要的 Service
```

本章先从插件开发者的角度理解这些概念，再通过实际实验观察它们之间的关系。

---

## 3.1 先认识仓库自带的官方 Cordis Tutorial

在开始学习之前，先介绍一个非常重要的学习资源：

**DeepSeek Harness 仓库已经提供了一套官方 Cordis Tutorial，其中包含多个可以直接运行的示例。**

教程位于：

```text
docs/cordis-tutorial/
```

这不是单纯的 API 文档，而是一套动手实践教程。每一章都会通过实际文件、插件代码和配置，带你了解 Cordis 的使用方式。

### 3.1.1 官方教程包含哪些内容？

官方教程提供以下章节：

| 官方教程文件 | 学习内容 | 对应知识 |
|---|---|---|
| `01-first-plugin.md` | 编写第一个插件 | Plugin |
| `02-lifecycle-and-effects.md` | 插件生命周期与 Effect | Fiber、Effect、Dispose |
| `03-services.md` | 注册和使用服务 | Service、Inject |
| `04-events.md` | 事件声明、监听和分发 | Events |
| `05-config.md` | 插件配置与 Schema 校验 | Config |
| `06-composition-and-hmr.md` | 插件组合与热更新 | Plugin、Fiber、HMR |

这些教程适合用来建立 Cordis 的基础认识。

其中：

- 第一章带你编写第一个插件。
- 第二章讲解插件生命周期以及资源清理。
- 第三章讲解 Service 和服务依赖。
- 第四章讲解事件系统。
- 第五章讲解插件配置。
- 第六章讲解插件组合和热更新。

### 3.1.2 如何运行官方教程？

官方教程提供了相应的环境准备和运行步骤。

首先，在仓库根目录安装依赖：

```bash
pnpm install
```

创建临时实验目录：

```bash
mkdir -p tmp/cordis-tutorial
cd tmp/cordis-tutorial
```

在完成官方教程要求的示例文件后，可以使用下面的命令启动 Cordis：

```bash
node --import tsx ../../vendor/cordis/bin.js
```

这条命令需要在：

```text
tmp/cordis-tutorial/
```

目录下执行。

它的作用是：

1. 使用 Node.js 启动 Cordis 启动器。
2. 通过 `tsx` 支持直接运行 TypeScript 文件。
3. 启动器创建 Cordis 运行环境。
4. 加载 Loader。
5. Loader 读取当前目录中的 `cordis.yml`。
6. 根据配置加载对应插件。

因此，运行教程时不需要自己编写一套启动框架。

只需要按照教程准备插件文件和 `cordis.yml` 即可。

> 注意：具体实验目录、文件名和命令参数应以对应官方教程中的说明为准。不要把不同教程的示例文件混在一起运行。

### 3.1.3 本教材与官方教程如何配合？

本教材不会把官方教程中的每一份示例原样抄写一遍。

两者的分工如下：

| 学习资源 | 主要任务 |
|---|---|
| 官方 Cordis Tutorial | 提供可运行的示例，带你实际操作 |
| 本章 | 帮助你理解 Context、Plugin、Service、Inject、Fiber 之间的关系 |
| 后续源码陪读篇 | 逐个分析 `vendor/cordis/src/` 中的真实实现 |

建议先阅读本章，再结合官方教程动手实践。

如果希望先通过代码建立直观认识，也可以直接从官方教程第一章开始。

---

## 3.2 Context：插件运行时所处的上下文

在 Cordis 中，插件并不是独立运行的。

当一个插件被加载时，Cordis 会向它传入一个 Context 对象，通常使用变量名 `ctx`。

例如：

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'hello'

export function apply(ctx: Context) {
  console.log('hello from my first plugin')
}
```

这里：

```ts
export function apply(ctx: Context)
```

就是插件的入口。

Cordis 加载插件时，会调用这个函数，并将 Context 传给它。

那么，Context 到底是什么？

### 3.2.1 Context 是插件访问运行环境的入口

可以先把 Context 理解为：

**插件运行时使用的能力访问入口。**

例如，在 Harness 中，你可能看到：

```ts
ctx.llm
ctx.tools
ctx.agents
ctx.logger
```

这些属性分别对应不同的服务。

插件不需要自己创建一个 LLM 服务，也不需要直接导入 LLM 服务提供方的内部实现。

只要相应的服务已经注册，并且当前插件具备访问权限，就可以通过 Context 使用它。

例如：

```ts
export const inject = ['llm']

export function apply(ctx: Context) {
  // 使用当前上下文中的 llm 服务
}
```

这里的 `ctx.llm` 不是普通的全局变量。

它是 Cordis 通过 Context 提供的服务访问入口。

### 3.2.2 Context 不只是一个普通对象

初学时，可能会认为 Context 就是下面这样的对象：

```ts
const ctx = {
  llm: {},
  tools: {},
  agents: {},
}
```

但 Cordis 的 Context 并不是简单地提前创建好所有属性。

它在运行时使用了 JavaScript Proxy。

因此，当插件访问：

```ts
ctx.llm
```

Cordis 可以介入这次属性访问，并根据当前插件的依赖关系、服务注册情况和上下文作用域，决定返回什么。

例如：

- 当前插件是否声明了 `inject: ['llm']`？
- `llm` 服务是否已经注册？
- 提供该服务的插件是否处于有效状态？
- 当前上下文是否处于独立的服务隔离范围？

如果一个插件没有声明必要的服务依赖，直接访问相应服务可能会得到异常。

这也是为什么不能把 Context 简单理解成一个随便读取属性的全局字典。

### 3.2.3 Context 可以创建子上下文

Cordis 允许从已有 Context 创建新的上下文。

例如：

```ts
const child = ctx.extend({
  label: 'child-context',
})
```

这里的 `child` 是一个新的上下文。

子上下文可以继承原 Context 的属性，同时拥有自己的附加属性。

注意：

```ts
const child = ctx.extend(...)
```

并不会简单地把新属性写回原来的 `ctx`。

Cordis 还提供两个容易混淆的方法：

```ts
ctx.isolate('shell')
```

```ts
ctx.intercept('some-service', {})
```

它们都与 Context 及服务作用域有关，但用途不同：

| 方法 | 作用 |
|---|---|
| `extend()` | 创建一个扩展上下文 |
| `isolate()` | 为指定服务创建独立的服务解析范围 |
| `intercept()` | 为指定服务增加拦截元数据 |

其中：

- `extend()` 主要解决上下文扩展问题。
- `isolate()` 主要解决同名服务需要使用不同实现的问题。
- `intercept()` 用于 Cordis 支持的服务拦截机制。

这三个方法的具体实现和作用域规则，留到后续源码陪读篇结合 `context.ts` 分析。

---

## 3.3 Plugin：插件究竟是什么？

理解 Context 之后，再看 Plugin。

在 Cordis 中，插件是向应用注册功能的基本单位。

官方教程介绍了不同形式的插件。

### 3.3.1 函数插件

最常见的插件形式是函数：

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'hello'

export function apply(ctx: Context) {
  console.log('hello from my first plugin')
}
```

这里有两个需要注意的地方。

第一：

```ts
export const name = 'hello'
```

`name` 是插件的名称。

第二：

```ts
export function apply(ctx: Context)
```

`apply` 是插件的实际执行入口。

当 Loader 加载这个模块时，会根据插件模块的导出内容识别插件，并在加载时调用相应入口。

### 3.3.2 对象插件

Cordis 也允许使用对象形式定义插件：

```ts
import type { Context } from '@deepseek-ai/cordis'

export const objectPlugin = {
  name: 'object-plugin',

  apply(ctx: Context) {
    console.log('object plugin loaded')
  },
}
```

对象插件需要提供 `apply` 方法。

### 3.3.3 类插件

第三种形式是类插件。

例如：

```ts
import { Service, type Context } from '@deepseek-ai/cordis'

export class GreeterService extends Service {
  constructor(ctx: Context) {
    super(ctx, 'greeter')
  }

  greet(who: string) {
    return `Hello, ${who}!`
  }
}
```

这里的 `GreeterService` 继承了 Cordis 的 `Service`。

它不仅是一个普通的 TypeScript 类，也可以作为 Cordis 插件被挂载。

后面讲解 Service 时，我们会进一步分析这种形式。

### 3.3.4 三种插件形式有什么区别？

| 插件形式 | 主要用途 | 入口形式 |
|---|---|---|
| 函数插件 | 执行普通插件逻辑 | 函数本身 |
| 对象插件 | 将插件逻辑组织在对象中 | `apply` 方法 |
| 类插件 | 通常用于提供 Service | 构造函数 |

对于刚开始编写插件的人，建议优先使用函数插件。

只有在需要向其他插件公开服务时，再考虑使用 Service 类。

### 3.3.5 插件如何被挂载？

除了通过 `cordis.yml` 加载，代码中也可以主动挂载插件：

```ts
const fiber = ctx.plugin(helloPlugin)
```

这里：

```ts
ctx.plugin()
```

就是 Cordis 提供的插件挂载接口。

调用之后会返回一个 Fiber。

先记住：

**Plugin 描述要加载什么功能，Fiber 则负责管理这个插件实例的运行状态和生命周期。**

---

## 3.4 Service：插件如何向其他插件提供能力？

前面已经知道，插件是 Cordis 中组织功能的基本单位。

但插件之间如何共享功能？

例如：

- 一个插件负责提供 LLM 能力。
- 一个插件负责提供工具注册能力。
- 另一个插件希望使用这些能力。

如果每个插件都直接导入其他插件的内部实现，那么插件之间就会产生紧密依赖。

Cordis 使用 Service 解决这个问题。

### 3.4.1 Service 是什么？

Service 可以理解为：

**一个插件向其他插件公开的具名能力。**

例如，某个插件提供了名为 `greeter` 的服务：

```ts
ctx.greeter
```

其他插件就可以通过这个名称访问它。

消费者不需要知道服务提供方内部是如何实现的。

只需要知道服务名称和服务提供的接口即可。

### 3.4.2 使用 Service 类提供服务

创建：

```text
greeter.ts
```

代码：

```ts
import { Service, type Context } from '@deepseek-ai/cordis'

declare module '@deepseek-ai/cordis' {
  interface Context {
    greeter: GreeterService
  }
}

export class GreeterService extends Service {
  constructor(ctx: Context) {
    super(ctx, 'greeter')
  }

  greet(who: string) {
    return `Hello, ${who}!`
  }
}

export const name = 'greeter'

export function apply(ctx: Context) {
  ctx.plugin(GreeterService)
}
```

这段代码看起来比较长，但核心只有几个部分。

**第一部分：继承 Service。**

```ts
export class GreeterService extends Service
```

说明这个类是一个 Cordis Service。

**第二部分：注册服务名称。**

```ts
constructor(ctx: Context) {
  super(ctx, 'greeter')
}
```

这里非常重要。

`super(ctx, 'greeter')` 会把当前 Service 实例注册到名为 `greeter` 的服务位置。

之后，其他符合依赖要求的插件就可以通过：

```ts
ctx.greeter
```

访问它。

**第三部分：实现服务功能。**

```ts
greet(who: string) {
  return `Hello, ${who}!`
}
```

这是我们自己定义的业务方法。

Cordis 并不要求 Service 必须提供 `greet()` 这样的固定方法。

服务具体能做什么，由插件开发者决定。

**第四部分：挂载 Service 类。**

```ts
export function apply(ctx: Context) {
  ctx.plugin(GreeterService)
}
```

这里把 `GreeterService` 作为类插件挂载。

Cordis 会创建该类的实例，并执行它的构造过程。

构造函数中的：

```ts
super(ctx, 'greeter')
```

负责注册服务。

### 3.4.3 为什么还需要 declare module？

代码中还有一段：

```ts
declare module '@deepseek-ai/cordis' {
  interface Context {
    greeter: GreeterService
  }
}
```

它看起来像是在修改 Cordis 的 Context。

实际上，这是 TypeScript 的声明合并机制。

它告诉 TypeScript：

Context 除了原本定义的属性，还可能存在：

```ts
ctx.greeter
```

并且这个属性的类型是：

```ts
GreeterService
```

这里要区分两个概念：

| 内容 | 作用 |
|---|---|
| `super(ctx, 'greeter')` | 在运行时注册服务 |
| `declare module` | 在编译时补充类型信息 |

`declare module` 不会在运行时创建服务。

即使删掉声明合并，服务仍然可以注册和运行，只是 TypeScript 无法正确识别 `ctx.greeter` 的类型。

### 3.4.4 Service 注册会随插件卸载而撤销

Service 并不是永久注册的全局对象。

当提供服务的插件卸载时，Cordis 会撤销对应的服务注册。

例如：

```text
GreeterService 插件加载
        │
        ▼
注册 greeter 服务
        │
        ▼
其他插件可以使用 ctx.greeter
        │
        ▼
GreeterService 插件卸载
        │
        ▼
撤销 greeter 服务
```

因此，Service 的注册和插件生命周期是关联的。

这也是为什么 Service 不应该简单理解为一个普通的全局单例。

---

## 3.5 Inject：插件如何声明服务依赖？

现在已经有了 Service。

接下来需要解决另一个问题：

如果一个插件需要使用其他插件提供的服务，Cordis 怎么知道它依赖什么？

答案就是：

`inject`

### 3.5.1 声明服务依赖

假设我们已经有了前面的 `greeter` 服务。

现在创建另一个插件：

```text
consumer.ts
```

代码：

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'consumer'

export const inject = ['greeter']

export function apply(ctx: Context) {
  console.log(ctx.greeter.greet('world'))
}
```

重点看：

```ts
export const inject = ['greeter']
```

它告诉 Cordis：

**这个插件需要 `greeter` 服务才能正常运行。**

因此，Cordis 不会在服务尚未就绪时直接执行 `apply()`。

只有当依赖满足时，插件才会进入加载过程。

### 3.5.2 inject 如何影响插件加载？

假设配置文件如下：

```yaml
- name: './greeter.ts'
- name: './consumer.ts'
```

加载关系可以理解为：

```text
greeter 插件
    │
    ▼
注册 greeter 服务
    │
    ▼
consumer 的依赖满足
    │
    ▼
执行 consumer.apply()
```

但这里还有一个非常重要的特点：

**Cordis 不保证按照 YAML 文件中的排列顺序加载插件。**

例如，把配置改成：

```yaml
- name: './consumer.ts'
- name: './greeter.ts'
```

消费者仍然可以在依赖满足后启动。

因为 Cordis 根据服务依赖关系决定插件什么时候能够启动，而不是简单按照配置文件中的上下顺序执行。

### 3.5.3 什么是 PENDING？

如果插件声明了：

```ts
export const inject = ['greeter']
```

但是整个应用没有任何插件提供 `greeter` 服务，会发生什么？

消费者不会正常执行。

它会处于：

```text
PENDING
```

也就是等待依赖满足的状态。

这不是普通的运行错误。

Cordis 允许插件先被声明，等到所需服务出现后再启动。

例如：

```text
consumer 插件被声明
        │
        ▼
检查 inject
        │
        ▼
greeter 服务不存在
        │
        ▼
PENDING
        │
        │ greeter 服务注册
        ▼
依赖满足
        │
        ▼
consumer 开始加载
        │
        ▼
ACTIVE
```

这也是为什么有时候插件明明已经写进配置文件，却没有任何输出。

不能只检查配置文件，还需要考虑它的依赖是否满足。

### 3.5.4 Inject 不只是启动时检查一次

Inject 还有一个重要特点：

**它会持续跟踪依赖服务的变化。**

假设应用已经运行：

```text
greeter 服务正常
        │
        ▼
consumer 正常运行
```

如果提供 `greeter` 的插件被卸载：

```text
greeter 服务消失
        │
        ▼
consumer 的依赖不再满足
        │
        ▼
consumer 需要卸载
```

之后，如果新的插件重新提供 `greeter`：

```text
新的 greeter 服务注册
        │
        ▼
consumer 的依赖重新满足
        │
        ▼
consumer 可以重新加载
```

这样可以避免消费者继续使用已经失效的服务引用。

所以，Inject 不只是告诉 Cordis：

“我启动之前需要这个服务。”

它还告诉 Cordis：

“运行期间，我的正常工作依赖这个服务。”

### 3.5.5 可选依赖：ctx.get()

并不是所有服务都属于必须依赖。

有时候插件即使没有某项服务，也应该能够正常运行。

这时可以使用：

```ts
ctx.get('greeter')
```

例如：

```ts
export function apply(ctx: Context) {
  const greeter = ctx.get('greeter')

  if (greeter) {
    console.log(greeter.greet('world'))
  } else {
    console.log('greeter service unavailable')
  }
}
```

这里没有声明：

```ts
export const inject = ['greeter']
```

因此插件不需要等待 `greeter` 服务才能启动。

可以把两种方式理解为：

| 方式 | 作用 |
|---|---|
| `inject: ['greeter']` | 必须等 greeter 服务可用，插件才能运行 |
| `ctx.get('greeter')` | 不要求服务一定存在，使用时自行判断 |

初学阶段先记住：

**必须依赖使用 inject，可选依赖使用 ctx.get()。**

---

## 3.6 Fiber：插件实例的运行时管理对象

现在已经理解了：

- Plugin 是插件本身。
- Service 是插件提供的能力。
- Inject 是插件声明的服务依赖。

那么 Fiber 又是什么？

### 3.6.1 Fiber 不等于 Plugin

先看一段代码：

```ts
const fiber = ctx.plugin(helloPlugin)
```

这里：

```ts
helloPlugin
```

是插件定义。

而：

```ts
fiber
```

是 Cordis 为这次插件挂载创建的运行时管理对象。

可以这样理解：

| 概念 | 含义 |
|---|---|
| Plugin | 插件的定义 |
| Fiber | 某次插件运行实例的管理对象 |

同一个插件定义可以在不同上下文中被挂载，因此不能简单认为一个 Plugin 永远只对应一个 Fiber。

Fiber 负责管理具体实例的运行状态、依赖关系、Effect 和清理过程。

### 3.6.2 Fiber 管理插件生命周期

Fiber 会记录插件实例的运行状态。

学习时可以重点关注以下状态：

| 状态 | 含义 |
|---|---|
| PENDING | 正在等待所需服务 |
| LOADING | 插件正在执行加载逻辑 |
| ACTIVE | 插件已经成功加载 |
| FAILED | 插件加载或配置校验失败 |
| UNLOADING | 插件正在执行卸载和清理 |
| DISPOSED | 插件实例已经释放 |

可以用下面的状态图理解主要关系：

```text
              依赖尚未满足
                    │
                    ▼
                 PENDING
                    │
                    │ 依赖满足
                    ▼
                 LOADING
                    │
                    ├──────── 加载失败 ────────┐
                    │                          │
                    ▼                          ▼
                  ACTIVE                    FAILED
                    │
                    │ 卸载
                    ▼
                UNLOADING
                    │
                    ▼
                 DISPOSED
```

注意：

这里展示的是帮助理解的主要状态关系，不代表所有运行场景都只能按照一条固定路径变化。

例如，处于 ACTIVE 的插件如果因为依赖服务消失而卸载，之后在服务恢复时，Cordis 可以重新加载该插件。

### 3.6.3 如何获取 Fiber？

通过：

```ts
const fiber = ctx.plugin(helloPlugin)
```

就可以获取 Fiber。

例如：

```ts
export function apply(ctx: Context) {
  const fiber = ctx.plugin(helloPlugin)

  console.log(fiber.state)
}
```

Fiber 提供状态信息和生命周期操作。

需要注意：

插件的加载过程可能是异步的，因此不能简单地认为调用 `ctx.plugin()` 后，插件一定已经完成加载。

如果需要等待加载过程，可以使用：

```ts
await ctx.plugin(helloPlugin)
```

Cordis 返回的 Fiber 支持 Promise-like 行为，可以等待加载过程结束。

### 3.6.4 使用 fiber.dispose() 卸载插件

Fiber 还提供：

```ts
await fiber.dispose()
```

用于释放插件实例。

它会处理该插件实例关联的资源和子插件，并等待相关清理工作完成。

例如：

```ts
export function apply(ctx: Context) {
  const fiber = ctx.plugin(helloPlugin)

  setTimeout(async () => {
    await fiber.dispose()
    console.log('plugin disposed')
  }, 3000)
}
```

这里：

1. 挂载 `helloPlugin`。
2. 保存返回的 Fiber。
3. 三秒后调用 `dispose()`。
4. 等待插件清理完成。
5. 输出释放完成的信息。

这就是 Fiber 的一个重要用途：

**管理某个插件实例从加载到卸载的过程。**

---

## 3.7 Effect：为什么插件卸载时能够自动清理资源？

前面已经提到，Service 注册会随着插件卸载而撤销。

事件监听器也是如此。

但如果插件自己创建了一个定时器呢？

例如：

```ts
export function apply(ctx: Context) {
  setInterval(() => {
    console.log('tick')
  }, 1000)
}
```

这个定时器并没有交给 Cordis 管理。

即使插件被卸载，定时器也可能继续运行。

这显然不是我们希望看到的结果。

### 3.7.1 使用 ctx.effect() 管理外部资源

Cordis 提供了：

```ts
ctx.effect()
```

用来管理那些没有被 Cordis 内置注册 API 自动管理的资源。

例如：

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'timer-demo'

export function apply(ctx: Context) {
  ctx.effect(() => {
    const timer = setInterval(() => {
      console.log('tick')
    }, 1000)

    return () => {
      clearInterval(timer)
      console.log('timer cleaned up')
    }
  })
}
```

这里分成两部分。

第一部分：

```ts
ctx.effect(() => {
  const timer = setInterval(...)
```

表示创建一个由当前插件生命周期管理的 Effect。

第二部分：

```ts
return () => {
  clearInterval(timer)
}
```

返回一个 disposer，也就是资源释放函数。

当插件卸载时，Cordis 会自动执行这个函数。

### 3.7.2 哪些操作已经属于 Effect？

很多 Cordis API 本身就会自动管理生命周期。

例如：

```ts
ctx.on('some-event', listener)
```

注册事件监听器。

```ts
ctx.plugin(childPlugin)
```

挂载子插件。

```ts
super(ctx, 'greeter')
```

注册 Service。

这些操作都会关联到当前插件的生命周期。

因此，插件卸载时，Cordis 会自动撤销相应的注册。

对于普通定时器、文件监听器、外部连接等 Cordis 不直接管理的资源，则需要使用 `ctx.effect()`。

### 3.7.3 Effect 的清理顺序

理解 Effect 时，还需要注意清理顺序。

如果一个插件注册了多个 Effect，清理时需要考虑它们之间的依赖关系。

例如：

```ts
ctx.effect(() => {
  console.log('effect A')
  return () => console.log('dispose A')
})

ctx.effect(() => {
  console.log('effect B')
  return () => console.log('dispose B')
})
```

清理函数的执行顺序和注册顺序有关。

如果多个异步 disposer 之间存在严格的先后依赖，应将相关清理步骤放在同一个 disposer 中，并在其中使用 `await` 控制顺序。

不要仅凭多个 Effect 的注册顺序，就假设多个异步清理过程一定会串行完成。

---

## 3.8 Context 的作用域：extend、isolate 与 intercept

前面已经介绍过 Context 可以创建子上下文。

这里进一步认识三个容易混淆的方法。

### 3.8.1 extend：扩展上下文

```ts
const child = ctx.extend({
  label: 'child',
})
```

它会创建一个新的 Context。

子上下文可以继承父上下文的属性，同时拥有自己的附加属性。

例如：

```text
父 Context
  │
  ├── logger
  ├── llm
  └── tools
        │
        ▼
    child Context
        │
        ├── 继承 logger
        ├── 继承 llm
        ├── 继承 tools
        └── 新增 label
```

这里展示的是概念关系。

实际服务能否访问，还需要满足 Cordis 的服务依赖和作用域规则。

### 3.8.2 isolate：隔离服务

假设两个插件组都需要名为：

```text
shell
```

的服务。

但它们希望使用不同的实现。

例如：

```text
应用
 │
 ├── 组 A
 │    └── shell A
 │
 └── 组 B
      └── shell B
```

如果所有插件都使用同一个全局 `shell` 服务，就很难让两个组分别使用不同的实现。

这时可以使用：

```ts
ctx.isolate('shell')
```

它会为指定服务创建一个独立的服务解析范围。

之后在该范围内注册的 `shell` 服务，与父级范围中的 `shell` 服务相互隔离。

需要注意：

`isolate()` 隔离的是指定服务名称的解析范围，不是复制整个应用，也不是自动创建新的服务提供方。

新的隔离范围仍然需要有对应的服务提供方。

### 3.8.3 intercept：增加服务拦截元数据

```ts
ctx.intercept('some-service', {
  // 服务支持的拦截配置
})
```

它为指定服务增加拦截元数据。

这类元数据最终如何使用，由具体 Service 的实现决定。

因此，不能认为所有 Service 都支持任意拦截配置。

`intercept()` 主要用于 Cordis 的服务拦截机制，不应当与普通插件的 `config` 配置混为一谈。

### 3.8.4 三者对比

| 方法 | 主要解决的问题 |
|---|---|
| `extend()` | 创建一个可以附加属性的子上下文 |
| `isolate()` | 让指定服务在不同作用域中使用不同实现 |
| `intercept()` | 为指定服务增加拦截元数据 |

这三个 API 的内部实现都涉及 Context 的继承和作用域处理。

后续源码陪读篇会结合 `vendor/cordis/src/` 中的 Context 实现逐个分析。

---

## 3.9 动手实验：创建一个 Service，并让另一个插件依赖它

前面介绍了 Context、Plugin、Service、Inject 和 Fiber。

现在通过一个完整实验，把它们串联起来。

这次不需要自己设计复杂插件，直接参考官方教程中的 Greeter 示例。

### 3.9.1 准备实验目录

在仓库根目录执行：

```bash
mkdir -p tmp/cordis-tutorial
cd tmp/cordis-tutorial
```

本实验会创建三个文件：

```text
tmp/cordis-tutorial/
│
├── greeter.ts
├── consumer.ts
└── cordis.yml
```

### 3.9.2 创建服务提供方 greeter.ts

```ts
import { Service, type Context } from '@deepseek-ai/cordis'

declare module '@deepseek-ai/cordis' {
  interface Context {
    greeter: GreeterService
  }
}

export class GreeterService extends Service {
  constructor(ctx: Context) {
    super(ctx, 'greeter')
  }

  greet(who: string) {
    return `Hello, ${who}!`
  }
}

export const name = 'greeter'

export function apply(ctx: Context) {
  ctx.plugin(GreeterService)
}
```

这个插件提供了一个名为 `greeter` 的 Service。

它只有一个方法：

```ts
greet(who: string)
```

调用后返回一段问候文本。

### 3.9.3 创建服务消费者 consumer.ts

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'consumer'

export const inject = ['greeter']

export function apply(ctx: Context) {
  console.log(ctx.greeter.greet('world'))
}
```

这个插件声明：

```ts
export const inject = ['greeter']
```

表示它依赖 `greeter` 服务。

在 `apply()` 中，通过：

```ts
ctx.greeter.greet('world')
```

调用服务提供方的方法。

### 3.9.4 创建 cordis.yml

```yaml
- name: './greeter.ts'

- name: './consumer.ts'
```

这里配置了两个插件：

- `greeter.ts`：提供服务。
- `consumer.ts`：使用服务。

### 3.9.5 运行实验

确保当前目录是：

```text
tmp/cordis-tutorial/
```

然后按照官方教程提供的方式启动：

```bash
node --import tsx ../../vendor/cordis/bin.js
```

预期可以看到：

```text
Hello, world!
```

如果能够看到这段输出，说明两个插件已经完成了服务注册和依赖调用。

### 3.9.6 实验一：交换插件顺序

将 `cordis.yml` 改成：

```yaml
- name: './consumer.ts'

- name: './greeter.ts'
```

重新运行：

```bash
node --import tsx ../../vendor/cordis/bin.js
```

如果当前示例的加载方式和依赖配置符合预期，消费者会在 `greeter` 服务可用后启动。

这说明：

**插件配置文件的排列顺序，不一定等于插件实际的加载顺序。**

### 3.9.7 实验二：移除服务提供方

现在把 `cordis.yml` 改成：

```yaml
- name: './consumer.ts'
```

重新运行。

这次不会看到：

```text
Hello, world!
```

原因是：

```ts
export const inject = ['greeter']
```

要求 `greeter` 服务必须存在。

但当前配置中没有任何插件提供它。

因此消费者无法满足启动条件。

### 3.9.8 实验三：观察依赖变化

再恢复原来的配置：

```yaml
- name: './greeter.ts'

- name: './consumer.ts'
```

现在可以观察：

```text
greeter 服务注册
        │
        ▼
consumer 依赖满足
        │
        ▼
consumer 执行
```

官方教程的生命周期与服务章节还提供了更完整的实验：

- 手动卸载子插件。
- 观察 Effect 清理。
- 观察服务消失后消费者的状态变化。
- 观察服务重新注册后的依赖变化。

建议继续阅读：

```text
docs/cordis-tutorial/02-lifecycle-and-effects.md
docs/cordis-tutorial/03-services.md
```

---

## 3.10 本章知识总结：五个概念如何协同工作？

到这里，已经认识了 Cordis 的五个核心概念。

再把它们放在一起看一次。

假设我们要实现一个提供问候能力的插件。

**第一步：编写 Plugin。**

```ts
export function apply(ctx: Context) {
  // 插件入口
}
```

Plugin 是功能的组织单位。

**第二步：通过 Context 访问运行环境。**

```ts
ctx
```

Context 是插件访问服务、事件和其他运行时能力的入口。

**第三步：提供 Service。**

```ts
super(ctx, 'greeter')
```

Service 是插件向其他插件公开的具名能力。

**第四步：声明 Inject。**

```ts
export const inject = ['greeter']
```

Inject 告诉 Cordis 当前插件需要哪些服务。

**第五步：通过 Fiber 管理运行实例。**

```ts
const fiber = ctx.plugin(helloPlugin)
```

Fiber 负责管理插件实例的加载、运行、依赖变化和卸载。

它们之间的关系可以概括为：

```text
                  Cordis Application
                         │
                         ▼
                      Context
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
          Plugin A                Plugin B
             │                       │
             │                       │
             ▼                       ▼
       提供 greeter              inject greeter
             │                       │
             ▼                       │
       注册 Service ─────────────────┘
                                     │
                                     ▼
                               依赖满足后加载
                                     │
                                     ▼
                                   Fiber
                                     │
                                     ▼
                              管理插件生命周期
```

记住这五句话：

1. **Context**：插件运行时访问能力的入口。
2. **Plugin**：组织和注册功能的基本单位。
3. **Service**：插件向其他插件公开的具名能力。
4. **Inject**：声明插件需要哪些服务。
5. **Fiber**：管理插件实例运行状态和生命周期的对象。

---

## 3.11 继续阅读官方 Cordis Tutorial

本章重点讲解了 Context、Plugin、Service、Inject 和 Fiber。

官方教程还提供了事件、配置、插件组合和热更新等内容。

建议按照下面的顺序继续实践：

| 顺序 | 官方教程 | 重点观察 |
|---|---|---|
| 1 | `01-first-plugin.md` | 插件如何被 Loader 加载 |
| 2 | `02-lifecycle-and-effects.md` | 插件卸载时如何清理资源 |
| 3 | `03-services.md` | Service 和 Inject 如何协作 |
| 4 | `04-events.md` | 插件之间如何通过事件通信 |
| 5 | `05-config.md` | 配置如何经过 Schema 校验 |
| 6 | `06-composition-and-hmr.md` | 插件组合、热更新和依赖诊断 |

特别建议认真完成官方教程的第四章和第六章。

第四章介绍的 Events 是 Cordis 插件之间另一种重要的通信方式。

第六章则把前面学习的插件、服务、依赖和生命周期组合起来，进一步展示 HMR（热模块替换）的工作方式。

### 本章暂时不做的事情

本章没有逐行分析以下目录：

```text
vendor/cordis/src/
```

这是有意安排的。

现在的重点是建立正确的概念认识，知道每个 API 的用途，以及它们在插件开发中如何协作。

等进入源码陪读篇，我们再从真实源码出发，逐个分析：

| 源码模块 | 后续重点 |
|---|---|
| Context 相关实现 | Context、Proxy、extend、isolate、intercept |
| Plugin 注册相关实现 | Plugin 注册、插件形态解析、Inject 依赖声明 |
| Service 相关实现 | Service 如何注册到 Context |
| 服务解析相关实现 | 服务注册、查找、访问和依赖关系 |
| Fiber 相关实现 | Fiber 状态、Effect、加载和卸载 |
| Events 相关实现 | 事件注册、监听和分发 |

届时不再只是介绍某个 API 的用途，而是沿着真实源码回答：

1. 这个对象在哪里创建？
2. 这个方法由哪个类实现？
3. 谁调用了它？
4. 调用之后修改了什么状态？
5. 它与其他模块之间是什么关系？
6. 仓库中的测试如何验证这些行为？

**本章负责建立概念，官方 Tutorial 负责动手实践。**