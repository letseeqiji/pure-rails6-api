# 使用rails6 开发纯后端 API 项目

## 用户状态管理：JWT系统引入与使用

---

##### **本节目标**

- **引入JWT机制实现用户状态管理；**
- **编写关键代码实现功能；**
- **利用JWT完善用户系统。**

---

上一节我们已经实现了用户相关的操作，在最后的总结中我们已经总结出现有的用户系统缺乏有效的权限管理。引入JWT机制后，我们将进一步完善我们的用户相关操作。

### 1. 文件版本管理

#### 1.1 创建新分支

```ruby
$ git checkout -b chapter05
```



### 2.  聊聊用户状态的管理

这里的`状态`是指对于网站而言用户是否登录在线还是已经退出的状态，站点通过识别用户的这些状态从而实现对用户权限的管理，比如有些敏感操作需要用户登录才能使用，而某些需要一定权限的操作还要通过获取用户的权限来限制用户操作进而达到限制用户和保护站点等作用。用户的状态管理实际上就是对用户状态的保存、识别和管理。

我们的站点采用网络传输协议是 http 协议，而http协议是一种无状态协议，也就是说从协议层角度讲是不能保存和识别用户状态的！这也就意味着要实现用户的状态管理就要通过我们自己来实现了。不过这种必要的功能已经有很多方案了。

常用的用户状态管理方案有：Cookie， Session 以及我们今天要使用的 JWT。下面就这三种常用方案做简单比较。

- Cookie：存储在客户端。服务器可以请求客户端存储必要信息，客户端访问服务端时会自动将cookie内容附加到请求头中，而符合http协议标准的服务器一般都能解析该头信息来识别用户，从而达到用户状态管理的目的。我们可以将 Cookie 看成是一张出入证，要访问服务器只要出示证明就行了，只要你有证就让你访问。如果要退出，就销毁这张证明，也就是当然是发送请求到客户端销毁Cookie即可。当然Cookie也是有限制的，一般需要同域才能使用，也就是说这张证明确实是我们网站的专有证明才行。
- Session： 存储在服务端。用户登录，服务端会将登录存储用户的信息打包成对象并生成唯一的sessionid来指向该信息并存储在指定存储系统中，例如内存、服务器文件、redis中，不过这不重要，我们只要知道session是存储在服务端的，而且是有一个唯一的sessionid来标示每一个用户的信息。Session默认情况下是依赖于cookie 的，之所以这么说，是因为针对用户的sessionid默认会存储在用户客户端的Cookie中，客户端每次访问服务端都会自动附加带有 sessionid 的Cookie内容，服务端获取到Cookie中的sessionid后，再到本地获取sessionid对应的用户信息，这样就实现了用户登录状态的保留。要退出，只要销毁本地对应的Session信息即可。我们可以将Session看成是一张公园门票，客户端手里握有票根，服务端存有副票。客户端每次访问都要带上票根来让服务端验证。
- JWT：在只是需要确认用户状态的场景下是不需要存储的，但这样会引发很多安全问题。对于JWT，大家或许更多的听到的是 token 这个概念。我们可以认为 JWT 是一种方案是一种标准，JWT会通过自己的一套逻辑 生成 token，然后颁发给客户端，客户端只要每次访问服务端携带上这个token即可，由于token的生成过程是可逆的，所以通过这个token，服务端可以计算出用户的信息。我们可以认为 token 是一句和当前用户绑定的暗号，但这个暗号的具体意义只有服务端知道，客户端只要携带这个暗号给服务端，服务端就能通过加密算法解析出暗号中的内容来实现对用户状态的管理。

其实通过认识上面几种用户管理的实现方案，其实要实现用户状态管理，基本思路都是一样的，客户端要访问服务端都要带一把唯一的“钥匙”，通过这把“钥匙”，服务端就能够知道并区分不同用户的身份。我们这里要使用 JWT 来实现用户状态管理，下面就让我们好好认识一下 JWT。



### 3. JWT详解

`JWT`全称`the JSON Web Token`， 中文理解为“json版网络token”， 实际上是一种 `token`实现标准或者说是一种`token`实现方案。在 [wiki](https://wikipedia.org/wiki/JSON_Web_Token_Web_Token) 中我们可以查阅该标准的具体说明。下面我们就仔细来剖析下 JWT 的具体构成。

##### 3.1 构成

一个 `JWT` 规范的 `token` 通常由3部分组成：

- header：token的头部，是一个结构体，通常包含了当前token的功能性描述，例如：生成时间，加密算法，数据类型等；
- payload：有很多文章称之为“载荷”，是存储我们自定义数据的地方，是一个数据容器；
- signature：签名，要保证一个token有效性而不是伪造的，这就是它的作用。

有了上面三部分内容，就可以生成唯一的具体的token:  

 	base64(header).base64(payload).base64(signature)

就是把每一部分都 base64 化，然后使用 “.” 进行连接，最终形成token字符串。

##### 3.2 安全性

有人或许疑惑，既然是一个公开的标准，那么算法也具有公共性特征，那不很容易被破解？ 其实在 `JWT`的安全机制中，一个 JWT token的生成是需要我们自定义一个秘钥字符串的。这个秘钥字符串是我们自定义的，只要不泄露，是不会被破解的。而且由于token的特殊性，秘钥字符串我们可以经常更换，用以更大程度的避免泄露。

##### 3.3 可逆性

JWT的加密和解密过程是可逆的，我们可以把具体的数据通过算法处理成token字符串，也可以通过一个token字符串通过有效的算法还原出具体的数据。

##### 3.4 API

在所有的 API 中，最重要的有两个：

- encode：利用具体的数据生成 token；
- decode：解析 token 到具体的数据。

##### 3.5 实现

在绝大部分的语言中，针对 JWT token 都有自己的实现方案，我们可以通过搜索引擎或者在语言或者框架的扩展包服务器上搜索相关的项目，参照文档使用这些具体的实现。在 Rails 中我们使用的是一个gem：`jwt`。我们可以在 [github](https://gems.ruby-china.com/gems/jwt) 上找到它的源码。



### 4. 在Rails中使用 `jwt`

---

**思路分析**

在 `Rails` 中使用 `jwt`，我们要使用它来生成token和解析token，并且利用获取到的用户信息进行用户状态和权限管理，所以我们 具体要做以下几 件事：

- 引入gem: `jwt` ；
- 编写我们自己的 `jwt` 公共类 ；
- 向用户暴露操作 token 的相关接口
  - 根据用户的邮箱和密码生成 token 给用户；
  - 接受用户的 token 并解析token来获取用户信息。
- 利用得到的用户信息进行其它验证功能的编写。

---



#### 4.1 引入gem: `jwt`

修改 项目目录下的Gemfile 文件

```ruby
# source 'https://rubygems.org'
source 'http://gems.ruby-china.com'
# ...
gem 'jwt'
# ...
```

在项目根目录下打开命令行，执行安装命令

```ruby
$ bundle
```

可以在 Rails console 中测试我们是否正常引入了 `jwt`

```shell
$ rails c   
2.7.2 :001 > secret_key = '123456abc'
 => "123456abc" 
2.7.2 :002 > payload = {message: 'Hello rails'}
 => {:message=>"Hello rails"} 
2.7.2 :003 > token = JWT.encode(payload, secret_key)
 => "eyJhbGciOiJIUzI1NiJ9.eyJtZXNzYWdlIjoiSGVsbG8gcmFpbHMifQ.S4K6N2UF5_cLdjmMsNFKuYSwsSzGeFTV4YlS2pX7wBU" 
2.7.2 :004 > JWT.decode(token, secret_key)
 => [{"message"=>"Hello rails"}, {"alg"=>"HS256"}]
```

从运行结果我们看到，我们已经可以正常使用 jwt 的相关功能了。



#### 4.2 编写公共类`JsonWebToken`

---

在正式编写代码前，我们需要了解：

- `jwt` 提供了两个重要的方法
  - `JWT.encode(payload, SECRET_KEY)`：添加数据生成具体的token；
  - `JWT.decode(token, SECRET_KEY)`：根据token解析出数据。
- 在Rails中，与具体业务无关的公用类通常定义在目录：`项目/lib`

所以我们要实现以下目标：

- 创建类库文件：`lib/json_web_token.rb` 
- 实现我们项目中方便使用的token相关方法
  - `JsonWebToken.encode(payload, exp = 24.hours.from_now)`: 根据具体的payload生成token，并可以设置过期时间，默认是24小时；
  - `JsonWebToken.decode(token)`: 根据token解析用户信息。
- 设置rails自动加载 lib 目录下的文件。

---



##### 4.2.1 创建类库文件：`lib/json_web_token.rb` 

```shell
$ touch lib/json_web_token.rb
```



##### 4.2.2 实现JsonWebToken类编写

`lib/json_web_token.rb`

```ruby
class JsonWebToken
	SECRET_KEY = Rails.application.credentials.secret_key_base.to_s

	def self.encode(payload, exp = 24.hours.from_now)
		payload[:exp] = exp.to_i
		JWT.encode(payload, SECRET_KEY)
	end

	def self.decode(token)
		decoded = JWT.decode(token, SECRET_KEY).first
		HashWithIndifferentAccess.new decoded
	end
end
```

> HashWithIndifferentAccess ：可以使针对hash指定访问使用的key允许使用字符串和符号。[官方文档](https://api.rubyonrails.org/classes/ActiveSupport/HashWithIndifferentAccess.html)



##### 4.2.3 设置`Rails`自动加载` lib` 目录下文件

`config/application.rb`

```ruby
# 下面两种方法，选择其中一个就好！！！
# 方式,
config.autoload_paths += Rails.root.join("lib")
# 方式2
config.autoload_paths << Rails.root.join("lib")
```

如果你正在运行服务器，更改了配置，请重启服务器。



##### 4.2.4 文件版本控制

```shell
$ git add .
$ git commit -m "add jwt gem"
```



### 5. `token`控制器开发

---

用户通过email和密码来获得token，这其实就是登录的过程！我们可以设置token有效期默认为24小时！也就是一个token可以用24小时，过期后需要重新登录，当然我们也可以允许用使用过期的token换取一个新的token，但在该项目中我们不添加此的功能。

基于以上功能我们可以确认我们要实现以下功能：

- 创建 tokens 控制器；
- 创建相应的路由;
- 编写测试;
- 编写create方法。
- 在tokens 控制器中添加 create 方法来响应用户的请求，该方法就收一个user对象，包含用户的 email 和 password 信息，如果信息都正确，生成token并返回，如果信息不正确，返回 401；

---

#### 5.1 创建 `tokens` 控制器

```ruby
$ rails generate controller api::v1::tokens create
Running via Spring preloader in process 4104
      create  app/controllers/api/v1/tokens_controller.rb
       route  namespace :api do
  namespace :v1 do
    get 'tokens/create'
  end
end
      invoke  test_unit
      create    test/controllers/api/v1/tokens_controller_test.rb
```



#### 5.2 创建相应的路由

其实在创建控制器时，`Rails` 已经帮我们自动在路由文件中添加了路由，但是不正确，我们可以手动修改好。

`config/routes.rb`

```ruby
Rails.application.routes.draw do
  namespace :api, defaults: { format: :json } do
    namespace :v1 do
      # 编写路由
      # ...
      resources :tokens, only: [:create]
    end
  end
end
```

我们这里使用 `only` 限制了只能访问 `create` 方法。

可以使用命令行查看 `tokens` 控制器能够使用的路由：

```ruby
$ rails routes -c tokens                       
--[ Route 1 ]-----------------------------------------------------
Prefix            | api_v1_tokens
Verb              | POST
URI               | /api/v1/tokens(.:format)
Controller#Action | api/v1/tokens#create {:format=>:json}
```



#### 5.3 编写测试

---

**思路分析**

我们要编写的是 创建 `token`的测试。

使用的路由是： `post: /api/v1/tokens`

请求参数是：`{"user" : {"email":"用户的邮箱", "password":"用户密码"}}`

期待用户名和密码正常的结果是： 

- 返回值：`{"error_code":0, "data":"token值", "message":"ok"}`
- http状态码：`201`

期待的用户名或密码异常的结果是：

- http状态码：`401`

---

##### 5.3.1 首先清空默认生成的测试方法

打开文件`test/controllers/api/v1/tokens_controller_test.rb` ， 可以看到，Rails也已经帮我们生成了一个测试 `should get create`, 显然这个测试内容也不正确，我们可以在其基础上修改，或者干脆删除该方法，保留测试的基本框架。我们还可以预先设置添加一个公共用户。

```ruby
require "test_helper"

class Api::V1::TokensControllerTest < ActionDispatch::IntegrationTest
  setup do
     @user_one = users(:one) 
  end
  # 编写测试
end
```

保存测试文件，下面就可以编写具体的测试了。



##### 5.3.2 编写测试

- 成功性测试：使用正确的email，正确的password请求，期望：返回token，http状态码返回201.

  ```ruby
  test "create success: create token with legal email and password" do
      post api_v1_tokens_path, params:{user:{email:@user_one.email, password:"123456"}}, as: :json
      assert_response 201
      
      json_response = JSON.parse(response.body)
      assert_not json_response['data']['token'].blank?
  end
  ```

- 失败性测试：使用正确的email，错误的password请求，期望：http状态码返回401.

  ```ruby
  test "create fail: create token with legal email and illegal password" do
      post api_v1_tokens_path, params:{user:{email:@user_one.email, password:"123"}}, as: :json
      assert_response 401
  end
  ```

运行测试会报错：`没有找到方法 create`。 



#### 5.4 编写`tokens#create`方法

---

**思路分析**

- 接受参数
- 验证信息正确性
  - 用户信息正确
    - 创建token并返回
  - 用于信息错误
    - 返回 401

---

`app/controller/api/v1/tokens_controller.rb`

```ruby
class Api::V1::TokensController < ApplicationController
  def create
    @user = User.find_by_email(user_params[:email])
    unless @user&.authenticate(user_params[:password])
      head 401 
      return
    end

    payload = {user_id: @user.id}
    exp_time = 24.hours.from_now
    if (JsonWebToken rescue nil) 
      token = JsonWebToken.encode(payload, exp_time) 
      render json: {error_code:0, data:{token:token, exp_time:exp_time}, message:"ok"}, status: 201
      return
    else
      head 401
    end
  end

  private
    def user_params
      params.require(:user).permit(:email, :password)
    end
end
```

现在如果客户端请求这个地址就会获取到token，获取到token客户端可以缓存到本地，以后遇到需要登录的接口，就可以把token附加到接口的参数中，可以是附加到一个请求参数上，也可以附加到请求头信息中！总之前后端要协调好，前端如何传输，那么后端就会按照传输的方式进行接收。

#### 5.5 测试

在命令行运行测试

```shell
$ rails test
# Running:
Error:
Api::V1::TokensControllerTest#test_create_fail:_create_token_with_legal_email_and_illegal_password:
BCrypt::Errors::InvalidHash: invalid hash
    app/controllers/api/v1/tokens_controller.rb:5:in `create'
    test/controllers/api/v1/tokens_controller_test.rb:17:in `block in <class:TokensControllerTest>'

rails test test/controllers/api/v1/tokens_controller_test.rb:16

......

Error:
Api::V1::TokensControllerTest#test_create_success:_create_token_with_legal_email_and_password:
BCrypt::Errors::InvalidHash: invalid hash
    app/controllers/api/v1/tokens_controller.rb:5:in `create'
    test/controllers/api/v1/tokens_controller_test.rb:9:in `block in <class:TokensControllerTest>'

rails test test/controllers/api/v1/tokens_controller_test.rb:8

Finished in 0.411657s, 34.0089 runs/s, 36.4381 assertions/s.
14 runs, 15 assertions, 0 failures, 2 errors, 0 skips
```

可以看到报错了，主要错误是 `BCrypt::Errors::InvalidHash: invalid hash`,  这个是由于我们现在使用了 `BCrypt`， 我们在测试环境中，Rails 会使用 `test/fixtures/users.yml`中的数据填充数据库，而用户的 `password_digest`字段是加密的密码而不是明文，我们需要修改 `test/fixtures/users.yml`文件。

 `test/fixtures/users.yml`

```ruby
one:
  email: 'user1@demo.com'
  # password_digest: '123456'
  password_digest: <%= BCrypt::Password.create('123456') %>
  role: 1

two:
  email: 'user2@demo.com'
  # password_digest: '123456'
  password_digest: <%= BCrypt::Password.create('123456') %>
  role: 1

```

保存文件，再次运行测试：

```shell
$ rails test
Running via Spring preloader in process 5318
Run options: --seed 63565

# Running:
..............

Finished in 1.013487s, 13.8137 runs/s, 17.7605 assertions/s.
14 runs, 18 assertions, 0 failures, 0 errors, 0 skips
```

现在就通过测试了。



#### 5.6 文件版本管理

```shell
$ git add .
$ git commit -m "setup tokens controller"
```



### 6. 用户状态管理

---

**思路分析**

现在客户端已经可以通过发送http请求来获取 `token`了，由于我们在`token`中已经存储了 用户的id，所以如果客户端在请求后端接口时如果附加提供了有效的 `token`，后端就可以接收`token`，然后解析出 `用户的id`，继而我们就可以确定用户信息！当确定了一个用户的信息，我们就可以按照我们的业务需求进行一些列的操作，比如：用户状态管理、用户权限控制等。

那么客户端要怎样传给后台`token`呢？

这个在工作中可以前后端进行约定，不过业界常用的方案有两种：

- 参数法

  把token当做请求参数传递，例如：

  ```ruby
  http://demo.com/api/v1/users?token=xxxxxxxxxxxxxxxxxxxxxxxx
  ```

- 请求头法

  把 token 放入到请求头中进行传递，例如：

  ```ruby
  request.headers['Authorization] = token值
  ```

我们这里使用请求头法，一旦确认了前台传值的方法，后端接受方式也就确定了。

现在我们接收到了token，我们要做的第一步就是解析它，由于在所有的控制器中我们都可能会用到这种场景，我们可以把控制器的公共方法提取到控制器目录下 `concerns` 中的 `authuser.rb` 中。在 `authuser.rb` 中我们主要就是解析 token 获取 当前用户并返回当前用户。当然我们要做一些测试工作，来保证功能的正确性。

所以我们需要完成以下工作：

- 定义公共方法解析token并获取当前访问用户；
- 编写测试

---

#### 6.1 获取当前用户

---

**思路分析**

首先需要创建文件：`app/controllers/concerns/authuser.rb`

然后我们需要在新建的文件中创建一个 `module`，并在其中定义 `current_user` 方法

---

##### 6.1.1 创建文件

```shell
$ touch app/controllers/concerns/authuser.rb
```

##### 6.1.2 开发获取当前用户方法

---

**思路分析**

获取请求头中的 Authorization 值，也就是token

判断是否存在Authorization 值

- 若存在
  - 解析 token 获取 用户id
  - 根据用户id查询用户信息并返回【有可能是nil】
- 若不存在
  - 返回nil

需要注意的是，为了防止重复调用导致的重复查询，我们在方法开始就应该判断下是否已经获取到当前用户了，如果已经获取到了直接返回用户信息，这样可以有效避免重复的查询。

---

```ruby
module Authuser
  def current_user
    return @current_user if @current_user
    
    token = request.headers['Authorization']
    return nil if token.nil?

    info= JsonWebToken.decode(token)
    @current_user = User.find_by_id(info[:user_id])
  end
end
```



#### 6.2 测试

---

**思路分析**

我们要测试的是针对获取到的token取得当前的用户，所以我们的测试应该包含：

- 成功的测试：传值合法的token，验证获取的用户不是 nil
- 失败的测试：传值非法的token，验证获取的用户是 nil

具体我们要完成以下步骤：

- 创建测试文件
- 编写测试
- 运行测试

---

##### 6.2.1 创建测试文件

```ruby
$ mkdir test/controllers/concerns
$ touch test/controllers/concerns/authuser_test.rb
```

准备基础框架

```ruby
require "test_helper"

class MockController
  include Authuser
  attr_accessor :request
end

class MockRequest
  attr_accessor :headers
  def initialize headers
    @headers = headers
  end
end

class AuthuserTest < ActionDispatch::IntegrationTest
  setup do
    @user = users(:one)
    # 构造： @authentication.request
    @authentication = MockController.new  
    # 构造: @authentication.request.headers = {}
    @authentication.request = MockRequest.new({})
  end

  # 编写测试
end
```

这里为了模拟request请求的数据结构而不是通过真实的http请求，需要创建两个辅助类。首先要明白我们想要的是某个对象具有 `request.headers["Authorization"]` 结构，为了创建这个对象我们定义了`MockController`类，为了是这个对象拥有`headers`属性，我们需要定义 `MockRequest` 类。



##### 6.2.2 编写测试

- 成功的测试：传值合法的token，验证获取的用户不是 nil

  ```ruby
  test "should get user from Authorization token" do
      @authentication.request.headers["Authorization"] = JsonWebToken.encode(user_id: @user.id)
      assert_not_nil @authentication.current_user
      assert_equal @user.id, @authentication.current_user.id
  end
  ```

  

- 失败的测试：传值非法的token，验证获取的用户是 nil

  ```ruby
  test "should not get user from empty Authorization token" do
      @authentication.request.headers["Authorization"] = nil
      assert_nil @authentication.current_user
  end
  ```



##### 6.2.3 运行测试

```shell
$ rails test
# Running:
................
Finished in 1.016836s, 15.7351 runs/s, 20.6523 assertions/s.
16 runs, 21 assertions, 0 failures, 0 errors, 0 skips
```

测试顺利通过了。



#### 6.3 在控制器中使用`token`

---

**思路分析**

现在我们已经定义好 了公共的token接受与解析方法，现在就可以在控制器使用该方法了。因为要在所有的控制器中都有可能用到登录用户的信息，而我们自定义的所有控制器都继承自 `app/controllers/application_controller.rb`中的`ApplicationController`，所以我们直接在`ApplicationController`中引入`Authuser`模块即可，这样继承了`ApplicationController`控制器的类都可以直接使用 `current_user`方法来获取当前登录的用户。

既然可以获取登录的用户信息了，我们就可以完成在之前用户模块中未能完成的权限控制部分。比如只有管理员才有权限查看用户列表和删除用户，只有用户自己或者管理员才能修改用户信息，还有有些操作只有登录的用户才能进行操作。我们就拿用户的相关操作举例来实现一些常用的验证。

我们的目标如下：

- 只有管理员才能删除用户
- 只有管理员才能查看用户列表
- 只有管理员或者用户自己才能修改用户信息

基于目标我们需要完成以下工作

- 在`ApplicationController`中引入`Authuser`模块；

- 在`UsersController`中定义检查用户是否是管理员的方法`check_admin`，并指定 删除用户前调用该方法；
- 指定执行查看用户列表操作前调用 `check_admin` 方法；
- 在`UsersController`中定义检查用户是否是管理员或者用户信息的拥有者的方法`check_admin_or_owner`，并指定 修改用户前调用该方法；
- 编写并完成测试。

---



##### 6.3.1 在`ApplicationController`中引入`Authuser`模块

`app/controllers/application_controller.rb`

```ruby
class ApplicationController < ActionController::API
  include Authuser
end
```



##### 6.3.2 定义检查用户是否是管理员的方法`check_admin`并调用

`app/controllers/api/v1/users_controller.rb`

```ruby
class Api::V1::UsersController < ApplicationController
  # 其它代码
  before_action :check_admin, only: [:index, :destroy]
  # 其它代码
    
  private
   	# 其它代码
    def is_admin?
      current_user&.role == 0
    end
    
    def check_admin
      head 403 unless is_admin?
    end
end
```



##### 6.3.3 定义`check_admin_or_owner`并调用

`app/controllers/api/v1/users_controller.rb`

```ruby
class Api::V1::UsersController < ApplicationController
  # 其它代码
  before_action :check_admin_or_owner, only: [:update]
  # 其它代码
    
  private
   	# 其它代码
    def is_owner?
      @user.id == current_user&.id
    end

    def check_admin_or_owner
      head 403 unless is_admin? || is_owner?
    end
end
```



##### 6.3.4 测试

---

**思路分析**

我们为用户的`destroy`和`update`方法添加了限制，而之前是没有的，所以如果现在运行测试，应该会报错。

不过这是对的，虽然看到红色报错会让你不舒服。

那现在我们需要对之前的测试进行修改并且还要写一写新的测试。

修改之前的主要是要添加上请求的token。这个比较简单。

新增加的测试主要是针对不合规的token应该得到拒绝，也就是http状态码是403，这个比较简单，我们可以直接进行书写。

---

- 准备测试数据

  我们首先需要将预设值的用户one的角色值设置为0，把他设置为管理员。

  `test/fixtures/users.yml`

  ```ruby
  one:
    email: 'user1@demo.com'
    # password_digest: '123456'
    password_digest: <%= BCrypt::Password.create('123456') %>
    role: 0
  ```

- 修改之前的测试

  `test/controllers/api/v1/users_controller_test.rb`

  ```ruby
  require "test_helper"
  
  class Api::V1::UsersControllerTest < ActionDispatch::IntegrationTest
    setup do
      @user = users(:one)
      @user2 = users(:two)
    end
  
    test "index_success: should show users" do
      get api_v1_users_path, 
        # 新增
        headers: { Authorization: JsonWebToken.encode(user_id: @user.id) },
        as: :json
      assert_response 200
    end
  
    # 未修改代码
  
    test "update_success: should update user" do
      put api_v1_user_path(@user), 
        params: {user:{email: 'test1@test.com', password: '123456'}},
        # 新增
        headers: { Authorization: JsonWebToken.encode(user_id: @user.id) },
        as: :json
    
      assert_response 202
    end
  
    test "destroy_success: should destroy user" do
      # 验证 User.count 变化 -1
      assert_difference('User.count', -1) do
        delete api_v1_user_path(@user2), 
          # 新增
          headers: { Authorization: JsonWebToken.encode(user_id: @user.id) },
          as: :json
      end
  
      assert_response 204
    end
  end
  ```

  现在运行测试，没有问题，全部通过。

- 新增测试

  - 非管理员无权查看用户列表，断言：http状态是403

    `test/controllers/api/v1/users_controller_test.rb`

    ```ruby
    test "index_forbidden: should forbiden show users cause not admin" do
        get api_v1_users_path, 
        # 新增
        headers: { Authorization: JsonWebToken.encode(user_id: @user2.id) },
        as: :json
        assert_response 403
    end
    ```

  - 非管理员无权修改用户，断言：http状态是403

    `test/controllers/api/v1/users_controller_test.rb`

    ```ruby
    test "update_forbidden: should forbiden update user cause not admin" do
        put api_v1_user_path(@user), 
        params: {user:{email: 'test1@test.com', password: '123456'}},
        headers: { Authorization: JsonWebToken.encode(user_id: @user2.id) },
        as: :json
    
        assert_response 403
    end
    ```

  - 非管理员无权删除用户，断言：http状态是403

    `test/controllers/api/v1/users_controller_test.rb`

    ```ruby
    test "destroy_forbidden: should forbiden destroy_ user cause not admin" do
        delete api_v1_user_path(@user), 
        headers: { Authorization: JsonWebToken.encode(user_id: @user2.id) },
        as: :json
    
        assert_response 403
    end
    ```

- 运行测试

  ```shell
  $ rails test
  
  Finished in 1.111107s, 17.1001 runs/s, 21.6001 assertions/s.
  19 runs, 24 assertions, 0 failures, 0 errors, 0 skips
  ```

  完美通过！！



### 7. 文件版本管理

#### 1.1 提交所有变动

```ruby
$ git add .
$ git commit -m "use jwt in users_controller"
```

#### 1.2 合并版本

```ruby
$ git checkout master
$ git merge chapter05
```



### 8. 总结

至此我们完成了JWT的引入，并且把它用到了我们的项目中，实现了用户的状态管理，用户的权限控制。在后面的章节中还会继续使用它来完成更多的工作。当然，JWT的应用不仅如此，还有更多的工作需要我们完成和完善！我们继续加油！
