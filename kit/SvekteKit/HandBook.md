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
	在使用路由的时候，我们使用的是svelte的基于文件系统的路由，路由存放的位置为`src/routes`,如果需要创建路由，则在当前文件夹下创建即可。（路由文件需要添加`+`前缀作为识别），下面是路由文件的介绍：
#### Page相关
- +page.svelte
- +page.js
- +page.server.js

#### Error相关
- +error.svelte
#### Layout相关
- +layout.svelte
- +layout.js
- +layout.server.js
#### Server相关
- +server.js
#### 其他说明
- $Types
  >具体是可以从`$ypes.d.ts`中去获取到数据类型（这个文件在SvelteKit自动生成的隐藏文件）
  >对以下两种情况会返回具体的data是什么类型的
  >  -  `page.svelte` 中的 `export let data` 上使用注解 `@type import('./$types').PageData}`  
  > - `layout.svelte` 的 `export let data` 上使用注解 `@type {import('./$types').LayoutData}`
  > 也可以在对应的`load`函数使用具体的类型,如下表
  >  | function |  file |
  >  | :--- | :--- |
  >  |PageLoad|+page.js |
  >  |PageServerLoad|+page.server.js|
  >  |LayoutLoad|+layout.js |
  >  |LayoutServerLoad|+layout.server.js |
  > 
- 其他文件
  >其他的文件会被SvelteKit忽略掉，不会参与到`route`的生成中，将这些文件放到他该有的地方去，例如`$lib`


### 高级路由

