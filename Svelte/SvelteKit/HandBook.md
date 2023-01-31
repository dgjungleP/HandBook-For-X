# SvelteKit

## 创建项目

```bash
npm create svelte project-name
```

## 文件结构

```yaml
my-project/
├ src/
│ ├ lib/
│ │ └ [your lib files]
│ ├ params/
│ │ └ [your param matchers]
│ ├ routes/
│ │ └ [your routes]
│ ├ app.html
│ ├ error.html
│ └ hooks.js
├ static/
│ └ [your static assets]
├ tests/
│ └ [your tests]
├ package.json
├ svelte.config.js
├ tsconfig.json
└ vite.config.js
```

> 特别注意`hooks.js`文件 ,`routes目录`,`params目录`

## 文件详解

- `lib/`
  > 包含自己写的库文件，而且可以在代码中使用 `$lib`作为引用使用
- `params/`
  > 用于存放一些路由参数的过滤规则
- `routes/`
  > 用来存放系统中的路由信息,详细见[路由详解](#route)
- `app.html`
  > 所有页面的样板文件，并且包含一下几个特殊的占位符
  >
  > > - `%sveltekit.head%` : 用以替换页面中的<svelte:head>部分的内容
  > > - `%sveltekit.body%` : 用以替换
  > > - `%sveltekit.assets%` : 用以表示 paths.assets 或者 paths.base (暂时没遇到过，遇到了再看是什么情况)
  > > - `%sveltekit.nonce%` : 用以[CSP](/appendix/CSP)使用(暂时没有用到，了解一下什么是 CSP)
- `error.html` （非必要）
  > 用以页面发生任何错误的时候的跳转，包含以下占位符
  >
  > > - `%sveltekit.status%` : http 请求的状态麻
  > > - `%sveltekit.message%` :错误信息
- `hooks.js` （非必要）
  > 用于管理系统的[hooks](Hoocks.md)
- `service-worker.js` （非必要）
  > 用以管理系统的[service worker](ServiceWorker.md)

## <span id="route"> 路由详解 </span>

### 简单路由

    在使用路由的时候，我们使用的是svelte的基于文件系统的路由，路由存放的位置为`src/routes`,如果需要创建路由，则在当前文件夹下创建即可。（路由文件需要添加`+`前缀作为识别），下面是路由文件的介绍：

#### Page 相关

- +page.svelte
  > 就是当前路由的页面，如果当前路由中有`+layout.svelte`的话，就会插入导layout中的`sot`中
- +page.js
>   主要是和`+page.svelte`合作，通过`load`方法返回数据和进行一些处理，同事还有一些页面的设置`orerender`,`ssr`,`csr`等
- +page.server.js
>   如果你的`load`函数只能在服务端使用，比如你有一些私有化的环境变量或者参数之类的，那么你就可以使用这个文件。
>   并且相比于`+page.js`他还能使用[actions](actions.md),可以让你用`<form>`的形式向服务器提交数据

#### Error 相关

- +error.svelte
> 如果在执行`load`函数的时候发生了任何问题，除了使用公共的错误页面以外，你可以针对于每个`route`自定义自己的错误页面

#### Layout 相关

- +layout.svelte
  > 可以用来创建页面固定的模板，而且支持嵌套
- +layout.js
  > 类比于 `+page.js`，可以为`+layout.svelte`提供数据
- +layout.server.js
>   类比于`+page.server.js`

#### Server 相关

- +server.js
  > 定义api的路由 或者说是端点信息 可以导出一些方法例如,`GET`, `POST`, `PATCH`, `PUT` 和 `DELETE`,并且会有一个入参`RequestEvnet`,和一个返回值`Response` ,
  > 同时在方法内部,你也可以用来自于`@sveltejs/kit`的`error`, `redirect` 和 `json`

#### 其他说明

- $Types
  > 具体是可以从`$ypes.d.ts`中去获取到数据类型（这个文件在 SvelteKit 自动生成的隐藏文件）
  > 对以下两种情况会返回具体的 data 是什么类型的
  >
  > - `page.svelte` 中的 `export let data` 上使用注解 `@type import('./$types').PageData}`
  > - `layout.svelte` 的 `export let data` 上使用注解 `@type {import('./$types').LayoutData}`
  >   也可以在对应的`load`函数使用具体的类型,如下表  
  >   | function | file |
  >   | :--- | :--- |
  >   |PageLoad|+page.js |
  >   |PageServerLoad|+page.server.js|
  >   |LayoutLoad|+layout.js |
  >   |LayoutServerLoad|+layout.server.js |
- 其他文件
  > 其他的文件会被 SvelteKit 忽略掉，不会参与到`route`的生成中，将这些文件放到他该有的地方去，例如`$lib`

### 高级路由
