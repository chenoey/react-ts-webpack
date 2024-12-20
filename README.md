
## 一.  前言

在前端项目工程日益复杂的今天，一套完善的开发环境配置可以极大的提升开发效率，提高代码质量，方便多人合作，以及后期的项目迭代和维护，项目规范分项目**目录结构规范**，**代码格式规范**和**git提交规范**，本文主要讲后两种。

本文将使用**webpack5**从零搭建一个完整的**react18+ts**开发和打包环境，配置完善的模块热替换以及**构建速度**和**构建结果**的优化。


## 二. 代码规范技术栈

### 2.1 代码格式规范和语法检测

1.  [vscode](http://vscode.bianjiqi.net/)：统一前端编辑器。
1.  [editorconfig](https://editorconfig.org/): 统一团队vscode编辑器默认配置。
1.  [prettier](https://www.prettier.cn/): 保存文件自动格式化代码。
1.  [eslint](https://eslint.bootcss.com/): 检测代码语法规范和错误。
1.  [lint-staged](https://github.com/okonet/lint-staged): 只检测暂存区文件代码，优化eslint检测速度。

### 2.2 代码git提交规范

1.  [husky](https://github.com/typicode/husky):可以监听[githooks](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)执行，在对应hook执行阶段做一些处理的操作。
1.  [pre-commit](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)：githooks之一， 在commit提交前使用tsc和eslint对语法进行检测。
1.  [commit-msg](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)：githooks之一，在commit提交前对commit备注信息进行检测。
1.  [commitlint](https://commitlint.js.org/#/)：在githooks的pre-commit阶段对commit备注信息进行检测。
1.  [commitizen](https://github.com/commitizen/cz-cli)：git的规范化提交工具，辅助填写commit信息。


## 三.  初始化项目

先手动初始化一个基本的**react**+**ts**项目，新建项目文件夹**react-ts**, 在项目下执行

```bash
npm init -y
```

初始化好**package.json**后,在项目下新增以下所示目录结构和文件

```yaml
├── public
│   └── index.html # html模板
├── src
|   ├── App.tsx 
│   └── index.tsx # react应用入口页面
├── tsconfig.json  # ts配置
└── package.json
```

安装**react**依赖

```sh
npm i react react-dom -S
```

安装**react**类型依赖

```sh
npm i @types/react @types/react-dom -D
```

添加**public/index.html**内容

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>webpack5-react-ts</title>
</head>
<body>
  <!-- 容器节点 -->
  <div id="root"></div>
</body>
</html>
```

添加**src/App.tsx**内容

```tsx

import React from 'react'

function App() {
  return <h2>webpack5-react-ts</h2>
}
export default App
```

添加**src/index.tsx**内容

```tsx

import React from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

const root = document.getElementById('root');
if(root) {
  createRoot(root).render(<App />)
}
```

添加**tsconfig.json**内容

```js

{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": false,
    "skipLibCheck": false,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react", // react18这里也可以改成react-jsx
  },
  "include": ["./src"]
}
```

添加 **.gitignore** 内容


```sh
# Dependency directories
node_modules

# Webstorm
.idea

# Build directory
dist
build
```
该文件决定了项目进行 git 提交时所需要忽略掉的文件或文件夹，编辑器如 vscode 也会监听 `.gitignore` 之外的所有文件，如果没有进行忽略的文件有所变动时，在进行 git 提交时就会被识别为需要提交的文件。

`node_modules` 是我们安装第三方依赖的文件夹，这个肯定要添加至 `.gitignore` 中，且不说这个文件夹里面成千上万的文件会给编辑器带来性能压力，也会给提交至远端的服务器造成不小损失，另外就是这个文件夹中的东西，完全可以通过简单的 `npm install` 就能得到～

所以不需要上传至 git 仓库的都要添加进来，比如我们常见的 `build` 、 `dist` 等，还有操作系统默认生成的，比如 MacOs 会生成存储项目文件夹显示属性的 `DS_Store` 文件。


## 四.**editorconfig**统一编辑器配置


由于每个人的**vsocde**编辑器默认配置可能不一样，比如有的默认缩进是**4**个空格，有的是**2**个空格，如果自己编辑器和项目代码缩进不一样，会给开发和项目代码规范带来一定影响，所以需要在项目中为编辑器配置下格式。

### 4.1 安装vscode插件EditorConfig

打开**vsocde**插件商店，搜索**EditorConfig for VS Code**，然后进行安装。

![image](https://github.com/user-attachments/assets/e8ffa81c-bb41-49a5-8314-2d797831023e)


### 4.2 添加.editorconfig配置文件

安装插件后，在根目录新增 **.editorconfig**配置文件:

```sh
root = true # 控制配置文件 .editorconfig 是否生效的字段

[**] # 匹配全部文件
indent_style = space # 缩进风格，可选space｜tab
indent_size = 2 # 缩进的空格数
charset = utf-8 # 设置字符集
trim_trailing_whitespace = true # 删除一行中的前后空格
insert_final_newline = true # 设为true表示使文件以一个空白行结尾
end_of_line = lf

[**.md] # 匹配md文件
trim_trailing_whitespace = false
```

上面的配置可以规范本项目中文件的缩进风格，和缩进空格数等，会覆盖**vscode**的配置，来达到不同编辑器中代码默认行为一致的作用。


## 五.**prettier**自动格式化代码

每个人写代码的风格习惯不一样，比如代码换行，结尾是否带分号，单双引号，缩进等，而且不能只靠口头规范来约束，项目紧急的时候可能会不太注意代码格式，这时候需要有工具来帮我们自动格式化代码，而**prettier**就是帮我们做这件事。

如果说 `EditorConfig` 帮你统一编辑器风格，那 `Prettier` 就是帮你统一项目风格的。 且能在发布流程中执行命令自动格式化，能够有效的使项目代码风格趋于统一。

在我们的项目中执行以下命令安装我们的第一个依赖包：

```sh
npm install prettier -D
```

### 5.1 添加.prettierrc配置文件

安装成功之后在根目录新建文件 `.prettierrc` ，输入以下配置：
```js
{
  printWidth: 100, // 一行的字符数，如果超过会进行换行
  tabWidth: 2, // 一个tab代表几个空格数，默认就是2
  useTabs: false, // 是否启用tab取代空格符缩进，.editorconfig设置空格缩进，所以设置为false
  semi: false, // 行尾是否使用分号，默认为true
  singleQuote: true, // 字符串是否使用单引号
  trailingComma: 'none', // 对象或数组末尾是否添加逗号 none| es5| all 
  jsxSingleQuote: true, // 在jsx里是否使用单引号，你看着办
  bracketSpacing: true, // 对象大括号直接是否有空格，默认为true，效果：{ foo: bar }
  arrowParens: "avoid", // 箭头函数如果只有一个参数则省略括号 默认值：always
}
```

-   一个是我们可以通过命令的形式去格式化某个文件下的代码，但是我们基本不会去使用，最终都是通过 `ESlint` 去检测代码是否符合规范。

-   二是当我们编辑完代码之后，按下 `ctrl+s` 保存就给我们自动把当前文件代码格式化了，既能实时查看格式化后的代码风格，又省去了命令执行代码格式化的多余工作。

需要做的是先安装扩展 [Prettier - Code formatter]

### 5.2 安装vscode插件Prettier

打开**vsocde**插件商店，搜索**Prettier - Code formatter**，然后进行安装。

![image](https://github.com/user-attachments/assets/325b4225-51c3-4ad7-9bb9-92af35f6a0bd)

### 5.3 添加.vscode/settings.json


配置前两步后，虽然已经配置**prettier**格式化规则，但还需要让**vscode**来支持保存后触发格式化

在项目根目录（一级目录）新建 **.vscode**文件夹，内部新建**settings.json**文件配置文件

该文件的配置优先于 vscode 全局的 `settings.json` ，这样别人下载了你的项目进行开发，也不会因为全局 `setting.json` 的配置不同而导致 `Prettier` 失效

```js

{
  "search.exclude": {
    "/node_modules": true,
    "dist": true,
    "npm-lock.sh": true
  },
  "editor.formatOnSave": true,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "vscode.json-language-features"
  },
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[less]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[scss]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
}
```

先配置了忽略哪些文件不进行格式化，又添加了保存代码后触发格式化代码配置，以及各类型文件格式化使用的格式。

这一步配置完成后，修改项目代码，把格式打乱，点击保存时就会看到编辑器自动把代码格式规范化了。



## 六.  **eslint**

`ESLint` 是主要为了解决**代码质量问题**，它能在我们编写代码时就检测出程序可能出现的隐性BUG，通过 `eslint --fix` 还能自动修复一些代码写法问题，比如你定义了 `var a = 3` ，自动修复后为 `const a = 3` 。还有许多类似的强制扭转代码最佳写法的规则，在无法自动修复时，会给出红线提示，强迫开发人员为其寻求更好的解决方案。

### 6.1 安装eslint依赖

首先在项目中安装 `eslint` ：

```sh
 npm install eslint -D
```

安装成功后，执行以下命令：

```sh
npx eslint --init
```

上述命令的功能为初始化 `ESLint` 的配置文件，采取的是问答的形式，特别人性化。

不过在我们介绍各个问答之前先来看看这句命令中 `npx` 是什么。实际上，要达到以上命令的效果还有两种方式。

一是直接找到我们项目中安装的 `eslint` 的可执行文件，然后根据该路径来执行命令：

```sh
./node_modules/.bin/eslint --init
```

二是先全局安装 `eslint` ，直接执行以下命令即可：

```sh
# 全局安装 eslint
npm install eslint -g

# eslint 配置文件初始化
eslint --init
```

现在让我们来说下这两种方式的缺点：

-   针对第一种，其实本质上来说和我们所推荐的 `npx` 形式没有区别，缺点是该命令太过于繁琐。
-   针对第二种，我们需要先全局进行 `eslint` 的安装，这会占据我们电脑的硬盘空间，且会将安装文件放到挺隐蔽的地方，对于有洁癖的人来说，是非常接受不了这种全局安装的，特别是越来越多全局包的时候。再有一个比较大的问题是，因为我们执行 `eslint --init` 是使用全局安装的版本去初始化的，这有可能会和你现在项目中的 `eslint` 版本不一致。

那么 `npx` 的作用就是抹掉了上述两个缺点，其是 `npm v5.2.0` 引入的一条命令，它在上述命令执行时：

-   会先去本地 `node_modules` 中找 `eslint` 的执行文件，如果找到了，就直接执行，相当于上面所说的第一种方式；
-   如果没有找到，就去全局找，找到了，就相当于上述第二种方式；
-   如果都没有找到，就下载一个临时的 `eslint` ，用完之后就删除这个临时的包，对本机完全无污染。

### 6.2 初始化 `ESLint` 的配置文件

除了上述方法对EsLint进行安装配置外，可以使用官网提供的命令直接进行安装并配置

```sh
npm init @eslint/config
```

执行**npm init @eslint/config**，选择自己需要的配置

![image](https://github.com/user-attachments/assets/506a2aab-1b63-4959-bac0-b83314bfd531)

这里我们选择了

-   使用**eslint**检测并问题
-   项目使用的模块规范是**es module**
-   使用的框架是**react**
-   使用了**typescript**
-   代码选择运行在浏览器端
-   是否现在安装相关依赖，选择是
-   使用**npm**包管理器安装依赖

选择完成后会在根目录下生成 `eslint.config.mjs`文件，默认配置如下


```js
import globals from "globals";
import pluginJs from "@eslint/js";
import tsEslint from "typescript-eslint";
import pluginReact from "eslint-plugin-react";


/** @type {import('eslint').Linter.Config[]} */
export default [
  {files: ["**/*.{js,mjs,cjs,ts,jsx,tsx}"]},
  {languageOptions: { globals: globals.browser }},
  pluginJs.configs.recommended,
  ...tsEslint.configs.recommended,
  pluginReact.configs.flat.recommended,
];

```
eslint 9 中支持 Common JS 和 ESM 两种配置文件格式，推荐使用 ESM。

eslint.config.js
```js
const eslint = require('@eslint/eslint');

module.exports = [
  eslint.configs.recommended,
  // your config
  {
    name: 'custom-lint-config',
    files: ['*.js'],
    rules: {
      'no-undef': 0,
    },
  },
];
```

eslint.config.mjs
```js
import eslint from '@eslint/eslint';

export default [
  eslint.configs.recommended,
  // your config
  {
    name: 'custom-lint-config',
    files: ['*.js'],
    rules: {
      'no-undef': 0,
    },
  },
];
```
在默认配置基础上新增如下配置

```js
import globals from "globals";
import pluginJs from "@eslint/js";
import tsEslint from "typescript-eslint";
import pluginReact from "eslint-plugin-react";


/** @type {import('eslint').Linter.Config[]} */
export default [
  { ignores: ['node_modules', 'build', 'dist'] },//忽略文件
  {files: ["**/*.{js,mjs,cjs,ts,jsx,tsx}"]},
  {languageOptions: { globals: globals.browser }},
  pluginJs.configs.recommended,
  ...tsEslint.configs.recommended,
  pluginReact.configs.flat.recommended,
];
```

### 6.3 安装vscode插件ESLint

`eslint` 由编辑器支持是有自动修复功能的，首先我们需要安装扩展，打开**vsocde**插件商店，搜索**ESLint**，然后进行安装。

![image](https://github.com/user-attachments/assets/9ef23620-58f0-44f5-8c1d-555986c0712a)

再到之前创建的 `.vscode/settings.json` 中添加以下代码
```js
{
  "eslint.validate": ["javascript", "javascriptreact", "typescript", "typescriptreact"],
  //"typescript.tsdk": "./node_modules/typescript/lib", // 代替 vscode 的 ts 语法智能提示
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
  },
}
```
这时候我们保存时，就会开启 `eslint` 的自动修复。

### 6.4 解决项目eslint语法错误

此时**eslint**基础配置就已经配置好了，此时要解决出现的几个小问题：

#### 1.看**App.tsx**页面会发现**jsx**部分有红色报红，提示 **'React' must be in scope when using JSX**

这是因为**React18**版本中使用**jsx**语法不需要再引入**React**了，根据[eslint-plugin-react](https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/react-in-jsx-scope.md)中的说明，如果使用了**react17**版本以上，不需要在使用**jsx**页面引入**React**时，在**eslint**配置文件添加插件**plugin:react/jsx-runtime**。

```js
import jsx from 'react/jsx-runtime'

export default [
  jsx.configs.recommended,
]

```

此时**App.tsx**就不会报错了。

#### 2.看到**index.tsx**文件带有警告颜色，看警告提示是**Forbidden non-null assertion**。

这个提示是不允许使用非空操作符!，但实际在项目中经常会用到，所以可以把该项校验给关闭掉。

在**eslint**配置文件 **.eslint.js**的**rules**字段添加插件 **'@typescript-eslint/no-non-null-assertion': 'off'** 。

```js
 rules: {
    '@typescript-eslint/no-non-null-assertion': 'off'
  }
```

然后就不会报警告了，如果为了避免代码出现异常，不想关闭该校验，可以提前做判断

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'

const root = document.getElementById('root')
// 如果root有值，才执行react渲染逻辑
if (root) {
  ReactDOM.createRoot(root).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  )
}
```

#### 3. 提示 **'React' refers to a UMD global, but the current file is a module. Consider adding an import instead**.

如果您使用自定义设置，则需要在编写 `jsx` 而不使用 `import React from 'react'` 时消除 TypeScript 错误：  


-   `typescript` 版本至少为 4.1 版
-   `react` 和 `react-dom` 至少版本 17
-   `tsconfig.json` 必须有一个 `jsx` 编译器选项 `react-jsx` 或 `react-jsxdev`

```js
示例：
// tsconfig.json
{
  "compilerOptions": {
    ...
    "jsx": "react-jsx"
    ...
  },
```


### 6.5 添加eslint语法检测脚本

前面的**eslint**报错和警告都是我们用眼睛看到的，有时候需要通过脚本执行能检测出来，在**package.json**的**scripts**中新增

```bash
"eslint": "eslint src/**/*.{ts,tsx}"
```

代表检测**src**目录下以**.ts**, **.tsx**为后缀的文件

除此之外再解决一个问题就是**eslint**报的警告**React version not specified in eslint-plugin-react settings**,需要告诉**eslint**使用的**react**版本，在 **eslint.config.mjs**和**rules**平级添加**settings**配置，让**eslint**自己检测**react**版本。

```js
 settings: {
    "react": {
      "version": "detect"
    }
  }
```

再执行**npm run eslint**就不会报这个未设置**react**版本的警告了。

## 七. lint-staged

在上面配置的**eslint**会检测**src**文件下所有的 **.ts**, **.tsx**文件，虽然功能可以实现，但是当项目文件多的时候，检测的文件会很多，需要的时间也会越来越长，但其实只需要检测提交到暂存区，就是**git add**添加的文件，不在暂存区的文件不用再次检测，而**lint-staged**就是来帮我们做这件事情的。

### 安装依赖

```sh
npm i lint-staged -D
```

### 随后在 `package.json` 中添加以下代码：

```js
{
  "lint-staged": {
    "src/**/*.{ts,tsx,js}": [
      "eslint --config eslint.config.mjs"
    ],
    "src/**/*.{ts,tsx,js,json,html,css,less,scss,md}": [
      "prettier --write"
    ]
  },
}
```
首先，我们会对暂存区后缀为 `.ts .tsx .js` 的文件进行 `eslint` 校验， `--config` 的作用是指定配置文件。

在使用 `prettier` 进行代码格式化时，完全可以添加 `--write` 来使我们的代码自动格式化，它不会更改语法层面上的东西，所以无需担心。

**lint-staged**只有检测到语法报错才会有提示而警告不会，如果需要出现警告也阻止代码提交，需要给eslint检测配置参数 **--max-warnings=0**

```js
//package.json
 "eslint": "eslint --max-warnings=0"
```

代表允许最多**0**个警告，就是只要出现警告就会报错。

把代码提交到暂存区后，执行**npx lint-staged**就可以看到检测结果。

## 八.husky

我们还需要另一个工具 **husky**，**husky**就是可以监听**githooks**的工具，它会提供一些钩子，比如执行 `git commit` 之前的钩子 `pre-commit` ，它会在**git commit**把代码提交到本地仓库之前执行，可以在这个阶段检测代码，如果检测不通过就退出命令行进程停止**commit**。借助这个钩子我们就能执行 `lint-staged` 所提供的代码文件格式化及 lint 规则校验！

### 安装依赖

```sh
npm i husky@8.0.1 -D
```

### 初始化生成 **.husky**配置文件夹

```sh
npx husky install
```

### 添加配置

```sh
npx husky add .husky/pre-commit 'npx lint-staged'


# .husky/pre-commit文件中添加

#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"
npx lint-staged
```



### 配置 script prepare

在 package.json 中增加 prepare 脚本，用于在安装依赖时自动安装 husky。


```js
{
  "scripts": {
    "prepare": "husky install"
    // other...
  }
}
```

## 九.commitlint

在提交代码时，良好的提交备注会方便多人开发时其他人理解本次提交修改的大致内容，也方便后面维护迭代，但每个人习惯都不一样，需要用工具来做下限制，在**git**提供的一系列的[githooks](https://git-scm.com/womdocs/githooks) 中，**commit-msg**会在**git commit**之前执行，并获取到**git commit**的备注，可以通过这个钩子来验证备注是否合理，而验证是否合理肯定需要先定义一套规范，而[commitlint](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fconventional-changelog%2Fcommitlint)就是用来做这件事情的，它会预先定义一套规范，然后验证**git commit**的备注是否符合定义的规范。

### 安装 commitlint 和 commitlint 配置 

```sh
npm i @commitlint/config-conventional @commitlint/cli -D
```

### 在根目录创建**commitlint.config.js**文件,添加配置如下

```sh
 module.exports = {
  extends: ["@commitlint/config-conventional"],
  rules: {
    // type 不允许为空
    "type-empty": [
      2, // 触发这条规则时 error 提示
      "never", // 满足这条规则时，则根据(level=2)进行 error 提示
    ],
    // type 允许的类型
    "type-enum": [
      2, // 触发这条规则时 error 提示
      "always", // 违背这条规则时，则根据 (level) 进行提示
      [
        "feat", // 新功能
        "fix", // bug 修复
        "docs", // 文档更新
        "style", // 样式调整
        "refactor", // 代码重构
        "test", // 编写测试用例
        "revert", // 代码回滚
        "chore", // 项目配置更新
        "perf", // 性能优化
        "build" // 打包
      ],
    ],
    // body 至少包含4个字符
    "body-min-length": [2, "always", 4],
  },
};
```

### 配置husky监听commit-msg

上面已经安装了**husky**,现在需要再配置下**husky**，让**husky**支持监听**commit-msg**钩子，在钩子函数中使用**commitlint**来验证

```sh
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'

#.husky/commit-msg

#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"
npx --no-install commitlint --edit
```

会在 **.husky**目录下生成 **commit-msg**文件，并且配置 **commitlint**命令对备注进行验证配置。


## 十.  配置webpack

```yaml
├── build
    ├── webpack.base.js # 公共配置
    ├── webpack.dev.js  # 开发环境配置
    └── webpack.prod.js # 打包环境配置
```

安装**webpack**依赖

```sh
npm i webpack webpack-cli -D
```

### 10.1 webpack公共配置

修改**webpack.base.js**

**1. 配置入口文件**

```js
// webpack.base.js
const path = require('path')

module.exports = {
  entry: path.join(__dirname, '../src/index.tsx'), // 入口文件
}
```

**2. 配置出口文件**

```js
// webpack.base.js
const path = require('path')

module.exports = {
  // ...
  // 打包文件出口
  output: {
    filename: 'static/js/[name].js', // 每个输出js的名称
    path: path.join(__dirname, '../dist'), // 打包结果输出路径
    clean: true, // webpack4需要配置clean-webpack-plugin来删除dist文件,webpack5内置了
    publicPath: '/' // 打包后文件的公共前缀路径
  },
}
```

**3. 配置loader解析ts和jsx**

由于**webpack**默认只能识别**js**文件,不能识别**jsx**语法,需要配置**loader**的预设 [**@babel/preset-typescript**](https://www.babeljs.cn/docs/babel-preset-typescript) 来先**ts**语法转换为 **js** 语法,再借助预设 [**@babel/preset-react**](https://www.babeljs.cn/docs/babel-preset-react) 来识别**jsx**语法。

**安装babel核心模块和babel预设**

```sh
npm i babel-loader @babel/core @babel/preset-react @babel/preset-typescript -D
```

在**webpack.base.js**添加**module.rules**配置

```js
// webpack.base.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /.(ts|tsx)$/, // 匹配.ts, tsx文件
        use: {
          loader: 'babel-loader',
          options: {
            // 预设执行顺序由右往左,所以先处理ts,再处理jsx
            presets: [
              '@babel/preset-react',
              '@babel/preset-typescript'
            ]
          }
        }
      }
    ]
  }
}
```

**4. 配置extensions**

**extensions**是**webpack**的**resolve**解析配置下的选项，在引入模块时不带文件后缀时，会来该配置数组里面依次添加后缀查找文件，因为**ts**不支持引入以 **.ts**, **tsx**为后缀的文件，所以要在**extensions**中配置，而第三方库里面很多引入**js**文件没有带后缀，所以也要配置下**js**

修改**webpack.base.js**，注意把高频出现的文件后缀放在前面

```js
// webpack.base.js
module.exports = {
  // ...
  resolve: {
    extensions: ['.js', '.tsx', '.ts'],
  }
}
```

这里只配置**js**, **tsx**和**ts**，其他文件引入都要求带后缀，可以提升构建速度。

**4. 添加html-webpack-plugin插件**

**webpack**需要把最终构建好的静态资源都引入到一个**html**文件中,这样才能在浏览器中运行,[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)就是来做这件事情的,安装依赖：

```sh
npm i html-webpack-plugin -D
```

因为该插件在开发和构建打包模式都会用到,所以还是放在公共配置**webpack.base.js**里面

```js
// webpack.base.js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../public/index.html'), // 模板取定义root节点的模板
      inject: true, // 自动注入静态资源
    })
  ]
}
```

到这里一个最基础的**react**基本公共配置就已经配置好了,需要在此基础上分别配置开发环境和打包环境了。

### 10.2 webpack开发环境配置

**1. 安装 webpack-dev-server**

开发环境配置代码在**webpack.dev.js**中,需要借助 [webpack-dev-server](https://www.npmjs.com/package/webpack-dev-server)在开发环境启动服务器来辅助开发,还需要依赖[webpack-merge](https://www.npmjs.com/package/webpack-merge)来合并基本配置,安装依赖:

```sh
npm i webpack-dev-server webpack-merge -D
```

修改**webpack.dev.js**代码, 合并公共配置，并添加开发模式配置

```js
// webpack.dev.js
const path = require('path')
const { merge } = require('webpack-merge')
const baseConfig = require('./webpack.base.js')

// 合并公共配置,并添加开发环境配置
module.exports = merge(baseConfig, {
  mode: 'development', // 开发模式,打包更加快速,省了代码优化步骤
  devtool: 'eval-cheap-module-source-map', // 源码调试模式,后面会讲
  devServer: {
    port: 3000, // 服务端口号
    compress: false, // gzip压缩,开发环境不开启,提升热更新速度
    hot: true, // 开启热更新，后面会讲react模块热替换具体配置
    historyApiFallback: true, // 解决history路由404问题
    static: {
      directory: path.join(__dirname, "../public"), //托管静态资源public文件夹
    }
  }
})
```

**2. package.json添加dev脚本**

在**package.json**的**scripts**中添加

```js
// package.json
"scripts": {
  "dev": "webpack-dev-server -c build/webpack.dev.js"
},
```

执行**npm run dev**,就能看到项目已经启动起来了，具体完善的**react**模块热替换在下面会讲到。

### 10.3 webpack打包环境配置

**1. 修改webpack.prod.js代码**

```js
// webpack.prod.js

const { merge } = require('webpack-merge')
const baseConfig = require('./webpack.base.js')
module.exports = merge(baseConfig, {
  mode: 'production', // 生产模式,会开启tree-shaking和压缩代码,以及其他优化
})
```

**2. package.json添加build打包命令脚本**

在**package.json**的**scripts**中添加**build**打包命令

```js
"scripts": {
    "dev": "webpack-dev-server -c build/webpack.dev.js",
    "build": "webpack -c build/webpack.prod.js"
},
```

执行**npm run build**,最终打包在**dist**文件中, 打包结果:

```yaml
dist                    
├── static
|   ├── js
|     ├── main.js
├── index.html
```

**3. 浏览器查看打包结果**

打包后的**dist**文件可以在本地借助**node**服务器**serve**打开,全局安装**serve**

```sh
npm i serve -g
```

然后在项目根目录命令行执行**serve -s dist**,就可以启动打包后的项目了。

到现在一个基础的支持**react**和**ts**的**webpack5**就配置好了,但只有这些功能是远远不够的,还需要继续添加其他配置。


## 十一.  常用功能配置

### 11.1 配置环境变量

环境变量按作用来分可分为两种。

1.  区分是开发模式还是打包构建模式
2.  区分项目业务环境,开发/测试/预测/正式环境

区分开发模式还是打包构建模式可以用**process.env.NODE_ENV**,因为很多第三方包里面判断都是采用的这个环境变量。

区分项目接口环境可以自定义一个环境变量**process.env.BASE_ENV**,设置环境变量可以借助[cross-env](https://www.npmjs.com/package/cross-env)和[webpack.DefinePlugin](https://www.webpackjs.com/plugins/define-plugin/)来设置。

-   **cross-env**：兼容各系统的设置环境变量的包
-   **webpack.DefinePlugin**：**webpack**内置的插件,可以为业务代码注入环境变量

安装**cross-env**

```sh
npm i cross-env -D
```

修改**package.json**的**scripts**脚本字段,删除原先的**dev**和**build**,改为

```js
"scripts": {
    "dev:dev": "cross-env NODE_ENV=development BASE_ENV=development webpack-dev-server -c build/webpack.dev.js",
    "dev:test": "cross-env NODE_ENV=development BASE_ENV=test webpack-dev-server -c build/webpack.dev.js",
    "dev:pre": "cross-env NODE_ENV=development BASE_ENV=pre webpack-dev-server -c build/webpack.dev.js",
    "dev:prod": "cross-env NODE_ENV=development BASE_ENV=production webpack-dev-server -c build/webpack.dev.js",
    
    "build:dev": "cross-env NODE_ENV=production BASE_ENV=development webpack -c build/webpack.prod.js",
    "build:test": "cross-env NODE_ENV=production BASE_ENV=test webpack -c build/webpack.prod.js",
    "build:pre": "cross-env NODE_ENV=production BASE_ENV=pre webpack -c build/webpack.prod.js",
    "build:prod": "cross-env NODE_ENV=production BASE_ENV=production webpack -c build/webpack.prod.js",
  },
```

**dev**开头是开发模式,**build**开头是打包模式,冒号后面对应的**dev**/**test**/**pre**/**prod**是对应的业务环境的**开发**/**测试**/**预测**/**正式**环境。

**process.env.NODE_ENV**环境变量**webpack**会自动根据设置的**mode**字段来给业务代码注入对应的**development**和**prodction**,这里在命令中再次设置环境变量**NODE_ENV**是为了在**webpack**和**babel**的配置文件中访问到。

在**webpack.base.js**中打印一下设置的环境变量

```
js
// webpack.base.js
// ...
console.log('NODE_ENV', process.env.NODE_ENV)
console.log('BASE_ENV', process.env.BASE_ENV)
```

执行**npm run build:dev**,可以看到打印的信息

```js
// NODE_ENV production
// BASE_ENV development
```

当前是打包模式，业务环境是开发环境，这里需要把**process.env.BASE_ENV**注入到业务代码里面，就可以通过该环境变量设置对应环境的接口地址和其他数据，要借助**webpack.DefinePlugin**插件。

修改**webpack.base.js**

```js
// webpack.base.js
// ...
const webpack = require('webpack')
module.export = {
  // ...
  plugins: [
    // ...
    new webpack.DefinePlugin({
      'process.env.BASE_ENV': JSON.stringify(process.env.BASE_ENV)
    })
  ]
}
```

配置后会把值注入到业务代码里面去,**webpack**解析代码匹配到**process.env.BASE_ENV**,就会设置到对应的值。测试一下，在**src/index.tsx**打印一下两个环境变量

```tsx
// src/index.tsx
// ...
console.log('NODE_ENV', process.env.NODE_ENV)
console.log('BASE_ENV', process.env.BASE_ENV)
```

执行**npm run dev:test**,可以在浏览器控制台看到打印的信息

```
js
// NODE_ENV development
// BASE_ENV test
```

当前是开发模式,业务环境是测试环境。

### 11.2 处理css文件

在**src**下新增**app.css**

```css
h2 {
    color: red;
    transform: translateY(100px);
}
```

在**src/App.tsx**中引入**app.css**

```tsx
import React from 'react'
import './app.css'

function App() {
  return <h2>webpack5-rea11ct-ts</h2>
}
export default App
```

执行打包命令**npm run build:dev**,会发现有报错, 因为**webpack**默认只认识**js**,是不识别**css**文件的,需要使用**loader**来解析**css**, 安装依赖

```sh
npm i style-loader css-loader -D
```

-   **style-loader**: 把解析后的**css**代码从**js**中抽离,放到头部的**style**标签中(在运行时做的)
-   **css-loader:** 解析**css**文件代码

因为解析**css**的配置开发和打包环境都会用到,所以加在公共配置**webpack.base.js**中

```js
// webpack.base.js
// ...
module.exports = {
  // ...
  module: { 
    rules: [
      // ...
      {
        test: /.css$/, //匹配 css 文件
        use: ['style-loader','css-loader']
      }
    ]
  },
  // ...
}
```

上面提到过,**loader**执行顺序是从右往左,从下往上的,匹配到**css**文件后先用**css-loader**解析**css**, 最后借助**style-loader**把**css**插入到头部**style**标签中。

配置完成后再**npm run build:dev**打包,借助**serve -s dist**启动后在浏览器查看,可以看到样式生效了。

### 11.3 支持less或scss

项目开发中,为了更好的提升开发体验,一般会使用**css**超集**less**或者**scss**,对于这些超集也需要对应的**loader**来识别解析。以**scss**为例,需要安装依赖:

```sh
npm i sass-loader sss -D
```

-   **sass-loader**: 解析**scss**文件代码,把**scss**编译为**css**
-   **sass**: **sass**核心

实现支持**scss**也很简单,只需要在**rules**中添加**scss**文件解析,遇到**scss**文件,使用**sass-loader**解析为**css**,再进行**css**解析流程,修改**webpack.base.js**：

```js
// webpack.base.js
module.exports = {
  // ...
  module: {
    // ...
    rules: [
      // ...
      {
        test: /.(css|scss)$/, //匹配 css和scss 文件
        use: ['style-loader','css-loader', 'sass-loader']
      }
    ]
  },
  // ...
}
```

测试一下,新增**src/app.scss**

```scss
#root {
  h2 {
    font-size: 20px;
  }
}
```

在**App.tsx**中引入**app.scss**,执行**npm run build:dev**打包,借助**serve -s dist**启动项目,可以看到**scss**文件编写的样式编译**css**后也插入到**style**标签了了。

### 11.4 处理css3前缀兼容

虽然**css3**现在浏览器支持率已经很高了, 但有时候需要兼容一些低版本浏览器,需要给**css3**加前缀,可以借助插件来自动加前缀, [postcss-loader](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.docschina.org%2Floaders%2Fpostcss-loader%2F)就是来给**css3**加浏览器前缀的,安装依赖：

```sh
npm i postcss-loader autoprefixer -D
```

-   **postcss-loader**：处理**css**时自动加前缀
-   **autoprefixer**：决定添加哪些浏览器前缀到**css**中

修改**webpack.base.js**, 在解析**css**和**scss**的规则中添加配置

```js
module.exports = {
  // ...
  module: { 
    rules: [
      // ...
      {
        test: /.(css|scss)$/, //匹配 css和scss 文件
        use: [
          'style-loader',
          'css-loader',
          // 新增
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['autoprefixer']
              }
            }
          },
          'sass-loader'
        ]
      }
    ]
  },
  // ...
}
```

配置完成后,需要有一份要兼容浏览器的清单,让**postcss-loader**知道要加哪些浏览器的前缀,在根目录创建 **.browserslistrc**文件

```sh
IE 9 # 兼容IE 9
chrome 35 # 兼容chrome 35
```

以兼容到**ie9**和**chrome35**版本为例,配置好后,执行**npm run build:dev**打包,可以看到打包后的**css**文件已经加上了**ie**和谷歌内核的前缀

上面可以看到解析**css**和**scss**有很多重复配置,可以进行提取**postcss-loader**配置优化一下

**postcss.config.js**是**postcss-loader**的配置文件,会自动读取配置,根目录新建**postcss.config.js**：

```js
module.exports = {
  plugins: ['autoprefixer']
}
```

修改**webpack.base.js**, 取消**postcss-loader**的**options**配置

```js
// webpack.base.js
// ...
module.exports = {
  // ...
  module: { 
    rules: [
      // ...
      {
        test: /.(css|scss)$/, //匹配 css和scss 文件
        use: [
          'style-loader',
          'css-loader',
          'postcss-loader',
          'sass-loader'
        ]
      },
    ]
  },
  // ...
}
```

提取**postcss-loader**配置后,再次打包,可以看到依然可以解析**css**, **less**文件, **css3**对应前缀依然存在。

### 11.5 babel预设处理js兼容

现在**js**不断新增很多方便好用的标准语法来方便开发,甚至还有非标准语法比如装饰器,都极大的提升了代码可读性和开发效率。但前者标准语法很多低版本浏览器不支持,后者非标准语法所有的浏览器都不支持。需要把最新的标准语法转换为低版本语法,把非标准语法转换为标准语法才能让浏览器识别解析,而**babel**就是来做这件事的,这里只讲配置,更详细的可以看[Babel 那些事儿](https://juejin.cn/post/6992371845349507108)。

安装依赖

```sh
npm i babel-loader @babel/core @babel/preset-env core-js -D
```

-   babel-loader: 使用 **babel** 加载最新js代码并将其转换为 **ES5**（上面已经安装过）
-   @babel/corer: **babel** 编译的核心包
-   @babel/preset-env: **babel** 编译的预设,可以转换目前最新的**js**标准语法
-   core-js: 使用低版本**js**语法模拟高版本的库,也就是垫片

修改**webpack.base.js**

```js
// webpack.base.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /.(ts|tsx)$/,
        use: {
          loader: 'babel-loader',
          options: {
            // 执行顺序由右往左,所以先处理ts,再处理jsx,最后再试一下babel转换为低版本语法
            presets: [
              [
                "@babel/preset-env",
                {
                  // 设置兼容目标浏览器版本,这里可以不写,babel-loader会自动寻找上面配置好的文件.browserslistrc
                  // "targets": {
                  //  "chrome": 35,
                  //  "ie": 9
                  // },
                   "useBuiltIns": "usage", // 根据配置的浏览器兼容,以及代码中使用到的api进行引入polyfill按需添加
                   "corejs": 3, // 配置使用core-js低版本
                  }
                ],
              '@babel/preset-react',
              '@babel/preset-typescript'
            ]
          }
        }
      }
    ]
  }
}
```

此时再打包就会把语法转换为对应浏览器兼容的语法了。

为了避免**webpack**配置文件过于庞大,可以把**babel-loader**的配置抽离出来, 新建**babel.config.js**文件,使用**js**作为配置文件,是因为可以访问到**process.env.NODE_ENV**环境变量来区分是开发还是打包模式。

```js
// babel.config.js
module.exports = {
  // 执行顺序由右往左,所以先处理ts,再处理jsx,最后再试一下babel转换为低版本语法
  "presets": [
    [
      "@babel/preset-env",
      {
        // 设置兼容目标浏览器版本,这里可以不写,babel-loader会自动寻找上面配置好的文件.browserslistrc
        // "targets": {
        //  "chrome": 35,
        //  "ie": 9
        // },
        "useBuiltIns": "usage", // 根据配置的浏览器兼容,以及代码中使用到的api进行引入polyfill按需添加
        "corejs": 3 // 配置使用core-js使用的版本
      }
    ],
    "@babel/preset-react",
    "@babel/preset-typescript"
  ]
}
```

移除**webpack.base.js**中**babel-loader**的**options**配置

```js
// webpack.base.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /.(ts|tsx)$/,
        use: 'babel-loader'
      },
      // 如果node_moduels中也有要处理的语法，可以把js|jsx文件配置加上
      // {
      //  test: /.(js|jsx)$/,
      //  use: 'babel-loader'
      // }
      // ...
    ]
  }
}
```

### 11.6 babel处理js非标准语法

现在**react**主流开发都是函数组件和**react-hooks**,但有时也会用类组件,可以用装饰器简化代码。

新增**src/components/Class.tsx**组件, 在**App.tsx**中引入该组件使用

```tsx
import React, { PureComponent } from "react";

// 装饰器为,组件添加age属性
function addAge(Target: Function) {
  Target.prototype.age = 111
}
// 使用装饰圈
@addAge
class Class extends PureComponent {

  age?: number

  render() {
    return (
      <h2>我是类组件---{this.age}</h2>
    )
  }
}

export default Class
```

需要开启一下**ts**装饰器支持,修改**tsconfig.json**文件

```js
// tsconfig.json
{
  "compilerOptions": {
    // ...
    // 开启装饰器使用
    "experimentalDecorators": true
  }
}
```

上面Class组件代码中使用了装饰器,目前**js**标准语法是不支持的,现在运行或者打包会报错,不识别装饰器语法,需要借助**babel-loader**插件,安装依赖

```sh
npm i @babel/plugin-proposal-decorators -D
```

在**babel.config.js**中添加插件

```js
module.exports = { 
  // ...
  "plugins": [
    ["@babel/plugin-proposal-decorators", { "legacy": true }]
  ]
}
```

现在项目就支持装饰器了。

### 11.7 复制public文件夹

一般**public**文件夹都会放一些静态资源,可以直接根据绝对路径引入,比如**图片**,**css**,**js**文件等,不需要**webpack**进行解析,只需要打包的时候把**public**下内容复制到构建出口文件夹中,可以借助[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin)插件,安装依赖

```sh
npm i copy-webpack-plugin -D
```

开发环境已经在**devServer**中配置了**static**托管了**public**文件夹,在开发环境使用绝对路径可以访问到**public**下的文件,但打包构建时不做处理会访问不到,所以现在需要在打包配置文件**webpack.prod.js**中新增**copy**插件配置。

```js
// webpack.prod.js
// ..
const path = require('path')
const CopyPlugin = require('copy-webpack-plugin');
module.exports = merge(baseConfig, {
  mode: 'production',
  plugins: [
    // 复制文件插件
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, '../public'), // 复制public下文件
          to: path.resolve(__dirname, '../dist'), // 复制到dist目录中
          filter: source => {
            return !source.includes('index.html') // 忽略index.html
          }
        },
      ],
    }),
  ]
})
```

在上面的配置中,忽略了**index.html**,因为**html-webpack-plugin**会以**public**下的**index.html**为模板生成一个**index.html**到**dist**文件下,所以不需要再复制该文件了。

测试一下,在**public**中新增一个[**favicon.ico**](https://guojiongwei.top/favicon.ico)图标文件,在**index.html**中引入

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- 绝对路径引入图标文件 -->
  <link data-n-head="ssr" rel="icon" type="image/x-icon" href="/favicon.ico">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>webpack5-react-ts</title>
</head>
<body>
  <!-- 容器节点 -->
  <div id="root"></div>
</body>
</html>
```

再执行**npm run build:dev**打包,就可以看到**public**下的**favicon.ico**图标文件被复制到**dist**文件中了。

### 11.8 处理图片文件

对于图片文件,**webpack4**使用**file-loader**和**url-loader**来处理的,但**webpack5**不使用这两个**loader**了,而是采用自带的[**asset-module**](https://webpack.js.org/guides/asset-modules/#root)来处理

修改**webpack.base.js**,添加图片解析配置

```js
module.exports = {
  module: {
    rules: [
      // ...
      {
        test:/.(png|jpg|jpeg|gif|svg)$/, // 匹配图片文件
        type: "asset", // type选择asset
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb转base64位
          }
        },
        generator:{ 
          filename:'static/images/[name][ext]', // 文件输出目录和命名
        },
      },
    ]
  }
}
```

测试一下,准备一张小于10kb的图片和大于10kb的图片,放在**src/assets/imgs**目录下, 修改**App.tsx**:

```js
import smallImg from './assets/imgs/5kb.png'
import bigImg from './assets/imgs/22kb.png'
import './app.scss'

function App() {
  return (
    <>
      <img src={smallImg} alt="小于10kb的图片" />
      <img src={bigImg} alt="大于于10kb的图片" />
    </>
  )
}
export default App
```

> 这个时候在引入图片的地方会报：**找不到模块“./assets/imgs/22kb.png”或其相应的类型声明**，需要添加一个图片的声明文件

新增**src/images.d.ts**文件，添加内容

```js
declare module '*.svg'
declare module '*.png'
declare module '*.jpg'
declare module '*.jpeg'
declare module '*.gif'
declare module '*.bmp'
declare module '*.tiff'
declare module '*.less'
declare module '*.css'
```

添加图片声明文件后,就可以正常引入图片了, 然后执行**npm run build:dev**打包,借助**serve -s dist**查看效果,可以看到可以正常解析图片了,并且小于**10kb**的图片被转成了**base64**位格式的。

**css**中的背景图片一样也可以解析,修改**app.tsx**。

```tsx
import React from 'react'
import smallImg from './assets/imgs/5kb.png'
import bigImg from './assets/imgs/22kb.png'
import './app.css'
import './app.scss'

function App() {
  return (
    <>
      <img src={smallImg} alt="小于10kb的图片" />
      <img src={bigImg} alt="大于于10kb的图片" />
      <div className='smallImg'></div> {/* 小图片背景容器 */}
      <div className='bigImg'></div> {/* 大图片背景容器 */}
    </>
  )
}
export default App
```

修改**app.scss**

```scss
// app.scss
#root {
  .smallImg {
    width: 69px;
    height: 75px;
    background: url('./assets/imgs/5kb.png') no-repeat;
  }
  .bigImg {
    width: 232px;
    height: 154px;
    background: url('./assets/imgs/22kb.png') no-repeat;
  }
}
```

可以看到背景图片也一样可以识别,小于**10kb**转为**base64**位。

### 11.9 处理字体和媒体文件

字体文件和媒体文件这两种资源处理方式和处理图片是一样的,只需要把匹配的路径和打包后放置的路径修改一下就可以了。修改**webpack.base.js**文件：

```js
// webpack.base.js
module.exports = {
  module: {
    rules: [
      // ...
      {
        test:/.(woff2?|eot|ttf|otf)$/, // 匹配字体图标文件
        type: "asset", // type选择asset
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb转base64位
          }
        },
        generator:{ 
          filename:'static/fonts/[name][ext]', // 文件输出目录和命名
        },
      },
      {
        test:/.(mp4|webm|ogg|mp3|wav|flac|aac)$/, // 匹配媒体文件
        type: "asset", // type选择asset
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb转base64位
          }
        },
        generator:{ 
          filename:'static/media/[name][ext]', // 文件输出目录和命名
        },
      },
    ]
  }
}
```


## 十二. 配置**react**模块热替换

热更新上面已经在**devServer**中配置**hot**为**true**, 在**webpack4**中,还需要在插件中添加了**HotModuleReplacementPlugin**,在**webpack5**中,只要**devServer.hot**为**true**了,该插件就已经内置了。

现在开发模式下修改**css**和**scss**文件，页面样式可以在不刷新浏览器的情况实时生效，因为此时样式都在**style**标签里面，**style-loader**做了替换样式的热替换功能。但是修改**App.tsx**,浏览器会自动刷新后再显示修改后的内容,但我们想要的不是刷新浏览器,而是在不需要刷新浏览器的前提下模块热更新,并且能够保留**react**组件的状态。

可以借助[@pmmmwh/react-refresh-webpack-plugin](https://www.npmjs.com/package/@pmmmwh/react-refresh-webpack-plugin)插件来实现,该插件又依赖于[react-refresh](https://www.npmjs.com/package/react-refresh), 安装依赖：

```sh
npm i @pmmmwh/react-refresh-webpack-plugin react-refresh -D
```

配置**react**热更新插件,修改**webpack.dev.js**

```js
// webpack.dev.js
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

module.exports = merge(baseConfig, {
  // ...
  plugins: [
    new ReactRefreshWebpackPlugin(), // 添加热更新插件
  ]
})
```

为**babel-loader**配置**react-refesh**刷新插件,修改**babel.config.js**文件

```js
const isDEV = process.env.NODE_ENV === 'development' // 是否是开发模式
module.exports = {
  // ...
  "plugins": [
    isDEV && require.resolve('react-refresh/babel'), // 如果是开发模式,就启动react热更新插件
    // ...
  ].filter(Boolean) // 过滤空值
}
```

测试一下,修改**App.tsx**代码

```tsx
import React, { useState } from 'react'

function App() {
  const [ count, setCounts ] = useState('')
  const onChange = (e: any) => {
    setCounts(e.target.value)
  }
  return (
    <>
      <h2>webpack5+react+ts</h2>
      <p>受控组件</p>
      <input type="text" value={count} onChange={onChange} />
      <br />
      <p>非受控组件</p>
      <input type="text" />
    </>
  )
}
export default App
```

会发现在不刷新浏览器的情况下,页面内容进行了热更新,并且**react**组件状态也会保留。

> 新增或者删除页面**hooks**时,热更新时组件状态不会保留。


## 十三. 优化构建速度

### 13.1 构建耗时分析

当进行优化的时候,肯定要先知道时间都花费在哪些步骤上了,而[speed-measure-webpack-plugin](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fspeed-measure-webpack-plugin)插件可以帮我们做到,安装依赖：

```sh
npm i speed-measure-webpack-plugin -D
```

使用的时候为了不影响到正常的开发/打包模式,我们选择新建一个配置文件,新增**webpack**构建分析配置文件**build/webpack.analy.js**

```js
const prodConfig = require('./webpack.prod.js') // 引入打包配置
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin'); // 引入webpack打包速度分析插件
const smp = new SpeedMeasurePlugin(); // 实例化分析插件
const { merge } = require('webpack-merge') // 引入合并webpack配置方法

// 使用smp.wrap方法,把生产环境配置传进去,由于后面可能会加分析配置,所以先留出合并空位
module.exports = smp.wrap(merge(prodConfig, {

}))
```

修改**package.json**添加启动**webpack**打包分析脚本命令,在**scripts**新增：

```js
{
  // ...
  "scripts": {
    // ...
    "build:analy": "cross-env NODE_ENV=production BASE_ENV=production webpack -c build/webpack.analy.js"
  }
  // ...
}
```

执行**npm run build:analy**命令

可以看到各**plugin**和**loader**的耗时时间,现在因为项目内容比较少,所以耗时都比较少,在真正的项目中可以通过这个来分析打包时间花费在什么地方,然后来针对性的优化。

### 13.2 开启持久化存储缓存

在**webpack5**之前做缓存是使用**babel-loader**缓存解决**js**的解析结果,**cache-loader**缓存**css**等资源的解析结果,还有模块缓存插件**hard-source-webpack-plugin**,配置好缓存后第二次打包,通过对文件做哈希对比来验证文件前后是否一致,如果一致则采用上一次的缓存,可以极大地节省时间。

**webpack5** 较于 **webpack4**,新增了持久化缓存、改进缓存算法等优化,通过配置 [webpack 持久化缓存](https%3A%2F%2Fwebpack.docschina.org%2Fconfiguration%2Fcache%2F%23root),来缓存生成的 **webpack** 模块和 **chunk**,改善下一次打包的构建速度,可提速 **90%** 左右,配置也简单，修改**webpack.base.js**

```js

// webpack.base.js
// ...
module.exports = {
  // ...
  cache: {
    type: 'filesystem', // 使用文件缓存
  },
}
```

当前文章代码的测试结果

| 模式     | 第一次耗时  | 第二次耗时 |
| ------ | ------ | ----- |
| 启动开发模式 | 2869毫秒 | 687毫秒 |
| 启动打包模式 | 5455毫秒 | 552毫秒 |

通过开启**webpack5**持久化存储缓存,再次打包的时间提升了**90%** 。

缓存的存储位置在**node_modules/.cache/webpack**,里面又区分了**development**和**production**缓存

### 13.3 开启多线程loader

**webpack**的**loader**默认在单线程执行,现代电脑一般都有多核**cpu**,可以借助多核**cpu**开启多线程**loader**解析,可以极大地提升**loader**解析的速度,[thread-loader](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.docschina.org%2Floaders%2Fthread-loader%2F%23root)就是用来开启多进程解析**loader**的,安装依赖

```sh
npm i thread-loader -D
```

使用时,需将此 **loader** 放置在其他 **loader** 之前。放置在此 **loader** 之后的 **loader** 会在一个独立的 **worker** 池中运行。

修改**webpack.base.js**

```js
// webpack.base.js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /.(ts|tsx)$/,
        use: ['thread-loader', 'babel-loader']
      }
    ]
  }
}
```

由于**thread-loader**不支持抽离css插件**MiniCssExtractPlugin.loader**(下面会讲),所以这里只配置了多进程解析**js**,开启多线程也是需要启动时间,大约**600ms**左右,所以适合规模比较大的项目。

### 13.4 配置alias别名

**webpack**支持设置别名**alias**,设置别名可以让后续引用的地方减少路径的复杂度。

修改**webpack.base.js**

```js
module.export = {
  // ...
   resolve: {
    // ...
    alias: {
      '@': path.join(__dirname, '../src')
    }
  }
}
```

修改**tsconfig.json**,添加**baseUrl**和**paths**

```js
{
  "compilerOptions": {
    // ...
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "src/*"
      ]
    }
  }
}
```

配置修改完成后,在项目中使用 **@/xxx.xx**,就会指向项目中**src/xxx.xx,**在**js/ts**文件和**css**文件中都可以用。

**src/App.tsx**可以修改为

```tsx
import React from 'react'
import smallImg from '@/assets/imgs/5kb.png'
import bigImg from '@/assets/imgs/22kb.png'
import '@/app.css'
import '@/app.scss'

function App() {
  return (
    <>
      <img src={smallImg} alt="小于10kb的图片" />
      <img src={bigImg} alt="大于于10kb的图片" />
      <div className='smallImg'></div> {/* 小图片背景容器 */}
      <div className='bigImg'></div> {/* 大图片背景容器 */}
    </>
  )
}
export default App
```

**src/app.scss**可以修改为

```scss
// app.scss
#root {
  .smallImg {
    width: 69px;
    height: 75px;
    background: url('@/assets/imgs/5kb.png') no-repeat;
  }
  .bigImg {
    width: 232px;
    height: 154px;
    background: url('@/assets/imgs/22kb.png') no-repeat;
  }
}
```

### 13.5 缩小loader作用范围

一般第三库都是已经处理好的,不需要再次使用**loader**去解析,可以按照实际情况合理配置**loader**的作用范围,来减少不必要的**loader**解析,节省时间,通过使用 **include**和**exclude** 两个配置项,可以实现这个功能,常见的例如：

-   **include**：只解析该选项配置的模块
-   **exclude**：不解该选项配置的模块,优先级更高

修改**webpack.base.js**

```js
// webpack.base.js
const path = require('path')
module.exports = {
  // ...
  module: {
    rules: [
      {
        include: [path.resolve(__dirname, '../src')], 只对项目src文件的ts,tsx进行loader解析
        test: /.(ts|tsx)$/,
        use: ['thread-loader', 'babel-loader']
      }
    ]
  }
}
```

其他**loader**也是相同的配置方式,如果除**src**文件外也还有需要解析的,就把对应的目录地址加上就可以了,比如需要引入**antd**的**css**,可以把**antd**的文件目录路径添加解析**css**规则到**include**里面。

### 13.6 精确使用loader

**loader**在**webpack**构建过程中使用的位置是在**webpack**构建模块依赖关系引入新文件时，会根据文件后缀来倒序遍历**rules**数组，如果文件后缀和**test**正则匹配到了，就会使用该**rule**中配置的**loader**依次对文件源代码进行处理，最终拿到处理后的**sourceCode**结果，可以通过避免使用无用的**loader**解析来提升构建速度，比如使用**sass-loader**解析**css**文件。

可以拆分上面配置的**sass**和**css**, 避免让**sass-loader**再去解析**css**文件

```js
// webpack.base.js
// ...
module.exports = {
  module: {
    // ...
    rules: [
      // ...
      {
        test: /.\css$/, //匹配所有的 css 文件
        include: [path.resolve(__dirname, '../src')],
        use: [
          'style-loader',
          'css-loader',
          'postcss-loader'
        ]
      },
      {
        test: /.\scss$/, //匹配所有的 scss 文件
        include: [path.resolve(__dirname, '../src')],
        use: [
          'style-loader',
          'css-loader',
          'postcss-loader',
          'sass-loader'
        ]
      },
    ]
  }
}
```

**ts**和**tsx**也是如此，**ts**里面是不能写**jsx**语法的，所以可以尽可能避免使用 **@babel/preset-react**对 **.ts** 文件语法做处理。

### 13.7 缩小模块搜索范围

**node**里面模块有三种

-   **node**核心模块
-   **node_modules**模块
-   自定义文件模块

使用**require**和**import**引入模块时如果有准确的相对或者绝对路径,就会去按路径查询,如果引入的模块没有路径,会优先查询**node**核心模块,如果没有找到会去当前目录下**node_modules**中寻找,如果没有找到会查从父级文件夹查找**node_modules**,一直查到系统**node**全局模块。

这样会有两个问题,一个是当前项目没有安装某个依赖,但是上一级目录下**node_modules**或者全局模块有安装,就也会引入成功,但是部署到服务器时可能就会找不到造成报错,另一个问题就是一级一级查询比较消耗时间。可以告诉**webpack**搜索目录范围,来规避这两个问题。

修改**webpack.base.js**

```js
// webpack.base.js
const path = require('path')
module.exports = {
  // ...
  resolve: {
     // ...
     // 如果用的是pnpm 就暂时不要配置这个，会有幽灵依赖的问题，访问不到很多模块。
     modules: [path.resolve(__dirname, '../node_modules')], // 查找第三方模块只在本项目的node_modules中查找
  },
}
```

### 13 .8 devtool 配置

开发过程中或者打包后的代码都是**webpack**处理后的代码,如果进行调试肯定希望看到源代码,而不是编译后的代码, [source map](http://blog.teamtreehouse.com/introduction-source-maps)就是用来做源码映射的,不同的映射模式会明显影响到构建和重新构建的速度, [**devtool**](https://webpack.js.org/configuration/devtool/)选项就是**webpack**提供的选择源码映射方式的配置。

**devtool**的命名规则为 **^(inline-|hidden-|eval-)?(nosources-)?(cheap-(module-)?)?source-map$**

| 关键字       | 描述                                           |
| --------- | -------------------------------------------- |
| inline    | 代码内通过 dataUrl 形式引入 SourceMap                 |
| hidden    | 生成 SourceMap 文件,但不使用                         |
| eval      | `eval(...)` 形式执行代码,通过 dataUrl 形式引入 SourceMap |
| nosources | 不生成 SourceMap                                |
| cheap     | 只需要定位到行信息,不需要列信息                             |
| module    | 展示源代码中的错误位置                                  |

开发环境推荐：**eval-cheap-module-source-map**

-   本地开发首次打包慢点没关系,因为 **eval** 缓存的原因, 热更新会很快
-   开发中,我们每行代码不会写的太长,只需要定位到行就行,所以加上 **cheap**
-   我们希望能够找到源代码的错误,而不是打包后的,所以需要加上 **module**

修改**webpack.dev.js**

```js
// webpack.dev.js
module.exports = {
  // ...
  devtool: 'eval-cheap-module-source-map'
}
```

打包环境推荐：**none**(就是不配置**devtool**选项了，不是配置**devtool**: '**none**')

```js
// webpack.prod.js
module.exports = {
  // ...
  // devtool: '', // 不用配置devtool此项
}
```

-   **none**话调试只能看到编译后的代码,也不会泄露源代码,打包速度也会比较快。
-   只是不方便线上排查问题, 但一般都可以根据报错信息在本地环境很快找出问题所在。

### 13.9 其他优化配置

除了上面的配置外，**webpack**还提供了其他的一些优化方式,本次搭建没有使用到，所以只简单罗列下

-   [**externals**](https://www.webpackjs.com/configuration/externals/): 外包拓展，打包时会忽略配置的依赖，会从上下文中寻找对应变量
-   [**module.noParse**](hhttps://www.webpackjs.com/configuration/module/#module-noparse): 匹配到设置的模块,将不进行依赖解析，适合**jquery**,**boostrap**这类不依赖外部模块的包
-   [**ignorePlugin**](https://webpack.js.org/plugins/ignore-plugin/#root): 可以使用正则忽略一部分文件，常在使用多语言的包时可以把非中文语言包过滤掉



## 十四. 优化构建结果文件

[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)是分析**webpack**打包后文件的插件,使用交互式可缩放树形图可视化 **webpack** 输出文件的大小。通过该插件可以对打包后的文件进行观察和分析,可以方便我们对不完美的地方针对性的优化,安装依赖：

```sh
npm install webpack-bundle-analyzer -D
```

修改**webpack.analy.js**

```js
// webpack.analy.js
const prodConfig = require('./webpack.prod.js')
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');
const smp = new SpeedMeasurePlugin();
const { merge } = require('webpack-merge')
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer') // 引入分析打包结果插件
module.exports = smp.wrap(merge(prodConfig, {
  plugins: [
    new BundleAnalyzerPlugin() // 配置分析打包结果插件
  ]
}))
```

配置好后,执行**npm run build:analy**命令,打包完成后浏览器会自动打开窗口,可以看到打包文件的分析结果页面,可以看到各个文件所占的资源大小。

### 14.1 抽取css样式文件

在开发环境我们希望**css**嵌入在**style**标签里面,方便样式热替换,但打包时我们希望把**css**单独抽离出来,方便配置缓存策略。而插件[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)就是来帮我们做这件事的,安装依赖：

```sh
npm i mini-css-extract-plugin -D
```

修改**webpack.base.js**, 根据环境变量设置开发环境使用**style-looader**,打包模式抽离**css**

```js
// webpack.base.js
// ...
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const isDev = process.env.NODE_ENV === 'development' // 是否是开发模式
module.exports = {
  // ...
  module: { 
    rules: [
      // ...
      {
        test: /.css$/, //匹配所有的 css 文件
        include: [path.resolve(__dirname, '../src')],
        use: [
          isDev ? 'style-loader' : MiniCssExtractPlugin.loader, // 开发环境使用style-looader,打包模式抽离css
          'css-loader',
          'postcss-loader'
        ]
      },
      {
        test: /.scss$/, //匹配所有的 scss 文件
        include: [path.resolve(__dirname, '../src')],
        use: [
          isDev ? 'style-loader' : MiniCssExtractPlugin.loader, // 开发环境使用style-looader,打包模式抽离css
          'css-loader',
          'postcss-loader',
          'sass-loader'
        ]
      },
    ]
  },
  // ...
}
```

再修改**webpack.prod.js**, 打包时添加抽离css插件

```js
// webpack.prod.js
// ...
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports = merge(baseConfig, {
  mode: 'production',
  plugins: [
    // ...
    // 抽离css插件
    new MiniCssExtractPlugin({
      filename: 'static/css/[name].css' // 抽离css的输出目录和名称
    }),
  ]
})
```

配置完成后,在开发模式**css**会嵌入到**style**标签里面,方便样式热替换,打包时会把**css**抽离成单独的**css**文件。

### 14.2 压缩css文件

上面配置了打包时把**css**抽离为单独**css**文件的配置,打开打包后的文件查看,可以看到默认**css**是没有压缩的,需要手动配置一下压缩**css**的插件。

可以借助[css-minimizer-webpack-plugin](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fcss-minimizer-webpack-plugin)来压缩css,安装依赖

```sh
npm i css-minimizer-webpack-plugin -D
```

修改**webpack.prod.js**文件， 需要在优化项[optimization](https://webpack.js.org/configuration/optimization/)下的[minimizer](https://webpack.js.org/configuration/optimization/#optimizationminimizer)属性中配置

```js
// webpack.prod.js
// ...
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')
module.exports = {
  // ...
  optimization: {
    minimizer: [
      new CssMinimizerPlugin(), // 压缩css
    ],
  },
}
```

再次执行打包就可以看到**css**已经被压缩了。

### 14.3 压缩js文件

设置**mode**为**production**时,**webpack**会使用内置插件[terser-webpack-plugin](https://www.npmjs.com/package/terser-webpack-plugin)压缩**js**文件,该插件默认支持多线程压缩,但是上面配置**optimization.minimizer**压缩**css**后,**js**压缩就失效了,需要手动再添加一下,**webpack**内部安装了该插件,由于**pnpm**解决了幽灵依赖问题,如果用的**pnpm**的话,需要手动再安装一下依赖。

```sh
npm i terser-webpack-plugin -D
```

修改**webpack.prod.js**文件

```js
// ...
const TerserPlugin = require('terser-webpack-plugin')
module.exports = {
  // ...
  optimization: {
    minimizer: [
      // ...
      new TerserPlugin({ // 压缩js
        parallel: true, // 开启多线程压缩
        terserOptions: {
          compress: {
            pure_funcs: ["console.log"] // 删除console.log
          }
        }
      }),
    ],
  },
}
```

配置完成后再打包,**css**和**js**就都可以被压缩了。

### 14.4 合理配置打包文件hash

项目维护的时候,一般只会修改一部分代码,可以合理配置文件缓存,来提升前端加载页面速度和减少服务器压力,而**hash**就是浏览器缓存策略很重要的一部分。**webpack**打包的**hash**分三种：

-   **hash**：跟整个项目的构建相关,只要项目里有文件更改,整个项目构建的**hash**值都会更改,并且全部文件都共用相同的**hash**值
-   **chunkhash**：不同的入口文件进行依赖文件解析、构建对应的**chunk**,生成对应的哈希值,文件本身修改或者依赖文件修改,**chunkhash**值会变化
-   **contenthash**：每个文件自己单独的 **hash** 值,文件的改动只会影响自身的 **hash** 值

**hash**是在输出文件时配置的,格式是**filename: "[name].[chunkhash:8][ext]"** , **[xx]** 格式是**webpack**提供的占位符, **:8**是生成**hash**的长度。

| 占位符         | 解释                 |
| ----------- | ------------------ |
| ext         | 文件后缀名              |
| name        | 文件名                |
| path        | 文件相对路径             |
| folder      | 文件所在文件夹            |
| hash        | 每次构建生成的唯一 hash 值   |
| chunkhash   | 根据 chunk 生成 hash 值 |
| contenthash | 根据文件内容生成hash 值     |

因为**js**我们在生产环境里会把一些公共库和程序入口文件区分开,单独打包构建,采用**chunkhash**的方式生成哈希值,那么只要我们不改动公共库的代码,就可以保证其哈希值不会受影响,可以继续使用浏览器缓存,所以**js**适合使用**chunkhash**。

**css**和图片资源媒体资源一般都是单独存在的,可以采用**contenthash**,只有文件本身变化后会生成新**hash**值。

修改**webpack.base.js**,把**js**输出的文件名称格式加上**chunkhash**,把**css**和图片媒体资源输出格式加上**contenthash**

```js
// webpack.base.js
// ...
module.exports = {
  // 打包文件出口
  output: {
    filename: 'static/js/[name].[chunkhash:8].js', // // 加上[chunkhash:8]
    // ...
  },
  module: {
    rules: [
      {
        test:/.(png|jpg|jpeg|gif|svg)$/, // 匹配图片文件
        // ...
        generator:{ 
          filename:'static/images/[name].[contenthash:8][ext]' // 加上[contenthash:8]
        },
      },
      {
        test:/.(woff2?|eot|ttf|otf)$/, // 匹配字体文件
        // ...
        generator:{ 
          filename:'static/fonts/[name].[contenthash:8][ext]', // 加上[contenthash:8]
        },
      },
      {
        test:/.(mp4|webm|ogg|mp3|wav|flac|aac)$/, // 匹配媒体文件
        // ...
        generator:{ 
          filename:'static/media/[name].[contenthash:8][ext]', // 加上[contenthash:8]
        },
      },
    ]
  },
  // ...
}
```

再修改**webpack.prod.js**,修改抽离**css**文件名称格式

```js
// webpack.prod.js
// ...
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports = merge(baseConfig, {
  mode: 'production',
  plugins: [
    // 抽离css插件
    new MiniCssExtractPlugin({
      filename: 'static/css/[name].[contenthash:8].css' // 加上[contenthash:8]
    }),
    // ...
  ],
  // ...
})
```

再次打包就可以看到文件后面的**hash**了

### 14.5 代码分割第三方包和公共模块

一般第三方包的代码变化频率比较小,可以单独把**node_modules**中的代码单独打包, 当第三包代码没变化时,对应**chunkhash**值也不会变化,可以有效利用浏览器缓存，还有公共的模块也可以提取出来,避免重复打包加大代码整体体积, **webpack**提供了代码分隔功能, 需要我们手动在优化项[optimization](https://webpack.js.org/configuration/optimization/)中手动配置下代码分隔[splitChunks](https://webpack.js.org/configuration/optimization/#optimizationsplitchunks)规则。

修改**webpack.prod.js**

```js
module.exports = {
  // ...
  optimization: {
    // ...
    splitChunks: { // 分隔代码
      cacheGroups: {
        vendors: { // 提取node_modules代码
          test: /node_modules/, // 只匹配node_modules里面的模块
          name: 'vendors', // 提取文件命名为vendors,js后缀和chunkhash会自动加
          minChunks: 1, // 只要使用一次就提取出来
          chunks: 'initial', // 只提取初始化就能获取到的模块,不管异步的
          minSize: 0, // 提取代码体积大于0就提取出来
          priority: 1, // 提取优先级为1
        },
        commons: { // 提取页面公共代码
          name: 'commons', // 提取文件命名为commons
          minChunks: 2, // 只要使用两次就提取出来
          chunks: 'initial', // 只提取初始化就能获取到的模块,不管异步的
          minSize: 0, // 提取代码体积大于0就提取出来
        }
      }
    }
  }
}
```

配置完成后执行打包,可以看到**node_modules**里面的模块被抽离到**verdors.ec685ef1.js**中,业务代码在**main.948bf38a.js**中。

测试一下,此时**verdors.js**的**chunkhash**是**ec685ef1**,**main.js**文件的**chunkhash**是**948bf38a**,改动一下**App.tsx**,再次打包,可以看到下图**main.js**的**chunkhash**值变化了,但是**vendors.js**的**chunkhash**还是原先的,这样发版后,浏览器就可以继续使用缓存中的**verdors.ec685ef1.js**,只需要重新请求**main.js**就可以了。

### 14.6 tree-shaking清理未引用js

[Tree Shaking](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.docschina.org%2Fguides%2Ftree-shaking%2F)的意思就是摇树,伴随着摇树这个动作,树上的枯叶都会被摇晃下来,这里的**tree-shaking**在代码中摇掉的是未使用到的代码,也就是未引用的代码,最早是在**rollup**库中出现的,**webpack**在**2**版本之后也开始支持。模式**mode**为**production**时就会默认开启**tree-shaking**功能以此来标记未引入代码然后移除掉,测试一下。

在**src/components**目录下新增**Demo1**,**Demo2**两个组件

```tsx
// src/components/Demo1.tsx
import React from "react";
function Demo1() {
  return <h3>我是Demo1组件</h3>
}
export default Demo1

// src/components/Demo2.tsx
import React from "react";
function Demo2() {
  return <h3>我是Demo2组件</h3>
}
export default Demo2
```

再在**src/components**目录下新增**index.ts**, 把**Demo1**和**Demo2**组件引入进来再暴露出去

```ts
// src/components/index.ts
export { default as Demo1 } from './Demo1'
export { default as Demo2 } from './Demo2'
```

在**App.tsx**中引入两个组件,但只使用**Demo1**组件

```tsx
// ...
import { Demo1, Demo2 } from '@/components'

function App() {
  return <Demo1 />
}
export default App
```

执行打包,可以在**main.js**中搜索**Demo**,只搜索到了**Demo1**, 代表**Demo2**组件被**tree-shaking**移除掉了。

### 14.7 tree-shaking清理未使用css

**js**中会有未使用到的代码,**css**中也会有未被页面使用到的样式,可以通过[purgecss-webpack-plugin](https://www.npmjs.com/package/purgecss-webpack-plugin)插件打包的时候移除未使用到的**css**样式,这个插件是和[mini-css-extract-plugin](https://www.npmjs.com/package/mini-css-extract-plugin)插件配合使用的,在上面已经安装过,还需要[**glob-all**](https://www.npmjs.com/package/glob-all)来选择要检测哪些文件里面的类名和**id**还有标签名称, 安装依赖:

```sh
npm i purgecss-webpack-plugin@4 glob -D
```

> 本文版本是4版本最新的5版本导入方式需要改为 const { PurgeCSSPlugin } = require('purgecss-webpack-plugin')

修改**webpack.prod.js**

```js
// webpack.prod.js
// ...
const glob = require('glob')
const PurgeCSSPlugin = require('purgecss-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports = {
  // ...
  plugins: [
    // 抽离css插件
    new MiniCssExtractPlugin({
      filename: 'static/css/[name].[contenthash:8].css'
    }),
    // 清理无用css
    new PurgeCSSPlugin({
      // 检测src下所有tsx文件和public下index.html中使用的类名和id和标签名称
      // 只打包这些文件中用到的样式
      paths: glob.sync([
        `${path.join(__dirname, '../src')}/**/*.tsx`,
        path.join(__dirname, '../public/index.html')
      ]),
    }),
  ]
}
```

测试一下,用上面配置解析图片文件代码拿过来,修改**App.tsx**

```tsx
import React from 'react'
import './app.css'
import './app.scss'

function App() {
  return (
    <>
      <div className='smallImg'></div>
      <div className='bigImg'></div>
    </>
  )
}
export default App
```

**App.tsx**中有两个**div**,类名分别是**smallImg**和**bigImg**,当前**app.scss**代码为

```scss
#root {
  .smallImg {
    width: 69px;
    height: 75px;
    background: url('./assets/imgs/5kb.png') no-repeat;
  }
  .bigImg {
    width: 232px;
    height: 154px;
    background: url('./assets/imgs/22kb.png') no-repeat;
  }
}
```

此时先执行一下打包,查看**main.css**

因为页面中中有**h2**标签, **smallImg**和**bigImg**类名,所以打包后的**css**也有,此时修改一下**app.scss**中的 **.smallImg**为 **.smallImg1**,此时 **.smallImg1**就是无用样式了,因为没有页面没有类名为 **.smallImg1**的节点,再打包后查看 **main.css**

可以看到**main.css**已经没有 **.smallImg1**类名的样式了,做到了删除无用**css**的功能。

但是**purgecss-webpack-plugin**插件不是全能的,由于项目业务代码的复杂,插件不能百分百识别哪些样式用到了,哪些没用到,所以请不要寄希望于它能够百分百完美解决你的问题,这个是不现实的。

插件本身也提供了一些白名单**safelist**属性,符合配置规则选择器都不会被删除掉,比如使用了组件库[antd](https://ant.design/), **purgecss-webpack-plugin**插件检测**src**文件下**tsx**文件中使用的类名和**id**时,是检测不到在**src**中使用**antd**组件的类名的,打包的时候就会把**antd**的类名都给过滤掉,可以配置一下安全选择列表,避免删除**antd**组件库的前缀**ant**。

```js
new PurgeCSSPlugin({
  // ...
  safelist: {
    standard: [/^ant-/], // 过滤以ant-开头的类名，哪怕没用到也不删除
  }
})
```

### 14.8 资源懒加载

像**react**,**vue**等单页应用打包默认会打包到一个**js**文件中,虽然使用代码分割可以把**node_modules**模块和**公共模块**分离,但页面初始加载还是会把整个项目的代码下载下来,其实只需要公共资源和当前页面的资源就可以了,其他页面资源可以等使用到的时候再加载,可以有效提升首屏加载速度。

**webpack**默认支持资源懒加载,只需要引入资源使用**import**语法来引入资源,**webpack**打包的时候就会自动打包为单独的资源文件,等使用到的时候动态加载。

以懒加载组件和**css**为例,新建懒加载组件**src/components/LazyDemo.tsx**

```tsx
import React from "react";

function LazyDemo() {
  return <h3>我是懒加载组件组件</h3>
}

export default LazyDemo
```

修改**App.tsx**

```tsx
import React, { lazy, Suspense, useState } from 'react'
const LazyDemo = lazy(() => import('@/components/LazyDemo')) // 使用import语法配合react的Lazy动态引入资源

function App() {
  const [ show, setShow ] = useState(false)
  
  // 点击事件中动态引入css, 设置show为true
  const onClick = () => {
    import('./app.css')
    setShow(true)
  }
  return (
    <>
      <h2 onClick={onClick}>展示</h2>
      {/* show为true时加载LazyDemo组件 */}
      { show && <Suspense fallback={null}><LazyDemo /></Suspense> }
    </>
  )
}
export default App
```

点击展示文字时,才会动态加载**app.css**和**LazyDemo**组件的资源。

### 14.9 资源预加载

上面配置了资源懒加载后,虽然提升了首屏渲染速度,但是加载到资源的时候会有一个去请求资源的延时,如果资源比较大会出现延迟卡顿现象,可以借助**link**标签的**rel**属性**prefetch**与**preload**,**link**标签除了加载**css**之外也可以加载**js**资源,设置**rel**属性可以规定**link**提前加载资源,但是加载资源后不执行,等用到了再执行。

**rel的属性值**

-   **preload**是告诉浏览器页面必定需要的资源,浏览器一定会加载这些资源。
-   **prefetch**是告诉浏览器页面可能需要的资源,浏览器不一定会加载这些资源,会在空闲时加载。

对于当前页面很有必要的资源使用 **preload** ,对于可能在将来的页面中使用的资源使用 **prefetch**。

**webpack v4.6.0+** 增加了对[预获取和预加载](https://webpack.docschina.org/guides/code-splitting/#prefetchingpreloading-modules)的支持,使用方式也比较简单,在**import**引入动态资源时使用**webpack**的魔法注释

```js
// 单个目标
import(
  /* webpackChunkName: "my-chunk-name" */ // 资源打包后的文件chunkname
  /* webpackPrefetch: true */ // 开启prefetch预获取
  /* webpackPreload: true */ // 开启preload预获取
  './module'
);
```

测试一下,在**src/components**目录下新建**PreloadDemo.tsx**, **PreFetchDemo.tsx**

```tsx
// src/components/PreloadDemo.tsx
import React from "react";
function PreloadDemo() {
  return <h3>我是PreloadDemo组件</h3>
}
export default PreloadDemo

// src/components/PreFetchDemo.tsx
import React from "react";
function PreFetchDemo() {
  return <h3>我是PreFetchDemo组件</h3>
}
export default PreFetchDemo
```

修改**App.tsx**

```tsx
import React, { lazy, Suspense, useState } from 'react'

// prefetch
const PreFetchDemo = lazy(() => import(
  /* webpackChunkName: "PreFetchDemo" */
  /*webpackPrefetch: true*/
  '@/components/PreFetchDemo'
))
// preload
const PreloadDemo = lazy(() => import(
  /* webpackChunkName: "PreloadDemo" */
  /*webpackPreload: true*/
  '@/components/PreloadDemo'
 ))

function App() {
  const [ show, setShow ] = useState(false)

  const onClick = () => {
    setShow(true)
  }
  return (
    <>
      <h2 onClick={onClick}>展示</h2>
      {/* show为true时加载组件 */}
      { show && (
        <>
          <Suspense fallback={null}><PreloadDemo /></Suspense>
          <Suspense fallback={null}><PreFetchDemo /></Suspense>
        </>
      ) }
    </>
  )
}
export default App
```

然后打包后查看效果,页面初始化时预加载了**PreFetchDemo.js**组件资源,但是不执行里面的代码,等点击展示按钮后从预加载的资源中直接取出来执行,不用再从服务器请求,节省了很多时间。

> 在测试时发现只有**js**资源设置**prefetch**模式才能触发资源预加载,**preload**模式触发不了,**css**和图片等资源不管设置**prefetch**还是**preload**都不能触发,不知道是哪里没配置好。

### 14.10 打包时生成gzip文件

前端代码在浏览器运行,需要从服务器把**html**,**css**,**js**资源下载执行,下载的资源体积越小,页面加载速度就会越快。一般会采用**gzip**压缩,现在大部分浏览器和服务器都支持**gzip**,可以有效减少静态资源文件大小,压缩率在 **70%** 左右。

**nginx**可以配置**gzip: on**来开启压缩,但是只在**nginx**层面开启,会在每次请求资源时都对资源进行压缩,压缩文件会需要时间和占用服务器**cpu**资源，更好的方式是前端在打包的时候直接生成**gzip**资源,服务器接收到请求,可以直接把对应压缩好的**gzip**文件返回给浏览器,节省时间和**cpu**。

**webpack**可以借助[compression-webpack-plugin](https://www.npmjs.com/package/compression-webpack-plugin) 插件在打包时生成 **gzip** 文章,安装依赖

```sh
npm i compression-webpack-plugin -D
```

添加配置,修改**webpack.prod.js**

```js
const CompressionPlugin  = require('compression-webpack-plugin')
module.exports = {
  // ...
  plugins: [
     // ...
     new CompressionPlugin({
      test: /.(js|css)$/, // 只生成css,js压缩文件
      filename: '[path][base].gz', // 文件命名
      algorithm: 'gzip', // 压缩格式,默认是gzip
      test: /.(js|css)$/, // 只生成css,js压缩文件
      threshold: 10240, // 只有大小大于该值的资源会被处理。默认值是 10k
      minRatio: 0.8 // 压缩率,默认值是 0.8
    })
  ]
}
```

配置完成后再打包,可以看到打包后js的目录下多了一个 **.gz** 结尾的文件

因为只有**verdors.js**的大小超过了**10k**, 所以只有它生成了**gzip**压缩文件,借助**serve -s dist**启动**dist**,查看**verdors.js**加载情况

我的**verdors.js**的原始大小是**182kb**, 使用**gzip**压缩后的文件只剩下了**60.4kb**,减少了**70%** 的大小,可以极大提升页面加载速度。


## 十五. 总结

到目前为止已经使用**webpack5**把**react18+ts**的开发环境配置完成，并且配置比较完善的保留组件状态的**热更新**，以及常见的**优化构建速度**和**构建结果**的配置。还有细节需要优化，比如把容易改变的配置单独写个**config.js**来配置，输出文件路径封装。这篇文章只是配置，如果想学好**webpack**，还需要学习**webpack**的构建原理以及**loader**和**plugin**的实现机制。

本文是总结自己使用**webpack5**搭建**react+ts**构建环境中使用到的配置, 肯定也很多没有做好的地方，后续有好的使用技巧和配置也会继续更新记录。

附上上面安装依赖的版本

```json
 "devDependencies": {
    "@babel/core": "^7.26.0",
    "@babel/preset-env": "^7.26.0",
    "@babel/preset-react": "^7.26.3",
    "@babel/preset-typescript": "^7.26.0",
    "@commitlint/cli": "^19.6.0",
    "@commitlint/config-conventional": "^19.6.0",
    "@eslint/js": "^9.16.0",
    "@pmmmwh/react-refresh-webpack-plugin": "^0.5.15",
    "@types/react": "^18.3.12",
    "@types/react-dom": "^18.3.1",
    "autoprefixer": "^10.4.20",
    "babel-loader": "^9.2.1",
    "compression-webpack-plugin": "^11.1.0",
    "copy-webpack-plugin": "^12.0.2",
    "core-js": "^3.39.0",
    "cross-env": "^7.0.3",
    "css-loader": "^7.1.2",
    "css-minimizer-webpack-plugin": "^7.0.0",
    "eslint": "^9.16.0",
    "eslint-plugin-react": "^7.37.2",
    "glob": "^11.0.0",
    "globals": "^15.13.0",
    "html-webpack-plugin": "^5.6.3",
    "husky": "^8.0.1",
    "lint-staged": "^15.2.10",
    "mini-css-extract-plugin": "^2.9.2",
    "postcss-loader": "^8.1.1",
    "prettier": "^3.4.2",
    "purgecss-webpack-plugin": "^4.1.3",
    "react-refresh": "^0.16.0",
    "sass": "^1.82.0",
    "sass-loader": "^16.0.4",
    "speed-measure-webpack-plugin": "^1.5.0",
    "style-loader": "^4.0.0",
    "terser-webpack-plugin": "^5.3.10",
    "thread-loader": "^4.0.4",
    "typescript": "^5.7.2",
    "typescript-eslint": "^8.17.0",
    "webpack": "^5.97.1",
    "webpack-bundle-analyzer": "^4.10.2",
    "webpack-cli": "^5.1.4",
    "webpack-dev-server": "^5.1.0",
    "webpack-merge": "^6.0.1"
  }
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
```

