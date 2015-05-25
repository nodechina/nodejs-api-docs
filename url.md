# URL

    稳定性: 3 - 稳定

这个模块包含分析和解析 URL 的工具。调用 `require('url')` 来访问模块。

解析 URL 对象有以下内容，依赖于他们是否在 URL 字符串里存在。任何不在 URL 字符串里的部分，都不会出现在解析对象里。例子如下：

`'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

* `href`: 准备解析的完整的 URL，包含协议和主机（小写）。

    例子： `'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

* `protocol`: 请求协议, 小写.

    例子： `'http:'`

* `slashes`: 协议要求的斜杠（冒号后）

    例子： true 或 false

* `host`: 完整的 URL 小写 主机部分，包含端口信息。

    例子： `'host.com:8080'`

* `auth`: url 中的验证信息。

    例子： `'user:pass'`

* `hostname`: 域名中的小写主机名

    例子： `'host.com'`

* `port`: 主机的端口号

    例子： `'8080'`

* `pathname`: URL 中的路径部分，在主机名后，查询字符前，包含第一个斜杠。

    例子： `'/p/a/t/h'`

* `search`: URL 中得查询字符串，包含开头的问号

    例子： `'?query=string'`

* `path`: `pathname` 和 `search` 连在一起

    例子： `'/p/a/t/h?query=string'`

* `query`: 查询字符串中得参数部分，或者使用 querystring.parse() 解析后返回的对象。

    例子： `'query=string'` or `{'query':'string'}`

* `hash`:  URL 的 “#” 后面部分（包括 # 符号）

    例子： `'#hash'`

URL 模块提供了以下方法：

<a name="url.parse"></a>
## url.parse(urlStr[, parseQueryString][, slashesDenoteHost])

输入 URL 字符串，返回一个对象。

第二个参数为 `true` 时，使用 `querystring` 来解析查询字符串。如果为 `true`，`query` 属性将会一直赋值为对象，并且  `search` 属性将会一直是字符串(可能为空)。默认为 `false`。

第三个参数为`true` ，把 `//foo/bar` 当做`{ host: 'foo', pathname: '/bar' }` ，而不是`{ pathname: '//foo/bar' }`。默认为 `false`。

## url.format(urlObj)

输入一个解析过的 URL 对象，返回格式化过的字符串。

格式化的工作流程：

* `href` 会被忽略
* `protocol` 无论是否有末尾的 : (冒号)，会同样的处理
  * `http`, `https`, `ftp`, `gopher`, `file` 协议会被添加后缀`://` 
  *  `mailto`, `xmpp`, `aim`, `sftp`, `foo`, 等协议添加后缀`:`
* `slashes` 如果协议需要 `://`，设置为 true
  * 仅需对之前列出的没有斜杠的协议，比如议 `mongodb://localhost:8000/`
* `auth` 如果出现将会使用.
* `hostname` 仅在缺少 `host` 时使用
* `port` 仅在缺少 `host` 时使用
* `host` 用来替换 `hostname` 和 `port`
* `pathname` 无论结尾是否有 / 将会同样处理
* `search` 将会替代 query属性
  * 无论前面是否有 / 将会同样处理
* `query` (对象; 参见 `querystring`) 如果没有 search,将会使用
* `hash` 无论前面是否有#，都会同样处理

## url.resolve(from, to)

给一个基础 URL， href URL，如同浏览器一样的解析它们可以带上锚点，例如：

    url.resolve('/one/two/three', 'four')         // '/one/two/four'
    url.resolve('http://example.com/', '/one')    // 'http://example.com/one'
    url.resolve('http://example.com/one', '/two') // 'http://example.com/two'
