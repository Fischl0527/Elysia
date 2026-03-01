学习网站：[https://elysia.zhcndoc.com/tutorial/](https://elysia.zhcndoc.com/tutorial/)

## 界面文字修改
```typescript
import { Elysia } from 'elysia'

const app = new Elysia()
  .get('/', 'Hello World!')
  .listen(3000)

console.log(
  `🦊 Elysia is running at ${app.server?.hostname}:${app.server?.port}`
)
```

效果：

![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765903808881-a14f97cd-7045-4db1-a9a1-8acd5b656876.png)

## 路由
### 静态路径 
静态路径是一个硬编码字符串，用于定位服务器上的资源。

新建路由直接就是在.get 下，修改第一个参数就行了。比如 `.get('**/elysia**','Hello Elysia!')`这样就创造了 elysia 的静态路径

```typescript
import { Elysia } from 'elysia'

const app = new Elysia()
  .get('/', 'Hello Elysia!')
  .get('/elysia','Hello Elysia!')
  .listen(3000)

console.log(
  `🦊 Elysia is running at ${app.server?.hostname}:${app.server?.port}`
)
```

效果：

![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765908086989-0944564d-b7be-421d-a48c-4c0396b6a429.png)

### 动态路径
动态路径匹配某部分内容并捕获该值得到额外信息。

定义动态路径时，可以使用冒号 `:` 后跟名称。比如

`:name`：动态参数语法（冒号 + 参数名）会匹配 URL 中 `/friends/` 后的**任意非空字符串**，例如

匹配：

`/friends/Alice` → `name = "Alice"`

`/friends/123` → `name = "123"`

`/friends/john_doe` → `name = "john_doe"`

不匹配：

`/friends/`（缺少参数值）

`/friends/Alice/Smith`（包含额外斜杠）

`{ params: { name } }`：从上下文提取 params 对象，并从 params 提取 name 属性，自动推断 name 为 string 类型，自动处理 URL 编码，当参数缺失时，路由不会匹配，即返回 404。

`=>`：箭头函数符号，表示这是一个匿名函数，等价于 function() { return ... }，但更简洁  
`` ``：反引号包裹的是模板字符串，允许在字符串中直接嵌入变量  
`${name}`：变量插值，把变量 name 的值插入到字符串中

把嵌入代码中得到

```typescript
import { Elysia } from 'elysia'

const app = new Elysia()
  .get('/', 'Hello Elysia!')
  .get('/elysia','Hello Elysia!')
  .get('/friends/:name', { params: { name } } => `Hello ${name}!`)
  .listen(3000)

console.log(
  `🦊 Elysia is running at ${app.server?.hostname}:${app.server?.port}`
)
```

这里通过 `/friends/:name` 创建了一个动态路径。通过解构 `({ params: { name } })` 获取路由参数，Elysia 会自动将 URL 中的参数值注入到 `params` 对象。使用模板字符串返回 `Hello ${name}!`

效果

![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765908129449-b0206906-2814-46ee-bf7b-a23f153f07c4.png)![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765908225518-24855307-db02-4b15-9a2e-891be9ffb396.png)

### 可选路径参数
在路由路径中，在参数名后加 `?` 可以让该参数变成可选的：

```typescript
// 常规动态参数（必需）
.get('/users/:id', ...)   // 必须提供 id → /users/123 ✅  /users/ ❌

// 可选动态参数（带?）
.get('/users/:id?', ...)  // id 可有可无 → /users/123 ✅  /users/ ✅  /users ✅
```

让`/friends/:name`参数编程可选的

```typescript
import { Elysia } from 'elysia'

const app = new Elysia()
  .get('/', 'Hello Elysia!')
  .get('/elysia','Hello Elysia!')
  //.get('/friends/:name', ({ params: { name } }) => `Hello ${name}!`)//注意不是单引号了
	.get('/friends/:name?', ({ params: { name } }) => `Hello ${name}!`)
  .listen(3000)

console.log(
  `🦊 Elysia is running at ${app.server?.hostname}:${app.server?.port}`
)
```

![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765908129449-b0206906-2814-46ee-bf7b-a23f153f07c4.png)![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765908239385-3f2b3190-1564-4cc5-8378-eab34498636c.png)

### 通配符
动态参数`:param`：单个路径段，即`/files/:name`→ 捕获`/files/report` 中的 `report`，但不捕获 `/files/docs/report`  
通配符`*`：剩余所有路径段，即`/files/*` → 捕获 `/files/report`中的`report`，同时捕获 `/files/docs/report`中的 `docs/report`

由于`*` 不是合法的 JavaScript 标识符（不能直接作为变量名）所以 Elysia 将捕获的路径存入 `params` 对象，键名为字符串 `'*'`，必须用方括号语法访问：`params['*']`

```typescript
import { Elysia } from 'elysia'

const app = new Elysia() //创建一个 Elysia 应用实例
	.get('/', 'Hello Elysia!')
	.get('/elysia','Hello Elysia!')
	//.get('/friends/:name', ({ params: { name } }) => `Hello ${name}!`)
	.get('/friends/:name?', ({ params: { name } }) => `Hello ${name}!`)
	.get('/flame-chasers/*', ({ params: {'*' : name } }) => `Hello ${name}!`)
  .listen(3000)

console.log(
	`🦊 Elysia is running at ${app.server?.hostname}:${app.server?.port}`
)
```

效果：

![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765908280915-693b93ec-7eb1-4e9f-abf0-718bfbc8cf59.png)![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765908299954-6986c0d8-0966-4a6f-a6aa-9eb5f98274ae.png)![](https://cdn.nlark.com/yuque/0/2025/png/50616406/1765908592581-193ec53e-8518-4d20-a5a7-ebb1229cce55.png)

## 处理器与上下文 
 处理器：一个资源或路由函数，用于向客户端发送数据。

```typescript
import { Elysia } from 'elysia'

new Elysia()
  // `() => 'hello world'` 是一个处理器    
  .get('/', () => 'hello world')    
  .listen(3000)
```

处理器也可以是一个字面值

```typescript
import { Elysia } from 'elysia'

new Elysia()
// `'hello world'` 是一个处理器   
  .get('/', 'hello world')   
  .listen(3000)
```

使用内联值对于静态资源如文件是有用的。

上下文：包含有关每个请求的信息。它作为处理器的唯一参数传递。

```typescript
import { Elysia } from 'elysia'

new Elysia()	
  .get('/', (context) => context.path)            
          // ^ 这是一个上下文
```

```typescript
import { Elysia } from 'elysia'

new Elysia()
	.get('/', 'Hello Elysia!')
	.get('/', () => 'hello world')
	.post('/',({body, query, headers}) =>{
		return{
			body,
			query,
			headers
		}
	}
	)
	.listen(3000)

```

