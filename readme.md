```
PSR 7 HTTP 消息接口
此文档遵循 RFC 7230 、RFC 7231 、RFC 3986 三个规范为 HTTP 消息和 HTTP 消息中使用的 URI 定义了通用的接口

HTTP 消息是 Web 技术发展的基石。浏览器或 HTTP 客户端如 cURL 生成发送 HTTP 请求消息到 Web 服务器，Web 服务器响应 HTTP 请求。服务端的代码接受 HTTP 请求消息后返回 HTTP 响应消息

对于终端用户来说，HTTP 消息通常比较抽象，但作为开发者，我们通常需要知道 HTTP 消息的组成结构、如何访问和操作，以便于更好的完成任务，无论是创建并发起到某个 HTTP API 的请求还是处理传入的请求

每一个 HTTP 请求消息都遵循着下面的格式

POST /path HTTP/1.1
Host: example.com

foo=bar&baz=bat
请求的第一行是 「 请求行 」，构成部分按照顺序分别为: HTTP 请求方法，请求目标，HTTP 协议版本。它们之间通过一个空格 ( " " ) 分隔

接下来，从第二行开始是一个或多个 HTTP 请求头，然后是一个空行，最后是消息正文

HTTP 响应消息有着相类似的结构

HTTP/1.1 200 OK
Content-Type: text/plain

This is the response body
第一行是 「 状态行 」 ，构成部分按照顺序分别为：HTTP 协议版本，HTTP 状态码，状态描述短语。它们之间通过一个空格 ( " " ) 分隔

接下来，跟请求消息一样，从第二行开始是一个或多个 HTTP 响应头，然后是一个空行，最后是消息正文

此规范中定义的接口是关于 HTTP 消息和围绕它的组成元素的抽象

本篇规范中的 必须，不得，需要，应，不应，应该，不应该，推荐，可能 和 可选 等词按照 RFC 2119 中的描述进行解释

参考文献
RFC 2119
RFC 3986
RFC 7230
RFC 7231
1. 规范
1.1 HTTP 消息
一个 HTTP 消息是指一个客户端到服务端的请求或者一个服务端到客户端的响应

此文档对 HTTP 消息中的请求和响应分别定义了两个接口 Psr\Http\Message\RequestInterface 和 Psr\Http\Message\ResponseInterface

Psr\Http\Message\RequestInterface 和 Psr\Http\Message\ResponseInterface 都继承自 Psr\Http\Message\MessageInterface

尽管实现类库 可能 会直接实现 Psr\Http\Message\MessageInterface 接口，但同时也 应该 实现 Psr\Http\Message\RequestInterface 和 Psr\Http\Message\ResponseInterface 接口

从这里开始，当描述这些接口时，将会省略命名空间 Psr\Http\Message

1.2 HTTP 请求头
不区分大小写的请求头字段名
HTTP 消息中的请求头字段名是不区分大小写的。可以根据不区分大小写的字段名从 MessageInterface 的实现类中获取请求头数据，例如，通过 foo 查找请求头和通过 Foo 查找请求头返回的结果是一样的，同理，通过 Foo 设置请求头也会覆盖之前通过 foo 设置的值

<?php 
$message = $message->withHeader('foo', 'bar');

echo $message->getHeaderLine('foo');
// 输出: bar

echo $message->getHeaderLine('FOO');
// 输出: bar

$message = $message->withHeader('fOO', 'baz');
echo $message->getHeaderLine('foo');
// 输出: baz
虽然获取请求头信息的时候可以不区分大小写，但实现类库 必须 始终保持请求头的大小写不可变更，尤其是使用 getHeaders() 方法获取所有的请求头的时候

也就是说，传入的时候时什么样的，那么获取的时候也必须是什么样的

一些非标准的 HTTP 应用程序，可能会依赖于大小写敏感的请求头信息，因此为了适应不同的场景，在创建请求和响应的时候，用户可以自己决定是否区分大小写

请求头部字段有多个值的情况
为了适用一个 HTTP 「键」可以包含多个值的情况，我们使用字符串配合数组来实现

你可以从一个 MessageInterface 取出数组或字符串，使用 getHeaderLine($name) 方法可以获取通过逗号分割的不区分大小写的字符串形式的所有值，还可以使用 getHeader($name) 获取数组形式的相应请求信息的所有值

<?php
$message = $message
    ->withHeader('foo', 'bar')
    ->withAddedHeader('foo', 'baz');

$header = $message->getHeaderLine('foo');
// $header 的值为 'bar, baz'

$header = $message->getHeader('foo');
// ['bar', 'baz']
注意：并不是所有的请求头信息都可以适用逗号分割 ( 如 Set-Cookie ) ，当处理这种请求头信息时候， MessageInterace 的实现类 应该 使用 getHeader($name) 方法获取这种多值的情况

Host 头部
在请求中，Host 头信息通常和 URI 的 host 信息，还有建立起 TCP 连接使用的 Host 信息一致

然而，HTTP 标准规范允许请求头中的 host 信息与其它两个不一样

在构建请求的时候，如果 host 头信息未提供的话，实现类库 必须 尝试从 URI 中提取 host 信息

默认的，RequestInterface::withUri() 会从传递的 UriInterface 实例的参数中提取 host ，并替代请求中原有的 host 信息

你可以将第二个参数 ($preserveHost) 设置为 true 来保证返回的消息实例中，原有的 host 头信息不会被替代掉

This table illustrates what getHeaderLine('Host') will return for a request returned by withUri() with the $preserveHost argument set to true for various initial requests and URIs.

下表说明了当 withUri() 的第二个参数被设置为 true 的时，返回的消息实例中调用 getHeaderLine('Host') 方法会返回的内容

请求头 Host 1	请求 URI 中的 Host 信息2	传参进去 URI 的 Host3	Result
''	''	''	''
''	foo.com	''	foo.com
''	foo.com	bar.com	foo.com
foo.com	''	bar.com	foo.com
foo.com	bar.com	baz.com	foo.com
1 当前请求的 Host 头信息

2 当前请求 URI 中的 Host 信息

3 通过 withUri() 传参进入的 URI 中的 host 信息 withUri()

1.3 流
HTTP 消息由开始行、请求头、请求正文三部分组成

HTTP 请求正文可以非常小也可以非常大，尝试使用字符串的形式来展示消息内容，会消耗大量的内存，使用数据流的形式来读取消息可以解决此问题

StreamInterface 接口用于隐藏读取或写入流数据的操作细节，在一些情况下，消息类型的读取方式为字符串是能容许的，可以使用 php://memory 或者 php://temp

StreamInterface 接口提供了一些方法用于读取、写入和遍历流的内容， 同时提供了另外三个方法用于表示对流的操作能力：isReadable(), isWritable() 和 isSeekable()。 这三个方法可以让那个数据流的操作者得知数据流能否能提供他们想要的功能

每一个数据流的实例，都会有多种功能：可以只读、可以只写、可读可写，可以随机读写，可以按顺序读取等

此外，StreamInterface 还定义了一个 __toString() 用于以字符串的形式一次性输出所有消息内容

与请求和响应的接口不同的是，StreamInterface 并不强调不可修改性。因为在 PHP 的实现内，基本上没有办法保证不可修改性，指针的指向，内容的变更等状态，都是不可控的

作为读取者，可以 调用只读的方法来返回数据流，以最大程度上保证数据流的不可修改性

使用者要时刻明确的知道数据流的可修改性，建议把数据流附加到消息实例中，来强迫不可修改的特性

1.4 请求目标和 URI
遵循 RFC 7230 协议规范, 请求消息可以在请求头第一行的第二部分添加 「请求目标」

请求头第一行的第一部分是请求方法...

请求目标可以是以下格式之一 ：

origin-form

origin-form 由路径和可选的查询字符串组成，这种组合通常也被称之为相对 URL

origin-form 的协议头 ( scheme ) 和授权信息一般存储在 CGI 变量里

通过 TCP 协议传输的消息通常都是 origin-form

absolute-form

absolute-form 由协议头 ( scheme ) 、授权信息 ( [user-info@]host[:port] ) 、可选的路径、可选的查询字符串、可选的哈希值 ( fragment ) 组成，这种组合通常也被称之为绝对 URL

absolute-form 也是唯一一个支持 RFC 3986 规定的完整 URI 格式的 form

这种 form 通常用在 HTTP 代理中

authority-form

authority-form 只有授权信息，通常用在协助 HTTP 客户端和代理服务器建立连接的 CONNECT 请求中

asterisk-form

asterisk-form 只有一个字符 * 号，用于发起 OPTIONS 请求确定服务器支持哪些服务

除了上面阐述的请求目标，通常会有一个与请求目标分开的 effective URL。

effective URL 并不包含在传输的 HTTP 消息内，它只用于确定发起的请求的协议 ( http / https )，端口和主机名是否有效

effective URL 也可以用 UriInterface 表示。UriInterface 同时支持 RFC 3986 中规定的 HTTP 和 HTTPS URI。

UriInterface 接口提供了提供了大量的操作 URI 属性的方法，这样就可以避免重复解析 URI。特别的，还提供了 __toString() 方法用于将 URI 转换为字符串类型

当通过 getRequestTarget() 方法获取请求目标时，默认的，该方法会使用 URI 对象中相应的属性去构建一个 origin-form. origin-form 是至今为止最常见的请求目标

如果最终用户想要使用其它三种形式之一，或者用户想修改请求目标，可以调用 withRequestTarget() 方法

调用 withRequestTarget() 方法并不会影响到 getUri() 返回的 URI

例如，用户可能向服务器发起一个 asterisk-form 请求

<?php
$request = $request
    ->withMethod('OPTIONS')
    ->withRequestTarget('*')
    ->withUri(new Uri('https://example.org/'));
上面的代码最终会转换成如下 HTTP 消息

OPTIONS * HTTP/1.1
客户端可以使用另一个 有效的 URL ( 通过 getUri() 方法获得 ) 来确定协议，主机名和 TCP 端口号

HTTP 客户端 必须 使用 getRequestTarget() 返回的结果替代 Uri::getPath() 和 Uri::getQuery() 两个方法的值

客户端可以有选择性的实现 4 个中的任意数量，但不管怎么样，必须 始终使用 getRequestTarget()

对于那些不支持的请求目标，客户端 必须 明确拒绝，并且 绝对不允许 回退到使用 getUri() 的返回值

RequestInterface 接口提供了用于获取请求目标或者根据指定请求目标创建新实例的方法

默认情况下，如果实例的请求目标未设置或不存在，getRequestTarget() 可以返回 origin-form 的 URI 或者 / ( 如果连 URI 也不存在的话 )

withRequestTarget($requestTarget) 方法可以根据指定的请求目标创建一个新的实例，从另一方面说，开发者可以使用这个方法创建另外三种类型的 forms ( absolute-form, authority-form, and asterisk-form)

当然了，也不是说 URI 属性没用，对于客户端来说，可以使用 URI 来创建一个到服务器的连接

1.5 服务器接收到的请求 ( Server-side Requests )
RequestInterface 接口定义了通用的 HTTP 请求消息

但是，服务器接收的 HTTP 请求则需要特别对待，主要还是因为服务器端环境，服务器端通过通用网关接口 ( CGI ) 处理请求。

更确切的说，是 PHP 通过服务器 API ( SPA ) 抽象和扩展了 CGI，使用超全局变量封装和简化了获取 HTTP 请求的数据，例如

$_COOKIE， 通过将 cookies 信息解析到哈希数组中，简化了访问 HTTP cookies

$_GET， 通过将 URL 中的查询字符串解析到哈希数组中，简化了对请求参数的访问

$_POST， 将 HTTP POST 请求传输的请求正文解析到哈希数组中，简化了对请求正文参数的访问

$_FILES， 存储了上传的文件的元数据

$_SERVER， 存储了 CGI/SAPI 环境变量，同时也包含了发起请求的方法，请求的传输协议，请求的 URI 和请求的头部信息

ServerRequestInterface 接口扩展了 RequestInterface 接口，提供了对这些超全局变量的封装。

这种做法有助于减少开发者对 PHP 超全局变量的依赖，也更加容易测试

ServerRequestInterface 接口还提供了属性 attributes ，用于收集从请求或应用程序衍生而来的信息， 比如路径匹配信息，URI scheme 匹配信息，主机匹配信息等，这样应用程序就可以在多个请求之间传递数据

1.6 上传文件
ServerRequestInterface 接口提供了一个方法用于获取所有上传文件组成的树，树的叶子节点是 UploadedFileInterface 的实例

众所周知，当处理一组上传文件时，全局变量 $_FILES 有着许许多多的问题。 例如有一个数组文件形式的上传表单，表单字段名称为 files，提交了两个文件 files[0] 和 files[1] ，那么在 $_FIELS 就会如下格式

<?php
array(
    'files' => array(
        'name' => array(
            0 => 'file0.txt',
            1 => 'file1.html',
        ),
        'type' => array(
            0 => 'text/plain',
            1 => 'text/html',
        ),
        /* etc. */
    ),
)
然而，我们期待的可能是：

<?php
array(
    'files' => array(
        0 => array(
            'name' => 'file0.txt',
            'type' => 'text/plain',
            /* etc. */
        ),
        1 => array(
            'name' => 'file1.html',
            'type' => 'text/html',
            /* etc. */
        ),
    ),
)
$_FIELS 中的上传文件格式非常独特，我们不得不牢记它的结构，然后写一些代码转换成我们期待的样子

更严重的问题是，当出现了下列情况，全局变量 $_FILES 就不会被填充数据

HTTP 请求方法不是 POST
单元测试
在非 SAPI 环境下，例如 ReactPHP
因此，我们需要从全新的角度去看待上传的数据，如：

一个从 HTTP 消息正文中解析出上传文件的程序，类库实现者可以自由决定是否将上传文件保存到文件系统，以减少内存占用，I/O 开销和存储开销

单元测试的时候，开发人员可以存储和/或模拟上传文件元数据，以验证和确认不同的场景

getUploadedFiles() 方法可以用来规范化上传的文件，类库实现该函数时要做以下事情：

收集整理每一个上传文件的信息，创建 Psr\Http\Message\UploadedFileInterface 的实例并填充数据
重新构建上传文件树，叶子节点必须是 Psr\Http\Message\UploadedFileInterface 的实例
上传文件树的结构应该类似于全局变量 $_FILES：

第一个维度的键名是上传表单字段名
第一个维度的值可以是一个数组或者 UploadedFileInterface 树的实例
举个简单的例子，假设有一个上传表单

<input type="file" name="avatar" />
表单提交后，全局变量 $_FILES 的内容类似于

<?php
array(
    'avatar' => array(
        'tmp_name' => 'phpUxcOty',
        'name' => 'my-avatar.png',
        'size' => 90996,
        'type' => 'image/png',
        'error' => 0,
    ),
)
使用 getUploadedFiles() 方法规范化后则返回

<?php
array(
    'avatar' => /* UploadedFileInterface 实例 */
)
如果上传表单使用的是哈希表，就像下面的 HTML 片段

<input type="file" name="my-form[details][avatar]" />
那么在全局变量 $_FILES 中则有着如下的结构

<?php
array(
    'my-form' => array(
        'details' => array(
            'avatar' => array(
                'tmp_name' => 'phpUxcOty',
                'name' => 'my-avatar.png',
                'size' => 90996,
                'type' => 'image/png',
                'error' => 0,
            ),
        ),
    ),
)
使用 getUploadedFiles() 方法规范化后则返回

<?php
array(
    'my-form' => array(
        'details' => array(
            'avatar' => /* UploadedFileInterface 实例 */
        ),
    ),
)
如果上传表单使用的是哈希表和数组的混合表示法，就像下面的 HTML 片段

上传头像: <input type="file" name="my-form[details][avatars][]" />
上传头像: <input type="file" name="my-form[details][avatars][]" />
(例如，可以使用 JavaScript 创建额外的文件上传域，一次可以上传多个文件 )

在这种情况下，类库实现者 必须 根据索引收集上传文件的所有信息，此时的 $_FILES 已经偏离了原来正常的结构

<?php
array(
    'my-form' => array(
        'details' => array(
            'avatars' => array(
                'tmp_name' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'name' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'size' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'type' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'error' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
            ),
        ),
    ),
)
使用 getUploadedFiles() 规范化后，上面的 $_FILES 中的内容应该具有如下格式

<?php
array(
    'my-form' => array(
        'details' => array(
            'avatars' => array(
                0 => /* UploadedFileInterface instance */,
                1 => /* UploadedFileInterface instance */,
                2 => /* UploadedFileInterface instance */,
            ),
        ),
    ),
)
然后可以通过下标索引访问上传的文件，例如访问索引为 1 的文件可以使用下面的代码

<?php
$request->getUploadedFiles()['my-form']['details']['avatars'][1];
正因为上传文件的数据都是通过衍变而来的 ( 从 $_FILES 或者请求正文衍变而来 )， 所以此接口中的另一个方法 withUploadedFiles() 的存在则允许将规范上传文件的的任务交给另一个程序

继续接着上面的范例，开发者就可以轻松的写出下面的代码

<?php
$file0 = $request->getUploadedFiles()['files'][0];
$file1 = $request->getUploadedFiles()['files'][1];

printf(
    "Received the files %s and %s",
    $file0->getClientFilename(),
    $file1->getClientFilename()
);

// "Received the files file0.txt and file1.html"
编撰此规范时，我们还意识到，实现类库可能在非 SAPI 环境下使用， 因此 UploadedFileInterface 提供的方法需要确保无论在何种环境都能正常工作，尤其是：

moveTo($targetPath)

一个推荐的安全的用于替代 move_uploaded_file() 函数的方法

类库实现时必须先判定环境然后调用正确的操作

getStream()

此方法会返回一个 StreamInterface 的实例

在非 SAPI 环境下，一种可能的解决办法是将单个上传文件解析为 php://temp 流而非直接解析为文件，因为不存在上传文件这回事了

这样，无论何种环境，getStream() 都能正常工作

为什么？ 请查阅前文提到的，PHP 解析上传文件到 $_FILES 里是有条件的，那就是必须为 POST 方法

范例
<?php
// 将文件移动到上传目录
$filename = sprintf(
    '%s.%s',
    create_uuid(),
    pathinfo($file0->getClientFilename(), PATHINFO_EXTENSION)
);
$file0->moveTo(DATA_DIR . '/' . $filename);

// 将文件存储到 Amazon S3.
// 假设 $s3wrapper 是一个将文件上传到 S3 的 PHP 数据流
// Psr7StreamWrapper 是一个将 `StreamInterface` 封装为 `StreamWrapper` 的类
$stream = new Psr7StreamWrapper($file1->getStream());
stream_copy_to_stream($stream, $s3wrapper);
2. 命名空间
上面讨论到的所有接口和类都放在 psr/http-message 命名空间下

3. 接口
3.1 Psr\Http\Message\MessageInterface
<?php
namespace Psr\Http\Message;

/**
 * HTTP 消息由客户端发往服务端的请求和服务器返回给客户端的响应两部分组成
 * 此接口定义了操作一些操作 HTTP 消息的通用的方法
 * 
 * HTTP 消息被视为不可变更的
 * 所有能修改当前 HTTP 消息状态的方法，都 **必须** 有一套机制，在内部保持好原有的内容，返回修改状态后的新的实例
 * 
 * @see http://www.ietf.org/rfc/rfc7230.txt
 * @see http://www.ietf.org/rfc/rfc7231.txt
 */
interface MessageInterface
{
    /**
     * 获取字符串类型的 HTTP 协议版本号
     *
     * 此接口 **必须** 且只能返回 HTTP 协议版本号中的数字，例如 `1.1` , `1.0`，
     * 也就是要去掉前导 `HTTP/` 字符串
     *
     * @return string HTTP 协议版本号
     */
    public function getProtocolVersion();

    /**
     * 返回包含指定 HTTP 协议版本号的新实例
     *
     * $version 参数 **必须** 只能是数字形式的 HTTP 协议版本号，例如 `1.1` , `1.0`
     *
     * 此方法 **必须** 实现为保持实例是不可变更的，任何对 HTTP 协议版本号的变更操作都 **必须** 返回一个新的实例
     *
     * @param string $version HTTP 协议版本号
     * @return static
     */
    public function withProtocolVersion($version);

    /**
     * 返回消息中所有的头部数据
     *
     * 返回的二维数组中，第一维数组的「键」代表单条头信息的名字，「值」是
     * 以数组形式返回的
     * 
     *     // 将头部数据转换为字符串并输出
     *     foreach ($message->getHeaders() as $name => $values) {
     *         echo $name . ': ' . implode(', ', $values);
     *     }
     *
     *     // 迭代输出所有头部数据
     *     foreach ($message->getHeaders() as $name => $values) {
     *         foreach ($values as $value) {
     *             header(sprintf('%s: %s', $name, $value), false);
     *         }
     *     }
     *
     * 虽然头部数据中的名称是不区分大小写的，`getHeaders()` 会保持它们原有的样子
     *
     * @return string[][] 返回由头部数据组成的关联数组，
     *     每一个键名都 **必须** 是一个头部名称，
     *     且每一个键值都 **必须** 是字符串类型的数组
     */
    public function getHeaders();

    /**
     * 检查头部信息中是否包含了此名称的值，$name 不区分大小写
     *
     * @param string $name 不区分大小写的头部字段名称
     * @return bool 存在则返回 true，否则返回 false
     */
    public function hasHeader($name);

    /**
     * 根据给定的不区分大小写的名称，获取一条头部信息，以数组形式返回
     *
     * 此方法以数组形式返回对应名称的头信息
     *
     * 如果没有对应的头信息，**必须** 返回一个空数组
     *
     * @param string $name 不区分大小写的头部字段名称
     * @return string[] 返回头部数据中指定的头信息，如果指定的头部不存在，则返回空数组
     */
    public function getHeader($name);

    /**
     * 根据给定的名称，获取一条头信息，以逗号分隔的形式返回响应的头信息
     * 
     * 此方法返回所有对应的使用逗号拼接的头信息
     *
     *   $header = $message->getHeader( $name );
     *   return implode( ',' $header )      
     *
     * 注意： 并非所有的头部字段都可以使用逗号拼接，对于那些不能使用的，
     * 可以先调用 `getHeader()` 方法获取值，然后使用特定的连接符来拼接
     *
     * 如果不存在对应的头部值，此方法 **必须** 返回空字符串
     *
     * @param string $name 不区分大小写的头部字段名称
     * @return string 返回使用逗号(,)拼接的字符串类型的相对应的头部的值，
     *        如果指定的头部不存在，此方法 **必须** 返回空字符串
     */
    public function getHeaderLine($name);

    /**
     * 返回包含了特定头部的新的实例
     *
     * 头部字段名称是不区分大小写的。但是此方法必须保留其传参时的大小写状态，并能够在
     * 调用 `getHeaders()` 的时候被取出
     *
     * 此方法 **必须** 实现为保持实例是不可变更的，任何对头部信息的添加或更新操作都 **必须** 返回一个新的实例
     *
     * @param string $name 不区分大小写的头部字段名称
     * @param string|string[] $value 头部字段的值
     * @return static
     * @throws \InvalidArgumentException 如果字段名称或值非法，则抛出此异常
     */
    public function withHeader($name, $value);

    /**
     * 返回添加了指定头部的新实例
     *
     * 如果对应的头部已经存在，则会追加在原有值的后面，如果不存在，则会新增加
     *
     * 此方法 **必须** 实现为保持实例是不可变更的，任何对头部信息的添加或更新操作都 **必须** 返回一个新的实例
     *
     * @param string $name 被添加的不区分大小写的头部字段名称
     * @param string|string[] $value 头部字段的值
     * @return static
     * @throws \InvalidArgumentException 如果字段名称非法，则抛出此异常
     * @throws \InvalidArgumentException 如果值非法，则抛出此异常
     */
    public function withAddedHeader($name, $value);

    /**
     * 返回删除了指定头部的新实例
     *
     * 头信息字段在解析的时候，**必须** 保证是不区分大小写的
     *
     * 此方法 **必须** 实现为保持实例是不可变更的，任何对头部信息的删除操作都 **必须** 返回一个新的实例
     *
     * @param string $name 要被删除的不区分大小写的头部字段名称
     * @return static
     */
    public function withoutHeader($name);

    /**
     * 获取 HTTP 消息的正文内容
     *
     * @return StreamInterface 返回 HTTP 消息的正文内容
     */
    public function getBody();

    /**
     * 返回变更了消息正文内容的新实例
     *
     * 消息正文内容 **必须** 是 `StreamInterface` 接口的实例
     *
     * 此方法 **必须** 实现为保持实例是不可变更的，任何对消息正文内容的变更操作都 **必须** 返回一个新的实例
     *
     * @param StreamInterface $body 消息正文内容
     * @return static
     * @throws \InvalidArgumentException 如果 $body 参数非法，则抛出此异常
     */
    public function withBody(StreamInterface $body);
}
3.2 Psr\Http\Message\RequestInterface
<?php
namespace Psr\Http\Message;

/**
 * 表示客户端请求的 HTTP 消息对象
 *
 * 遵循 HTTP 协议规范, 该接口包含以下属性
 *
 * - Protocol version HTTP 协议版本号
 * - HTTP method   HTTP 请求方法
 * - URI           URI
 * - Headers       请求头部
 * - Message body  请求正文
 *
 * 在构造此接口的实例时，如果没有传递 Host 信息，实现类库 **必须** 解析传入的 URI 数据并提取 Host 信息
 * 
 * 该接口的实例 ( RequestInterface 的实例 ) 被视为是不可变更的。
 * 所有能修改状态的方法，都 **必须** 有一套机制，在内部保持好原有的内容，返回修改状态后的新的实例
 * 
 */
interface RequestInterface extends MessageInterface
{
    /**
     * 获取该请求对象的目标地址
     *
     * 返回该请求对象的请求目标(客户端)或请求对象的 URI(服务器端)或当前实例设置的值(请查阅 withRequestTarget())
     *
     * 大部分情况下，此方法会返回完整的 URI，除非调用 `withRequestTarget()` 设置过新值
     *
     * 如果没有提供 URI，并且没有提供任何的请求目标，此方法 **必须** 返回 "/"
     *
     * @return string
     */
    public function getRequestTarget();

    /**
     * 返回包含了指定请求目标的新实例
     *
     * 如果请求需要一个 `non-origin-form` 的请求目标，例如 `absolute-form` , 
     * `authority-form`, 或 `asterisk-form`
     * 则可以使用此方法来创建包含了特定请求目标的实例
     *
     * 此方法 **必须** 实现为保持实例是不可变更的，任何对请求目标的变更操作都 **必须** 返回一个新的实例
     *
     * @see http://tools.ietf.org/html/rfc7230#section-5.3 (各种合法的请求目标)
     * @param mixed $requestTarget
     * @return static
     */
    public function withRequestTarget($requestTarget);

    /**
     * 获取 HTTP 请求使用的方法名
     *
     * @return string 返回 HTTP 请求所用的方法名
     */
    public function getMethod();

    /**
     * 返回包含了指定 HTTP 请求方法的新实例
     *
     * HTTP 请求方法一般都是大写字母且是大小敏感的，因此实现该方法时 **不应该** 修改传递的值
     *
     * 此方法 **必须** 实现为保持实例是不可变更的，任何对 HTTP 请求方法的变更操作都 **必须** 返回一个新的实例
     *
     * @param string $method 大小写敏感的方法名
     * @return static
     * @throws \InvalidArgumentException 当传入不合法的方法名时抛出此异常
     */
    public function withMethod($method);

    /**
     * 获取 URI 实例
     *
     * 此方法 **必须** 返回一个 `UriInterface` 接口的实例
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @return UriInterface 返回与当前请求的 URL 对应的 UriInterface 实例
     */
    public function getUri();

    /**
     * 返回包含了指定 URI 的新实例
     *
     * 当传入的 `URI` 包含有 `HOST` 信息时，此方法 **必须** 更新返回的新请求中的 `HOST` 头信息
     * 如果 `URI` 实例没有附带 `HOST` 信息，任何之前存在的 `HOST` 信息 **必须** 作为候补，应用更改到返回的消息实例里
     *
     * 你可以通过传入第二个参数 `$preserveHost` 来来干预方法的处理行为。
     * 当 `$preserveHost` 设置为 `true` 的时候，此方法会遵循以下规则：
     *
     * - 如果 Host 请求头不存在或为空值且新的 URI 包含 host 信息，
     *     此方法 **必须** 更新返回的新请求中的 Host 信息
     * - 如果 Host 请求头不存在或为空值且新的 URI 不包含 host 信息，
     *     此方法 **绝对不能** 修改返回的新请求中的 Host 信息
     * - 如果 Host 请求头存在且不为空，此方法 **绝对不能** 修改返回的新请求中的 Host 信息
     * 
     *
     * 此方法 **必须** 实现为保持实例是不可变更的，任何对 URI 的变更操作都 **必须** 返回一个新的实例
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @param UriInterface $uri `UriInterface` 类型的 URI 实例
     * @param bool $preserveHost 是否保留原有的 HOST 头信息
     * @return static
     */
    public function withUri(UriInterface $uri, $preserveHost = false);
}
3.2.1 Psr\Http\Message\ServerRequestInterface
<?php
namespace Psr\Http\Message;

/**
 * 服务端请求对象
 *
 * 根据 HTTP 规范，该接口应该包含下列属性
 *
 * - Protocol version 协议版本号
 * - HTTP method  HTTP 请求方法
 * - URI          请求 URI 
 * - Headers      请求头部
 * - Message body 请求正文
 *
 * 此外，它还封装了从 CGI 和/或 PHP 环境传递给应用程序的所有数据，包括
 * 
 * - $_SERVER 超全局变量中的数据
 * - 任何 cookies 数据 ( 一般来自 $_COOKIE )
 * - 查询字符串参数 ( 一般来自 $_GET 或 parse_str() 函数)
 * - 上传的文件，如果有 ( 一般来自 $_FILES )
 * - 反序列化请求正文参数 ( 一般来自 $_POST )
 *
 * $_SERVER 超全局变量 **必须** 被视为不可更改的，因为它们表示请求时应用程序的状态，也没有允许修改这些值方法
 * 其它的数据如请求方法，可能在 $_SERVER 或请求正文中被重写，
 * 也可能在应用程序执行期间被更改 ( 例如请求正文会根据 content-type 头来反序列化 )
 * 
 * 此外，该接口可以从一个请求中解析并派生出额外的参数 ( 例如，URL 路径匹配, 解析 cookie 信息, 反序列化非表单格式请求正文, 
 * 根据授权头部信息检查用户是否授权等等)。这些数据应该存储在 `attributes` 属性中
 * 
 * 服务器端请求对象 ( ServerRequestInterface 的实例 ) 被视为是不可变更的。
 * 所有能修改状态的方法，都 **必须** 有一套机制，在内部保持好原有的内容，返回修改状态后的新的实例
 * 
 */
interface ServerRequestInterface extends RequestInterface
{
    /**
     * 获取服务器端参数
     * 
     * 获取与当前请求相关的环境数据，通常来自于 $_SERVER 全局变量，但却不一定都来自 $_SERVER
     *
     * @return array
     */
    public function getServerParams();

    /**
     * 获取 cookies 信息
     *
     * 获取客户端传送给服务端的 cookies 信息
     *
     * 返回的数据 **必须** 兼容全局变量 $_COOKIE 的数据结构
     *
     * @return array
     */
    public function getCookieParams();

    /**
     * 返回一个包含指定 cookies 信息的实例
     *
     * $cookies 数据不一定都来自全局变量 $_COOKIE，但却 **必须** 和 $_COOKIES 有着相同的数据结构
     * 通常，这些数据会在实例话时自动注入
     *
     * 此方法 **绝对不能** 修改请求实例中的相关 Cookie 或者服务器端变量中的相关 Cookie
     *
     * 此方法 **必须** 被实现为不可变更的。如果变更了 cookie 信息则 **必须** 返回一个新的实例
     * 
     * @param array $cookies 由键值对 cookies 信息所组成的数组
     * @return static
     */
    public function withCookieParams(array $cookies);

    /**
     * 获取解析后的查询参数
     *
     * 注意：URI 中的查询参数和服务器变量中的查询参数可能不一样
     * 如果想要获得原始的数据，可以通过 `getUri()->getQuery()` 方法或者解析服务器变量中的 `QUERY_STRING` 参数来获得
     *
     * @return array
     */
    public function getQueryParams();

    /**
     * 返回一个包含指定查询字符串参数的新实例
     *
     * 这些值 **应该** 在整个请求的生命周期内保持不变，它们可以在实例化的时候就被注入，
     * 数据来源 **可以** 是全局变量 $_GET 或者通过其它的数据比如 URI，衍生而来
     *
     * 如果数据是从解析 URI 中得到的，那么这些数据格式 **必须** 兼容 PHP 内置的 
     * `parse_str()` 函数的返回值，以便去掉重复的参数和处理嵌套的参数

     * 
     *
     * 设置查询字符串参数的操作 **绝对不能** 更新请求对象中的 URI 参数或服务器变量中的 URI 参数
     *
     * 该方法 **必须** 实现为保持实例是不可变更的，任何变更查询字符串参数的操作都 **必须** 返回一个新的实例
     *
     * @param array $query 查询字符串参数组成的数组，一般从全局变量 $_GET 处获得
     * @return static
     */
    public function withQueryParams(array $query);

    /**
     * 获取结构化的上传文件的数据
     *
     * 此方法返回包含所有的上传文件元数据组成的哈希表，
     * 上传文件的元数据据必须是 `Psr\Http\Message\UploadedFileInterface` 接口的实例
     *
     * 返回的数据 **可以** 从 `$_FILES` 转换而来，也可以是实例化时从请求正文解析得到，
     * 还 **可以** 是通过调用 `withUploadedFiles()` 方法注入得到
     *
     * @return array 返回由 `UploadedFileInterface` 实例构成的哈希表，
     * 如果不存在数据，则 **必须** 返回一个空数组
     */
    public function getUploadedFiles();

    /**
     * 创建一个包含指定上传文件的新实例
     *
     * 该方法 **必须** 实现为保持实例是不可变更的，任何变更请求正文参数的操作都 **必须** 返回一个新的实例
     *
     * @param array $uploadedFiles 由 UploadedFileInterface 实例组成的哈希数组
     * @return static
     * @throws \InvalidArgumentException 如果参数结构错误，则抛出此异常
     */
    public function withUploadedFiles(array $uploadedFiles);

    /**
     * 返回解析后的请求正文参数
     *
     * 如果请求头 `Content-Type` 的值为 `pplication/x-www-form-urlencoded` 或 
     * `multipart/form-data`，且请求方法为 `POST` ，那么调用此方法只会返回 `$_POST` 中的内容
     *
     * 否则，该方法可以返回任何解析请求正文得到的结果，返回的结果**必须** 只能是数组或者对象
     * 或者 `null` 值，如果请求正文不存在的话
     *
     * @return null|array|object 解析请求正文得到的结果
     */
    public function getParsedBody();

    /**
     * 返回包含指定请求正文参数的新实例
     *
     * 请求正文参数可以在实例化时自动注入
     *
     * 如果请求头 `Content-Type` 的值为 `pplication/x-www-form-urlencoded` 或 
     * `multipart/form-data`，且请求方法为 `POST` ，那么调用此方法只会注入 `$_POST` 中的内容
     *
     * 变量 $data 中的数据 **不要求** 都来自 `$_POST`，但 **必须** 是解析请求正文的到的数据
     * 
     * 此方法 **只** 接受数组或者对象或者 `null` 值作为参数
     *
     * 例如，如果能判定请求正文的格式为 `JSON`，该方法就可以根据解析后的 JSON 数据创建一个新的请求
     *
     * 该方法 **必须** 实现为保持实例是不可变更的，任何变更请求正文参数的操作都 **必须** 返回一个新的实例
     *
     * @param null|array|object $data 解析请求正文得到的数据
     * @return static
     * @throws \InvalidArgumentException 如果传递的参数类型错误，则抛出此异常
     * 
     */
    public function withParsedBody($data);

    /**
     * 返回所有的衍生属性
     *
     * 变量 `attributes` 用来保存从 request 衍生出来的任何参数。
     * 例如：路径匹配结果，cookies 解析结果，非表单请求正文解析结果等等
     *
     * 所有的衍生属性都与 应用程序 和 传入的请求有关，而且可以随时变更
     * 
     * @return mixed[] 从传入的请求衍生出来的所有属性
     */
    public function getAttributes();

    /**
     * 获取单个衍生属性
     *
     * 获取在 `getAttributes()` 返回结果中的单个衍生属性，如果属性没有值，那么返回传递的 $default
     *
     * 该方法可以避免先调用 `hasAttribute()` 方法，因为它有一个允许当属性不存在时返回一个默认的值
     *
     * @see getAttributes()
     * @param string $name 属性名
     * @param mixed $default 如果属性不存在则返回默认值
     * @return mixed
     */
    public function getAttribute($name, $default = null);

    /**
     * 更新某个衍生属性并返回一个新的实例
     *
     * 此方法允许更新在 `getAttributes()` 返回结果中的单个衍生属性
     *
     * 此方法 **必须** 实现为保持实例为不可变更的，如果更新了某个衍生属性则 **必须** 返回新的实例
     *
     * @see getAttributes()
     * @param string $name 属性名
     * @param mixed $value 属性值
     * @return static
     */
    public function withAttribute($name, $value);

    /**
     * 删除指定的衍生属性并返回一个新的实例
     *
     * 此方法允许删除在 `getAttributes()` 返回结果中的指定的衍生属性
     *
     * 此方法 **必须** 实现为保持实例为不可变更的，如果删除了某个衍生属性则 **必须** 返回新的实例
     *
     * @see getAttributes()
     * @param string $name 衍生属性名称
     * @return static
     */
    public function withoutAttribute($name);
}
3.3 Psr\Http\Message\ResponseInterface
<?php
namespace Psr\Http\Message;

/**
 * HTTP 服务端响应对象
 *
 * 根据 HTTP 规范，该对象应该包含下列属性
 *
 * - Protocol version  协议版本号
 * - Status code and reason phrase 状态码和原因短语
 * - Headers  响应头部
 * - Message body 响应正文
 *
 * 响应 ( Responses ) 被视为是不可变更的。
 * 所有能修改状态的方法，都 **必须** 有一套机制，在内部保持好原有的内容， 返回修改状态后的新的实例
 * 
 */
interface ResponseInterface extends MessageInterface
{
    /**
     * 获取响应状态码
     *
     * 状态码由三个数字组成的整数，表示服务器对请求的理解和的回应
     *
     * @return int 状态码
     */
    public function getStatusCode();

    /**
     * 返回一个包含指定状态码和可选的原因短语的 ResponseInterface 实例
     *
     * 如果没有传递原因短语，实现类库 **可以** 默认使用 RFC 7231 或 IANA 推荐标准中与状态码相对应的原因短语
     * 
     * 此方法 **必须** 实现为确保响应实例不可变更的，而且当状态码或原因短语有变更时 **必须** 返回一个新的实例
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @param int $code 由三个数字组成的状态码
     * @param string $reasonPhrase 状态码相对应的原因短语
     *   如果没有提供，实现类库 **可以** 默认使用 RFC 7231 或 IANA 推荐标准中与状态码相对应的原因短语
     * @return static
     * @throws \InvalidArgumentException 当状态码错误时抛出此异常
    public function withStatus($code, $reasonPhrase = '');

    /**
     * 根据状态码获取响应状态原因短语
     *
     * 原因短语不是响应状态行的必须组成部分，因此 **可以** 为空字符串。
     * 实现类库 **可以** 返回 RFC 7231 推荐标准中的原因短语或者返回 IANA HTTP Status Code Registry 中的原因短语
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @return string 原因短语; 如果不存在，则返回空字符串
     */
    public function getReasonPhrase();
}
3.4 Psr\Http\Message\StreamInterface
<?php
namespace Psr\Http\Message;

/**
 * 数据流 ( stream ) 接口
 *
 * 此接口封装了 PHP 数据流 ( stream ) 并提供了对数据流的通用操作，包括将流的内容转换为字符串
 */
interface StreamInterface
{
    /**
     * 将流的内容转换为字符串
     *
     * 此方法 **必须** 先寻址到流的开始之处，读取流的数据直到结束，再将读取到的数据转换为字符串
     *
     * **警告**：要转换的数据可能非常大
     *
     * 此方法 **绝对不可以** 抛出任何异常，因为 PHP 不允许 `__toString()` 抛出任何异常
     *
     * @see http://php.net/manual/en/language.oop5.magic.php#object.tostring
     * @return string
     */
    public function __toString();

    /**
     * 关闭数据流并释放资源
     *
     * @return void
     */
    public function close();

    /**
     * 将流与底层资源分离
     *
     * 当流被分离出来，流就进入不可用状态
     *
     * @return resource|null 底层资源，如果存在的话，否则返回 `null`
     */
    public function detach();

    /**
     * 获取流内容的大小
     *
     * @return int|null 返回流内容的大小，如果可知，否则返回 `null`
     */
    public function getSize();

    /**
     * 返回当前文件的读写位置
     *
     * @return int 文件游标的位置
     * @throws \RuntimeException 如果发生错误，则抛出此异常
     */
    public function tell();

    /**
     * 判断是否已经到了流结束的位置
     *
     * @return bool
     */
    public function eof();

    /**
     * 判断流是否可以寻址
     *
     * @return bool
     */
    public function isSeekable();

    /**
     * 寻址到流的指定位置
     *
     * @see http://www.php.net/manual/en/function.fseek.php
     * @param int $offset 偏移字节数
     * @param int $whence 指定游标从哪个位置开始寻址，值和内建函数 `fseek()` 的 $whence 参数一样  
     *     SEEK_SET: 从流的开始位置往后寻址 
     *     SEEK_CUR: 从当前位置往后寻址
     *     SEEK_END: 从流的结束位置往前寻址 offset.
     * @throws \RuntimeException on failure.
     */
    public function seek($offset, $whence = SEEK_SET);

    /**
     * 寻址到流的开始位置
     *
     * 如果流不可以被寻址，则抛出一个异常，否则和调用方法 `seek(0)` 有着同样的效果
     *
     * @see seek()
     * @see http://www.php.net/manual/en/function.fseek.php
     * @throws \RuntimeException 如果发生错误则抛出异常
     */
    public function rewind();

    /**
     * 判断流是否可写入数据
     *
     * @return bool
     */
    public function isWritable();

    /**
     * 往流里写入数据
     *
     * @param string $string 要被写入流的字符串数据
     * @return int 返回写入了多少数据到流里
     * @throws \RuntimeException 如果发生错误，则抛出此异常
     */
    public function write($string);

    /**
     * 判断流是否可读
     *
     * @return bool
     */
    public function isReadable();

    /**
     * 从流中读取数据
     *
     * @param int $length 要读取的数据的长度，返回数据的长度可能小于 $length，因为底层读操作返回的数据长度就是小于的
     * @return string 从流中读取到的数据，如果流中没有数据可读，则返回空字符串
     * @throws \RuntimeException 如果发生错误则抛出异常
     */
    public function read($length);

    /**
     * 以字符串格式的返回流中剩余的所有数据
     *
     * @return string
     * @throws \RuntimeException 如果流不可读，则抛出此异常
     * @throws \RuntimeException 读取过程中发生了错误，则抛出此异常
     */
    public function getContents();

    /**
     * 获取流的全部元数据或者特定的元数据
     *
     * 对于同一个 key ，此函数返回的数据和 PHP 内建函数 `stream_get_meta_data()` 返回的结果中 key 对应的值应该是一样的
     *
     * @see http://php.net/manual/en/function.stream-get-meta-data.php
     * @param string $key 要获取的指定元数据的键名.
     * @return array|mixed|null 
     *    如果 $key 为空则返回所有的元数据
     *    如果 $key 不为空且 $key 指定的键名存在则返回 $key 相关联的元数据
     *    如果 $key 指定的键名不存在则返回 `null` 
     */
    public function getMetadata($key = null);
}
3.5 Psr\Http\Message\UriInterface
<?php
namespace Psr\Http\Message;

/**
 * URI 数据接口
 *
 * 此接口按照 RFC 3986 来构建 HTTP URI，并提供了一些通用操作
 * 
 * 你可以自由的对此接口进行扩展，可以使用此 URI 接口来做 HTTP 相关的操作，也可以使用此接口做任何 URI 相关的操作
 *
 * 此接口的实例化对象被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的实例返回。
 *
 * 一般情况下，HTTP 请求中都会设置 Host 相关的头部，应用程序可以从服务端参数中获取 scheme
 *
 * @see http://tools.ietf.org/html/rfc3986 ( URI 标准规范 )
 */
interface UriInterface
{
    /**
     * 从 URI 中获取 scheme
     *
     * 如果 scheme 不存在，那么此方法 **必须** 返回一个空字符串
     *
     * 返回的值 **必须** 是小写字母，遵循 RFC 3986 3.1 章节规范
     *
     * 结尾部分的 ":" 字符不属于 scheme ，**一定不可** 出现在返回的数据中
     *
     * @see https://tools.ietf.org/html/rfc3986#section-3.1
     * @return  URI scheme 的值
     */
    public function getScheme();

    /**
     * 获取 URI 中的授权信息
     *
     * 如果授权信息不存在，此方法 **必须** 返回空字符串
     *
     * URI 中授权信息的语法格式如下：
     *
     * <pre>
     * [user-info@]host[:port]
     * </pre>
     *
     * 如果端口号没有设置，或者是当前 scheme 的默认端口号, 那么就 **不应该** 包含在返回值内
     *
     * @see https://tools.ietf.org/html/rfc3986#section-3.2
     * @return string URI 中的授权信息，格式为： "[user-info@]host[:port]" 
     */
    public function getAuthority();

    /**
     * 获取 URI 中的用户信息
     *
     * 如果用户名不存在， 此方法 **必须** 返回空字符串
     *
     * 如果用户名存在，则返回其值，如果密码也存在，则必须追加在用户名后，以冒号 ":" 分隔，然后一起返回
     *
     * "@" 字符不属于用户信息， **一定不可** 出现在返回值中
     *
     * @return string URI 中的用户信息，格式为："username[:password]"
     */
    public function getUserInfo();

    /**
     * 从 URI 中获取 host 值
     *
     * 如果 host 不存在，此方法 **必须** 返回空字符串
     *
     * 返回的值 **必须** 是小写字母，遵循 RFC 3986  3.2.2. 章节规范
     *
     * @see http://tools.ietf.org/html/rfc3986#section-3.2.2
     * @return string URI 中的 host 值
     */
    public function getHost();

    /**
     * 获取 URI 中的 port (端口号) 值
     *
     * 如果端口号存在且不是当前 scheme 的标准端口号， 此方法 **必须** 将其转化为 int 类型并返回
     * 
     * 如果端口号存在且是当前 scheme 的标准端口号，则 **应该** 返回 `null`
     *
     * 如果端口号不存在且 scheme 也不存在， 此方法 **必须** 返回 `null`
     *
     * 如果端口号不存在，但 scheme 存在， 此方法 **可以** 返回 scheme 对应的标准端口号，但 **应该** 返回 null
     *
     * @return null|int URI 中的端口号
     */
    public function getPort();

    /**
     * 获取 URI 中的 path (路径) 值
     *
     * 路径可以是空字符串 (`""`)，或绝对路径 (以 `/` 开始)，或相对路径 (不以 `/` 开始)
     * 此方法的实现 **必须** 支持这三种格式
     * 
     * 如果遵循 RFC 7230 章节 2.7.3. 规范，空路径(`""`) 和绝对路径 (`"/"`) 是一样的
     * 但此方法 **绝对不能** 自动的做此转换，因为在去除了根路径的环境中，例如前端控制器，两者的区别非常大
     * 应该让开发者自己去处理 `""` 和 `"/"`
     *
     * 返回的数据 **必须** 是经过 URL 编码，但 **绝对不能** 重复编码
     * 检查某个字符是否需要 URL 编码的方法，请查阅 RFC 3986, 章节 2 and 3.3.
     *
     * 例如，如果数据中包含反斜杠 (`/`)，又不是作为路径分隔符而出现，那么就应该将它编码为 `%2F`
     * 
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.3
     * @return string  URL 中的路径
     */
    public function getPath();

    /**
     * 获取 URI 中的查询字符串
     *
     * 如果不存在查询字符串，此方法 **必须** 返回空字符串
     *
     * 前导的 "?" 字符不属于查询字符串，**绝对不能** 出现在返回的数据中
     *
     * 返回的数据 **必须** 是经过 URL 编码，但 **绝对不能** 重复编码
     * 检查某个字符是否需要 URL 编码的方法，请查阅 RFC 3986, 章节 2 and 3.4.
     *
     * 例如，如果查询字符串键值对中的值包含了 `&` 字符，又不是作为键值对的分隔符而出现，那么就应该将它编码为 `%26`
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.4
     * @return string URI 中的查询字符串
     */
    public function getQuery();

    /**
     * 获取 URI 中的 fragment ( 中文译为哈希值，`#` 后面的数据 )
     *
     * 如果不存在 fragment 数据，此方法 **必须** 返回空字符串
     *
     * 前导的 `#` 不属于 fragment，**一定不能** 包含在返回的数据中
     *
     * 返回的数据 **必须** 是经过 URL 编码，但 **绝对不能** 重复编码
     * 检查某个字符是否需要 URL 编码的方法，请查阅 RFC 3986, 章节 2 and 3.5.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.5
     * @return string URI 中的 fragment
     */
    public function getFragment();

    /**
     * 返回包含了指定 scheme 的实例
     *
     * 此方法 **必须** 保存当前的实例状态，然后返回包含了指定 scheme 的新实例
     *
     * 实现类库 **必须** 同时支持大小写不敏感的 "http" 和 "https"。如有必要，还 **可以** 兼容其它的 scheme
     * 
     * 如果 `$scheme` 为空值，则表示要移除 scheme 的值
     *
     * @param string $scheme 新实例使用的 scheme
     * @return static 包含了指定 scheme 的新实例
     * @throws \InvalidArgumentException 如果 $scheme 参数错误，则抛出此异常
     * @throws \InvalidArgumentException 如果 scheme 不被支持，则抛出此异常
     */
    public function withScheme($scheme);

    /**
     * 返回包含了指定授权信息的新实例
     *
     * 此方法 **必须** 保存当前的实例状态，然后返回包含了指定授权信息的新实例
     *
     * 密码信息是可选的，但用户信息 **必须** 包含用户名
     * 
     * 如果 $user 为空值，则表示要移除用户信息
     *
     * @param string $user 授权信息中的用户名
     * @param null|string $password $user 对应的密码信息
     * @return static 包含了指定授权信息的新实例
     */
    public function withUserInfo($user, $password = null);

    /**
     * 返回包含了指定 host 的新实例
     *
     * 此方法 **必须** 保存当前的实例状态，然后返回包含了指定 HOST 的新实例
     *
     * 如果 $host 为空值，则表示要移除 host 值
     *
     * @param string $host 新实例中要设置的主机信息
     * @return static 包含了指定主机信息的新实例
     * @throws \InvalidArgumentException 如果 $host 参数错误，则抛出此异常
     */
    public function withHost($host);

    /**
     * 返回包含了指定端口号的新实例
     *
     * 此方法 **必须** 保存当前的实例状态，然后返回包含了指定端口号的新实例
     *
     * 如果 $port 的值超出了 TCP 和 UDP 端口号的范围，实现类库 **必须** 抛出异常
     * 
     * 如果 $port 为空值，则表示要移除 port 值
     *
     * @param null|int $port 新实例要使用的端口号，如果传递了空值，则表示删除端口号
     * @return static 包含了指定端口号的新的实例
     * @throws \InvalidArgumentException 如果 $port 参数错误，则抛出此异常
     */
    public function withPort($port);

    /**
     * 返回包含了指定路径的新实例
     * 
     * 此方法 **必须** 保存当前的实例状态，然后返回包含了指定路径的新实例
     *
     * 路径可以是空字符串 (`""`)，或绝对路径 (以 `/` 开始)，或相对路径 (不以 `/` 开始)
     * 此方法的实现 **必须** 支持这三种格式
     *
     * HTTP 中的路径一般使用以 `/` 开始的绝对路径，而不是相对路径
     * 如果 HTTP 中的路径不以 `/` 开头，那么表示应用程序或者用户知晓它们相对于哪个根路径
     *
     * 参数 $path 中既可以包含 URI 编码过的字符，也可以包含未编码过的字符
     * 实现类库要确保 `getPath()` 方法返回正确的编码后的数据
     *
     * 
     * @param string  新实例要使用的路径信息
     * @return static 包含了指定路径信息的新实例
     * @throws \InvalidArgumentException 如果 $path 参数错误则抛出此异常
     */
    public function withPath($path);

    /**
     * 返回包含了指定查询字符串的新实例
     *
     * 此方法 **必须** 保存当前的实例状态，然后返回包含了指定查询字符串的新实例
     *
     * 参数 $query 中既可以包含 URI 编码过的字符，也可以包含未编码过的字符
     * 实现类库要确保 `getQuery()` 方法返回正确的编码后的数据

     * 如果 $query 为空值，则表示要移除 query 值
     * 
     * @param string $query 新实例要使用的查询字符串
     * @return static 包含了指定查询字符串的新实例
     * @throws \InvalidArgumentException 如果 $query 参数错误则抛出此异常
     */
    public function withQuery($query);

    /**
     * 返回包含了指定哈希值 ( fragment ) 的新实例
     *
     * 此方法 **必须** 保存当前的实例状态，然后返回包含了指定哈希值 ( fragment ) 的新实例
     *
     * 参数 $fragment 中既可以包含 URI 编码过的字符，也可以包含未编码过的字符
     * 实现类库要确保 `getFragment()` 方法返回正确的编码后的数据
     *
     * 如果 $fragment 为空值，则表示要移除 fragment 值
     *
     * @param string $fragment 新实例要使用的哈希值 ( fragment )
     * @return static 包含了指定哈希值 ( fragment ) 的新实例
     */
    public function withFragment($fragment);

    /**
     * 将 URI 转换为字符串并返回
     *
     * 返回值可以是一个完整的 URI 或者遵循 RFC 3986 中章节 4.1. 的相对的 URI
     * 返回的数据取决于 URI 实例中哪些属性有值，此方法需要使用正确的连接符将 URI 中的各个部分组合在一起
     *
     * - 如果 scheme 存在，则 **必须** 以 `:` 结尾
     * - 如果授权信息存在，则 **必须** 以 `//` 开头
     * - 路径 ( path ) 可以不使用任何连接符，但如果出现了下面的两种情况，则 **必须** 调整路径来让路径有效，因为 PHP 不允许在 `__toString()` 函数中引发异常
     *     - 如果 path 是相对路径且授权信息存在，路径 **必须** 以 `/` 开头
     *     - 如果 path 以多个 `/` 开头且授权信息不存在，反斜杠 `/` 必须裁剪到一个，也就是说路径只能以一个 `/` 开头
     * - 如果查询字符串存在，则查询字符串 **必须** 以 `?` 开头
     * - 如果哈希值存在，则哈希值必须以 `#` 号开头
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.1
     * @return string
     */
    public function __toString();
}
3.6 Psr\Http\Message\UploadedFileInterface
<?php
namespace Psr\Http\Message;

/**
 * 通过 HTTP 请求上传的文件内容
 *
 * 此接口的实例是被视为无法修改的
 * 所有能修改状态的方法都 **必须** 有一套机制，在内部保持好原有的内容，然后返回修改状态后的新的实例
 */

interface UploadedFileInterface
{
    /**
     * 获取上传文件的数据流
     *
     * 此方法 **必须** 返回一个 `StreamInterface` 实例
     * 
     * 此方法的目的在于允许 PHP 对获取到的数据流直接操作，例如使用 `stream_copy_to_stream()` 函数
     *
     * 
     * @return StreamInterface 上传文件的数据流
     * @throws \RuntimeException 没有数据流时抛出异常
     * @throws \RuntimeException 无法创建数据流是抛出异常
     */
    public function getStream();

    /**
     * 将上传的文件移动到新的地址
     *
     * 此方法可以作为 `move_uploaded_file()` 方法的替代方法. 该方法保证能同时在 SAPI 和 非 SAPI 环境下工作。
     * 
     * 实现类库 **必须** 判断当前处于什么环境下，并且使用合适的方法来处理 ( move_uploaded_file(), rename() 或者数据库操作 )
     *
     * $targetPath 可以是绝对路径，也可以是相对路径，如果是相对路径，效果和 PHP 的 rename() 函数应该是一样的
     *
     * 如果移动完成，**必须** 移除原文件或数据流
     *
     * 如果该方法被调用多次，一次以后的后续调用都 **必须** 抛出异常
     * 
     * 如果在 SAPI 环境下使用，$_FILES 有值，当使用  moveTo(), is_uploaded_file() 和 move_uploaded_file() 移动文件时 **应该** 确保权限和上传状态的准确性
     *
     * 如果你将文件移动到数据流的话，请使用 `getStream()` 方法，因为在 SAPI 场景下，无法
     * 保证书写入数据流目标
     * 
     * @see http://php.net/is_uploaded_file
     * @see http://php.net/move_uploaded_file
     * @param string $targetPath 目标文件路径
     * @throws \InvalidArgumentException 如果参数 $targetPath 有问题时抛出异常
     * @throws \RuntimeException 移动文件时发生其它错误抛出异常
     * @throws \RuntimeException 多次调用此方法时抛出异常
     */
    public function moveTo($targetPath);

    /**
     * 获取文件的大小
     *
     * 实现类库 **应该** 返回存储在 $_FILE 数组中 `size` 键的值。PHP 会根据文件传输的实际大小来设置此值
     *
     * @return int|null 以 bytes 为单位的文件的大小，或者 null 未知的情况下
     */
    public function getSize();

    /**
     * 获取文件上传时出现的错误
     *
     * 返回值 **必须** 是 PHP 的 UPLOAD_ERR_XXX 常量之一
     *
     * 如果文件上传成功，此方法 **必须** 返回 UPLOAD_ERR_OK
     *
     * 实现类库 **应该** 返回存储在 $_FILES 数组中的 `error` 键的值
     *
     * @see http://php.net/manual/en/features.file-upload.errors.php
     * @return int PHP 的 UPLOAD_ERR_XXX 常量
     */
    public function getError();

    /**
     * 获取客户端提交的文件名
     *
     * 永远不要相信该方法返回的数据，客户端可能发送一个恶意的文件名来攻击你的应用程序
     *
     * 实现类库 **应该** 返回存储在 $_FILES 数组中的 `name` 键的值
     * 
     * @return string|null 客户端上传的文件名，或者 null 如果没有此值
     */
    public function getClientFilename();

    /**
     * 获取客户端提交的文件类型
     *
     * 永远不要相信该方法返回的数据，客户端有可能发送了一个恶意的文件类型名称来攻击你的程序
     *
     * 实现类库 **应该** 返回存储在 $_FILES 数组中 `type` 的值。
     *
     * @return string|null 用户上传的类型，或者 null 如果没有此值
     */
    public function getClientMediaType();
}
```
