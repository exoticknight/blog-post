配合 angular 和 angular-material 来开发基于 electron 的应用 · 2--node 库的使用和抓取代码的编写
=====================================

![thumbnail](https://i.imgur.com/kvb5CTf.png)

##使用 nodejs 的库

python 脚本的详细编写，请看之前的博文：[radioit计划——动画广播辅助脚本radioitScript][1]。

需要用 node 实现脚本中的某些逻辑是获取和提取广播的信息，整合成 JSON 格式的数据。

而用一些库就能轻松做到。

[1]: http://blog.e10t.net/radioit-plan-animate-radio-script-radioitscript/

###superagent

[superagent][superagent] 是一个极其简单的 AJAX 库。

使用方法简单得令人发指。

```javascript
var request = require( 'superagent' );

request
    .get( 'http://xxx.com' )
    .end( function ( err, res ) {
    // Do something
});
```

还用介绍吗？不用了。

[superagent]: https://github.com/visionmedia/superagent

###bluebird

[bluebird][bluebird] 是一个 Promise 库。

凡是类似 IO 的操作，必定需要异步。经典的解决方法是回调，然而是时候用 Promise 了！

bluebird 声称拥有无与伦比的速度。其实更实用的功能是它支持能够将一些本身是不支持 Promise 的库转化为支持 Promise 的库。

然而，要配合之前的 superagent，则需要另外一个库 [superagent-bluebird-promise][superagent-bluebird-promise]。superagent 本身不支持 Promise，从上面的代码来看就是使用回调的方法，这个库就是将 superagent 和 bluebird 融合在一起的“融合卡”。

使用的时候只需要：

```javascript
var Promise = require( 'bluebird' );
var request = require( 'superagent-bluebird-promise' );

request
    .get( 'http://xxxx.com' )
    .then( function ( res ) {
        // do something when resolved
    }, function ( err ) {
        // do something when rejected
    });
```

立刻就可以使用上 `then` 了，方便吧。

[bluebird]: https://github.com/petkaantonov/bluebird/
[superagent-bluebird-promise]: https://github.com/KyleAMathews/superagent-bluebird-promise

###cherrio

[cheerio][cheerio] 是一个语法类似 jQuery，为服务端提供 jQuery 核心功能的库。这里用到的是它的 CSS 选择器功能。

代码同样很简单，使用过 jQuery 的人会倍感亲切。

```javascript
var cheerio = require( 'cheerio' ),
    $ = cheerio.load( '<h2 class="title">Hello world</h2>' );

$( '.title' ).text(); // Hello world
```

使用 cheerio 有比较推荐的做法就是添加上 `decodeEntities` 和 `lowerCaseAttributeNames` 这个两个 options 配置，能避免各种 HTML 文本的奇怪问题。

```javascript
$ = cheerio.load( HTMLtext, {
                    'decodeEntities': true,
                    'lowerCaseAttributeNames': true
                });
```

[cheerio]: https://github.com/cheeriojs/cheerio

综上，四个库的混合使用例子如下：

```javascript
var Promise = require( 'bluebird' );
var request = require( 'superagent-bluebird-promise' );
var cheerio = require( 'cheerio' );

request
    .get( 'http://xxxx.com' )
    .then( function ( res ) {
        var $, text;
        $ = cheerio.load( res.text, {
                    'decodeEntities': true,
                    'lowerCaseAttributeNames': true
                });
        text = $( 'p' ).text();
    }, function ( err ) {
        console.log( err );
    });
```

##编写逻辑

> npm 安装库的过程略。

因为是信息整合，那么必定需要有一个统一的数据格式。于是先来确定数据格式。

广播站中所有广播的信息整合数据格式。

```javascript
// data will be formated as a json object in following structure:
// {
//     'name': 'String, name of the channel',
//     'url': 'String, url of the channel',
//     'timestamp': 'Number, timestamp of this data',
//     'bangumi': {
//         'mon': [
//             {
//                 'id': 'String, id of the bangumi',
//                 'homepage': 'URL, homepage of the bangumi',
//                 'name': 'String, name of the bangumi',
//                 'image': 'String, image url  of the bangumi, optional',
//                 'status': 'String, new / normal',
//                 ...
//             },
//             {...}
//         ],
//         'tue': [{...},{...}],
//         'wed': [{...},{...}],
//         'thu': [{...},{...}],
//         'fri': [{...},{...}],
//         'sat': [{...},{...}],
//         'sun': [{...},{...}],
//         'irr': [{...},{...}],
//     }
// }
```

单个广播的信息整合数据格式。

```javascript
// data will be formated as a json object in following structure:
// {http://hibiki-radio.jp
//     'timestamp': 'Number',
//     'name': 'String, name of the bangumi',
//     'homepage': 'URL, homepage of the bangumi',
//     'description': 'String, description of the bangumi',
//     'title': 'String, title of the newest episode',
//     'comment': 'String, comment of the newest episode',
//     'schedule': 'String, schedule of the bangumi or the update date of the newest pisode',
//     'personality': 'String, personality of the bangumi',
//     'guest': 'String, guest of the newest episode',
//     'images': 'String Array, array of images' url',
//     'audio': 'String, url of audio'
// }
```

有了输出的数据格式，抓取信息的时候就能有的放失。

以[響 - HiBiKi Radio Station -][響 - HiBiKi Radio Station -]为例。因为在之前编写脚本的时候已经得到了页面上信息的位置，所以可以直接应用在代码中。

```javascript
// 一些固定的信息和变量
var NAME = '響 - HiBiKi Radio Station -';
var HOST = 'http://hibiki-radio.jp';

var URLs = {
    'catalogue': 'http://hibiki-radio.jp/program',
    'bangumi': 'http://hibiki-radio.jp/description/'
}
```

以下开始获取广播站的广播。

```javascript
// 获取信息的对象
var hibiki = {
    catalogueName: NAME,
    host: HOST,

    // 异步取得所有广播的基本信息，返回一个 promise 对象
    getCatalogueAsync: function () {
        return request
            .get( URLs.catalogue )
            .then( function ( res ) {
                var $,
                    days,
                    bangumi,
                    data;

                days = 'mon tue wed thu fri sat sun irr'.split( ' ' );

                $ = cheerio.load( res.text, {
                    'decodeEntities': true,
                    'lowerCaseAttributeNames': true
                });

                // Extract html and structure data
                // 准备数据结构
                data = {};
                data.bangumi = {};
                days.forEach( function ( el ) {
                    data.bangumi[el] = [];
                });


                // Structure daily bangumis
                // 一个 .hbkProgramTable 包含一天的广播
                $( '.hbkProgramTable' ).each( function ( i, el ) {
                    var _;

                    // 一个 .hbkProgramTitleNew 或 .hbkProgramTitle 为一个广播
                    data.bangumi[days[i]] = $( this ).find( '.hbkProgramTitleNew, .hbkProgramTitle' ).map( function ( _, el ) {
                        _ = $( this );

                        // 一个广播的基本信息
                        return {
                            'id': _.parent().attr( 'href' ).slice( 35 ),
                            'homepage': _.parent().attr( 'href' ),
                            'name': _.text(),
                            'image': _.prev().children().eq( 0 ).attr( 'src' ),
                            'status': _.attr( 'class' ) === 'hbkProgramTitleNew' ? 'new' : 'normal'
                        };
                    }).get();

                    _ = null;
                });

                // add extra data
                data.name = NAME;
                data.url = HOST;
                data.timestamp = Date.now();

                return data;

            }, function ( err ) {
                console.log( 'hibiki:get catalogue error: ' + err );
                throw new Error( err );
            });
    }
}
```

以下开始获取某个广播的详细信息，函数定义在上面的对象中。

```javascript
    getBangumiAsync: function ( id ) {
        return request
            .get( url.resolve( URLs.bangumi, id ) )
            .then( function ( res ) {
                var $,
                    data;

                $ = cheerio.load( res.text, {
                    'decodeEntities': true,
                    'lowerCaseAttributeNames': true
                });

                // Extract html and structure data
                // 某个广播详细页的信息提取，信息的位置在 python 脚本中已经确定好了
                data = {
                    'timestamp': Date.now(),
                    'name': $( 'title' ).text().slice( 27, -5 ),
                    'homepage': url.resolve( URLs.bangumi, id ),
                    'description': $( 'table.hbkTextTable td:nth-of-type(1) div:nth-of-type(1)' ).eq( 0 ).text().trim(),
                    'title': $( 'table.hbkTextTable td:nth-of-type(1) div:nth-of-type(3) table:nth-of-type(1) div' ).eq( 0 ).text().trim(),
                    'comment': $( 'table.hbkTextTable td:nth-of-type(1) div:nth-of-type(3) table:nth-of-type(2) td' ).eq( 0 ).text().trim(),
                    'schedule': (function () {
                        var _, text;
                        _ = $( 'table.hbkTextTable > tr > td:nth-of-type(2) > div' );
                        if ( !( text = _.eq( -5 ).text().trim() ) ) {
                            text = _.eq( -3 ).text();
                        }
                        return text;
                    })(),
                    'update': $( '.hbkDescriptonContents' ).eq( -1 ).prev().prev().find( 'span' ).eq( 0 ).text(),
                    'personality': $( 'table.hbkTextTable td:nth-of-type(1) > table table td:nth-of-type(2n) a' ).map( function () {return $( this ).text();} ).get().join( ' ' ),
                    'guest': '',
                    'images': $( 'table.hbkTextTable td:nth-of-type(1) div:nth-of-type(3) table:nth-of-type(2) td img' ).map( function () {return $( this ).attr( 'src' );}).get(),
                    'audio': $( 'div.hbkDescriptonContents embed' ).eq( -1 ).attr( 'src' )
                };

                return data;

            }, function ( err ) {
                console.log( 'hibiki:get bangumi error: ' + err );
                throw new Error( err );
            });
    },
```

代码看似很多，其实就是多了信息提取的部分，其他代码完全就是上一节中四个库的混合使用。

要注意的有一点，就是 promise 链中的 `then( fulfilledHandler, rejectedHandler )`。其中 `fulfilledHandler` 在最后需要使用 `return data;` 将数据传出去，而 `rejectedHandler` 也需要使用 `throw new Error( err );` 重新抛出错误，不然 promise 链中下一个函数将不会得到处理好的数据或者异常（因为已经处理掉了）。

最后别忘了将对象导出。

```javascript
module.exports = hibiki;
```

同理，另外两个广播站的代码基本都一样，不同的只是信息提取的部分。

[響 - HiBiKi Radio Station -]: http://hibiki-radio.jp

###整合

对于取数据的调用者而言，是无需理会数据从哪来的，只需要知道使用什么 API 就够了。

再者，既然有“整合”之名，就要行“整合”之实。因此要将这三个或者日后出现的更多个广播站提取代码整合起来，只提供一个调用入口。

新建目录 `provider`，将三个广播站的脚本都放进去。

再新建一个 `provider.js` 文件，写入以下代码。

```javascript
var catalogue = {
    'hibiki': require( './provider/hibiki.js' ),
    'onsen': require( './provider/onsen.js' ),
    'animate': require( './provider/animate.js' )
};

var provider = {
    /**
     * Get the list of catalogue
     * @return {Array} list of catalogue
     */
    getCatalogueList: function () {
        var arr = [],
            item;

        for ( item in catalogue ) {
            arr.push({
                id: item,
                name: catalogue[item].catalogueName,
                host: catalogue[item].host
            });
        }

        return arr;
    },
    getCatalogueAsync: function ( id ) {
        var c;

        if ( !( c = catalogue[id] ) ) {
            return;
        }

        return c.getCatalogueAsync();
    },
    getBangumiAsync: function ( catalogueID, bangumiID ) {
        var c;

        if ( !( c = catalogue[catalogueID] ) ) {
            return;
        }

        return c.getBangumiAsync( bangumiID );
    }
};

module.exports = provider;
```

整体思路是提供一个可调用的列表，然后根据参数调用相应脚本的功能，就是一个 `dispatcher` 的功能。

如此，就实现了应用的一大部分主要功能了。