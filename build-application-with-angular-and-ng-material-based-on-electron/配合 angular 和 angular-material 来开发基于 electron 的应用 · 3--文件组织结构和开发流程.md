配合 angular 和 angular-material 来开发基于 electron 的应用 · 3--文件组织结构和开发流程
========================================

##文件组织结构

良好的文件组织结构不仅能帮助我们更快地定位文件，更能配合开发工具形成流畅的开发流程，从而提高编程效率。

以下的目录和文件都放在存放应用的根目录 `app` 下。

###package.json

Electron 应用的配置文件，经常做 node 开发的人应该很熟悉了。稍微说明一下一些字段：

`name`: 应用的名字，本项目就是 radioit 了

`description`: 应用的描述

`version`: 应用的版本号

`author`: 作者名字

`email`: 作者的邮箱

###main.js

Electron 应用的入口点，可以在 package.json 的 `main` 字段自定义

###node_modules/

node 库的目录，一般不用手动管理，而是使用 npm 来安装和卸载库。

###lib/

存放 node 模块的目录。

###src/

存放源代码的目录。

###src/css/

存放待编译的 css 代码，比如本项目用的 .styl 文件。

###src/modules/

存放浏览器端的 javascript 源代码。

因为使用 AngularJS，所以此目录的结构就照搬 AngularJS 项目的结构。

通常来说有两种：按 service / controller / directive 分目录存放，按功能模块存放。

本项目选择按功能模块存放。

###src/modules/entry.js

供 browserify 打包的入口点。最终浏览器端的 javascript 代码会打包成一个名为 `bundle.js` 的文件。

###static/css/

存放编译好的 CSS 文件。

###static/font/

存放字体文件。因为 Electron 可以访问本地文件，所以自定义字体也基本不需要考虑网络传输问题。

###static/image/

存放图片文件。

###static/js/

存放客户端的 javascript 库，比如 jQuery，underscore，AngularJS 等。

###static/js/bundle.js

browserify 编译 javascript 代码后输出的文件。

###static/view/

存放 HTML 模板文件或者包含 HTML 代码的文件。

##开发流程 -- node 相关

有关 node 的开发，跟普通的项目并没有什么两样，需要什么库就直接使用 npm 安装，然后再代码中使用 `require` 就可以了。

然而虽然 Electron 为 webkit 内核提供了 `io.js` 的运行环境，但是最好还是避免在客户端（浏览器）的 javascript 代码内混杂需要 node 依赖的代码。换句话说，最好将需要 node 依赖的 javascript 代码和平常在网页中使用的 javascript 代码分开。这样做的好处是不会搞混相关的 API 和设计模式，毕竟 node 大部分时候都是用在服务端上的。

本项目将 node 相关的代码放在 `lib/` 目录下，负责应用的业务逻辑，其既有可能被主进程所用，也有可能被渲染进程所用。node 相关的代码不需要编译合并。编写时在目录下新建 `xxx.js` 文件，写好需要 exports 的内容，在其他文件中则使用 `require( './xxx.js' )` 就可以了。

##开发流程 -- 界面相关

因为界面的渲染采用 webkit 引擎，所以 javascript 的编写和网页开发没有分别。

在 `package.json` 的 `scripts` 字段中增加一条命令：

```language-javascript
"build:js": "browserify src/modules/entry.js -o static/js/bundle.js"
```

然后在编写好 javascript 代码的时候，执行 `npm run build:js` 进行编译。

CSS 的编译则是加入如下：

```language-javascript
"build:css": "stylus -u nib src/css/app.styl -o static/css/app.css"
```

最后需要运行应用来测试，增加命令：

```language-javascript
"test": "electron main.js 2>&1 | silence-chromium",
"start": "npm run build:js && npm run build:css && electron main.js 2>&1 | silence-chromium"
```

需要测试的时候使用 `npm run test`，需要运行则使用 `npm run start` 进行重新编译和运行。