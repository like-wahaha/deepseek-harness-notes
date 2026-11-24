- [第 5 章：Tool Registry 与执行边界](#第-5-章tool-registry-与执行边界)
    - [5.1.1 为什么不能相信模型生成的工具名称？](#511-为什么不能相信模型生成的工具名称)
    - [5.1.2 为什么工具定义和工具执行必须分开？](#512-为什么工具定义和工具执行必须分开)
  - [5.2 Tool Registry：工具注册中心](#52-tool-registry工具注册中心)
    - [5.2.1 Registry 的职责](#521-registry-的职责)
    - [5.2.2 找到 ToolsService](#522-找到-toolsservice)
    - [5.2.3 register()：注册工具](#523-register注册工具)
    - [5.2.4 Scope：同一个工具注册表，为什么不同 Agent 实例看到的工具不同？](#524-scope同一个工具注册表为什么不同-agent-实例看到的工具不同)
    - [5.2.5 restrict()：限制工具可见性](#525-restrict限制工具可见性)
    - [5.2.6 resolve()：解析当前步骤可用的工具](#526-resolve解析当前步骤可用的工具)
    - [5.2.7 schemas()：Registry 如何告诉模型有哪些工具？](#527-schemasregistry-如何告诉模型有哪些工具)
  - [5.3 ToolDefinition：一个工具到底包含什么？](#53-tooldefinition一个工具到底包含什么)
    - [5.3.1 parameters：输入参数定义](#531-parameters输入参数定义)
    - [5.3.2 execute：真正的执行逻辑](#532-execute真正的执行逻辑)
    - [5.3.3 output.schema：输出也必须有约束](#533-outputschema输出也必须有约束)
    - [5.3.4 render：把结构化结果转换成模型内容](#534-render把结构化结果转换成模型内容)
  - [5.4 defineTool：为什么不直接手写 ToolDefinition？](#54-definetool为什么不直接手写-tooldefinition)
    - [5.4.1 参数类型推导](#541-参数类型推导)
    - [5.4.2 运行时参数校验](#542-运行时参数校验)
    - [5.4.3 输出类型推导](#543-输出类型推导)
    - [5.4.4 不要混淆三个操作](#544-不要混淆三个操作)
  - [5.5 参数校验与权限检查是两件事](#55-参数校验与权限检查是两件事)
    - [5.5.1 参数校验解决什么问题？](#551-参数校验解决什么问题)
    - [5.5.2 权限检查解决什么问题？](#552-权限检查解决什么问题)
    - [5.5.3 不要只依赖工具参数 Schema](#553-不要只依赖工具参数-schema)
    - [5.5.4 文件系统工具还要考虑什么？](#554-文件系统工具还要考虑什么)
    - [5.5.5 网络工具还要考虑什么？](#555-网络工具还要考虑什么)
  - [5.6 从 Tool Registry 进入 ToolExecutionPipeline](#56-从-tool-registry-进入-toolexecutionpipeline)
    - [5.6.1 为什么要有 ToolExecution？](#561-为什么要有-toolexecution)
    - [5.6.2 参数为什么要做快照？](#562-参数为什么要做快照)
    - [5.6.3 为什么不能在 pre-execute 中修改 arguments？](#563-为什么不能在-pre-execute-中修改-arguments)
  - [5.7 五个核心工具事件：真正的执行边界](#57-五个核心工具事件真正的执行边界)
    - [5.7.1 tools/pre-execute：执行前决策](#571-toolspre-execute执行前决策)
    - [一个重要区别](#一个重要区别)
    - [5.7.2 tools/execute：环绕实际工具执行](#572-toolsexecute环绕实际工具执行)
    - [5.7.3 tools/post-execute：执行后处理](#573-toolspost-execute执行后处理)
    - [5.7.4 tools/result：观察最终结果](#574-toolsresult观察最终结果)
    - [5.7.5 tools/change：工具注册变化通知](#575-toolschange工具注册变化通知)
  - [5.8 把真实执行流水线重新串起来](#58-把真实执行流水线重新串起来)
  - [5.9 超时不等于取消](#59-超时不等于取消)
    - [5.9.1 Promise.race 只决定谁先返回](#591-promiserace-只决定谁先返回)
    - [5.9.2 AbortSignal：协作式取消](#592-abortsignal协作式取消)
    - [5.9.3 当前 Harness 如何处理取消？](#593-当前-harness-如何处理取消)
    - [5.9.4 取消前后有不同错误](#594-取消前后有不同错误)
    - [5.9.5 当前 Harness 的超时策略在哪里？](#595-当前-harness-的超时策略在哪里)
    - [5.9.6 本节结论](#596-本节结论)
  - [5.10 工具结果必须可记录、可展示、可恢复](#510-工具结果必须可记录可展示可恢复)
    - [5.10.1 为什么需要规范化结果？](#5101-为什么需要规范化结果)
    - [5.10.2 为什么错误不能转换成空字符串？](#5102-为什么错误不能转换成空字符串)
    - [5.10.3 输出校验](#5103-输出校验)
    - [5.10.4 Session 记录](#5104-session-记录)
    - [5.10.5 为什么要区分执行结果和展示结果？](#5105-为什么要区分执行结果和展示结果)
  - [5.11 ToolExecutor：为什么 Registry 不直接执行所有工具？](#511-toolexecutor为什么-registry-不直接执行所有工具)
    - [5.11.1 describe()](#5111-describe)
    - [5.11.2 execute()](#5112-execute)
    - [5.11.3 并发与独占执行](#5113-并发与独占执行)
  - [5.12 实战：编写巡检插件的两个工具](#512-实战编写巡检插件的两个工具)
    - [5.12.1 目标](#5121-目标)
    - [5.12.2 目录结构](#5122-目录结构)
    - [5.12.3 package.json](#5123-packagejson)
    - [5.12.4 实现 inspect\_file](#5124-实现-inspect_file)
    - [5.12.5 实现 check\_service](#5125-实现-check_service)
    - [5.12.6 创建插件入口](#5126-创建插件入口)
    - [5.12.7 主动调用两个工具](#5127-主动调用两个工具)
    - [5.12.8 查看工具执行结果](#5128-查看工具执行结果)
    - [5.12.9 关于 timeoutMs](#5129-关于-timeoutms)
    - [5.12.10 测试失败场景](#51210-测试失败场景)
    - [5.12.11 本节总结](#51211-本节总结)
    - [5.12.12 本节验收](#51212-本节验收)
  - [5.13 从测试反向理解执行边界](#513-从测试反向理解执行边界)
    - [测试中应该重点关注什么？](#测试中应该重点关注什么)
  - [5.14 本章源码阅读路线](#514-本章源码阅读路线)
    - [第一阶段：认识 Tool Registry](#第一阶段认识-tool-registry)
    - [第二阶段：理解 ToolDefinition](#第二阶段理解-tooldefinition)
    - [第三阶段：理解参数与输出校验](#第三阶段理解参数与输出校验)
    - [第四阶段：理解事件扩展点](#第四阶段理解事件扩展点)
    - [第五阶段：理解真正的执行流水线](#第五阶段理解真正的执行流水线)
    - [第六阶段：理解批次执行](#第六阶段理解批次执行)
    - [第七阶段：追踪 Agent Loop](#第七阶段追踪-agent-loop)
    - [第八阶段：理解超时与审批](#第八阶段理解超时与审批)
  - [5.15 本章需要形成的架构认识](#515-本章需要形成的架构认识)
  - [本章验收](#本章验收)
    - [Tool Registry](#tool-registry)
    - [ToolDefinition](#tooldefinition)
    - [执行流水线](#执行流水线)
    - [安全边界](#安全边界)
    - [Session 与恢复](#session-与恢复)
  - [本章小结](#本章小结)


# 第 5 章：Tool Registry 与执行边界

在前面的章节中，我们已经了解了 Agent 的运行过程：Agent 接收消息，组织上下文，调用模型，并根据模型返回的内容决定下一步操作。

模型除了可以返回普通文本，还可以提出工具调用请求。

例如，用户要求：

> 删除 `/important/data.txt` 文件。

模型可能返回一个工具调用：

```json
{
  "name": "delete_file",
  "arguments": {
    "path": "/important/data.txt"
  }
}

但这不意味着 Harness 必须执行。

真正的执行者必须重新检查：

- 工具是否存在？
- 当前 Agent 是否能看到这个工具？
- 当前步骤是否允许使用这个工具？
- 参数是否符合工具定义？
- 参数指向的资源是否在允许范围内？
- 当前调用是否已经被取消？
- 是否存在执行策略拒绝？
- 工具执行是否发生异常？
- 返回值能否被规范化？
- 最终结果是否正确写入 Session？

这些检查共同构成工具执行边界。

本章不只是学习如何注册一个工具，而是要理解：

**模型提出调用，Registry 确认能力，Policy 决定是否允许，Executor 负责执行，Session 负责记录。**

---

## 5.1 模型提出调用，不代表系统批准调用

先理解一个基本事实：

模型并不直接执行工具。

模型只会在响应中产生一个 Tool Call，例如：

```json
{
  "type": "tool-call",
  "id": "call-001",
  "name": "inspect_file",
  "arguments": "{\"path\":\"README.md\"}"
}
```

这里包含：

| 字段 | 含义 |
|---|---|
| `type` | 表示这是工具调用 |
| `id` | 本次调用的标识 |
| `name` | 模型希望调用的工具名称 |
| `arguments` | 模型生成的参数 |

这只是模型提出的一项请求。

真正执行之前，Harness 还需要经过自己的处理流程。

例如：

```text
模型输出 Tool Call
       |
       v
解析工具调用
       |
       v
查找工具定义
       |
       v
检查当前 Agent 的工具可见性
       |
       v
验证参数
       |
       v
执行前策略检查
       |
       v
调用工具实现
       |
       v
处理执行结果
       |
       v
写入 Session
       |
       v
下一轮模型请求
```

注意：

这张图是理解工具执行过程的阅读模型，不代表每一步都对应一个独立函数。

实际源码可能把多个步骤封装在同一个模块中，也可能通过事件、接口和插件共同完成。

阅读源码时必须找到真正的调用者和被调用者。

### 5.1.1 为什么不能相信模型生成的工具名称？

假设当前系统只注册了：

```text
inspect_file
check_service
search_workspace
```

但是模型输出：

```json
{
  "name": "delete_database",
  "arguments": "{}"
}
```

Harness 不能因为模型要求调用，就自动寻找某个相似工具执行。

如果工具不存在，就应该拒绝本次调用，并形成明确的失败结果。

当前 Harness 使用：

```ts
ToolNotFoundError
```

表示工具不存在。

其错误码为：

```text
UNKNOWN_TOOL
```

未知工具属于结构化工具失败，而不是直接让模型请求本身失去所有后续处理能力。

这意味着：

模型提出了一个无法执行的工具调用，Harness 可以将失败结果反馈给模型，让模型根据错误信息决定下一步。

### 5.1.2 为什么工具定义和工具执行必须分开？

一个工具通常包含两部分：

```ts
{
  name: 'inspect_file',

  description: 'Inspect a file.',

  parameters: {
    path: {
      type: 'string',
      required: true,
    },
  },

  execute: async (args) => {
    // 真正执行工作
  },
}
```

其中：

- `name`：工具名称。
- `description`：告诉模型工具能做什么。
- `parameters`：告诉模型需要提供什么参数。
- `execute`：宿主程序真正执行的代码。

模型需要知道前三项。

但模型不应该获得 `execute` 函数本身。

因此，Harness 会把工具定义投影成模型能够理解的 Tool Schema。

后面阅读 `schemas()` 时，需要重点理解这种投影。

---

## 5.2 Tool Registry：工具注册中心

首先阅读：

```text
packages/core/tools/src/index.ts
```

这是理解工具注册和管理的核心文件。

先不要急着阅读执行流水线。

先弄清楚 Registry 到底管理什么。

### 5.2.1 Registry 的职责

Tool Registry 不是简单的 Map。

它至少负责以下工作：

1. 注册工具。
2. 删除工具。
3. 管理不同 Scope 中的工具。
4. 处理工具名称冲突。
5. 管理工具可见性。
6. 管理工具限制规则。
7. 根据当前 Agent 解析可用工具。
8. 生成模型可以看到的工具 Schema。
9. 选择实际负责执行工具的 Executor。

可以把它理解为：

**整个 Harness 的工具目录和能力入口。**

### 5.2.2 找到 ToolsService

打开：

```text
packages/core/tools/src/index.ts
```

搜索：

```ts
class ToolsService
```

重点观察它暴露了哪些方法。

其中比较重要的是：

```ts
register()
restrict()
resolve()
definitions()
schemas()
setExecutor()
execute()
guard()
checkGuards()
```

不要试图一次记住所有方法。

先按照职责分类：

| 方法 | 主要职责 |
|---|---|
| `register()` | 注册工具 |
| `restrict()` | 限制某个 Scope 能看到的全局工具 |
| `resolve()` | 解析当前步骤允许使用的工具集合 |
| `definitions()` | 获取可见工具定义 |
| `schemas()` | 生成模型可见的工具 Schema |
| `setExecutor()` | 注册工具执行器 |
| `execute()` | 将工具调用交给执行器 |
| `guard()` | 注册单调拒绝策略 |
| `checkGuards()` | 执行已经注册的 Guard |

这里有一个重要区别：

`register()` 管理工具定义。

`execute()` 管理执行器调用。

两者不是同一个概念。

### 5.2.3 register()：注册工具

找到：

```ts
register(definition, options)
```

核心代码可以概括为：

```ts
register(definition: ToolDefinition, options: ToolRegistrationOptions = {}) {
  const name = definition.name

  // 检查 output 声明
  // 检查 output.schema
  // 检查 timeoutMs
  // 检查保留名称

  return this.layers.effect(
    this.ctx,
    layer => layer.tools.insert(name, {
      definition,
      when: options.when,
    }),
    { label: 'tools.register()' },
  )
}
```

这里需要理解几个细节。

**第一，注册时会检查输出声明。**

当前 ToolDefinition 必须提供：

```ts
output: {
  schema: ...,
  render: ...,
}
```

不能只定义输入参数和 execute。

因为 Harness 还需要知道工具返回值应该满足什么结构，以及如何转换成模型可以理解的内容。

**第二，注册时会检查 timeoutMs。**

如果工具声明了：

```ts
timeoutMs: 5000
```

它必须是一个正的有限数值。

但要特别注意：

**注册 timeoutMs 不代表 Registry 自己会执行超时控制。**

真正的超时由专门的策略插件实现，后面会详细讲。

**第三，注册时会检查保留名称。**

例如：

```text
run_code
```

是保留的 PTC 模式传输名称，不能被普通插件注册或覆盖。

**第四，register() 返回 disposer。**

```ts
const dispose = ctx.tools.register(tool)

dispose()
```

这样可以提前注销工具。

正常情况下，注册操作也会关联插件生命周期，插件卸载时清理对应注册。

### 5.2.4 Scope：同一个工具注册表，为什么不同 Agent 实例看到的工具不同？
先区分两个概念：
- Agent：负责组织模型请求、处理消息和驱动工具调用的运行时对象。
- Scope：工具可用范围的限制机制，决定某次工具调用能够使用哪些工具。

假设系统中有三个工具：

```text
read_file
write_file
run_command
```

现在有两个 Agent：

```text
Agent A
Agent B
```

Agent A 可能允许：

```text
read_file
write_file
```

Agent B 可能只允许：

```text
read_file
```

这意味着：

工具是否注册，和某个 Agent 是否能看到工具，是两件不同的事情。

当前 Harness 使用 Scope 管理工具的可见性。

阅读：

```text
packages/core/tools/src/index.ts
```

搜索：

```ts
view()
```

以及：

```ts
scopeOf()
scopeChainOf()
```

重点理解：

- 全局注册的工具可以被多个 Scope 继承。
- Scope 自己注册的工具属于自己的作用域。
- 子 Scope 可以通过限制规则过滤继承来的全局工具。
- 不同 Scope 可以有自己的工具执行器。

### 5.2.5 restrict()：限制工具可见性

当前工具系统提供：

```ts
ctx.tools.restrict()
```

例如：

```ts
ctx.tools.restrict({
  allow: ['read_file'],
})
```

这表示当前 Agent Scope 只保留允许列表中的全局工具。

也可以使用：

```ts
ctx.tools.restrict({
  deny: ['write_file'],
})
```

表示禁止使用指定的全局工具。

注意：

`restrict()` 不是全局删除工具。

它只影响当前 Scope 继承到的工具。

Scope 自己注册的工具不会被这类全局工具限制直接过滤。

另外，多个限制规则会进行交集处理。

因此，后面的限制不能通过简单增加 allow 项目，撤销前面已经施加的限制。

### 5.2.6 resolve()：解析当前步骤可用的工具

这是本节非常重要的方法。

搜索：

```ts
async resolve(context)
```

当前 Agent Loop 在准备一个新步骤时，会调用：

```ts
const toolSet = await this.loopCtx.tools.resolve({
  agent: this,
  modelSelection: selection,
  signal,
})
```

这里传入三个信息：

| 参数 | 作用 |
|---|---|
| `agent` | 当前 Agent |
| `modelSelection` | 当前步骤使用的模型选择 |
| `signal` | 当前步骤的取消信号 |

resolve() 会根据当前可见工具，处理注册时声明的条件：

```ts
when?: (context) => boolean | Promise<boolean>
```

例如：

```ts
ctx.tools.register(
  tool,
  {
    when: async ({ agent, signal }) => {
      signal.throwIfAborted()

      return await checkToolAvailability(agent)
    },
  },
)
```

如果条件返回 `false`，工具不会进入当前步骤的工具集合。

如果条件执行失败，Registry 会抛出：

```ts
ToolActivationError
```

如果执行过程中收到取消信号，也不会继续发布一个部分解析的工具集合。

这里要特别注意：

`resolve()` 不是每次模型提出工具调用时才简单查询一次 Map。

它是在 Agent 准备步骤时解析工具集合。

得到的 `ResolvedToolSet` 会在当前步骤的工具批次完成后释放。

### 5.2.7 schemas()：Registry 如何告诉模型有哪些工具？

搜索：

```ts
schemas()
```

核心逻辑：

```ts
schemas(scope, toolSet) {
  return this.definitions(scope, toolSet)
    .map(definition => projectToolSchema(definition, true))
}
```

这一步非常关键。

Registry 不会把整个 ToolDefinition 原样交给模型。

它只投影允许发送给模型的字段。

主要包括：

```text
name
description
parameters
```

而下面这些内容不会进入模型请求：

```text
execute
output
render
projectContent
finalizeContent
timeoutMs
isConcurrencySafe
presentCall
presentResult
```

这说明：

**工具定义是宿主程序内部的完整能力描述，Tool Schema 是提供给模型的受限描述。**

模型只能根据 Schema 提出调用。

它不能通过 Schema 直接访问宿主程序内部的执行函数。

---

## 5.3 ToolDefinition：一个工具到底包含什么？

继续阅读：

```text
packages/core/tools/src/contracts.ts
```

重点找到：

```ts
interface ToolDefinition
```

当前 ToolDefinition 不只是：

```ts
{
  name,
  description,
  parameters,
  execute,
}
```

它还包含规范化输出、超时声明、并发分类和展示相关能力。

可以按照下面的结构理解：

```text
ToolDefinition
│
├── name
├── description
├── parameters
│
├── output
│   ├── schema
│   ├── render
│   └── presentationMeta
│
├── execute
├── projectContent
├── finalizeContent
├── timeoutMs
├── isConcurrencySafe
├── presentCall
└── presentResult
```

不需要现在掌握每一个字段。

先掌握下面几个核心字段。

### 5.3.1 parameters：输入参数定义

例如：

```ts
parameters: {
  path: {
    type: 'string',
    required: true,
    description: 'File path',
  },
}
```

它描述模型应该提供什么参数。

但它不能保证路径安全。

例如：

```json
{
  "path": "../../etc/passwd"
}
```

完全可能是一个符合字符串类型的参数。

Schema 可以判断它是不是字符串。

但不能仅凭字符串类型判断这个路径是否允许访问。

### 5.3.2 execute：真正的执行逻辑

例如：

```ts
async execute(args, exec) {
  return await readFileMetadata(args.path, exec.signal)
}
```

`execute` 才是真正执行工具工作的地方。

它接收：

- 已经进入执行流水线的参数。
- 当前工具调用的执行上下文。

其中：

```ts
exec.signal
```

是特别重要的取消信号。

工具实现应该把它传递给真正执行工作的底层 API。

### 5.3.3 output.schema：输出也必须有约束

例如：

```ts
output: {
  schema: {
    type: 'object',
    properties: {
      path: { type: 'string' },
      size: { type: 'number' },
    },
    required: ['path', 'size'],
    additionalProperties: false,
  },

  render(args, value) {
    return [{
      type: 'text',
      text: JSON.stringify(value),
    }]
  },
}
```

为什么要约束输出？

因为工具执行成功，不代表它返回的内容就一定符合系统要求。

例如：

工具声明返回：

```json
{
  "path": "README.md",
  "size": 100
}
```

但实际返回：

```json
{
  "path": "README.md",
  "size": "100"
}
```

这里 `size` 类型错误。

Harness 需要识别这个问题，而不是直接把错误结构当作正常结果交给模型。

### 5.3.4 render：把结构化结果转换成模型内容

工具执行返回的是规范化 JSON 值。

`render()` 负责将它转换成 ContentBlock。

例如：

```ts
render(args, value) {
  return [{
    type: 'text',
    text: `文件 ${value.path} 的大小为 ${value.size} 字节`,
  }]
}
```

这里有三个不同概念：

| 内容 | 作用 |
|---|---|
| execute 返回值 | 工具的规范化结构化结果 |
| output.schema | 检查结构化结果是否合法 |
| render | 将结构化结果转换成模型内容 |

不要把这三件事情混为一谈。

---

## 5.4 defineTool：为什么不直接手写 ToolDefinition？

阅读：

```text
packages/core/tools/src/schema.ts
```

搜索：

```ts
defineTool()
```

Harness 提供了 `defineTool()`，帮助开发者定义类型安全的工具。

例如：

```ts
import { defineTool } from '@deepseek-ai/dsh-tools'

const tool = defineTool({
  name: 'greet',

  description: 'Greet a person.',

  parameters: {
    name: {
      type: 'string',
      required: true,
      description: 'Person name',
    },
  },

  output: {
    schema: {
      type: 'string',
    },

    render: (_args, value) => [{
      type: 'text',
      text: value,
    }],
  },

  async execute(args) {
    return `Hello, ${args.name}!`
  },
})
```

它比手写普通对象多做了一些事情。

### 5.4.1 参数类型推导

根据：

```ts
parameters
```

推导：

```ts
args
```

的类型。

因此：

```ts
async execute(args) {
  return `Hello, ${args.name}!`
}
```

TypeScript 可以知道：

```ts
args.name
```

是字符串。

### 5.4.2 运行时参数校验

`defineTool()` 不只是帮助 TypeScript 推导类型。

它还会生成运行时验证逻辑。

在执行时：

```ts
const violations = validate(args)

if (violations.length > 0) {
  throw new ToolArgsError(violations)
}
```

如果参数不合法，就抛出：

```ts
ToolArgsError
```

错误码：

```text
INVALID_ARGS
```

例如：

工具要求：

```ts
name: {
  type: 'string',
  required: true,
}
```

模型却传入：

```json
{
  "name": 123
}
```

就会进入参数验证失败路径。

### 5.4.3 输出类型推导

`defineTool()` 也会根据：

```ts
output.schema
```

推导 execute 的返回类型。

这样开发者在编写工具时，可以同时获得输入和输出的类型检查。

### 5.4.4 不要混淆三个操作

```ts
const tool = defineTool(...)
```

```ts
ctx.tools.register(tool)
```

```ts
await ctx.tools.execute(...)
```

它们分别表示：

| 操作 | 作用 |
|---|---|
| `defineTool()` | 定义工具 |
| `register()` | 将工具注册到 Registry |
| `execute()` | 通过执行器处理工具调用 |

定义工具不等于注册工具。

注册工具也不等于执行工具。

---

## 5.5 参数校验与权限检查是两件事

这是工具安全设计中最容易混淆的地方。

假设有一个文件工具：

```ts
parameters: {
  path: {
    type: 'string',
    required: true,
  },
}
```

下面这些参数都符合字符串类型：

```json
{
  "path": "README.md"
}
```

```json
{
  "path": "../../etc/passwd"
}
```

```json
{
  "path": "C:\\Windows\\System32\\config\\SAM"
}
```

Schema 只能说明：

参数是不是字符串。

它不能说明：

这个字符串指向的文件是否允许读取。

### 5.5.1 参数校验解决什么问题？

参数校验回答：

**输入数据是否符合工具声明的格式？**

例如：

- 是否缺少必填字段？
- 字符串是否符合要求？
- 数字是否在允许范围？
- 数组元素类型是否正确？
- 对象是否包含不允许的字段？

这些属于数据结构检查。

### 5.5.2 权限检查解决什么问题？

权限检查回答：

**当前调用者是否有权执行这个操作？**

例如：

- 当前 Agent 是否允许读取这个目录？
- 当前 Session 是否允许写文件？
- 当前用户是否需要审批？
- 当前工具是否允许访问这个主机？
- 当前操作是否违反 Sandbox Policy？

这属于执行授权。

### 5.5.3 不要只依赖工具参数 Schema

例如：

```ts
parameters: {
  path: {
    type: 'string',
    required: true,
  },
}
```

即使 Schema 校验通过，仍然需要在工具内部检查：

```ts
const target = await realpath(args.path)

if (!isInsideAllowedRoot(target)) {
  throw new Error('path is outside allowed directory')
}
```

而且要注意：

仅仅检查字符串是否以某个目录前缀开头是不够的。

例如：

```text
/home/user/work
/home/user/work-backup
```

如果直接使用：

```ts
target.startsWith('/home/user/work')
```

第二个路径也会通过。

应该使用真实路径和路径边界判断。

### 5.5.4 文件系统工具还要考虑什么？

至少检查：

| 风险 | 需要处理的情况 |
|---|---|
| 路径穿越 | `../` |
| 绝对路径 | 绕过预设目录 |
| 符号链接 | 指向允许目录之外 |
| 大小限制 | 读取超大文件 |
| 文件类型 | 设备文件、目录、特殊文件 |
| 文件变化 | 检查与实际读取之间发生替换 |
| 并发修改 | 多个工具同时操作同一个文件 |

如果要抵抗恶意本地进程在检查和打开之间替换符号链接，仅使用 `realpath()` 后再 `readFile()` 也不够。

更严格的实现需要使用适当的文件描述符、打开标志和目录边界控制。

### 5.5.5 网络工具还要考虑什么？

至少检查：

- URL 协议。
- 主机 allowlist。
- 端口。
- 重定向。
- DNS 解析结果。
- 私有地址和本地地址。
- 请求大小。
- 响应大小。
- 连接超时和整体超时。

特别注意：

只检查 URL 中的 hostname，不一定能完全防止 SSRF。

如果允许访问的主机可以通过 DNS 指向内网地址，还需要考虑 DNS 解析和实际连接目标的一致性。

对于高安全要求的系统，应该使用具备出口控制能力的网络层，而不是只在工具函数里判断字符串。

---

## 5.6 从 Tool Registry 进入 ToolExecutionPipeline

现在进入真正的执行过程。

阅读：

```text
packages/core/tools/src/pipeline.ts
```

以及：

```text
packages/core/tools/src/events.ts
```

当前工具系统把单次工具调用的核心处理集中在共享执行流水线中。

在 README 中，可以看到它将过程划分为：

```text
prepare
dispatch
complete
```

这是本章需要掌握的核心结构。

### 5.6.1 为什么要有 ToolExecution？

工具执行过程中需要携带的信息很多：

```ts
interface ToolExecutionInput {
  callId: ToolCallId

  name: string

  arguments: unknown

  agent?: Agent

  toolSet?: ResolvedToolSet

  signal: AbortSignal
}
```

这只是输入结构的主要部分。

流水线还会为执行分配自己的执行身份：

```ts
interface ToolExecution extends ToolExecutionInput {
  rootCallId: ToolCallId

  token: ToolExecutionToken
}
```

其中：

- `callId`：工具调用标识。
- `rootCallId`：根工具调用标识。
- `token`：流水线生成的内部关联标识。
- `agent`：调用工具的 Agent。
- `toolSet`：当前步骤允许使用的工具集合。
- `signal`：调用者提供的取消信号。

为什么需要 `token`？

因为某些工具可能会在内部继续调用其他工具。

例如：

```text
run_code
   |
   +--> inspect_file
   |
   +--> search_workspace
   |
   +--> check_service
```

这时需要区分：

- 外层模型直接请求的调用。
- 内层程序产生的子调用。

`token` 是内部关联机制的一部分。

它不是模型自己生成的，也不是让模型决定的。

### 5.6.2 参数为什么要做快照？

模型产生的参数进入流水线后，不能继续把任意可变对象直接交给后续处理。

当前工具系统要求参数通过 lossless JSON materialization 边界。

之后参数会被冻结。

这样做有几个目的：

1. 防止执行期间参数被意外修改。
2. 保证策略检查和实际执行使用相同的数据。
3. 让审计记录与真正执行的参数保持一致。
4. 避免循环引用等非 JSON 数据进入模型调用记录。

如果输入不是能够无损表示的 JSON 数据，就不能直接把它当作正常工具参数处理。

### 5.6.3 为什么不能在 pre-execute 中修改 arguments？

当前设计明确禁止：

```ts
exec.arguments = newArguments
```

原因是：

工具调用参数已经进入调用记录和展示流程。

如果策略插件在后面偷偷修改参数，就会出现：

```text
Session 记录的参数
        !=
真正执行的参数
```

这会破坏审计一致性。

因此，当前 `tools/pre-execute` 允许做允许、拒绝、取消和审批决策，但不提供任意参数重写能力。

如果确实需要修改参数，应当在明确的工具定义或独立的转换边界中设计，而不是偷偷修改已经记录的调用参数。

---

## 5.7 五个核心工具事件：真正的执行边界

阅读：

```text
packages/core/tools/src/events.ts
```

当前工具系统定义了五个核心事件：

| 事件 | 模式 | 主要职责 |
|---|---|---|
| `tools/pre-execute` | waterfall | 执行前决策 |
| `tools/execute` | waterfall | 环绕实际分发 |
| `tools/post-execute` | waterfall | 处理执行结果 |
| `tools/result` | emit | 观察最终结果 |
| `tools/change` | emit | 通知工具注册或限制发生变化 |

这五个事件不是五种不同的日志通知。

它们处于工具生命周期的不同位置。

### 5.7.1 tools/pre-execute：执行前决策

声明：

```ts
'tools/pre-execute'(
  exec,
  next,
): Promise<PreToolDecision>
```

它采用：

```text
waterfall
```

这个事件负责决定工具能否进入执行阶段。

当前支持的决策类型：

```ts
type PreToolDecision =
  | { kind: 'allow' }
  | { kind: 'deny'; reason: string; info?: ToolErrorInfo }
  | { kind: 'cancel' }
  | {
      kind: 'ask'
      reason?: string
      displayReason?: {
        readonly en: string
        readonly [locale: string]: string
      }
    }
```

分别理解：

**allow**

允许继续执行。

**deny**

拒绝执行，并给出拒绝原因。

**cancel**

将本次调用归类为取消，而不是普通策略拒绝。

**ask**

请求用户审批。

如果当前系统没有可用的审批支持，不能默认批准。

应该失败关闭。

### 一个重要区别

审批机制和审批策略不是同一件事情。

审批服务负责：

```text
提出审批请求
等待审批结果
返回封闭的审批结果
```

而谁需要审批、什么情况下询问、由谁回答，则由其他策略插件和通道插件决定。

可以阅读：

```text
packages/interaction/user-approval/
```

重点理解：

```text
ctx.approval
```

以及：

```text
approval/request
```

这可以帮助理解为什么一个工具调用的授权决策不是模型自己决定的。

### 5.7.2 tools/execute：环绕实际工具执行

声明：

```ts
'tools/execute'(
  exec,
  next,
): Promise<ToolExecutionResult>
```

它同样采用：

```text
waterfall
```

它不是简单的“工具开始执行”通知。

它允许插件包装真正的工具分发过程。

例如：

```ts
ctx.on('tools/execute', async (exec, next) => {
  console.log('工具执行开始')

  const result = await next()

  console.log('工具执行完成')

  return result
})
```

这里：

```ts
next()
```

才是真正继续向下执行的关键。

如果监听器不调用 `next()`，后续工具执行就不会继续。

这个位置适合实现：

- 超时控制。
- 指标统计。
- 执行包装。
- 重试策略。

但是必须注意：

`tools/execute` 的 `exec.signal` 可以由包装器替换。

Registry 会将包装器提供的信号与原始调用者信号重新融合，确保包装器不能通过替换信号解除调用者取消。

### 5.7.3 tools/post-execute：执行后处理

声明：

```ts
'tools/post-execute'(
  exec,
  result,
  next,
): Promise<PostToolDecision>
```

同样采用：

```text
waterfall
```

它负责检查已经规范化的执行结果。

当前决策包括：

```ts
type PostToolDecision =
  | {
      kind: 'accept'
      content?: ContentBlock[]
      additionalContexts?: UserMessage[]
    }
  | {
      kind: 'accept'
      value: JsonValue
      additionalContexts?: UserMessage[]
    }
  | {
      kind: 'block'
      feedback: ContentBlock[]
      info?: ToolErrorInfo
      additionalContexts?: UserMessage[]
    }
```

可以理解为：

**执行成功并不代表结果一定会被最终接受。**

例如：

工具读取到了一份文件。

执行本身成功，但执行后的策略可能判断：

- 结果包含不应该返回的内容。
- 结果需要替换。
- 结果需要附加上下文。
- 结果不应该被接受。

这时可以通过 `post-execute` 进行处理。

需要注意：

工具执行抛出的异常也会进入这个 waterfall，并作为错误结果供后续处理。

### 5.7.4 tools/result：观察最终结果

声明：

```ts
'tools/result'(
  exec,
  result,
): undefined
```

模式：

```text
emit
```

它的职责是观察最终结果。

这里的执行对象和结果都是只读快照。

结果在通知监听器之前会被深度冻结。

监听器失败会被隔离，不会因为某个审计监听器出错，就改变已经形成的工具结果。

因此它适合：

- 审计。
- 统计。
- 监控。
- 追踪。
- 记录执行结果。

不适合：

- 决定是否执行工具。
- 修改参数。
- 修改最终结果。

### 5.7.5 tools/change：工具注册变化通知

声明：

```ts
'tools/change'(): void
```

模式：

```text
emit
```

当工具注册、注销或 Scope 限制发生变化时，可以通过它通知其他组件。

例如：

- 工具列表缓存需要刷新。
- 某个界面需要重新显示可用工具。
- 需要重新计算工具集合。

它不是工具执行事件。

不要把 `tools/change` 和 `tools/result` 混为一谈。

---

## 5.8 把真实执行流水线重新串起来

现在可以把前面的内容放到一张图中。

```text
模型产生 Tool Call
       |
       v
Agent Loop 收到工具调用
       |
       v
ToolExecutor 接收调用批次
       |
       v
Tool Registry 提供当前工具定义
       |
       v
ToolExecutionPipeline
       |
       v
参数快照与执行身份分配
       |
       v
工具定位与参数验证
       |
       v
tools/pre-execute
       |
       v
审批与单调 Guard 检查
       |
       v
tools/execute
       |
       v
实际工具 execute()
       |
       v
输出验证与结果规范化
       |
       v
tools/post-execute
       |
       v
finalizeContent
       |
       v
tools/result
       |
       v
Executor 返回批次结果
       |
       v
Agent Loop 写入 Session
       |
       v
下一轮模型请求
```

注意几个容易误解的地方：

1. 工具定义不等于工具执行。
2. Registry 不等于 Executor。
3. `tools/execute` 是环绕调用，不是简单通知。
4. `tools/result` 是最终观察点，不是修改结果的地方。
5. Session 记录不只是控制台日志，而是 Agent 对话历史的一部分。

实际代码中的函数边界和调用顺序，应以 `pipeline.ts`、`batch.ts` 和 Agent Loop 的实现为准。

---

## 5.9 超时不等于取消

这是本章最重要的工程问题之一。

先看一个常见写法：

```ts
const result = await Promise.race([
  doSomething(),
  timeout(),
])
```

假设：

```ts
async function doSomething() {
  await writeLargeFile()

  return 'done'
}
```

而 timeout 在 5 秒后触发。

那么：

```ts
Promise.race()
```

可能在 5 秒后返回超时结果。

但是：

```ts
writeLargeFile()
```

不一定停止。

它可能继续运行。

这意味着：

```text
调用者已经收到超时
        |
        v
底层任务仍然运行
        |
        v
文件仍然可能被修改
```

这就是为什么：

**超时不等于取消，取消也不等于任务已经结束。**

### 5.9.1 Promise.race 只决定谁先返回

例如：

```ts
const task = new Promise(resolve => {
  setTimeout(() => {
    console.log('底层任务完成')
    resolve('done')
  }, 10000)
})

const timeout = new Promise((_, reject) => {
  setTimeout(() => {
    reject(new Error('timeout'))
  }, 1000)
})

await Promise.race([task, timeout])
```

一秒后，调用者收到超时异常。

但十秒后，底层任务仍然会打印：

```text
底层任务完成
```

`Promise.race()` 不会自动取消其他 Promise。

### 5.9.2 AbortSignal：协作式取消

Node.js 提供：

```ts
AbortController
AbortSignal
```

创建：

```ts
const controller = new AbortController()

const signal = controller.signal
```

请求取消：

```ts
controller.abort()
```

工具实现应该接收并使用：

```ts
async execute(args, exec) {
  exec.signal.throwIfAborted()

  // 将 signal 继续传给底层操作
}
```

如果底层 API 支持 AbortSignal：

```ts
await fetch(url, {
  signal: exec.signal,
})
```

就应该把它传下去。

但要注意：

AbortSignal 不是强制终止线程的机制。

它是一种协作式取消协议。

被调用的 API 必须主动响应它。

### 5.9.3 当前 Harness 如何处理取消？

阅读：

```text
packages/core/tools/src/contracts.ts
packages/core/tools/src/events.ts
packages/core/tools/src/pipeline.ts
```

当前工具执行上下文中：

```ts
exec.signal
```

是必须存在的调用者取消信号。

工具实现需要：

- 观察取消信号。
- 将信号传递给底层 API。
- 在取消后尽快停止工作。
- 等待自己拥有的工作真正结束。

当前设计强调 quiescence，也就是：

**执行器不能因为外层已经返回取消结果，就假设底层工作已经停止。**

如果工具已经启动了一个子进程，还需要确认子进程被终止。

如果工具打开了文件句柄，还需要确认句柄被关闭。

如果工具启动了后台任务，还需要等待或明确隔离它。

### 5.9.4 取消前后有不同错误

当前工具系统定义了两个重要错误码：

```text
ABORTED_BEFORE_DISPATCH
ABORTED
```

含义：

| 错误码 | 含义 |
|---|---|
| `ABORTED_BEFORE_DISPATCH` | 工具函数尚未真正调用时，调用已经被取消 |
| `ABORTED` | 工具函数已经开始执行，随后发生取消 |

为什么要区分？

因为它们代表不同的执行事实。

例如：

```text
工具还没有开始
       |
       v
收到取消
       |
       v
ABORTED_BEFORE_DISPATCH
```

另一种情况：

```text
工具已经开始执行
       |
       v
正在读取文件
       |
       v
收到取消
       |
       v
工具停止并完成清理
       |
       v
ABORTED
```

不能把这两种情况简单地都记录成一个模糊的 timeout。

### 5.9.5 当前 Harness 的超时策略在哪里？

阅读：

```text
packages/guard/timeout-policy/
```

包名：

```text
@deepseek-ai/dsh-tool-call-timeout-policy
```

它通过：

```text
tools/execute
```

这个 waterfall 包装工具执行。

工具定义可以声明：

```ts
timeoutMs: 5000
```

但 Registry 本身不会执行超时。

超时策略插件会读取这个声明，并为本次调用派生新的执行信号。

它的核心思路：

```text
原始调用 signal
       |
       v
创建带期限的派生 signal
       |
       v
tools/execute 包装
       |
       v
next()
       |
       v
工具实现
       |
       v
等待工具真正结束
       |
       v
如果超时已触发，形成 TOOL_TIMEOUT
```

特别注意：

当前策略不是简单地在 Promise.race 超时后就放弃底层 Promise。

它要求工具实现能够响应取消，并在自己的工作达到 quiescence 后再完成。

这就是 timeoutMs 被定义为 cooperative timeout budget 的原因。

### 5.9.6 本节结论

设计工具时要明确回答：

1. 谁创建 AbortController？
2. 谁负责调用 abort()？
3. signal 是否传递给真正执行工作的 API？
4. 底层 API 是否真的支持取消？
5. 如果不支持取消，如何隔离？
6. 子进程是否会被终止？
7. 资源是否完成清理？
8. Executor 是否等待任务真正结束？

如果这些问题回答不清楚，就不能认为工具已经具备可靠的超时和取消能力。

---

## 5.10 工具结果必须可记录、可展示、可恢复

工具的返回值不一定能直接进入 Session。

例如：

```ts
return {
  name: 'README.md',
  size: 1024,
}
```

这是比较简单的 JSON 数据。

但是下面这些值就存在问题：

```ts
return new Error('failed')
```

```ts
return Buffer.from('hello')
```

```ts
const value: any = {}

value.self = value

return value
```

它们分别涉及：

- Error 对象。
- Buffer。
- 循环引用。

如果直接把任意 JavaScript 对象交给 JSON 序列化，就可能失败或丢失信息。

### 5.10.1 为什么需要规范化结果？

当前 Harness 使用结构化执行结果：

```ts
type ToolExecutionResult =
  | ToolExecutionSuccess
  | ToolExecutionFailure
```

成功结果：

```ts
interface ToolExecutionSuccess {
  isError: false

  value: JsonValue

  content: ContentBlock[]

  meta?: JsonValue
}
```

失败结果：

```ts
interface ToolExecutionFailure {
  isError: true

  error: ToolFailure

  content: ContentBlock[]

  meta?: JsonValue
}
```

这里最重要的是：

成功和失败使用不同的结构。

成功时有：

```ts
value
```

失败时有：

```ts
error
```

这样就不会出现：

```ts
{
  isError: true,
  value: 'success'
}
```

这种语义混乱的结构。

### 5.10.2 为什么错误不能转换成空字符串？

假设工具执行失败：

```ts
throw new Error('permission denied')
```

如果系统把它转换成：

```text
''
```

那么模型收到的结果就是一个空字符串。

模型无法知道：

- 工具执行失败。
- 用户没有权限。
- 文件不存在。
- 请求被取消。
- 工具返回了空内容。

因此，失败应该保留明确的错误语义。

例如：

```ts
{
  isError: true,

  error: {
    message: 'permission denied',

    info: {
      name: 'PermissionError',
      code: 'PERMISSION_DENIED',
    },
  },

  content: [{
    type: 'text',
    text: 'permission denied',
  }],
}
```

这里：

`content` 是模型能够看到的内容。

`error.info` 是结构化错误身份，供系统内部策略和持久化诊断使用。

不要随意把内部诊断字段全部暴露给模型。

### 5.10.3 输出校验

当前工具系统会根据：

```ts
output.schema
```

检查成功结果。

如果工具返回值不符合输出声明，会产生：

```text
INVALID_TOOL_OUTPUT
```

对应：

```ts
ToolOutputError
```

这意味着：

工具执行成功，不代表输出一定通过验证。

例如：

```ts
output.schema = {
  type: 'number',
}
```

但工具返回：

```ts
return '100'
```

就不应该被当作正常的数值结果。

### 5.10.4 Session 记录

继续阅读：

```text
packages/core/agent-loop/src/agent.ts
```

搜索：

```ts
this.loopCtx.tools.execute
```

这里可以看到 Agent Loop 将模型产生的工具调用交给工具执行系统。

工具执行完成后，Agent Loop 还需要将工具调用和工具结果写入 Session。

当前工具执行器也支持接收：

```ts
log: {
  turn,
  step,
}
```

它用于确定工具调用属于哪个 Agent、Turn 和 Step。

工具批次执行结果还会返回：

```ts
interface ToolBatchResult {
  results: readonly ToolCallResult[]

  additionalContexts: readonly UserMessage[]

  concluded: boolean
}
```

其中：

- `results`：与原始调用关联的执行结果。
- `additionalContexts`：工具为后续模型请求提供的附加上下文。
- `concluded`：是否有成功工具结果要求结束当前 Agent Turn。

这里要特别理解：

**Session 是 Agent 对话过程的持久化事实来源，不是普通控制台日志。**

工具调用和结果需要保持对应关系。

即使工具执行失败，也应该让后续模型请求能够看到相应的失败结果。

### 5.10.5 为什么要区分执行结果和展示结果？

工具可能返回结构化数据：

```json
{
  "path": "README.md",
  "size": 1024
}
```

模型可能需要文本：

```text
README.md 的文件大小是 1024 字节。
```

UI 可能希望显示：

```text
文件信息卡片
```

三者并不完全相同。

因此，Harness 将：

- 规范化执行值。
- 模型内容。
- UI 展示信息。

分开处理。

相关字段：

```ts
output.schema
output.render()
output.presentationMeta()
presentCall()
presentResult()
```

不要认为工具的 UI 展示对象就是工具的实际执行结果。

---

## 5.11 ToolExecutor：为什么 Registry 不直接执行所有工具？

阅读：

```text
packages/core/tools/src/executor.ts
packages/core/tools/src/batch.ts
packages/core/tools/src/index.ts
```

在当前架构中：

Registry 负责工具管理。

Executor 负责执行一批工具调用。

当前接口可以概括为：

```ts
interface ToolExecutor {
  describe(tools): ToolModelInput

  execute(input: ToolBatchInput): Promise<ToolBatchResult>
}
```

### 5.11.1 describe()

负责描述当前 Executor 支持的工具集合。

它根据可见工具定义，生成模型请求需要的工具信息。

### 5.11.2 execute()

负责处理一批工具调用。

它需要考虑：

- 工具调用解析。
- 工具调用顺序。
- 并发执行。
- 取消。
- Session 记录。
- 结果关联。
- 执行失败后的恢复。

为什么需要批次执行？

因为模型一次响应可能包含多个工具调用。

例如：

```text
模型响应
   |
   +--> inspect_file
   |
   +--> check_service
   |
   +--> search_workspace
```

系统不能简单地假设永远只有一个工具调用。

### 5.11.3 并发与独占执行

当前工具系统支持两种调度模式：

```ts
type ToolExecutionMode =
  | { kind: 'parallel' }
  | { kind: 'exclusive' }
```

工具可以通过：

```ts
isConcurrencySafe()
```

声明自己是否允许与同批其他工具重叠执行。

例如：

只读文件元信息的工具，在满足其他条件时可能适合并发。

而修改同一个文件的工具，通常不应该默认允许并发。

这里有一个重要原则：

**并发安全不是由工具名称决定的，而是由工具实际副作用决定的。**

不能因为两个工具名字不同，就认为它们可以安全地同时执行。

---

## 5.12 实战：编写巡检插件的两个工具

现在开始本章实战。

我们要编写一个简单的巡检插件，包含两个工具：

```text
inspect_file
check_service
```

第一版不执行任意 Shell 命令。

不允许模型自由访问任意路径和任意网络地址。

两个工具都采用 allowlist（允许列表）设计。

本节不仅要完成工具定义和注册，还要主动调用工具，观察 Harness 的真实执行结果。

### 5.12.1 目标

**工具一：inspect_file**

读取允许目录中的文件元信息。

只返回：

- 文件路径。
- 文件类型。
- 文件大小。
- 修改时间。

不读取完整文件内容。

另外，限制单个文件的最大大小为 1 MiB。

**工具二：check_service**

检查允许列表中的服务端点。

只返回：

- 服务名称。
- HTTP 状态码。
- 是否可访问。
- 检查耗时。

不允许模型指定任意 URL。

模型只能选择预先配置的服务名称，例如：

```text
api
storage
```

两个工具都需要支持取消操作，通过 `exec.signal` 将取消信号传递给底层文件或网络操作。

### 5.12.2 目录结构

在仓库中创建：

```text
packages/experimental/inspection-tools/
├── package.json
└── src/
    ├── index.ts
    ├── inspect-file.ts
    └── check-service.ts
```

另外，在项目根目录创建一个用于测试的目录：

```text
inspection-data/
└── sample.txt
```

`sample.txt` 内容：

```text
This is a test file.
```

最终形成两层目录：

- `packages/experimental/inspection-tools/`：插件源码。
- `inspection-data/`：允许 `inspect_file` 访问的测试目录。

注意：`inspection-data` 是本实验约定的运行目录，不是 Harness 自动创建的目录。

### 5.12.3 package.json

创建：

```text
packages/experimental/inspection-tools/package.json
```

内容：

```json
{
  "name": "@deepseek-ai/inspection-tools",
  "version": "0.0.1",
  "type": "module",
  "private": true,
  "main": "./src/index.ts",
  "peerDependencies": {
    "@deepseek-ai/cordis": "workspace:^",
    "@deepseek-ai/dsh-tools": "workspace:^",
    "@deepseek-ai/dsh-llm": "workspace:^"
  }
}
```

这里声明三个依赖：

| 依赖 | 用途 |
|---|---|
| `@deepseek-ai/cordis` | 提供插件开发所需的 Context 类型 |
| `@deepseek-ai/dsh-tools` | 提供 `defineTool()` 和工具注册服务 |
| `@deepseek-ai/dsh-llm` | 提供工具调用使用的 `CallId` |

`peerDependencies` 表示插件需要宿主环境提供这些依赖，而不是在插件内部各自安装一份运行时实例。

这里使用 `workspace:^`，与仓库中的 workspace 依赖声明方式保持一致。

### 5.12.4 实现 inspect_file

创建：

```text
packages/experimental/inspection-tools/src/inspect-file.ts
```

先定义允许目录和文件大小限制：

```ts
import { realpath, stat } from 'node:fs/promises'
import { isAbsolute, relative, resolve, sep } from 'node:path'

const ROOT_PATH = resolve(process.cwd(), 'inspection-data')

const MAX_FILE_SIZE = 1024 * 1024
```

这里：

- `ROOT_PATH` 是允许访问目录的路径。
- `MAX_FILE_SIZE` 限制文件大小为 1 MiB。
- `process.cwd()` 表示当前 Node.js 进程的工作目录。

注意，这里先保存目录路径，不在模块加载时立即执行 `realpath()`。这样插件导入时不会因为测试目录尚未创建而直接失败。

定义路径检查：

```ts
function isInsideRoot(root: string, target: string) {
  const rel = relative(root, target)

  return rel === ''
    || (
      rel !== '..'
      && !rel.startsWith(`..${sep}`)
      && !isAbsolute(rel)
    )
}
```

这里不能使用简单的：

```ts
target.startsWith(root)
```

因为目录前缀相似不代表路径属于同一个目录。

例如：

```text
允许目录：/app/inspection-data
目标路径：/app/inspection-data-backup/secret.txt
```

字符串前缀检查可能错误地认为目标路径属于允许目录。

`relative()` 才能正确判断目标路径相对于根目录的位置。

再定义元信息读取函数：

```ts
async function inspectFile(
  input: string,
  signal: AbortSignal,
) {
  signal.throwIfAborted()

  if (input.length === 0) {
    throw new Error('path is required')
  }

  // 获取允许目录的真实路径。
  const root = await realpath(ROOT_PATH)

  // 将输入路径解析到允许目录下。
  const candidate = resolve(root, input)

  // 获取目标文件的真实路径。
  const target = await realpath(candidate)

  signal.throwIfAborted()

  // 检查真实路径是否仍然位于允许目录中。
  if (!isInsideRoot(root, target)) {
    throw new Error('path is outside allowed directory')
  }

  const info = await stat(target)

  signal.throwIfAborted()

  if (!info.isFile()) {
    throw new Error('target is not a regular file')
  }

  if (info.size > MAX_FILE_SIZE) {
    throw new Error('file is too large')
  }

  return {
    path: relative(root, target),
    type: 'file',
    size: info.size,
    modifiedAt: info.mtime.toISOString(),
  }
}
```

这里有几个重要步骤：

1. `resolve()` 将用户输入转换成目标路径。
2. `realpath()` 获取真实路径，解析符号链接。
3. `isInsideRoot()` 检查真实路径是否在允许目录内。
4. `stat()` 获取文件元信息。
5. `isFile()` 确保目标是普通文件。
6. 检查文件大小是否超过限制。
7. 返回元信息，不读取文件内容。

注意：

这个示例用于教学，演示路径边界、文件类型、大小限制和取消检查。

如果要抵抗恶意本地进程在路径检查与文件元信息读取之间替换符号链接，还需要更严格的文件描述符和目录边界控制。

不能把这个简化示例直接当作高安全等级的文件沙箱。

最后导出工具定义：

```ts
import { defineTool } from '@deepseek-ai/dsh-tools'

export const inspectFileTool = defineTool({
  name: 'inspect_file',

  description: 'Inspect metadata of a file inside the allowed inspection directory.',

  parameters: {
    path: {
      type: 'string',
      required: true,
      description: 'Relative path under the inspection root.',
    },
  },

  output: {
    schema: {
      type: 'object',
      properties: {
        path: {
          type: 'string',
          required: true,
        },
        type: {
          type: 'string',
          required: true,
        },
        size: {
          type: 'number',
          required: true,
        },
        modifiedAt: {
          type: 'string',
          required: true,
        },
      },
      additionalProperties: false,
    },

    render: (_args, value) => [{
      type: 'text',
      text: JSON.stringify(value, null, 2),
    }],
  },

  async execute(args, exec) {
    return inspectFile(args.path, exec.signal)
  },
})
```

这里使用 `defineTool()` 定义工具。

注意输出 Schema 的写法：

```ts
output: {
  schema: {
    type: 'object',
    properties: {
      path: {
        type: 'string',
        required: true,
      },
    },
    additionalProperties: false,
  },
}
```

这里的 `required: true` 是属性自身的声明，不是 JSON Schema 中对象的 `required` 数组。

`additionalProperties: false` 表示输出对象不允许出现未声明的属性。

因此，工具最终返回的对象必须符合声明的输出结构。

### 5.12.5 实现 check_service

创建：

```text
packages/experimental/inspection-tools/src/check-service.ts
```

先定义服务 allowlist：

```ts
const SERVICES = {
  api: 'https://api.example.internal/health',
  storage: 'https://storage.example.internal/health',
} as const
```

这里使用 `as const` 保留服务名称的字面量类型。

模型只能选择：

```text
api
storage
```

而不是自己传入任意 URL。

定义检查函数：

```ts
async function checkService(
  serviceName: keyof typeof SERVICES,
  signal: AbortSignal,
) {
  signal.throwIfAborted()

  const url = SERVICES[serviceName]

  const startedAt = Date.now()

  const response = await fetch(url, {
    method: 'GET',
    redirect: 'manual',
    signal,
  })

  signal.throwIfAborted()

  // 本实验不读取响应正文，尽量释放响应资源。
  await response.body?.cancel()

  return {
    service: serviceName,
    status: response.status,
    ok: response.ok,
    elapsedMs: Date.now() - startedAt,
  }
}
```

这里有几个重要设计：

**第一，模型不能自由指定 URL。**

模型传入的是服务名称：

```ts
serviceName: 'api'
```

工具内部再通过：

```ts
const url = SERVICES[serviceName]
```

找到实际 URL。

这样，模型不能通过修改参数访问任意网络地址。

**第二，禁止自动跟随重定向。**

使用：

```ts
redirect: 'manual'
```

避免 `fetch()` 自动跟随服务端返回的重定向地址。

**第三，传递取消信号。**

```ts
signal
```

让 `fetch()` 能够响应 Harness 的取消操作。

**第四，限制返回数据。**

不把整个 `Response` 对象返回给模型。

只返回：

- 服务名称。
- HTTP 状态码。
- 是否可访问。
- 检查耗时。

这里的 `ok` 表示 HTTP 状态是否处于成功范围，不代表网络请求一定能够成功完成。

例如，服务器返回 HTTP 503 时，工具仍然可以正常返回一个巡检结果，只是 `ok` 为 `false`。

如果 DNS 解析失败、连接失败或者请求被取消，则会进入工具错误处理路径。

这个示例还假设 allowlist 中的端点是可信的固定基础设施地址。

如果允许用户控制 DNS 或服务端点，还需要在网络出口层实施 DNS、IP 和连接目标约束，不能仅依赖 URL allowlist。

最后导出工具定义：

```ts
import { defineTool } from '@deepseek-ai/dsh-tools'

export const checkServiceTool = defineTool({
  name: 'check_service',

  description: 'Check an approved service endpoint.',

  // 由超时策略插件读取并执行的协作式超时预算。
  timeoutMs: 5000,

  parameters: {
    service: {
      type: 'string',
      required: true,
      enum: ['api', 'storage'],
      description: 'Approved service name.',
    },
  },

  output: {
    schema: {
      type: 'object',
      properties: {
        service: {
          type: 'string',
          required: true,
        },
        status: {
          type: 'number',
          required: true,
        },
        ok: {
          type: 'boolean',
          required: true,
        },
        elapsedMs: {
          type: 'number',
          required: true,
        },
      },
      additionalProperties: false,
    },

    render: (_args, value) => [{
      type: 'text',
      text: JSON.stringify(value, null, 2),
    }],
  },

  async execute(args, exec) {
    return checkService(args.service, exec.signal)
  },
})
```

这里的 `enum` 限制了参数的可选值：

```ts
enum: ['api', 'storage']
```

因此，模型即使传入：

```json
{
  "service": "google"
}
```

也无法通过参数 Schema 校验。

另外，`args.service` 已经根据 `enum` 推导为：

```ts
'api' | 'storage'
```

不需要再手动使用类型断言。

### 5.12.6 创建插件入口

创建：

```text
packages/experimental/inspection-tools/src/index.ts
```

内容：

```ts
import type { Context } from '@deepseek-ai/cordis'

import { inspectFileTool } from './inspect-file.ts'
import { checkServiceTool } from './check-service.ts'

export const name = 'inspection-tools'

export const inject = ['tools']

export function apply(ctx: Context) {
  ctx.tools.register(inspectFileTool)

  ctx.tools.register(checkServiceTool)

  console.log('[inspection-tools] registered')
}
```

这里：

```ts
export const name = 'inspection-tools'
```

表示插件名称。

```ts
export const inject = ['tools']
```

表示插件依赖 `tools` 服务。

下面两行：

```ts
ctx.tools.register(inspectFileTool)

ctx.tools.register(checkServiceTool)
```

分别注册两个工具。

到这里，插件已经具备两个工具，但它们还没有被调用。

**注册工具和执行工具是两件不同的事情。**

`ctx.tools.register()` 负责把工具放进注册表。

`ctx.tools.execute()` 才负责提交一次工具执行请求。

插件卸载时，注册操作产生的清理函数会由 Cordis 的插件生命周期管理。

### 5.12.7 主动调用两个工具

现在开始验证工具是否真的能够执行。

为了不让插件每次启动都自动执行巡检，我们单独编写一个测试函数。

继续修改：

```text
packages/experimental/inspection-tools/src/index.ts
```

增加导入：

```ts
import { CallId } from '@deepseek-ai/dsh-llm'
```

然后在文件末尾增加：

```ts
export async function runInspectionDemo(ctx: Context) {
  const signal = new AbortController().signal

  // 调用第一个工具。
  const fileResult = await ctx.tools.execute({
    callId: CallId('inspection-demo-file-1'),
    name: 'inspect_file',
    arguments: {
      path: 'sample.txt',
    },
    signal,
  })

  console.log('[inspection-demo] inspect_file result:')

  console.log(JSON.stringify(fileResult, null, 2))

  // 调用第二个工具。
  const serviceResult = await ctx.tools.execute({
    callId: CallId('inspection-demo-service-1'),
    name: 'check_service',
    arguments: {
      service: 'api',
    },
    signal,
  })

  console.log('[inspection-demo] check_service result:')

  console.log(JSON.stringify(serviceResult, null, 2))
}
```

现在有一个需要特别理解的调用：

```ts
ctx.tools.execute({
  callId: CallId('inspection-demo-file-1'),
  name: 'inspect_file',
  arguments: {
    path: 'sample.txt',
  },
  signal,
})
```

它提交的是一次工具执行请求。

参数含义：

| 字段 | 作用 |
|---|---|
| `callId` | 本次工具调用的关联 ID |
| `name` | 要执行的工具名称 |
| `arguments` | 工具参数对象 |
| `signal` | 本次调用使用的取消信号 |

这里的 `arguments` 是对象，不是 JSON 字符串。

正确：

```ts
arguments: {
  path: 'sample.txt',
}
```

不是：

```ts
arguments: JSON.stringify({
  path: 'sample.txt',
})
```

另外：

```ts
CallId('inspection-demo-file-1')
```

创建的是工具调用关联 ID。

它不是工具执行流水线内部的 `ToolExecutionToken`。后者由工具注册表自己分配。

**如何实际运行这个测试？**

在确认插件已经加载、两个工具已经注册之后，可以临时在 `apply()` 末尾增加：

```ts
void runInspectionDemo(ctx).catch(error => {
  console.error('[inspection-demo] failed:', error)
})
```

这样插件加载时就会主动执行两个工具。

观察完成后，把这段测试调用删除。

不要把它作为正式插件的默认行为，否则每次启动都会自动发起巡检。

### 5.12.8 查看工具执行结果

`ctx.tools.execute()` 返回的是 `ToolExecutionResult`。

成功结果包含：

```ts
{
  isError: false,
  value: {
    path: 'sample.txt',
    type: 'file',
    size: 21,
    modifiedAt: '实际文件修改时间'
  },
  content: [
    {
      type: 'text',
      text: '...'
    }
  ]
}
```

失败结果包含：

```ts
{
  isError: true,
  error: {
    message: '错误信息'
  },
  content: [
    {
      type: 'text',
      text: '...'
    }
  ]
}
```

成功结果有 `value`，失败结果没有 `value`。

因此，实际开发时应该先判断 `isError`：

```ts
if (fileResult.isError) {
  console.error('文件检查失败:', fileResult.error)
} else {
  console.log('文件检查成功:', fileResult.value)
}
```

第二个工具也一样：

```ts
if (serviceResult.isError) {
  console.error('服务检查失败:', serviceResult.error)
} else {
  console.log('服务检查成功:', serviceResult.value)
}
```

这里需要区分两个字段：

| 字段 | 作用 |
|---|---|
| `value` | 工具返回的规范结构化数据 |
| `content` | 经过渲染后，提供给模型的内容块 |

例如，`inspect_file` 返回的规范数据是一个对象：

```json
{
  "path": "sample.txt",
  "type": "file",
  "size": 21,
  "modifiedAt": "实际文件修改时间"
}
```

经过 `render()` 处理后，模型收到的是文本内容块。

因此，工具开发时不能只考虑最终显示出来的字符串，还要考虑规范输出结构。

### 5.12.9 关于 timeoutMs

在 `check_service` 中，我们增加了：

```ts
timeoutMs: 5000,
```

它表示该工具声明了一个 5000 毫秒的协作式超时预算。

但要注意：

`timeoutMs` 本身不会自动终止工具。

Harness 中由专门的策略插件负责读取这个字段，并在 `tools/execute` 环绕分发阶段设置超时信号。

对应插件：

```text
@deepseek-ai/dsh-tool-call-timeout-policy
```

它在仓库中的位置：

```text
packages/guard/timeout-policy/
```

需要在实际运行配置中加载这个策略插件。

另外，工具自身也必须正确传递：

```ts
exec.signal
```

例如：

```ts
await fetch(url, {
  signal: exec.signal,
})
```

如果工具忽略取消信号，即使声明了 `timeoutMs`，也不能认为底层工作已经安全停止。

这里还要区分：

- `timeoutMs`：工具声明的超时预算。
- `exec.signal`：工具执行时接收的取消信号。
- 超时策略插件：负责将超时预算转化为实际的取消信号。

三者共同构成工具超时机制。

### 5.12.10 测试失败场景

工具开发不能只测试成功路径。

还需要测试以下情况。

**实验一：访问允许目录之外的文件**

调用：

```ts
const result = await ctx.tools.execute({
  callId: CallId('inspection-invalid-path'),
  name: 'inspect_file',
  arguments: {
    path: '../package.json',
  },
  signal: new AbortController().signal,
})

console.log(result.isError)
console.log(result.content)
```

预期：

```text
isError = true
```

观察：

- 参数是否通过 Schema 校验？
- 真正阻止路径越界的是哪段代码？

答案是：Schema 只能检查参数类型和结构，路径边界由 `inspectFile()` 内部的路径检查负责。

**实验二：传入非法服务名称**

调用：

```ts
const result = await ctx.tools.execute({
  callId: CallId('inspection-invalid-service'),
  name: 'check_service',
  arguments: {
    service: 'google',
  },
  signal: new AbortController().signal,
})

console.log(result.isError)
console.log(result.content)
```

预期：

```text
isError = true
```

观察：

- `service` 是否符合参数 Schema？
- 工具主体是否会执行？

由于 `google` 不属于枚举值，参数校验会失败，工具主体不会执行。

**实验三：传入错误参数类型**

调用：

```ts
const result = await ctx.tools.execute({
  callId: CallId('inspection-invalid-type'),
  name: 'inspect_file',
  arguments: {
    path: 123,
  },
  signal: new AbortController().signal,
})

console.log(result.isError)
console.log(result.content)
```

观察：

- Schema 能否识别类型错误？
- 参数校验失败后，是否还会执行文件读取逻辑？

**实验四：访问不存在的文件**

调用：

```ts
const result = await ctx.tools.execute({
  callId: CallId('inspection-missing-file'),
  name: 'inspect_file',
  arguments: {
    path: 'missing.txt',
  },
  signal: new AbortController().signal,
})

console.log(result.isError)
console.log(result.content)
```

观察：

- 参数是否通过 Schema 校验？
- 文件不存在时，错误从哪里产生？
- 工具失败后是否仍然返回结构化结果？

**实验五：访问不存在的服务**

将 `SERVICES.api` 临时改成一个无法访问的测试端点。

观察：

- 网络请求失败时，工具如何返回错误？
- 取消信号是否能够传递到 `fetch()`？
- 网络失败和 HTTP 503 有什么区别？

### 5.12.11 本节总结

现在回顾整个工具开发过程：

```text
inspect-file.ts
    |
    v
defineTool()
    |
    v
生成 inspectFileTool
    |
    |
check-service.ts
    |
    v
defineTool()
    |
    v
生成 checkServiceTool
    |
    v
index.ts
    |
    v
ctx.tools.register()
    |
    v
两个工具进入工具注册表
    |
    v
ctx.tools.execute()
    |
    v
提交工具执行请求
    |
    v
参数校验与工具执行流水线
    |
    v
执行工具主体
    |
    v
输出校验与渲染
    |
    v
ToolExecutionResult
```

本节最重要的是区分三个操作：

| 操作 | 对应 API | 作用 |
|---|---|---|
| 定义工具 | `defineTool()` | 描述工具名称、参数、输出和执行逻辑 |
| 注册工具 | `ctx.tools.register()` | 将工具加入注册表 |
| 执行工具 | `ctx.tools.execute()` | 主动提交一次工具执行请求 |

还要记住：

**执行工具不等于调用模型。**

本节直接构造 `ToolExecutionInput`，跳过了模型调用。

真实 Agent 工作流程中：

1. Agent Loop 组织消息和模型请求。
2. 模型根据工具 Schema 决定是否提出工具调用。
3. Agent Loop 接收模型返回的 Tool Call。
4. Agent Loop 将调用请求交给工具执行流水线。
5. 工具执行完成后，Agent Loop 再组织工具结果和后续模型请求。

本节只是为了单独研究工具执行机制，才直接调用 `ctx.tools.execute()`。

### 5.12.12 本节验收

完成本节后，应该能够独立回答：

1. `defineTool()` 和 `ctx.tools.register()` 有什么区别？
2. 为什么注册工具之后，它不会自动执行？
3. 为什么 `ctx.tools.execute()` 不需要调用 LLM？
4. `ToolExecutionInput` 中的 `name` 和 `arguments` 分别有什么作用？
5. 为什么参数符合 Schema，不代表文件路径一定安全？
6. `exec.signal` 与 `AbortController` 有什么关系？
7. `timeoutMs` 为什么不能单独保证工具停止？
8. `output.schema` 和 `output.render()` 分别负责什么？
9. `ToolExecutionResult` 中的 `value` 和 `content` 有什么区别？
10. `CallId` 和 `ToolExecutionToken` 有什么区别？

如果这些问题都能回答，说明已经掌握了从定义工具、注册工具到主动执行工具的基本过程。

下一步回到 Agent Loop，追踪模型返回的 Tool Call 是如何进入这条工具执行流水线的。


## 5.13 从测试反向理解执行边界

源码阅读不应该只看实现。

测试可以告诉我们：

哪些行为是系统明确保证的。

建议按下面的顺序阅读：

```text
packages/core/tools/
packages/core/tool-testkit/
packages/core/agent-loop/
```

重点寻找以下测试：

| 测试主题 | 需要验证的行为 |
|---|---|
| Registry | 注册、注销、名称冲突 |
| Scope | 全局工具、局部工具、限制规则 |
| Schema | 参数合法性和输出合法性 |
| Pipeline | 执行前、执行中、执行后策略 |
| Guard | 单调拒绝是否生效 |
| Cancellation | 取消前后错误分类 |
| Timeout | 超时是否等待底层任务结束 |
| Batch | 多工具调用顺序和并发 |
| Session | 工具调用与结果是否正确记录 |

### 测试中应该重点关注什么？

例如，测试一个未知工具时，不要只看：

```ts
expect(result.isError).toBe(true)
```

还要检查：

```ts
expect(toolExecute).not.toHaveBeenCalled()
```

因为真正重要的是：

未知工具没有被错误地执行。

再例如，测试超时时：

不能只检查：

```ts
expect(result.error.code).toBe('TOOL_TIMEOUT')
```

还要检查：

```ts
expect(backgroundTaskFinished).toBe(true)
```

因为外层已经返回超时，不代表后台任务已经结束。

测试应该验证真正的执行边界，而不是只验证一个表面返回值。

---

## 5.14 本章源码阅读路线

本章源码不要从整个仓库目录树开始。

按照真实调用关系阅读。

### 第一阶段：认识 Tool Registry

阅读：

```text
packages/core/tools/src/index.ts
```

顺序：

```text
ToolsService
   |
   v
register()
   |
   v
restrict()
   |
   v
resolve()
   |
   v
definitions()
   |
   v
schemas()
   |
   v
setExecutor()
   |
   v
execute()
```

目标：

理解工具从注册到进入当前 Agent 工具集合的过程。

### 第二阶段：理解 ToolDefinition

阅读：

```text
packages/core/tools/src/contracts.ts
```

重点：

```ts
ToolDefinition
ToolExecutionInput
ToolExecution
ToolDispatchExecution
ToolRunContext
ToolExecutionResult
PreToolDecision
PostToolDecision
ToolGuard
```

目标：

理解工具执行过程中，各阶段传递的数据结构。

### 第三阶段：理解参数与输出校验

阅读：

```text
packages/core/tools/src/schema.ts
packages/core/tools/src/json-schema.ts
```

重点：

```ts
defineTool()
validateArgs()
ToolArgsError
ToolOutputError
```

目标：

理解参数校验和输出校验分别在哪里发生。

### 第四阶段：理解事件扩展点

阅读：

```text
packages/core/tools/src/events.ts
```

重点：

```text
tools/pre-execute
tools/execute
tools/post-execute
tools/result
tools/change
```

目标：

理解每个事件的分发模式、参数和返回值。

### 第五阶段：理解真正的执行流水线

阅读：

```text
packages/core/tools/src/pipeline.ts
```

重点：

```text
prepare
dispatch
complete
```

目标：

找到每个阶段真正执行的代码。

不要只看函数名称。

要回答：

- 参数在哪里被快照？
- 工具在哪里被定位？
- 参数在哪里被校验？
- Guard 在什么时候执行？
- 工具函数在哪里被调用？
- 输出在哪里被规范化？
- 最终事件在哪里触发？

### 第六阶段：理解批次执行

阅读：

```text
packages/core/tools/src/batch.ts
packages/core/tools/src/executor.ts
```

目标：

理解一个模型响应中包含多个 Tool Call 时，Executor 如何调度。

### 第七阶段：追踪 Agent Loop

阅读：

```text
packages/core/agent-loop/src/agent.ts
```

搜索：

```ts
this.loopCtx.tools.execute
```

目标：

理解模型响应中的工具调用如何进入 Registry 和 Executor。

以及执行结果如何重新进入 Session。

### 第八阶段：理解超时与审批

阅读：

```text
packages/guard/timeout-policy/
packages/interaction/user-approval/
packages/sandbox/sandbox-policy/
```

目标：

理解：

- 超时策略如何包装工具执行。
- 审批服务如何返回封闭决策。
- Sandbox Policy 如何向工具和执行边界提供安全约束。

---

## 5.15 本章需要形成的架构认识
    
学习完本章，应该能区分下面几个对象：

| 对象 | 负责什么 |
|---|---|
| ToolDefinition | 描述工具的输入、输出和执行逻辑 |
| Tool Registry | 管理工具注册、可见性和执行器 |
| ResolvedToolSet | 当前 Agent 当前步骤允许使用的工具集合 |
| ToolExecutionPipeline | 处理单次工具调用的执行流程 |
| ToolExecutor | 调度一批工具调用 |
| Policy | 决定工具调用是否允许 |
| AbortSignal | 将取消意图传递给实际执行工作 |
| Session | 记录模型调用和工具结果 |

特别要记住：

```text
Tool Registry != Tool Executor

Tool Schema != Tool Definition

参数校验 != 权限检查

超时 != 取消

取消请求 != 底层工作已经停止

工具执行成功 != 结果已经被最终接受

工具结果 != UI 展示对象
```

这些区别是理解 Harness 工具系统的基础。

---

## 本章验收

完成本章后，应该能够回答下面的问题。

### Tool Registry

1. Tool Registry 主要负责哪些工作？
2. `register()` 和 `execute()` 有什么区别？
3. `resolve()` 为什么需要 Agent 和 AbortSignal？
4. `ResolvedToolSet` 是什么？
5. `schemas()` 为什么不能直接把 ToolDefinition 交给模型？
6. `restrict()` 和注销工具有什么区别？
7. 全局工具和 Scope 自己注册的工具有什么区别？

### ToolDefinition

8. `defineTool()` 做了什么？
9. `parameters` 和 `output.schema` 分别约束什么？
10. `execute()` 和 `render()` 有什么区别？
11. 为什么输出也需要 Schema？
12. 为什么 `timeoutMs` 不是模型可见字段？

### 执行流水线

13. ToolExecution 是什么？
14. 为什么需要 `callId` 和内部 `token`？
15. 为什么工具参数需要做快照？
16. `tools/pre-execute` 能做什么？
17. 为什么 `tools/execute` 是环绕调用？
18. `tools/post-execute` 能否修改最终结果？
19. `tools/result` 为什么适合审计？
20. Guard 为什么设计成单调拒绝？

### 安全边界

21. 参数 Schema 为什么不能代替权限检查？
22. 为什么路径字符串符合 Schema 仍然可能不安全？
23. 为什么网络工具不能允许模型自由指定 URL？
24. 为什么不能只用 `Promise.race()` 实现可靠超时？
25. AbortSignal 为什么属于协作式取消？
26. `ABORTED` 和 `ABORTED_BEFORE_DISPATCH` 有什么区别？
27. 为什么 Executor 必须等待已经启动的工作达到 quiescence？

### Session 与恢复

28. 为什么工具结果必须是可序列化的数据？
29. 为什么不能把工具异常转换为空字符串？
30. 工具调用和结果为什么需要关联？
31. 为什么执行结果和 UI 展示结果需要分开？
32. 工具执行失败后，Agent 为什么还需要正确记录失败结果？

---

## 本章小结

本章真正要掌握的不是如何写一个工具，而是理解工具从模型请求到宿主程序执行的完整边界。

第一，模型只能提出工具调用。

它不能决定工具是否存在，也不能决定自己是否有权限执行。

第二，Tool Registry 负责管理工具能力。

它管理注册、Scope、可见性、当前步骤的工具集合，以及执行器选择。

第三，ToolExecutionPipeline 负责处理单次调用。

它将参数、策略、实际执行、结果规范化和最终通知组织成一条执行流程。

第四，ToolExecutor 负责一批调用。

它需要考虑多个工具之间的执行顺序、并发、取消和结果记录。

第五，工具结果必须能够被规范化并持久化。

不能把任意 JavaScript 对象直接交给 Session，也不能把执行失败伪装成成功。

第六，超时和取消必须落实到真正的执行工作。

外层返回超时，不代表文件操作、网络请求或子进程已经停止。

最后记住这条主线：

```text
模型提出调用
       |
       v
Tool Registry 确认能力
       |
       v
Schema 验证参数
       |
       v
Policy 判断是否允许
       |
       v
Executor 调度执行
       |
       v
ToolExecutionPipeline 处理调用
       |
       v
结果规范化
       |
       v
Session 记录结果
       |
       v
Agent 继续运行
```

以后阅读 Harness 中任何工具插件，都可以沿着这条主线追踪：

先看工具在哪里定义，再看在哪里注册，然后看它进入哪个执行器，接着看执行前后的策略，最后确认结果如何进入 Session。

这才是理解 Tool Registry 与执行边界的正确方式。