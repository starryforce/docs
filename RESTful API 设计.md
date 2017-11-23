# RESTful架构背景

> URI应该只代表"资源"的位置。它的具体表现形式,
> 应该在HTTP请求的头信息中用Accept和Content-Type字段指定，

> 综合上面的解释，我们总结一下什么是RESTful架构：
> 1. 每一个URI代表一种资源；
> 2. 客户端和服务器之间，传递这种资源的某种表现层；
> 3. 客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

# 协议
API与用户的通信协议，总是使用HTTPs协议。用于增强网站安全性
加密的连接简化了用户认证的工作 - 你可以使用简单的access token，而不需要对每个API请求进行签名
NOTICE：不要把以非SSL的形式访问API的URL的请求跳转到它们的SSL版本上。直接抛出一个严重错误！

# 域名
尽量将API部署在专用域名之下。

    https://api.example.com

# 版本号
将API的版本号放入URL。或者将版本号放在HTTP头信息中（不直观）。

    https://api.example.com/v1/

# URI
URI 表示资源，资源是一类或者一个实体，应该始终为名词。
在服务器端资源对应着数据库的表,而数据库的表是实体的集合。
因此API中的实体集合用复数形式表示。
URL是大小写敏感的。所以为了避免歧义，尽量使用小写字母。
/在url中表达层级，用于按实体关联关系进行对象导航，一般根据id导航。

举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。

    https://api.example.com/v1/zoos
    https://api.example.com/v1/animals
    https://api.example.com/v1/employees

## URI失效

随着系统发展，总有一些API失效或者迁移，对失效的API，返回404 not found 或 410 gone；对迁移的API，返回 301 重定向。

# HTTP动词
常用的HTTP动词有下面五个（括号里是对应的SQL命令）。
>   * GET（SELECT）：从服务器获取资源（一项或多项）。
>   * POST（CREATE）：在服务器新建一个资源。
>   * PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
>   * PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
>   * DELETE（DELETE）：从服务器删除资源。

还有两个不常用的HTTP动词。
>   * HEAD：获取资源的元数据。
>   * OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

下面是一些例子。
>   * `GET /zoos`：列出所有动物园
>   * `POST /zoos`：新建一个动物园 （POST一般向“资源集合”型uri发起）
>   * `GET /zoos/ID`：获取某个指定动物园的信息
>   * `PUT /zoos/ID`：更新某个指定动物园的信息（提供该动物园的全部信息）（PUT/PATCH一般向“单个资源”型uri发起）
>   * `PATCH /zoos/ID`：更新某个指定动物园的信息（提供该动物园的部分信息）
>   * `DELETE /zoos/ID`：删除某个动物园
>   * `GET /zoos/ID/animals`：列出某个指定动物园的所有动物
>   * `DELETE /zoos/ID/animals/ID`：删除某个指定动物园的指定动物

## 安全性和幂等性

安全性：不会改变资源状态，可以理解为只读的；
幂等性：执行1次和执行N次，对资源状态改变的效果是等价的。

|        | 安全性 | 幂等性 |
| ------ | ----- | ----- |
| GET    |   √   |   √   |
| POST   |   ×   |   ×   |
| PUT    |   ×   |   √   |
| DELETE |   ×   |   √   |

安全性和幂等性均不保证反复请求能拿到相同的response。
以 DELETE 为例，第一次DELETE返回200表示删除成功，第二次返回404提示资源不存在，这是允许的。

## Bookmarker

经常使用的、复杂的查询标签化，降低维护成本。

如：

    GET /trades?status=closed&sort=created,desc
快捷方式：

    GET /trades#recently-closed

## Format

只用以下常见的3种body format：

1. Content-Type: application/json

        POST /v1/animal HTTP/1.1
        Host: api.example.org
        Accept: application/json
        Content-Type: application/json
        Content-Length: 24

        {
          "name": "Gir",
          "animalType": "12"
        }

2. Content-Type: application/x-www-form-urlencoded (浏览器POST表单用的格式)

        POST /login HTTP/1.1
        Host: example.com
        Content-Length: 31
        Accept: text/html
        Content-Type: application/x-www-form-urlencoded

        username=root&password=Zion0101
3. Content-Type: multipart/form-data; boundary=—-RANDOM_jDMUxq4Ot5 (表单有文件上传时的格式)

## Content Negotiation

资源可以有多种表示方式，如json、xml、pdf、excel等等，客户端可以指定自己期望的格式，通常有两种方式：

1. http header Accept：

        Accept:application/xml;q=0.6,application/atom+xml;q=1.0
    q为各项格式的偏好程度

2. url中加文件后缀：/zoo/1.json

# 过滤信息（Filtering）
如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。
下面是一些常见的参数。
>   * `?limit=10`：指定返回记录的数量
>   * `?offset=10`：指定返回记录的开始位置。
>   * `?page=2&per_page=100`：指定第几页，以及每页的记录数。
>   * `?sortby=name&order=asc`：指定返回结果按照哪个属性排序，以及排序顺序。
>   * `?animal_type_id=1`：指定筛选条件

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的。

# 不符合 CRUD 的情况/服务型资源
在实际资源操作中，总会有一些不符合 CRUD（Create-Read-Update-Delete） 的情况，

如果某些动作是HTTP动词表示不了的，应该把动作做成一种资源。比如网上汇款，从账户1向账户2汇款500元，错误的URI是：

    POST /accounts/1/transfer/500/to/2

正确的写法是把动词transfer改成名词transaction，资源不能是动词，但是可以是一种服务：

    POST /transaction HTTP/1.1
    Host: 127.0.0.1
    from=1&to=2&amount=500.00

# 数据格式
* 对于响应返回的格式，JSON 因为它的可读性、紧凑性以及多种语言支持等优点，
* **最好采用 JSON 作为返回内容的格式。**
* 如果用户需要其他格式，比如 xml，应该在请求头部 Accept 中指定。
* 对于不支持的格式，服务端需要返回正确的 status code，并给出详细的说明。
* 返回数据不作包装,response 的 body 直接就是数据
* json格式的约定：
    1. 时间用长整形(毫秒数)，客户端自己按需解析（moment.js）
    2. 不传null字段

# 操作返回结果
针对不同操作，服务器向用户返回的结果应该符合以下规范。
>   * `GET /collection`：返回资源对象的列表（数组）
>   * `GET /collection/resource`：返回单个资源对象
>   * `POST /collection`：返回新生成的资源对象
>   * `PUT /collection/resource`：返回完整的资源对象
>   * `PATCH /collection/resource`：返回完整的资源对象
>   * `DELETE /collection/resource`：返回一个空文档

# 状态码（Status Codes）
HTTP 应答中，需要带一个很重要的字段：`status code`。它说明了请求的大致情况，是否正常完成、需要进一步处理、出现了什么错误，对于客户端非常重要。状态码都是三位的整数，大概分成了几个区间：

>   * 2XX：请求正常处理并返回
>   * 3XX：重定向，请求的资源位置发生变化
>   * 4XX：客户端发送的请求有错误
>   * 5XX：服务器端错误

服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）。
>   * 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
>   * 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
>   * 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
>   * 204 NO CONTENT - [DELETE]：用户删除数据成功。
>   * 301 Moved Permanently	请求的资源已经永久性地移动到另外一个地方，后续所有的请求都应该直接访问新地址。服务端会把新地址写在 Location 头部字段，方便客户端使用。允许客户端把 POST 请求修改为 GET。
>   * 304 Not Modified	请求的资源和之前的版本一样，没有发生改变。用来缓存资源，和条件性请求（conditional request）一起出现
>   * 307 Temporary Redirect	目标资源暂时性地移动到新的地址，客户端需要去新地址进行操作，但是不能修改请求的方法。
>   * 308 Permanent Redirect	和 301 类似，除了客户端不能修改原请求的方法
>   * 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
>   * 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
>   * 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
>   * 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
>   * 405 Method Not Allowed	服务端接收到了请求，而且要访问的资源也存在，但是不支持对应的方法。服务端必须返回 Allow 头部，告诉客户端哪些方法是允许的
>   * 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
>   * 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
>   * 415 Unsupported Media Type	服务端不支持客户端请求的资源格式，一般是因为客户端在 Content-Type 或者 Content-Encoding 中申明了希望的返回格式，但是服务端没有实现。比如，客户端希望收到 xml返回，但是服务端只支持 Json
>   * 429 Too Many Requests	客户端在规定的时间里发送了太多请求，在进行限流的时候会用到
>   * 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
>   * 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
>   * 503 Service Unavailable	服务器因为负载过高或者维护，暂时无法提供服务。服务器端应该返回 Retry-After 头部，告诉客户端过一段时间再来重试


状态码的完全列表参见[详细说明](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。

# 错误处理（Error handling）
如果状态码是4xx，就应该向用户返回出错信息。

比如客户端发送的请求有错误，一般会返回 4XX Bad Request 结果。这个结果很模糊，给出错误 message 的话，能更好地让客户端知道具体哪里有问题，进行快速修改。基本的思路就是尽可能提供更准确的错误信息：比如数据不是正确的 json，缺少必要的字段，字段的值不符合规定…… 而不是直接说“请求错误”之类的信息。

Java 服务器端一般用异常表示 RESTful API 的错误。API 可能抛出两类异常：业务异常和非业务异常。业务异常由自己的业务代码抛出，表示一个用例的前置条件不满足、业务规则冲突等，比如参数校验不通过、权限校验失败。非业务类异常表示不在预期内的问题，通常由类库、框架抛出，或由于自己的代码逻辑错误导致，比如数据库连接失败、空指针异常、除0错误等等。

业务类异常必须提供2种信息：

1. 如果抛出该类异常，HTTP 响应状态码应该设成什么；
>有的Web应用对正确的REST响应使用200，对错误的REST响应使用400，
>
>这样，客户端即是静态语言，也可以根据HTTP响应码判断是否出错，出错时直接把结果反序列化为APIError对象。

2. 异常的文本描述；
>在Controller层使用统一的异常拦截器：
>
>   1. 设置 HTTP 响应状态码：对业务类异常，用它指定的 HTTP code；对非业务类异常，统一500；
>   2. Response Body 的错误码：异常类名，建议使用字符串作为错误码（日志/问题追查）
>   3. Response Body 的错误描述：对业务类异常，用它指定的错误文本；对非业务类异常，线上可以统一文案如“服务器端错误，请稍后再试”，开发或测试环境中用异常的 stacktrace，服务器端提供该行为的开关。（展示给用户）
>
>           {
>               "code": "错误代码",
>               "message": "错误描述信息"
>           }
>
>其中，错误代码命名规范为大类:子类，例如，口令不匹配的登录错误代码为`auth:bad_password`，用户名不存在的登录错误代码为`auth:user_not_found`。这样，客户端既可以简单匹配某个类别的错误，也可以精确匹配某个特定的错误。

# 异步任务

对耗时的异步任务，服务器端接受客户端传递的参数后，应返回创建成功的任务资源，其中包含了任务的执行状态。客户端可以轮询该任务获得最新的执行进度。

    提交任务请求：
    POST /batch-publish-msg
    [{"from":0,"to":1,"text":"abc"},{},{}...]

    返回：
    {"taskId":3,"createBy":"Anonymous","status":"running"}

    GET /task/3
    {"taskId":3,"createBy":"Anonymous","status":"success"}

如果任务的执行状态包括较多信息，可以把“执行状态”抽象成组合资源，客户端查询该状态资源了解任务的执行情况。

    提交任务请求：
    POST /batch-publish-msg
    [{"from":0,"to":1,"text":"abc"},{},{}...]

    返回：
    {"taskId":3,"createBy":"Anonymous"}

    GET /task/3/status
    {"progress":"50%","total":18,"success":8,"fail":1}


# 身份认证
使用[OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)框架。
一般来说，让任何人随意访问公开的 API 是不好的做法。验证和授权是两件事情：

>   * 验证（Authentication）是为了确定用户是其申明的身份，比如提供账户的密码。不然的话，任何人伪造成其他身份（比如其他用户或者管理员）是非常危险的
>   * 授权（Authorization）是为了保证用户有对请求资源特定操作的权限。比如用户的私人信息只能自己能访问，其他人无法看到；有些特殊的操作只能管理员可以操作，其他用户有只读的权限等等
如果没有通过验证（提供的用户名和密码不匹配，token 不正确等），需要返回 401 Unauthorized状态码，并在 body 中说明具体的错误信息；而没有被授权访问的资源操作，需要返回 403 Forbidden 状态码，还有详细的错误信息。

NOTE：Github API 对某些用户未被授权访问的资源操作返回 404 Not Found，目的是为了防止私有资源的泄露（比如黑客可以自动化试探用户的私有资源，返回 403 的话，就等于告诉黑客用户有这些私有的资源）。

# 限流 rate limit
如果对访问的次数不加控制，很可能会造成 API 被滥用，甚至被 DDos 攻击。根据使用者不同的身份对其进行限流，可以防止这些情况，减少服务器的压力。

对用户的请求限流之后，要有方法告诉用户它的请求使用情况，Github API 使用的三个相关的头部：

>   * X-RateLimit-Limit: 用户每个小时允许发送请求的最大值
>   * X-RateLimit-Remaining：当前时间窗口剩下的可用请求数目
>   * X-RateLimit-Rest: 时间窗口重置的时候，到这个时间点可用的请求数量就会变成 X-RateLimit-Limit 的值

如果允许没有登录的用户使用 API（可以让用户试用），可以把 X-RateLimit-Limit 的值设置得很小，比如 Github 使用的 60。没有登录的用户是按照请求的 IP 来确定的，而登录的用户按照认证后的信息来确定身份。

对于超过流量的请求，可以返回 429 Too many requests 状态码，并附带错误信息。而 Github API 返回的是 403 Forbidden，虽然没有 429 更准确，也是可以理解的。

# Hypermedia API
RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，这种设计也被称为 HATEOAS。使得用户不查文档，也知道下一步应该做什么。
比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档。

    {"link": {
      "rel":   "collection https://www.example.com/zoos",
      "href":  "https://api.example.com/zoos",
      "title": "List of zoos",
      "type":  "application/vnd.yourformat+json"
    }}
上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API了。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。
Hypermedia API的设计被称为HATEOAS。Github的API就是这种设计，访问api.github.com会得到一个所有可用API的网址列表。

    {
      "current_user_url": "https://api.github.com/user",
      "authorizations_url": "https://api.github.com/authorizations",
      // ...
    }
从上面可以看到，如果想获取当前用户的信息，应该去访问api.github.com/user，然后就得到了下面结果。

    {
      "message": "Requires authentication",
      "documentation_url": "https://developer.github.com/v3"
    }
上面代码表示，服务器给出了提示信息，以及文档的网址。

# 编写优秀的文档

API 最终是给人使用的，不管是公司内部，还是公开的 API 都是一样。即使我们遵循了上面提到的所有规范，设计的 API 非常优雅，用户还是不知道怎么使用我们的 API。最后一步，但非常重要的一步是：为你的 API 编写优秀的文档。

对每个请求以及返回的参数给出说明，最好给出一个详细而完整地示例，提醒用户需要注意的地方……反正目标就是用户可以根据你的文档就能直接使用 API，而不是要发邮件给你，或者跑到你的座位上问你一堆问题。
