mocha + chai + Travis CI + Codecov 使用流程
====================================

###[mocha](https://mochajs.org/)
a feature-rich JavaScript test framework running on Node.js and the browser

###[chai](http://chaijs.com/)
a BDD / TDD assertion library for node and the browser that can be delightfully paired with any javascript testing framework

###[Travis CI](https://travis-ci.org/)
Free continuous integration platform for GitHub projects

###[Codecov](https://codecov.io/)
Continuous Code Coverage

##编写测试

简单来说，就是使用 mocha 作为测试框架，chai 作为断言库，将项目交给 Travis CI 做自动测试，交给 Codecov 做覆盖率测试。

以我自己的项目 [simpleTemplate.js](https://github.com/exoticknight/simpleTemplate.js) 为例。

先给项目装上 mocha 和 chai。

```bash
npm i mocha chai --save-dev
```

在 `package.json` 文件中添加测试脚本命令行。

```javascript
"scripts": {
    "test": "mocha"
}
```

项目根目录新建文件 `test.js`。

引入 chai 以及三个要测试的库。

```javascript
var expect = require( 'chai' ).expect;

var bare = require( './simpleTemplate.bare.js' ),
    normal = require( './simpleTemplate.normal.js' ),
    advanced = require( './simpleTemplate.advanced.js' );
```

定义必需的数据。

```javascript
var testData = {
    'text1': 'mocha tastes good, chai tastes good too.',
    'html': '<hr>',
    'list': [1, 2, 3],
    'objectList': [
        { 'name': 'a' },
        { 'name': 'b' },
        { 'name': 'c' }
    ],
    'obj': { 'name': 'obj' },
    'boolFalse': false,
    'boolTrue': true
}
```

mocha 用起来其实也不复杂，常用的就是使用 `describe` 定义一个项目，使用 `it` 来执行一项测试。

```javascript
describe( 'bare', function () {
    describe( 'string template', function () {
        it( 'should output correct string', function () {
            var template = bare( '<p>{=text1}</p>' );

            expect( template.fill( testData ).render() ).to.equal( '<p>mocha tastes good, chai tastes good too.</p>' );
        });
    });
});
```

这里就是定义了一个 `bare` 项，里面再定义一个 `string template` 项，然后在 `it` 的回调函数中写断言，第一个参数可以写上断言描述。如果断言失败，测试就会失败。

OK，执行 `npm test`，可以看到结果输出。

```markup
  bare
    string template
      √ should output correct string


  1 passing (9ms)
```

`expect(...).to.equal(...)` 就是用到了 chai 了。

剩下的测试编写就不再详述了，基本都一样。

##自动测试

去 Travis-CI 官网使用 github 帐号登录，开启对应项目的访问权限。

然后在 `package.json` 同目录（根目录）下，新建文件 `.travis.yml`，写入如下内容。

```yaml
language: node_js
node_js:
  - "0.12"
```

`git push` 一次，再访问 Travis-CI，会发现已经给你显示出测试结果了。

那么好了，测试通过，何不贴个奖章 show off 一下呢？

在项目旁边有一个黑色加绿色的按钮，点一下，弹框中选择 markdown 格式，将代码贴进 `readme`，再 `git push`，去 github 的项目页一看，是不是高大上起来了呢？

##代码覆盖率

代码覆盖率其实也没必要到 100%，只要不是太低的值就可以了。

去 Codecov 官网使用 github 帐号登录，开启对应项目的访问权限。

然后在 `package.json` 同目录（根目录）下的 `.travis.yml` 加入如下内容。

```yaml
env:
  global:
    - CODECOV_TOKEN: your-uuid
script:
  - istanbul cover node_modules/mocha/bin/_mocha
  - cat ./coverage/coverage.json | node_modules/codecov.io/bin/codecov.io.js
```

your-uuid 替换成开启项目时生成的 `Repository Upload Token`。

继续 `git push` 一次，再访问 Codecov 就可以看到项目的代码覆盖率了。

照样里添加上 badge，点击右边齿轮的 `badge`，就可以得到 markdown 代码了。

##打完收工

以后每次 `push`，都会自动运行测试和代码率覆盖统计，去查看一下就知道代码有没有错误或者改进了。

参考文章：

[Basic Front End Testing With Mocha & Chai](http://callmenick.com/post/basic-front-end-testing-with-mocha-chai)

[折腾 Coffee + mocha + Travis-CI 单元测试与覆盖率报告](https://cnodejs.org/topic/5443b8342be2db9d42e8f685)

[Example Node with Codecov](https://github.com/codecov/example-node)