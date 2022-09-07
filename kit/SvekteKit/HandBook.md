# SvelteKit

## 创建项目

``` bash
npm create svelte project-name
```

## 文件结构

``` yaml
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

>特别注意`hooks.js`文件 ,`routes目录`,`params目录`

## 文件详解
- `lib/`
  >  包含自己写的库文件，而且可以在代码中使用 `$lib`作为引用使用
- `params/`
  >  用于存放一些路由参数的过滤规则
- `routes/`
  > 用来存放系统中的路由信息,详细见[路由详解](#route)
- `app.html`
  > 所有页面的样板文件，并且包含一下几个特殊的占位符
  >>   -  `%sveltekit.head%` : 用以替换页面中的<svelte:head>部分的内容
  >>   -  `%sveltekit.body%` : 用以替换
  >>   -  `%sveltekit.assets%` : 用以表示 paths.assets 或者 paths.base (暂时没遇到过，遇到了再看是什么情况)
  >>   -  `%sveltekit.nonce%` : 用以[CSP](/appendix/CSP)使用(暂时没有用到，了解一下什么是CSP)
  
 - `error.html` （非必要）
   > 用以页面发生任何错误的时候的跳转，包含以下占位符
   > >  - `%sveltekit.status%` : http请求的状态麻
   > >  - `%sveltekit.message%` :错误信息
- `hooks.js` （非必要）
>  用于管理系统的[hooks](./hoocks)
- `service-worker.js` （非必要）
  > 用以管理系统的[service worker](./serviceWorker)


## <span id="route"> 路由详解 </span>

### 简单路由

### 高级路由

