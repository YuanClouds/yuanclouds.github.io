---
title: 初遇pythonWeb之Flask的简洁
date: 2019-02-19 23:51:04
tags: python
---

# 一、前言

    作为一个客户端开发人员，python基础初级，web基础初级。以前在ssh、ssm的年代，框架搭建成本、代码臃肿成本等等因素影响开发效率，也带
    来了学习的阻力（深刻体会ed）。在了解python基本语法后，选择了了解这个轻量级的web框架Flask。

# 二、Flask 简言

   Flask是一个使用python编写的轻量级web应用框架，基于Werkzeug WSGI工具箱和Jinja2 模板引擎。相对Django，Flask最大的亮点就是它保持
   一个最简单的核心，可以通过扩展的特性新增必要的核心功能，例如文件上传、身份验证、缓存处理等。很适合小团队web开发，轻量可定制并且效率快。

# 三、学习输出

### 时间：

+ 3 day

### 相关知识点：

+ Flask jinja2
+ Flask Blueprint、视图
+ Flask Cache、HttpAuth、MongoDb

### 实现结果
#### 0x01 app接口

+ 用户注册
+ 用户登录，获取令牌
+ 获取用户信息，需要校验令牌，有10秒缓存（不直接查询数据库）

#### 0x02 web管理后台

+ 查看注册用户列表

#### 0x03 其他

+ 数据库支持mongo
+ 使用蓝图管理服务

#### 效果图
+ 注册

![76264992.png](http://pn5gfsan2.bkt.clouddn.com/flasksample/0.png)

+ 登录

![76283238.png](http://pn5gfsan2.bkt.clouddn.com/flasksample/1.png)

+ 获取个人信息(未登录无权限情况)

![76359005.png](http://pn5gfsan2.bkt.clouddn.com/flasksample/2.png)

+ 获取个人信息（已登录有权限情况）

![76399823.png](http://pn5gfsan2.bkt.clouddn.com/flasksample/3.png)

+ 后台管理系统（查看注册用户信息）

![76431753.png](http://pn5gfsan2.bkt.clouddn.com/flasksample/4.png)

# 三、Flask 技术点
## Flask的路由

Flask相对于SSM常见的框架，Flask的路由基于Werkzeug路由模块，路由中的url配置是直接写在执行的函数上的。不过对于轻量级的系统来说，灵活配置的意义也不大，相反在这一块维护性更强，例子：

``` java 
app = serviceEngine = Flask(__name__)

@app.route('/')
@app.route('/index')
@app.route('/index/<name>')
def index(name=None):
        if name is None:
             return'Hello World'
        return 'Hello %s' % name
```
默认运行在5000端口，这上面的代码表示 http:localhost:5000 和 http:localhost:5000/index 并且两个url都会返回‘Hello World'这个字符串。如果访问 http:localhost:5000/index/siven，url则会返回 ’Hello siven‘

这里涉及到三个知识点：

+ 默认路由

 默认路由就是@app.route('/')，这个应该好理解

+ 多URL的路由

  多URL路由表示一个函数可以支持多个URL路由，例如index函数就支持上面代码的三个URL

+ 带参数的路由

  路由URL支持带参数，@app.route('/index/<name>')，如果路由发现index后面又进行传参，则会返回到 'Hello %s' % name

详细可以参考文档 [@Flask路由](https://dormousehole.readthedocs.io/en/latest/quickstart.html#id6)

## Flask的 Flask-RESTful
从上面我们可以了解到在Flask中定义一个http接口非常简单，但是Flask还支持更简单的方法，就是Flask-RESTful扩展。
首先因为Flask-RESTful是官方拓展库，需要先进行引用

```
$ pip install Flask-RESTful 
```
实现代码举例：

``` 
def abort_if_value_invaild(value):
    if value < 0:
        abort(404, message="value {} is invaild !!!".format(value))

class Test(Resource):
    def post(self):
         value = request.args.get('value')  # 获取请求参数
         abort_if_value_invaild(value)
          return '请求返回 %d' % value   

api.add_resource(Test, '/test','/test2')
```

这里涉及三个知识点

+ RESTful 的基本用法

首先RESTful通过“api.add_resource”方法进行添加路由，其中方法的第一个形参是一个类名，并且这个类继承于Resource类，并且实现不同的HTTP方法，例如get、post、delete等请求。第二个参数则表示当前这个api的URL路径，例如上面代码案例，请求实现类为其父类Resource的Test类，并且实现post逻辑，最后通过add_resource进行api注册，api路径表示'/test'

+ 支持多个URL路由的RESTful

RESTful支持多个URL路由相对较为简单，只需要在add_resource注册api的时候进行多URL路径设置即可

+ 关于abort

在某些场景，例如本代码案例中要校验令牌token，如果在校验失败的时候我们希望有一个统一的逻辑去处理这种状况，abort函数除了异常抛出外，abort无疑是一个你最佳的一个选择。“abort()”可以直接退回出请求

## Flask的 flask-mongo
mongo属于文档型数据库，因其快速且有很高的扩展性被大多数开发者的认同，mongo存储格式为json格式，在数据处理来说较为方便。这里就不详细介绍mongo，详情了解可以[@mongo](https://www.mongodb.com/cn)

同样因为Flask-PyMongo是官方拓展库，需要先进行引用

```
$ pip install Flask-PyMongo
``
实现代码举例：

开发过java web的同学应该知道JDBC这个连接驱动包，通过配置指定的数据库URL链接、用户名以及密码后即可连接Mysql数据库。在PyMongo同理是通过 app.config.update进行连接配置，例如下面代码：

+ mongo配置

```
# 实例化mongo对象
mongo = PyMongo()

# 配置mongo
app.config.update(
    MONGO_URI='mongodb://localhost:27017/flask', # 表示MongoURL连接
    MONGO_USERNAME='siven', # 表示连接用户名
    MONGO_PASSWORD='123456' # 表示连接用户密码
)

# 初始化数据库
mongo.init_app(app)

```
+ mongo应用实例

数据库插入 api

``` 
    user = {'username':username,'userinfo':{'age':18,'sex':0}}
    # 数据库插入成功后会返回一个id
    insertedId = mongo.db.users.insert_one(user).inserted_id
```

数据库查询api

```
    # 根据username的key查询当前这个文档的json字符串，如果使用find进行查询是返回一个游标，需要开发者进行遍历
    user = mongo.db.users.find_one({'username':username}) 
```
其他api可以参考[@PyMongo](http://api.mongodb.com/python/current/api/pymongo/)

## Flask的 flask-httpauth
flask为开发者提供了一个很简便的Restful API认证库“flask-httpauth”，其中支持校验方式有HTTPBasicAuth、HTTPTokenAuth、HTTPDigestAuth等，可以查看[@flask-httpauth](http://flask-httpauth.readthedocs.io/en/latest/)

+ HTTPBasicAuth

并没有启动cookie的功能，验证只是在头部设置了 Authorization 字段来校验用户

+  HTTPTokenAuth

通过令牌的校验用户

+ ...

应用实例，模拟一个请求用户信息的api

```
# token 校验，这里依赖了 itsdangerous 库管理token
@auth.verify_token
def verify_token(token):
    print('verify_token ing... %s ' % token)
    g.user = None
    try:
        data = serializer.loads(token)
    except:
        return False
    if 'username' in data:
        g.user = data['username']
        return True
    return False

# 统一处理验证失败处理
@auth.error_handler
def unauthorized():
    return make_response(jsonify({'error': 'Unauthorized access'}), 401)

# 设置一个需要验证的api
@app.route('/getmyuserinfo')
@auth.login_required
def getmyuserinfo():
    return "Hello, %s!" % auth.username()
```
这里涉及到三个知识点
+ login_required修饰符
   通过login_required修饰的api，则意味着用户请求api都需要进行验证，验证方式由auth初始化实例的时候决定
+ verify_token修饰符
    通过verify_token修饰的函数，则意味着login_required修饰的api需要校验的逻辑在该函数上处理
+ error_handler修饰符
    首先看下verify_token的返回值为布尔值，通过error_handler修饰的函数表示统一处理验证失败处理逻辑，如果验证成功继续走api的逻辑

[@flask-httpauth](http://flask-httpauth.readthedocs.io/en/latest/)

## Flask的 blueprint蓝图
在web的应用场景，面向前端有apiServer层，面向运维有web管理后台。两者虽然都存在同一个应用中，但是业务与风格又有所不同。开发者如果想区分两个web应用，总有一些代码想进行重用。所以flask提供了blueprint蓝图的功能。blueprint蓝图管理的应用相当于应用里面的自营业，可以有自己的模板、静态目录、URL规则等，并且每个蓝图直接没有任何影响，可以共用应用的配置。
图

@详情在案例项目讲解说明

## Flask的装饰器与cache
Flask的装饰器可以理解类似java平台的动态代理，典型的AOP思想，在打点、日志场景用得较多。装饰器模式Decorator可以动态的扩充一个类或者函数的功能，实现的方法一般是在原有的类或者函数上包裹一层修饰类或修饰函数

+ 基础用例：

```
# 案例
def multiply(number):
    def in_func(value):
        return value * number
    return in_func

double = multiply(2)
triple = multiply(3)
print double(5)
print triple(3)
```
顾名思义，装饰器指的是在原来的函数或类进行一层逻辑装饰，案例代码里面 in_func 函数由multiply进行装饰
multiply(2)返回的double函数，其实实质是in_func函数，执行double(5)的时候相当于返回了 2 * 5
同理 triple(3) 函数执行相当于等同 3 * 3

+ 装饰器传入函数

借鉴AOP思维，如果要在一个函数里面增加日志打印方法，可以这么做:

``` 
def add_log(func):
    # *args matches all arguments without key
    # **kwargs matches all key=value arguments
    def newFunc(*args, **kwargs):
        logger.debug("Before %s() call" % func.__name__)
        ret = func(*args, **kwargs)
        logger.debug("After %s() call" % func.__name__)
        return ret
    return newFunc

def funcC(x, y):
    print x + y
newFuncC = add_log(funcC)
newFuncC(2, y=3)
```
这里args代表是一个任意类型的数组数据，kwargs代表是一个字典类型(k=v)的数组数据
上面装饰器的思想是：在逻辑函数里面装饰一层日志打印逻辑，最后通过返回一个新的函数给调用方
+ 装饰器注解操作符
根据上面改造，可以这么做：
```
def add_log(func):
    # *args matches all arguments without key
    # **kwargs matches all key=value arguments
    def newFunc(*args, **kwargs):
        logger.debug("Before %s() call" % func.__name__)
        ret = func(*args, **kwargs)
        logger.debug("After %s() call" % func.__name__)
        return ret
    return newFunc

@add_log
def funcC(x, y):
    print x + y

// 直接这么调用
funcC(2, y=3)
```
说那么多装饰器的实例，其实就是想表达flask中的cache底层原理就是装饰器。在下面项目案例中，会通过装饰器实现了一个简单的token校验处理，读者可以多关心看下。

# 四、项目讲解
### 项目描述
相对App服务，实现基本的注册登录以及token校验。相对后台管理员人员，实现可查看注册用户。详细在上面**实现结果**有说明

### 项目架构
![bf7b5f49-db6f-467a-be2c-4be479830e2a.jpg](http://pn5gfsan2.bkt.clouddn.com/flasksample/6.jpg)

+ ServiceCore

ServiceCore表示项目中的公用服务层，常用语其他子服务的公共服务。其他子web服务通过flask提供的蓝图BlurePrint进行注入

+ AppService

AppService表示客户端api层，有部分api是需要经过Token令牌层校验的。其中AppService里面包括其他子服务程序，例如userService，用户相关的服务(注册、登录等)

+ WebService

WebService表示后台管理层，demo展示类暂时没做安全校验

+ DataBase

DataBase表示数据模型层

+ 项目目录

![81361055.png](http://pn5gfsan2.bkt.clouddn.com/flasksample/5.png)

### appService部分讲解

+ 主服务配置

```
# 应用服务引擎
serviceEngine = Flask(__name__)
# 注册第三方组件
thirds_register(serviceEngine)
# 注册蓝图
serviceEngine.register_blueprint(web,url_prefix='/webservice') # 后台管理系统
serviceEngine.register_blueprint(app,url_prefix='/appservice') # app接口服务
```

+ 注册蓝图

```
serviceEngine.register_blueprint(web,url_prefix='/webservice') # 后台管理系统
serviceEngine.register_blueprint(app,url_prefix='/appservice') # app接口服务
```

+ 程序入口

```
from flask import Blueprint
from app.appService import service

app = Blueprint('app', __name__, template_folder='./templates')
# 注册接口服务
service.register(app)
```

程序初始化通过flask的Blueprint注入到主服务程序，最后通过appService api注册业务接口服务

+ 业务接口服务

```
# userService
def register(api):
    api.add_resource(Test,'/user/test')
    api.add_resource(UserRegister,'/user/register')
    api.add_resource(UserLogin,'/user/login')
    api.add_resource(UserInfo,'/user/getUserInfo')
```

在程序初始化时会遍历所有需要注册的业务接口服务，其中上面代码举例的是用户相关的接口服务userService，其中接口包括注册、登录以及获取用户信息

+ 业务接口-用户接口登录

查看实例相关实例代码：

``` 
class UserLogin(Resource):
    def post(self):
        # 1.获取用户的账号密码，这里由于是用例，密码是明文
        account = request.json.get('account')
        password = request.json.get('password')
        # 2.数据库查询校验密码
        result = useModelService.loginUser(account,password)
        # 3.如果结果不对，直接交给abort进行异常处理
        abort_if_error(result)
        # 4.登录成功刷新token
        token = User.generate_auth_token(account)
        #....
        return result
```

useModelService是数据mode层的接口实例，里面代理了mongo的实例，提供对user用户数据模型的相关数据库操作
login接口中，通过获取用户post的账号密码，如果在数据库校验正确，则刷新缓存的token，并且将token返回给请求用户，否则抛出并且返回异常给用户

+ 业务接口-用户获取用户信息

查看相关实例代码：

```
class UserInfo(Resource):
    @token.verify
    @cache.cached(timeout=10) # 缓存时间10秒
    def post(self):
        account = request.json.get('account')
        result = useModelService.getUserInfo(account)
        abort_if_error(result)
        return jsonify(result)
```

首先这个接口相对较为特别的对方，是两个修饰器，一个是token.verify，另一个则是cache.cached。cache较为好理解，表示这个请求数据有10秒的缓存时间，超过10秒后才会去查下数据库（根据用户当前请求的cookie），这里不做详细讲解

token.verify 顾名思义，修饰表示这个接口需要进行token校验。我们经过阅读上面修饰器的原理，可以了解修饰器是AOP的一种思维模式，通过一个函数修饰另外一个函数。因此这里我们可以查看到token实例的verify方法

``` 
    def verify(self,func):
        def actionFun(self):
            token = request.json.get('token')
            result = User.verify_auth_token(token)
            if result:
                return func(self)
                pass
            else:
                abort_if_token_invaild()
        return actionFun
```

从上面我们可以看到verify方法的形参是一个方法，结果返回是一个新的方法。也就是当请求A的时候，修饰器token.verify会先执行actionFun的方法逻辑，通过从request的token信息进行校验后，如果校验结果是正确的才真正的去执行原方法逻辑A，否则抛出abort

# 五、总结

本文举例了一个简单的web案例实例应用与讲解（在大型的web后台开发中当然考虑的东西不能这么少了），大概地从数据库mongo、令牌HttpAuth、蓝图管理子服务等层面描述了flask的一些拓展应用。

以前接触过java的ssm、ssh框架，相对前者而言，flask给我最大的体验就是轻量化、拓展性强以及可快速快发。虽然在实例中并没有很深入开拓flask的种种拓展功能，但是入门的restful api体验已经让我喜欢上flask这个框架。而且综合学习成本和性能，flask已经算是一个不错的轻量级web框架。给个赞~
