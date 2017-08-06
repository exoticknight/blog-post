# Mocha + Chai + istanbul in ES2015

使用 JavaScript 写代码的时候，无论是使用 TDD 方法还是为了保证代码的质量和可维护性，都应该考虑加上单元测试。在博文 [mocha + chai + Travis CI + Codecov 使用流程](https://blog.e10t.net/mocha-chai-travis-ci-codecov-workflow/) 中有简单地介绍了如何使用流行的 JavsScript 库来对代码进行自动测试，检查代码覆盖率。

在那篇文章中，使用的是 Mocha／Chai／istanbul 和在线的 Codecov，以及和 Github 关系密切的 Travis CI，而且测试的 JavaScript 代码是 es5。现在 es2015 已经标准化了，那么教程也需要更新一下了。另外如果项目是私有项目，那么还是使用完备的离线测试环境比较好。接下来就是一个快速可行的教程。

## 安装和配置

测试框架无需变更，还是 Mocha + Chai 的组合，但是 istanbul 需要稍微变动一下。

> 如果你不需要 istanbul 做覆盖率测试，那么需要使用 `npm install --save-dev babel-register` 和 `mocha --require babel-register` 使 mocha 能识别 es2015 的代码

使用新套件，直接安装 `npm i -S mocha chai cross-env nyc babel-plugin-istanbul babel-register babel-preset-env`。

mocha 和 chai 不用解释了，`nyc` 可以理解是 istanbul 的命令行工具；`babel-plugin-istanbul` 是在 babel 中插入 istanbul，`babel-register` 是 istanbul 使用的 babel 接口，这样两个库就打通了；最后 `babel-preset-env` 是 babel 的运行配置。

先来配置 `babel-plugin-istanbul`，新建 `.babelrc`：

```json
{
  "presets": [
    "env"
  ],
  "env": {
    "test": {
      "plugins": [ "istanbul" ]
    }
  }
}
```

`presets` 配置告诉 babel 使用 `babel-preset-env`。当然也可以用 'es2015' + 'stage-0' 的组合，具体可以自行斟酌。

`env.test.plugins` 告诉 babel 在 `NODE_ENV=test` 的情况下使用插件 `babel-plugin-istanbul`。

接下来配置 `babel-register`，新建 `.nycrc`：

> `.nycrc` 是 nyc 的配置文件，和 `.babelrc` 类似，当然配置也是可以直接写进 `package.json` 的。

```json
{
  "require": [
    "babel-register"
  ],
  "reporter": [
    "lcov",
    "text-summary"
  ],
  "sourceMap": false,
  "instrument": false
}
```

以上的配置直接用了官方的配置。`require` 字段告诉 nyc 使用 `babel-register`，`reporter` 字段的 'lcov' 会让 nyc 生成 `lcov.info` 文件和对应的 HTML 报告，如果使用 `lcovonly` 则只生成 'lcov.info'。'text-summary' 则是会在控制台输出覆盖率等信息。

> 文件会默认生成在 `/coverage` 下，可以使用 `report-dir` 字段指定。

最后，在 `package.json` 中加入：

```json
  "scripts": {
    "test": "cross-env NODE_ENV=test nyc mocha test/**/*.spec.js"
  },
```

## 测试编写事例

比如有 `index.js`：

```javascript
export class Test {
  constructor () {
    this.data = 'a'
  }
}
```

那么可以写 `test/index.spec.js`，可以直接上 ES2015 的语法：

```javascript
import { expect } from 'chai'
import { Test } from '../index'

describe('index test', function() {
  it('should be a string', function() {
    let test = new Test
    expect(test.data).to.be.a('string')
  })
})
```

使用 `npm test` 运行测试，得到：

```text
> cross-env NODE_ENV=test nyc mocha test/**/*.spec.js



  index test
    ✓ should be a string


  1 passing (8ms)


=============================== Coverage summary ===============================
Statements   : 100% ( 1/1 )
Branches     : 100% ( 0/0 )
Functions    : 100% ( 1/1 )
Lines        : 100% ( 1/1 )
================================================================================
```

## Bonus

在测试中经常需要测试 promise 等异步操作，虽然 Mocha 库是可以使用回调来完成测试的，但是我们当然要用 async／await 啦。

比如，需要测试 `index.js` 中的 `requestAsync`：

```javascript
export class Test {
  constructor () {
    this.data = 'a'
  }

  requestAsync () {
    return new Promise((res, rej) => {
      res(this.data)
    })
  }
}
```

那么需要先 `npm i -S babel-polyfill babel-plugin-transform-async-to-generator`。

然后配置 `.nycrc`，加上 'babel-polyfill' 支持 generator 运行时：

```json
{
  "require": [
    "babel-polyfill",
    "babel-register"
  ],
  ...
}
```

配置 `.babelrc`，加上 'transform-async-to-generator'，将 async 模式转换为 generator 模式。

```json
{
  ...
  "env": {
    "test": {
      "plugins": [ "istanbul", "transform-async-to-generator" ]
    }
  }
}
```

接着这样测试异步：

```javascript
import { expect } from 'chai'
import { Test } from '../index'

describe('test#requestAsync', function() {
  it('should get a string', async function() {
    const test = new Test
    const ret = await test.requestAsync()
    expect(ret).to.be.a('string')
  })
})
```

在测试项 `it` 的第二个函数前加 'async' 标志异步，然后在返回 promise 的调用前加上 'await'，OK。

```text
> cross-env NODE_ENV=test nyc mocha test/**/*.spec.js



  test#requestAsync
    ✓ should get a string


  1 passing (14ms)


=============================== Coverage summary ===============================
Statements   : 100% ( 3/3 )
Branches     : 100% ( 0/0 )
Functions    : 100% ( 3/3 )
Lines        : 100% ( 3/3 )
================================================================================
```