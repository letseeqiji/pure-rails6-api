# 使用rails6 开发纯后端 API 项目

## API版本管理

今天我们聊聊 API 版本管理的话题。

### 1. API版本管理浅析

##### 那什么是 API 的版本管理？为什么要进行 API  版本管理呢？

程序开发必然伴随着bug的修改，功能的升级，而对于后端 API 的升级，我们需要采用合理的方式。通常情况下，我们是对修改关闭的，对新增开放的 ！所以对于程序的维护，特别是对 API 的维护，对于较大功能的升级，我们采用通过升级 API 版本的方式，从而避免对源代码的功能性修改！

##### 常见的API版本管理方案

- **域名前缀 + 版本号**

  - 例如：

    ```ruby
    http://api.demo.com/v1/xxx
    ```

  - 个人认为这个是最佳的方案，但是需要二级域名的支持，需要服务器的支持以及对二级域名的相关设置，如果咱们的服务器不支持，这个就用不了了。

- **命名空间 + 版本号**

  - 例如：

    ```ruby
    http://www.demo.com/api/v1/xxx
    ```

  - 这个方案也非常常见，并且也特别灵活，只需要我们在代码级别上操作即可。这种形式也是我们在书中要采用的方式！



### 2. API 版本管理功能实现

在Rails中要实现 API 版本的管理，需要完成两件事：

- 路由命名空间的增加；
- 对应命名空间文件夹的创建。



#### 2.1 知识点预热

##### 2.1.1 Rails 相关

- rails路由默认存放文件路径

  ```shell
  项目目录/config/routes.rb
  ```

- rails以命令行形式启动项目

  ```shell
  $ rails console
  # 简写形式
  $ rails c
  ```

- 创建一个复数的资源路由

  ```ruby
  # 项目目录/config/routes.rb
  Rails.application.routes.draw do
      # users_controller.rb
      resources :users
  end
  ```

- 一个复数资源路由提供的路由

  - 例如： `resources :users `

  | HTTP 方法     | 路径              | 辅助方法               | 控制器#动作     | 用途                         |
  | :------------ | :---------------- | :--------------------- | :-------------- | :--------------------------- |
  | `GET`         | `/users`          | `users_path`           | `users#index`   | 显示所有用户的列表           |
  | `GET`         | `/users/new`      | `new_users_path`       | `users#new`     | 返回用于新建用户的 HTML 表单 |
  | `POST`        | `/users`          |                        | `users#create`  | 新建用户                     |
  | `GET`         | `/users/:id`      | `users_path(:id)`      | `users#show`    | 显示指定用户                 |
  | `GET`         | `/users/:id/edit` | `edit_users_path(:id)` | `users#edit`    | 返回用于修改用户的 HTML 表单 |
  | `PATCH`/`PUT` | `/users/:id`      |                        | `users#update`  | 更新指定用户                 |
  | `DELETE`      | `/users/:id`      |                        | `users#destroy` | 删除指定用户                 |

- 为路由添加命名空间

  ```ruby
  # 项目目录/config/routes.rb
  Rails.application.routes.draw do
      namespace :api do
          # 这里添加路由
      end
  end
  ```
  
  上面的关键词 `namespace`本意就是`命名空间`, 在这里定义了名为 `api`的路由命名空间，这样就会在路由前面加上 `api`前缀：
  
  ```ruby
  http://www.demo.com/users  -->  http://www.demo.com/api/users
  ```
  
  而对于命名空间`api`, 我们应该建立文件夹   `项目/app/controller/api`   与其对应。



##### 2.1.2 Git 相关

- 生成新的分支

  ```shell
  $ git checkout -b 新分支名字
  ```

- 切换到制定分支

  ```shell
  $ git checkout 目标分支名字
  ```

- 切换到主分支

  ```shell
  $ git checkout master
  ```

- 合并分支到主分支

  ```shell
  $ git checkout master
  $ git merge 要合并的分支名
  ```

  

#### 2.2 文件版本管理

##### 2.1.1 切换分支到 chapter02

```
$ git checkout -b chapter02
```



#### 2.3 API版本管理功能开发

---

##### **目标**

- **路由地址以 `http://127.0.0.1/api/v1/`为前缀**
- **请求参数类型为 `json`**

---



##### 2.3.1 创建命名空间：`api`

```ruby
Rails.application.routes.draw do
    namespace :api do
        # 编写路由
    end
end
```

此时路由应该成为以 `http://www.demo.com/api` 开头的样式。



##### 2.3.2 创建命名空间`api`对应的文件夹 `app/controllers/api`

```shell
$ mkdir app/controllers/api
```



##### 2.3.3 创建版本`v1`的命名空间

```ruby
Rails.application.routes.draw do
    namespace :api do
        namespace :v1 do
            # 编写路由
        end
    end
end
```

此时路由应该成为以 `http://www.demo.com/api/v1` 开头的样式。



##### 2.3.4 创建命名空间`v1`对应的文件夹 `app/controllers/api/v1`

```shell
$ mkdir app/controllers/api/v1
```

>  以上命令会创建目录： app/controllers/api/v1



##### 2.3.5 设置请求参数格式为 `json`

```ruby
Rails.application.routes.draw do
	namespace :api, defaults: { format: :json } do
		namespace :v1 do
		# 编写路由
		end
	end
end
```



#### 2.4 文件版本管理

##### 2.4.1 添加所有变动文件到git管理

```shell
$ git add .
```

> 以上命令是我常使用的命令，您可以其它相同命令来代替



##### 2.4.2 提交所有变动文件

```shell
$ git commit -m "set the routes constraints for the API"
```



##### 2.4.3 合并当前分支到主分支

```shell
$ git checkout master
$ git merge chapter02
```

> 以上命令会把 chaptor02 分支合并到 master 分支



### 3. 总结

我们完成了项目API的版本管理功能，事情再难，练上万遍也会熟练！一定要跟上脚步，认真练习！