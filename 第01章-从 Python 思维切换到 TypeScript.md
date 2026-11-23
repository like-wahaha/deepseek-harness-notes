- [第 1 章：从 Python 思维切换到 TypeScript](#第-1-章从-python-思维切换到-typescript)
  - [1.1 TypeScript 的类型不是运行时校验](#11-typescript-的类型不是运行时校验)
  - [1.2 需要熟悉的类型语法](#12-需要熟悉的类型语法)
    - [1.2.1 `interface`：描述对象的形状](#121-interface描述对象的形状)
    - [1.2.2 `type`：给类型表达式命名](#122-type给类型表达式命名)
    - [1.2.3 泛型：让类型跟随输入变化](#123-泛型让类型跟随输入变化)
    - [1.2.4 联合类型与交叉类型](#124-联合类型与交叉类型)
    - [1.2.5 常见工具类型](#125-常见工具类型)
  - [1.3 `unknown`、`any` 与类型收窄](#13-unknownany-与类型收窄)
    - [1.3.1 `any`：关闭这部分类型检查](#131-any关闭这部分类型检查)
    - [1.3.2 `unknown`：可以接收任何值，但不能未经检查就使用](#132-unknown可以接收任何值但不能未经检查就使用)
    - [1.3.3 什么是类型收窄（narrowing）](#133-什么是类型收窄narrowing)
    - [1.3.4 从 `unknown` 安全构造业务对象](#134-从-unknown-安全构造业务对象)
    - [1.3.5 `as Config` 是类型断言，不是验证](#135-as-config-是类型断言不是验证)
  - [1.4 Promise 与异步控制](#14-promise-与异步控制)
    - [1.4.1 Promise 表示一个未来才会完成的结果](#141-promise-表示一个未来才会完成的结果)
    - [1.4.2 `await`、`return` 和 `void` 的区别](#142-awaitreturn-和-void-的区别)
    - [1.4.3 异常如何传播](#143-异常如何传播)
    - [1.4.4 顺序执行与并发执行](#144-顺序执行与并发执行)
    - [1.4.5 `AbortController`：把取消意图传给任务](#145-abortcontroller把取消意图传给任务)
  - [1.5 ESM、模块和声明合并](#15-esm模块和声明合并)
    - [1.5.1 模块解决什么问题](#151-模块解决什么问题)
    - [1.5.2 常见导入导出语法](#152-常见导入导出语法)
    - [1.5.3 声明合并与 Cordis Context 类型扩展](#153-声明合并与-cordis-context-类型扩展)
    - [1.5.4 workspace package 与依赖方向](#154-workspace-package-与依赖方向)
  - [1.6 Node.js 必须掌握的部分](#16-nodejs-必须掌握的部分)
    - [1.6.1 进程参数与环境变量](#161-进程参数与环境变量)
    - [1.6.2 文件系统与路径](#162-文件系统与路径)
    - [1.6.3 stdout、stderr 与退出码](#163-stdoutstderr-与退出码)
    - [1.6.4 子进程](#164-子进程)
    - [1.6.5 AbortController 与子进程清理](#165-abortcontroller-与子进程清理)
  - [本章实验：配置验证、泛型缓存与可取消子进程](#本章实验配置验证泛型缓存与可取消子进程)
    - [实验目录](#实验目录)
    - [1. 创建项目](#1-创建项目)
    - [2. 验证配置：`src/config.ts`](#2-验证配置srcconfigts)
    - [3. 泛型缓存：`src/cache.ts`](#3-泛型缓存srccachets)
    - [4. 子进程工作脚本：`src/worker.ts`](#4-子进程工作脚本srcworkerts)
    - [5. 启动并取消子进程：`src/main.ts`](#5-启动并取消子进程srcmaints)
    - [6. 运行与预期观察](#6-运行与预期观察)
    - [7. 故障注入](#7-故障注入)
    - [本章练习与参考答案](#本章练习与参考答案)
    - [本章源码映射：把基础知识带回 Harness](#本章源码映射把基础知识带回-harness)



# 第 1 章：从 Python 思维切换到 TypeScript

本章不是要求 Python 开发者重新学习一门语言的全部语法，而是建立阅读 Harness 源码所需的心智模型。重点是：TypeScript 的类型系统如何帮助维护大型程序、哪些检查只发生在编译期、Node.js 如何组织模块和异步任务，以及这些知识如何影响插件、Tool、Session 和 Agent Loop 的阅读。

学完本章，读者应当能看懂常见的 TypeScript 类型声明，判断外部数据是否经过运行时验证，追踪一个 Promise 的等待与错误传播，理解 ESM 模块边界，并能解释 Node.js CLI 的进程、路径和子进程行为。

## 1.1 TypeScript 的类型不是运行时校验

TypeScript 是 JavaScript 的静态类型检查层。开发时，编译器根据类型信息检查代码是否存在明显的不一致；运行时，Node.js 执行的仍然是 JavaScript。通常情况下，接口、类型别名、泛型参数和类型断言都会在编译输出中消失。

先看一个最小例子：

```ts
function add(a: number, b: number): number {
  return a + b
}

console.log(add(1, 2))
```

这里的三个 `number` 分别表示两个参数应当是数字、函数返回值应当是数字。它们能帮助编辑器提示错误，也能让编译器拒绝一部分错误调用：

```ts
// 编译期会报错：第二个参数不是 number
add(1, '2')
```

但类型标注不会在运行时替你检查 JSON：

```ts
const raw = JSON.parse('{"a":"hello","b":2}') as {
  a: number
  b: number
}

console.log(raw.a + raw.b)
```

这段代码可能通过 TypeScript 编译，因为 `as` 告诉编译器“把它当成这个类型看待”；但 JSON 中的 `a` 实际上仍然是字符串。运行时执行的是 JavaScript 的加法规则，结果可能是字符串拼接，而不是数字相加。

因此必须区分两件事：

- **静态类型检查**：在开发和构建阶段，检查代码如何使用值。它主要保护程序内部的代码协作。
- **运行时数据校验**：程序运行时检查实际收到的数据是否符合要求。它保护配置文件、网络请求、模型输出、数据库记录和插件输入等边界。

一个有用的判断原则是：**凡是从当前进程之外进入系统的数据，都先当作不可信数据处理。** 例如：

- 用户编辑的 JSON/YAML 配置；
- Remote Client 发来的请求；
- LLM 生成的 Tool 参数或结构化输出；
- 磁盘上历史版本留下的 Session 记录；
- 第三方插件返回的对象。

读 Harness 时，看到 `interface` 或 `type` 不要立刻认为数据已验证。继续向上追踪：数据从哪里来？是否经过 schema parser、显式检查或解码器？校验失败时是抛异常、返回错误对象，还是拒绝请求？

在真实项目中，运行时校验可以用手写检查，也可以用 schema 库。无论使用哪一种，核心流程都一样：

1. 接收原始值；
2. 把它视为未知结构；
3. 检查对象形状、字段类型、范围和业务约束；
4. 校验成功后，构造内部可信的数据对象；
5. 后续业务逻辑只处理通过边界检查的数据。

## 1.2 需要熟悉的类型语法

### 1.2.1 `interface`：描述对象的形状

```ts
interface User {
  id: string
  name: string
  enabled?: boolean
}

const user: User = {
  id: 'u-001',
  name: 'Ada',
}
```

`interface` 描述一个对象应当具备哪些属性、每个属性是什么类型。`enabled?: boolean` 中的问号表示该属性可以缺省；如果提供，它必须是布尔值。

注意，“可选”不等于“可以随便填”。下面的对象仍然不符合 `User`：

```ts
// 错误：id 应为 string
const badUser: User = { id: 1, name: 'Ada' }
```

接口主要用于描述对象契约，常见于配置、事件载荷、服务 API 和插件 Context。接口本身不会创建对象，也不会在运行时自动检查对象。

### 1.2.2 `type`：给类型表达式命名

```ts
type UserId = string

type Result =
  | { ok: true; value: string }
  | { ok: false; error: Error }
```

类型别名可以给基础类型、对象类型、联合类型、交叉类型、函数类型等命名。上面的 `Result` 是一个联合类型，表示结果只能是成功形态或失败形态之一。

```ts
function printResult(result: Result): void {
  if (result.ok) {
    console.log(result.value)
  } else {
    console.error(result.error.message)
  }
}
```

这里的 `ok` 是区分字段（discriminant）。判断 `result.ok` 后，TypeScript 能推断当前分支是哪一种对象，因此成功分支可以访问 `value`，失败分支可以访问 `error`。

这比设计一个包含大量可选字段的对象更安全。例如，不推荐用 `{ ok?: boolean; value?: string; error?: Error }` 表达结果，因为它允许出现“成功和失败字段同时存在”或“什么字段都没有”的模糊状态。

### 1.2.3 泛型：让类型跟随输入变化

泛型不是某一种固定的数据类型，而是类型层面的参数。可以把它理解为“写函数时先不决定具体类型，调用时再把类型代入”。

```ts
function first<T>(items: T[]): T | undefined {
  return items[0]
}

const n = first([10, 20, 30])   // T 被推导为 number
const s = first(['a', 'b'])    // T 被推导为 string
```

`T[]` 表示 T 类型的数组；`T | undefined` 表示返回值要么是 T，要么是 `undefined`。数组可能为空，因此不能承诺一定返回一个元素。

如果没有泛型，初学者可能写成：

```ts
function firstNumber(items: number[]): number | undefined {
  return items[0]
}

function firstString(items: string[]): string | undefined {
  return items[0]
}
```

两个函数的逻辑完全相同，只是数据类型不同。泛型把重复逻辑抽象出来，同时保留调用者的具体类型。

泛型也可以有约束：

```ts
function getId<T extends { id: string }>(value: T): string {
  return value.id
}
```

`T extends { id: string }` 表示 T 可以是任意类型，但必须至少具备 `id: string`。这让函数能够安全访问 `value.id`，同时不丢失对象的其他字段类型。

在 Harness 中，泛型常用于事件载荷、工具结果、服务接口、缓存和通用容器。读到 `<T>` 时，要问：T 是从参数推导出来的，还是由调用者显式指定？有没有 `extends` 约束？返回值是否保留了输入的类型信息？

### 1.2.4 联合类型与交叉类型

联合类型 `A | B` 表示一个值可以是 A 或 B；交叉类型 `A & B` 表示一个值同时满足 A 和 B 的结构要求。

```ts
type StringOrNumber = string | number

type Identified = { id: string }
type Named = { name: string }
type Entity = Identified & Named

const entity: Entity = { id: 'e-1', name: 'document' }
```

联合类型通常需要先判断具体是哪一种，再使用该分支独有的成员。交叉类型常用于组合能力或扩展对象形状。

### 1.2.5 常见工具类型

TypeScript 内置工具类型可以从已有类型派生新类型，减少重复声明：

| 工具类型 | 作用 | 示例 |
| --- | --- | --- |
| `Promise<T>` | 描述异步结果最终得到的 T | `Promise<User>` |
| `Record<K, V>` | 创建键为 K、值为 V 的对象类型 | `Record<string, number>` |
| `Partial<T>` | 将 T 的所有属性变成可选 | `Partial<User>` |
| `Required<T>` | 将 T 的所有属性变成必选 | `Required<User>` |
| `Readonly<T>` | 将 T 的属性标记为只读 | `Readonly<User>` |
| `Pick<T, K>` | 从 T 中选取 K 指定的属性 | `Pick<User, 'id'>` |
| `Omit<T, K>` | 从 T 中排除 K 指定的属性 | `Omit<User, 'enabled'>` |
| `ReturnType<typeof fn>` | 获取函数返回值类型 | `ReturnType<typeof loadUser>` |
| `Parameters<typeof fn>` | 获取函数参数元组类型 | `Parameters<typeof loadUser>` |
| `Exclude<T, U>` | 从联合类型 T 中排除可赋给 U 的成员 | `Exclude<'a' \\| 'b', 'a'>` |
| `Extract<T, U>` | 从 T 中提取可赋给 U 的成员 | `Extract<'a' \\| 'b', 'a'>` |
| `Awaited<T>` | 递归解开 Promise 的最终值类型 | `Awaited<Promise<User>>` |

看几个具体例子：

```ts
interface Config {
  endpoint: string
  token: string
  timeout: number
}

type ConfigPatch = Partial<Config>
// { endpoint?: string; token?: string; timeout?: number }

type PublicConfig = Omit<Config, 'token'>
// { endpoint: string; timeout: number }

type ConfigByKey = Record<'dev' | 'prod', Config>
// 必须有 dev 和 prod 两个键，每个值都是 Config
```

`Partial<T>` 常用于“局部更新”参数，但不要把它误当成已经验证过的配置。`Readonly<T>` 也只是类型层面的限制，并不等于深度冻结对象；嵌套对象仍可能被修改，运行时也可以通过其他引用改变它。

## 1.3 `unknown`、`any` 与类型收窄

这是阅读配置解析、Remote API、Tool 参数和 LLM 输出时最重要的基础之一。

### 1.3.1 `any`：关闭这部分类型检查

`any` 相当于告诉 TypeScript：“不用继续检查这个值，我自己保证它是什么。” 它能让代码快速通过编译，但也会让后续操作失去类型保护。

```ts
let a: any = 123

a.foo()                  // TypeScript 不报错
let text: string = a     // TypeScript 允许赋值

console.log(a.foo())     // 运行时：TypeError，a.foo 不是函数
```

问题在于：编译器无法再帮助你发现错误。`any` 还会沿着赋值和函数调用传播，使原本有类型保护的代码逐渐变成“相信开发者不会写错”。

`any` 并非绝对不能用。它可能出现在迁移旧项目、接入类型缺失的第三方库或编写非常底层的兼容代码时。但应把它限制在很小的边界，并尽快转换成明确类型。不要为了消除红色波浪线，就把一串变量全部标成 `any`。

### 1.3.2 `unknown`：可以接收任何值，但不能未经检查就使用

```ts
let u: unknown = 123

// u.foo()                 // 编译错误：不能直接访问未知值的属性
// const s: string = u     // 编译错误：不能直接把 unknown 当 string

if (typeof u === 'string') {
  // 进入这个分支后，TypeScript 知道 u 是 string
  console.log(u.length)
}
```

`unknown` 与 `any` 都可以接收任意值，但使用规则不同：

- `any`：允许直接访问属性、调用方法、传给其他类型，编译器基本不再保护你。
- `unknown`：允许接收和保存值，但在读取属性、调用方法或赋给更具体类型之前，必须先证明它符合要求。

所以外部输入优先声明为 `unknown`。这会迫使代码在真正使用数据前完成检查。

### 1.3.3 什么是类型收窄（narrowing）

类型收窄是指：通过条件判断、控制流或显式检查，把一个较宽泛的类型缩小成更具体的类型。

```ts
function printLength(value: string | number): void {
  if (typeof value === 'string') {
    // 此处 value 是 string
    console.log(value.length)
  } else {
    // 此处 value 是 number
    console.log(value.toFixed(2))
  }
}
```

函数参数一开始是 `string | number`。`typeof` 判断把程序分成两个分支，编译器根据分支条件推断每个分支里的类型。

常见收窄方式包括：

```ts
function inspect(value: unknown): string {
  if (typeof value === 'string') {
    return value.trim()
  }

  if (typeof value === 'number' && Number.isFinite(value)) {
    return value.toFixed(2)
  }

  if (Array.isArray(value)) {
    return `array(${value.length})`
  }

  if (value === null) {
    return 'null'
  }

  if (typeof value === 'object') {
    return 'object'
  }

  return 'other'
}
```

注意两个细节：

1. `typeof null === 'object'` 是 JavaScript 历史遗留行为，所以检查普通对象时通常要同时排除 `null`。
2. `Array.isArray()` 可以识别数组；数组本身也是对象，但通常需要单独处理。

### 1.3.4 从 `unknown` 安全构造业务对象

假设 Tool 接收一个搜索参数：

```ts
interface SearchConfig {
  query: string
  topK: number
}

function parseSearchConfig(raw: unknown): SearchConfig {
  if (typeof raw !== 'object' || raw === null || Array.isArray(raw)) {
    throw new TypeError('config must be a plain object')
  }

  // 先把对象视为“属性值仍未知”的字典。
  const record = raw as Record<string, unknown>

  if (typeof record.query !== 'string' || record.query.trim() === '') {
    throw new TypeError('query must be a non-empty string')
  }

  if (
    typeof record.topK !== 'number' ||
    !Number.isInteger(record.topK) ||
    record.topK < 1 ||
    record.topK > 20
  ) {
    throw new TypeError('topK must be an integer between 1 and 20')
  }

  // 返回新对象，而不是把未经验证的 raw 直接冒充为 SearchConfig。
  return {
    query: record.query,
    topK: record.topK,
  }
}

const valid = parseSearchConfig(JSON.parse('{"query":"Harness","topK":5}'))
console.log(valid.query, valid.topK)
```

这里的断言 `raw as Record<string, unknown>` 只是在完成“它是非 null 对象且不是数组”的检查后，告诉编译器按字典方式读取属性；每个属性仍然是 `unknown`，因此还要分别验证 `query` 和 `topK`。最后重新构造一个 `SearchConfig`，让返回值只包含经过验证的字段。

如果数据结构复杂、字段多、需要复用错误信息，实际项目通常使用运行时 schema 库。手写校验适合教学和简单边界；复杂系统应避免散落着几十段不一致的手写判断。

### 1.3.5 `as Config` 是类型断言，不是验证

```ts
const config = raw as SearchConfig
```

这句话只影响 TypeScript 编译器如何看待 `raw`，不会检查 `raw.query` 是否是字符串，也不会补上缺失字段。它不等于转换数据，更不等于验证数据。

类型断言适合以下情况：

- 已经通过运行时校验，但编译器无法从控制流中推导出结果；
- 某个外部 API 有明确保证，而它的类型声明不完整；
- 操作 DOM 等场景中，开发者确实掌握额外信息，并会处理断言不成立的情况。

不适合的用法是：收到模型输出后，直接 `JSON.parse(text) as SearchConfig`，然后相信字段一定正确。

TypeScript 还支持尖括号形式的断言：

```ts
const a = raw as SearchConfig
const b = <SearchConfig>raw
```

两者在普通 `.ts` 文件中表达相同的类型断言意图；在 `.tsx` 文件中，尖括号还可能被解析成 JSX，因此通常使用 `as`。再次强调：两种写法都不执行运行时验证。

**本节记忆规则：** 外部输入用 `unknown` 接住；用 `typeof`、`Array.isArray`、字段检查或 schema 做验证；验证成功后构造可信对象；不要用 `as` 代替验证。

## 1.4 Promise 与异步控制

### 1.4.1 Promise 表示一个未来才会完成的结果

文件读取、网络请求、数据库访问和子进程执行都可能需要时间。Node.js 不应该为了等待它们而阻塞整个 JavaScript 执行流程，因此很多 API 返回 Promise。

可以把 Promise 理解为一个“未来结果的句柄”：它最终会成功得到一个值（fulfilled），或失败并携带原因（rejected），也可能暂时还没有完成（pending）。

```ts
import { readFile } from 'node:fs/promises'

async function loadText(path: string): Promise<string> {
  const value = await readFile(path, 'utf8')
  return value
}
```

`Promise<string>` 表示函数返回一个 Promise，成功完成时得到字符串。`async` 函数总是返回 Promise；即使函数体里直接 `return 'hello'`，调用者拿到的仍是 Promise。

`await` 的作用是：在当前 async 函数中暂停后续语句，等 Promise 完成后再继续。它不会把 Node.js 进程的所有工作都冻结；事件循环仍然可以处理其他任务。

### 1.4.2 `await`、`return` 和 `void` 的区别

```ts
async function a() {
  const result = await task()
  return result
}

function b() {
  return task()
}

function c() {
  void task()
}
```

- `await task()`：当前函数等待任务完成；如果任务 reject，异常会在 `await` 位置抛出，可以由当前函数的 `try/catch` 捕获。
- `return task()`：不在当前函数里等待它，但把 Promise 返回给调用者；调用者可以继续等待或处理错误。
- `void task()`：明确丢弃表达式结果，不等待它。它不会自动处理拒绝；如果没有 `.catch()` 或统一错误处理机制，可能产生未处理的 Promise rejection。

一个常见错误：

```ts
async function saveLater() {
  void writeFile('data.json', '{}')
  console.log('function finished')
}
```

函数可能在文件真正写完之前就结束；写文件失败也没有在这里处理。若任务必须完成，应当 `await`；若确实是后台任务，应有明确的任务管理器、错误日志、取消策略和关闭时清理机制。

### 1.4.3 异常如何传播

```ts
async function loadConfig(): Promise<string> {
  try {
    return await readFile('config.txt', 'utf8')
  } catch (error) {
    console.error('读取配置失败', error)
    throw error
  }
}
```

Promise reject 经 `await` 后表现为当前 async 函数中的异常。`catch` 可以记录、转换或恢复错误；如果重新 `throw`，调用者仍然会收到失败的 Promise。

不要无意义地吞掉异常：

```ts
try {
  await runAgent()
} catch {
  // 什么也不做：调用者会误以为任务成功
}
```

如果决定恢复，应该返回明确的失败结果或采取可验证的降级策略。错误处理的责任必须清楚：谁记录日志、谁向用户报告、谁决定重试、谁负责清理资源。

### 1.4.4 顺序执行与并发执行

```ts
const a = await loadA()
const b = await loadB()
```

这通常表示先完成 A，再开始等待 B。如果 A 和 B 互不依赖，可以并发启动：

```ts
const [a, b] = await Promise.all([loadA(), loadB()])
```

`Promise.all` 在全部成功后返回结果数组；其中一个任务失败时，整体 Promise 会 reject。但要注意：`Promise.all` 本身不会自动取消其他已经启动的任务。需要取消时，必须由任务 API 支持 `AbortSignal`，并由调用者发出取消信号。

### 1.4.5 `AbortController`：把取消意图传给任务

```ts
const controller = new AbortController()

const timer = setTimeout(() => {
  controller.abort(new Error('time limit exceeded'))
}, 2_000)

try {
  await fetch('https://example.com', {
    signal: controller.signal,
  })
} finally {
  clearTimeout(timer)
}
```

`AbortController` 创建取消控制器，`signal` 是传给任务的取消信号，`abort()` 发出取消请求。它不是强制杀死任意 JavaScript 函数的魔法：被调用的 API 必须支持该信号，或者你的任务代码必须主动监听它并停止工作。

阅读 Harness 的异步代码时，逐个问：

1. 谁创建 Promise？
2. 谁负责 `await` 或把 Promise 返回给上层？
3. 谁捕获 reject？
4. 是否有超时、重试和并发限制？
5. 取消信号有没有一路传到真正执行 I/O 或模型请求的地方？
6. 任务取消后，流、子进程、监听器和临时文件是否被清理？

Agent Loop、LLM 流式生成、Tool 执行和 Session 持久化都离不开这些问题。

## 1.5 ESM、模块和声明合并

### 1.5.1 模块解决什么问题

模块把程序拆成可独立维护的文件，并通过 `export` 暴露能力、通过 `import` 使用其他模块的能力。

```ts
// math.ts
export function add(a: number, b: number): number {
  return a + b
}

// main.ts
import { add } from './math.js'

console.log(add(1, 2))
```

在 ESM 项目中，模块依赖关系是显式的。读一个 package 时，要从 `package.json` 的 `exports`、`main`、`types` 等字段了解外部入口，再沿着 `src/index.ts` 看公开导出。内部文件存在，不代表它就是稳定公开 API。

### 1.5.2 常见导入导出语法

```ts
import { createAgent } from './agent.js'
import type { AgentOptions } from './types.js'

export { createAgent }
export type { AgentOptions }
```

`import type` 明确表示只导入类型信息，避免把纯类型依赖误当成运行时依赖。TypeScript 的模块解析和最终 JavaScript 输出还会受 `tsconfig.json`、Node.js 版本和构建工具影响，因此不能只凭某一行 import 就推断运行时文件路径。

Node.js ESM 项目通常会在 `package.json` 中使用：

```json
{
  "type": "module"
}
```

这表示该 package 中符合规则的 `.js` 文件按 ES Module 解释。`.mjs` 明确表示 ESM，`.cjs` 明确表示 CommonJS。不要在没有检查 package 配置和构建设置的情况下混用两套模块系统。

### 1.5.3 声明合并与 Cordis Context 类型扩展

Cordis 插件经常通过 Context 提供服务。插件作者希望写出 `ctx.greeter.greet()` 时有类型提示，因此会使用 TypeScript 的模块声明扩展：

```ts
declare module '@deepseek-ai/cordis' {
  interface Context {
    greeter: {
      greet(name: string): string
    }
  }
}
```

这段代码的含义是：在编译器看来，`@deepseek-ai/cordis` 模块导出的 `Context` 接口还具有 `greeter` 属性。它只扩展类型声明，**不会在运行时创建 `ctx.greeter` 服务**。

运行时仍然必须由插件真正注册服务。概念上类似：

```ts
ctx.greeter = {
  greet(name) {
    return `Hello, ${name}`
  },
}
```

真实 Cordis API 的注册方式应以仓库当前版本为准；这里的代码只说明“类型声明”和“运行时对象”是两回事。若只写 `declare module`，编译器可能允许访问一个运行时根本不存在的属性，最终仍会报错。

### 1.5.4 workspace package 与依赖方向

大型仓库通常由多个 package 组成。workspace 让这些 package 在一个仓库里共同开发，但不意味着它们可以随意互相依赖。应通过 package 的公开导出建立依赖，而不是让一个 package 深入引用另一个 package 的内部文件。

阅读依赖时重点检查：

- package 的 `name`、`exports`、`dependencies`、`peerDependencies`；
- workspace 配置和包管理器锁文件；
- 是否出现循环依赖；
- 哪些 API 是公开契约，哪些只是内部实现。

在 Harness 二开中，优先使用公开插件 API；如果必须依赖内部路径，要把它记录为升级风险。

## 1.6 Node.js 必须掌握的部分

Node.js 不只是“运行 JavaScript 的程序”。它还提供进程、文件系统、路径、子进程、流和信号等能力。Harness CLI、构建脚本、插件加载、日志输出和任务取消都会使用这些 API。

### 1.6.1 进程参数与环境变量

```ts
console.log(process.argv)
console.log(process.env.NODE_ENV)
console.log(process.cwd())
```

- `process.argv`：当前 Node.js 进程的命令行参数数组。通常前两项分别是 Node 可执行文件路径和入口脚本路径，后面才是用户传入的参数。
- `process.env`：环境变量字典，值通常是字符串或 `undefined`。不要假设 `"false"` 会自动变成布尔值。
- `process.cwd()`：进程当前工作目录。它取决于启动命令时所在的目录，不一定是脚本所在目录，也不一定是仓库根目录。
- `process.exitCode`：设置进程最终退出码，通常比在业务逻辑中立即调用 `process.exit()` 更利于完成异步清理。

示例：

```ts
const portText = process.env.PORT ?? '3000'
const port = Number(portText)

if (!Number.isInteger(port) || port < 1 || port > 65535) {
  console.error('PORT must be an integer between 1 and 65535')
  process.exitCode = 1
} else {
  console.log(`listening port: ${port}`)
}
```

环境变量必须解析和验证。`process.env.PORT` 是字符串，不会因为变量名叫 PORT 就自动成为数字。

### 1.6.2 文件系统与路径

```ts
import { readFile } from 'node:fs/promises'
import path from 'node:path'

const configPath = path.resolve(process.cwd(), 'config.json')
const text = await readFile(configPath, 'utf8')
```

`node:fs/promises` 提供基于 Promise 的文件系统 API；`node:path` 用于按当前操作系统规则处理路径。不要手动用字符串拼接路径，例如 `cwd + '/config.json'`，因为 Windows 和 POSIX 的路径分隔符不同。

相对路径最容易造成 CLI bug。下面两个概念不同：

- 相对于 `process.cwd()`：相对于用户启动命令时所在的目录；
- 相对于当前模块：相对于代码文件自身的位置。

设计 CLI 时要明确配置文件采用哪一种规则，并在帮助文档和错误信息中说明。

### 1.6.3 stdout、stderr 与退出码

- `stdout`：正常输出，适合机器读取的结果或进度信息（具体取决于 CLI 约定）。
- `stderr`：诊断信息、警告和错误。
- 退出码 `0` 通常表示成功，非 `0` 表示失败；具体数值可由程序定义。

如果 CLI 要支持管道，例如 `tool list | jq ...`，就不要把装饰性进度条和调试日志混入 stdout，否则会破坏下游程序解析。错误信息通常写到 stderr，并设置非零退出码。

### 1.6.4 子进程

Node.js 可以通过 `node:child_process` 启动外部命令。常见方式包括：

- `spawn()`：适合长时间运行、需要持续读取 stdout/stderr 或交互的子进程；
- `exec()`：通过 shell 执行命令并缓冲输出，适合输出量可控的简单命令；
- `execFile()`：直接执行指定程序，通常不经过 shell。

执行外部输入拼接出的 shell 命令有注入风险。能传递可执行文件和参数数组时，优先使用不经过 shell 的方式，并对可执行文件和参数做白名单约束。

### 1.6.5 AbortController 与子进程清理

取消任务需要区分“发出取消请求”和“任务确实停止”。例如，向子进程发送终止信号后，仍要等待它退出、读取最终退出码，并处理它可能启动的子孙进程。不同操作系统的进程树行为也可能不同。

对任何长任务，都要明确：

- 谁持有子进程句柄；
- 谁监听 `error`、`exit`、`close`；
- 超时后发送什么信号；
- 是否需要终止整个进程组；
- 关闭主程序时是否清理仍在运行的子进程。

## 本章实验：配置验证、泛型缓存与可取消子进程

实验目标是把本章知识串起来：读取 JSON、把外部数据当作 `unknown`、完成运行时校验、使用泛型缓存、启动一个每秒输出进度的子进程，并在两秒后请求取消。

### 实验目录

```text
ts-node-lab/
  package.json
  tsconfig.json
  config.json
  src/
    config.ts
    cache.ts
    worker.ts
    main.ts
```

### 1. 创建项目

```bash
mkdir ts-node-lab
cd ts-node-lab
npm init -y
npm install --save-dev typescript tsx @types/node
```

将 `package.json` 中的 scripts 和 type 设置为：

```json
{
  "type": "module",
  "scripts": {
    "dev": "tsx src/main.ts",
    "typecheck": "tsc --noEmit"
  }
}
```

创建 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist"
  },
  "include": ["src/**/*.ts"]
}
```

这里用 `tsx` 直接运行 TypeScript 实验代码，用 `tsc --noEmit` 单独检查类型。真实仓库应使用它自己的 package manager、脚本和 tsconfig，不要把本实验命令套用到 Harness 仓库。

`config.json`：

```json
{
  "name": "local-lab",
  "maxItems": 5
}
```

### 2. 验证配置：`src/config.ts`

```ts
import { readFile } from 'node:fs/promises'
import path from 'node:path'

export interface AppConfig {
  name: string
  maxItems: number
}

function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value)
}

export async function loadConfig(): Promise<AppConfig> {
  const filePath = path.resolve(process.cwd(), 'config.json')
  const text = await readFile(filePath, 'utf8')
  const raw: unknown = JSON.parse(text)

  if (!isRecord(raw)) {
    throw new TypeError('config must be an object')
  }

  if (typeof raw.name !== 'string' || raw.name.trim() === '') {
    throw new TypeError('name must be a non-empty string')
  }

  if (
    typeof raw.maxItems !== 'number' ||
    !Number.isInteger(raw.maxItems) ||
    raw.maxItems < 1
  ) {
    throw new TypeError('maxItems must be a positive integer')
  }

  return { name: raw.name, maxItems: raw.maxItems }
}
```

这里的 `isRecord` 使用了类型谓词 `value is Record<string, unknown>`。当函数返回 true，编译器就允许调用者把 value 当作属性值未知的对象字典使用。字段本身仍需单独检查。

### 3. 泛型缓存：`src/cache.ts`

```ts
export class Cache<T> {
  private readonly values = new Map<string, T>()

  set(key: string, value: T): void {
    this.values.set(key, value)
  }

  get(key: string): T | undefined {
    return this.values.get(key)
  }
}
```

`Cache<T>` 表示缓存中存放的值统一属于 T。创建 `new Cache<AppConfig>()` 后，缓存只能保存 `AppConfig`，读取时得到 `AppConfig | undefined`。这里的 `undefined` 表示键可能不存在。

### 4. 子进程工作脚本：`src/worker.ts`

```ts
let count = 0

const timer = setInterval(() => {
  count += 1
  console.log(`worker tick ${count}`)
}, 1_000)

function stop(): void {
  clearInterval(timer)
  console.log('worker stopped')
}

process.on('SIGTERM', stop)
process.on('SIGINT', stop)
```

### 5. 启动并取消子进程：`src/main.ts`

```ts
import { spawn } from 'node:child_process'
import { setTimeout as delay } from 'node:timers/promises'
import path from 'node:path'
import { fileURLToPath } from 'node:url'
import { loadConfig } from './config.js'
import { Cache } from './cache.js'

const here = path.dirname(fileURLToPath(import.meta.url))

async function main(): Promise<void> {
  const config = await loadConfig()
  console.log(`config: ${config.name}, maxItems=${config.maxItems}`)

  const cache = new Cache<typeof config>()
  cache.set('app', config)
  console.log(`cache hit: ${cache.get('app')?.name}`)

  const workerFile = path.join(here, 'worker.ts')
  const child = spawn(
    process.execPath,
    ['--import', 'tsx', workerFile],
    {
      stdio: ['ignore', 'inherit', 'inherit'],
    },
  )

  child.on('error', (error) => {
    console.error('failed to start worker:', error)
  })

  const exitPromise = new Promise<{ code: number | null; signal: NodeJS.Signals | null }>(
    (resolve) => {
      child.once('close', (code, signal) => resolve({ code, signal }))
    },
  )

  await delay(2_500)
  console.log('requesting worker shutdown...')
  child.kill('SIGTERM')

  const result = await exitPromise
  console.log(`worker closed: code=${result.code}, signal=${result.signal}`)
}

main().catch((error: unknown) => {
  console.error(error)
  process.exitCode = 1
})
```

注意：本实验使用当前 Node.js 通过 `--import tsx` 加载子进程脚本，要求安装了 `tsx`，并使用支持该选项的 Node.js 版本。如果本机 Node.js 版本较旧，可以先将 `worker.ts` 编译为 JavaScript，再让子进程执行编译后的 `.js` 文件。

### 6. 运行与预期观察

```bash
npm run typecheck
npm run dev
```

预期输出大致如下（进程调度可能导致 tick 数量略有差异）：

```text
config: local-lab, maxItems=5
cache hit: local-lab
worker tick 1
worker tick 2
requesting worker shutdown...
worker stopped
worker closed: code=0, signal=null
```

不同 Node.js 版本和平台对信号退出结果的呈现可能不同；重点不是死记 `code` 和 `signal` 的某个固定组合，而是确认：父进程发出了终止请求、子进程执行了清理逻辑、父进程等待到了 `close` 事件。若子进程没有停止，先检查它是否真的注册了信号监听器，以及父进程是否监听并等待了正确的退出事件。

### 7. 故障注入

把 `config.json` 改成：

```json
{
  "name": 123,
  "maxItems": -1
}
```

再次运行，应看到 `name must be a non-empty string`，且程序不会启动 worker。修复 name 后，再把 maxItems 改成 `0`，应看到 maxItems 校验错误。

### 本章练习与参考答案

**练习一：** 为什么 `JSON.parse(text) as AppConfig` 不足以保护配置？

**参考答案：** `JSON.parse` 的结果来自外部文本，运行时可能包含错误字段或错误类型；`as` 只改变编译器的看法，不执行字段检查。应先将结果视为 `unknown`，验证每个字段和业务范围，再构造 `AppConfig`。

**练习二：** 为什么 `Cache<T>.get()` 返回 `T | undefined`，而不是直接返回 T？

**参考答案：** Map 中可能不存在指定 key。类型必须反映这一事实，调用者应处理未命中情况，不能假设缓存一定有值。

**练习三：** `child.kill('SIGTERM')` 是否代表子进程已经退出？

**参考答案：** 不代表。它表示父进程发出了信号请求。应监听 `close` 或 `exit`，并根据结果确认子进程是否结束；生产代码还要考虑超时、强制终止、进程树和关闭清理。

**练习四：** 如果把 `void loadConfig()` 放在 `main()` 中而不等待，会有什么风险？

**参考答案：** 主流程可能在配置加载完成前继续执行或退出；加载失败也可能无人捕获。需要等待配置结果，或把后台任务交给具有错误处理和生命周期管理能力的任务系统。

### 本章源码映射：把基础知识带回 Harness

| 本章知识 | 在 Harness 源码中重点追踪的问题 |
| --- | --- |
| 静态类型与运行时校验 | 配置、Tool 参数、Remote 请求和模型输出在哪里验证？ |
| 联合类型与泛型 | 事件、消息、工具结果和状态如何用类型表达？ |
| `unknown` 与类型收窄 | 外部输入进入内部逻辑前经过哪些检查？ |
| Promise 与异常传播 | 调用者是否等待任务？异常由哪一层处理？ |
| AbortSignal | 取消从 CLI / Session / Agent Loop 传到哪个底层操作？ |
| ESM 与 package exports | 哪些入口是公开 API，哪些是内部实现？ |
| Node.js 进程与子进程 | CLI 如何读取参数、定位配置、启动和清理外部任务？ |

**本章完成标准：** 不只是能解释语法，而是能拿一段真实 Harness 代码，指出它的输入边界、类型约束、运行时验证、异步等待者、错误处理者和取消传播路径。

------------------------------------------------------------------------
