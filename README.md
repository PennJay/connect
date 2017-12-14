# connect
# connect是一个可扩展HTTP服务器框架，用于使用称为中间件的“插件”的节点。
    var connect = require('connect');
    var http = require('http');

    var app = connect();

    // gzip/deflate outgoing responses
    var compression = require('compression');
    app.use(compression());

    // store session state in browser cookie
    var cookieSession = require('cookie-session');
    app.use(cookieSession({
        keys: ['secret1', 'secret2']
    }));

    // parse urlencoded request bodies into req.body
    var bodyParser = require('body-parser');
    app.use(bodyParser.urlencoded({extended: false}));

    // respond to all requests
    app.use(function(req, res){
      res.end('Hello from Connect!\n');
    });

    //create node.js http server and listen on port
    http.createServer(app).listen(3000);
# 入门指南
connect是一个简单的框架，可以将各种“中间件”粘在一起处理请求。
# 安装 Connect
    $ npm install connect
# 创建 an app
主要组件是Connect“app”。这将存储所有添加的中间件，并且本身就是一个函数。
    var app = connect();
# 使用中间件
Connect的核心是“使用”中间件。中间件被添加为“堆栈”，传入的请求将逐个执行每个中间件，直到中间件不调用其中的next ()。
    app.use(function middleware1(req, res, next) {
      // middleware 1
      next();
    });
    app.use(function middleware2(req, res, next) {
      // middleware 2
      next();
    });
# 安装中间件
那个。use ()方法还采用与传入请求URL开头匹配的可选路径字符串。这允许基本路由。
    app.use('/foo', function fooMiddleware(req, res, next) {
      // req.url starts with "/foo"
      next();
    });
    app.use('/bar', function barMiddleware(req, res, next) {
      // req.url starts with "/bar"
      next();
    });
# 错误中间件
存在“错误处理”中间件的特殊情况。在中间件中，函数只需要4个参数。当中间件将错误传递给下一个中间件时，应用程序将继续查找在该中间件之后声明的错误中间件并调用它，跳过该中间件之上的任何错误中间件和下面的任何非错误中间件。
    // regular middleware
    app.use(function (req, res, next) {
      // i had an error
      next(new Error('boom!'));
    });

    // error middleware for errors that occurred in middleware
    // declared before this
    app.use(function onerror(err, req, res, next) {
      // an error occurred!
    });
# 从应用程序创建服务器
最后一步是在服务器中实际使用Connect应用程序。那个。listen ()方法为启动HTTP服务器提供了方便(与HTTP相同)。运行的node .js版本中的服务器侦听方法)。
    var server = app.listen(port);
这个应用程序本身实际上只是一个包含三个参数的函数，所以它也可以被传递给。在node . js中创建服务器( )
    var server = http.createServer(app);
# 中间件
    Connect / Express团队正式支持这些中间件和库:
    body-parser - previous bodyParser, json, and urlencoded. You may also be interested in:
    body
    co-body
    raw-body
    compression - previously compress
    connect-timeout - previously timeout
    cookie-parser - previously cookieParser
    cookie-session - previously cookieSession
    csurf - previously csrf
    errorhandler - previously error-handler
    express-session - previously session
    method-override - previously method-override
    morgan - previously logger
    response-time - previously response-time
    serve-favicon - previously favicon
    serve-index - previously directory
    serve-static - previously static
    vhost - previously vhost
其中大多数都是Connect 2 .x等效端口的精确端口。主要异常是cookie会话。
Connect /Express团队不再支持以前包含在Connect中的某些中间件，而是替换为其他模块，或者应该替换为更好的模块。请改为使用以下选项之一:
    cookieParser
    cookies and keygrip
    limit
    raw-body
    multipart
    connect-multiparty
    connect-busboy
    query
    qs
    staticCache
    st
    connect-static
 # API
 ConnectAPI非常简单，足以创建应用程序并添加中间件链。
当需要connect模块时，将返回一个函数，该函数将在调用时构造一个新应用程序。
    // require module
    var connect = require('connect')

    // create app
    var app = connect()
 # app(req, res[, next])
 应用程序本身就是一个函数。这只是app .handle的别名
 # app.handle(req, res[, out])
 调用函数将针对给定的node . js http请求( req )和响应( res)对象运行中间件堆栈。可以提供可选的函数out，如果请求(或错误)未被中间件堆栈处理，则调用该函数out。
 # app.listen([...])
 启动应用程序侦听请求。此方法将在内部创建node . jsHTTP服务器并调用。听着。
这是运行的node . js版本中server . listen ( )方法的别名，因此请参阅node .js文档了解所有不同的变体。最常见的签名是app . listen ( port)。
# app.use(fn)
在应用程序上使用一个函数，该函数表示中间件。将按调用app .use的顺序为每个请求调用该函数。函数使用三个参数调用:
    app.use(function (req, res, next) {
      // req is the Node.js http request object
      // res is the Node.js http response object
      // next is a function to call to invoke the next middleware
    })
除了plan函数之外，fn参数还可以是node . jsHTTP服务器实例或其他Connect应用程序实例。
# app.use(route, fn)
在应用程序上使用一个函数，该函数表示中间件。对于URL ( req . URL属性)以调用app .use的顺序以给定路由字符串开头的每个请求，都会调用该函数。函数使用三个参数调用:
    app.use('/foo', function (req, res, next) {
      // req is the Node.js http request object
      // res is the Node.js http response object
      // next is a function to call to invoke the next middleware
    })
 除了plan函数之外，fn参数还可以是node . jsHTTP服务器实例或其他Connect应用程序实例。
路由始终终止于路径分隔符( / )或点( .)字符。这意味着给定的routes / foo /和/ foo是相同的，它们都将匹配URL为/ foo、/ foo /、/ foo / bar和/ foo . bar的请求，但不匹配URL为/ foobar的请求。
路由以不区分大小写的方式匹配。
为了使中间件更易于编写而不知道路由，当调用fn时，req . URL将被更改以删除路由部分(并且原始的将作为req . originallurl提供)。例如，如果在route / foo中使用fn，则/ foo / bar的请求将使用req . URL = = ' / bar '和req . original URL = = ' / foo / bar'调用fn。
# 运行测试
    npm install
    npm test
# People
如果没有所有相关人员，Connect项目将会不一样。
Connect的原始作者是tjholwaychuk TJ的grati pay
目前的主要维护者是Doug Christopher Wilson Doug的grati pay
所有捐助者名单
# Node Compatibility

Connect < 1.x - node 0.2
Connect 1.x - node 0.4
Connect < 2.8 - node 0.6
Connect >= 2.8 < 3 - node 0.8
Connect >= 3 - node 0.10, 0.12, 4.x, 5.x, 6.x, 7.x, 8.x; io.js 1.x, 2.x, 3.x
# License

MIT
