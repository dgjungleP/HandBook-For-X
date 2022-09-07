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
  > 所有页面的样板文件
  >>   -  `%sveltekit.head%`
  >>   -  `%sveltekit.body%`
  >>   -  `%sveltekit.assets%`
  >>   -  `%sveltekit.nonce%`
  
- `error.html`
- `hooks.js`
- `service-worker.js`


## <span id="route"> 路由详解 </span>

### 简单路由

### 高级路由

