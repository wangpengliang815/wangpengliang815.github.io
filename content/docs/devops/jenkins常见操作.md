# 通过 ssh 连接目标服务器

## 插件安装

1. Manage Jenkins => Manage Plugins

2. 下载  `Publish over SSH` 插件

下载后的本地路径：`$JENKINS_HOME/plugins/`

## 全局配置

1. 系统管理 => 系统设置 => 拉到底部找到 Publish over SSH

2. Passphrase  和 Key 二选一即可

- `Passphrase` 密码方式登录服务器

- `Key` 免密方式登录服务器。具体步骤：（将 Jenkins 所在主机的 `public_key` 添加到目标服务器的 `~/.ssh/authorized_keys` 即可）：

```bash
  cat id_rsa.pub >> authorized_keys # 生成authorized_keys
  scp authorized_keys root@10.80.188.130:/root/.ssh # 将authorized_keys拷贝到目标服务器 root/.ssh
```

- Remote Directory： 填写当前用户有权限操作并且必须是已经存在的路径

- SSH 服务器默认端口号是22，点击高级即可自定义端口号

配置完成后，点击 `Test Configuration` 返回 `Success` 证明 Jenkins 所在宿主机可以正常连接到目标服务器。

# 视图权限管理

目标：jenkins 配置不同用户显示不同视图。

## 添加用户

Manage Jenkins => Manage Users => 新建用户

## 插件安装

1. Manage Jenkins => Manage Plugins

2. 下载  `Role-based Authorization Strategy` 插件
3. Manage Jenkins => Configure Global Security => 授权策略 => 选择启用 Role-Based Strategy

## 权限配置

Manage Jenkins => Manage And Assign Roles

- Manage Roles：编辑权限
- Assign Roles：  把编辑好的权限分配给不同用户
- Role Strategy Macros： 角色策略宏，没有用到

![](/images/2022-07-12-16-40-50.png)

### Manage Roles

编辑全局用户权限 `Global roles`：添加角色 `normal` 并分配相关权限

![](/images/2022-07-12-16-42-36.png)

编辑项目权限 `Item roles`：添加项目组，根据需要自己调整权限范围

![](/images/2022-07-12-16-47-04.png)

使用通配符配置之后，添加 Job 时按照设定的前缀作为名称，则可以实现按照设定权限显示给不同用户。

> `pattern` 为正则表达式，语法为 java 正则表达式语法


比如：`group_dev` 组对应的项目以 `dev` 开头，用户创建 Job 时必须以 `dev` 开头，否则对应的授权用户无法看到此Job。

> 注意： 这里的模糊匹配时不能写成 `*` 要写成  `.*` 

### Assign Roles 

Global roles：指定用户有哪个用户组权限

![](/images/2022-07-12-16-54-08.png)

Item roles ：指定用户有哪个项目组权限

![](/images/2022-07-12-16-55-25.png)

## 添加视图


管理账户中点击标签栏“+”号添加新视图。需要注意：视图名大小写要与通配符一致，根据之前添加项目角色时通配符配置的前缀创建视图。一定要选择列表视图。

![](/images/2022-07-12-16-57-09.png)

# 视图复制

```java
import hudson.model.*
        //源view
        def str_view = "CeriOS.Core.DataDir"
        //目标view
        def str_new_view = "CeriOS.Core.Gateway"
        //源job名称(模糊匹配)
        def str_search = "CeriOS.Core.Datadir"
        //目标job名称(模糊匹配后替换)
        def str_replace = "CeriOS.Core.Gateway"

        def view = Hudson.instance.getView(str_view)
        //copy all projects of a view
        for(item in view.getItems())
        {
          //create the new project name
          newName = item.getName().replace(str_search, str_replace)
          // copy the job, disable and save it
          def job
          try {
                //因为第一次导入后报错，所以添加了try-catch 跳过已存在的job
                job = Hudson.instance.copy(item, newName)
          } catch(IllegalArgumentException e) {
             println(e.toString())
             println("$newName job is exists")
             continue
          } catch(Exception e) {
            println(e.toString())
            continue
          }
      //是否禁用任务，false不禁用，true禁用
          job.disabled = false
          job.save() 
          Hudson.instance.getView(str_new_view).add(job)
          println(" $item.name copied as $newName")
        }
```

