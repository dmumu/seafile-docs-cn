### 关于

- [Docker](https://docker.com/) 是一个可以打包应用以及依赖包到一个可移植的容器中，以更轻量的方式运行Linux应用程序, 这比传统的虚拟机要更快.

- Docker 可以让您在服务器上更轻松的部署以及更新 [Seafile服务](https://github.com/haiwen/seafile).

- 在镜像中会使用 Seafile 团队推荐的配置作为默认配置.

如果您并不熟悉 Docker 的命令，请参考[docker文档](https://docs.docker.com/engine/reference/commandline/cli/).

### 快速开始

运行 Seafile 服务容器:

```sh
docker run -d --name seafile \
  -e SEAFILE_SERVER_HOSTNAME=seafile.example.com \
  -v /opt/seafile-data:/shared \
  -p 80:80 \
  seafileltd/seafile:latest
```

第一次运行会进行初始化，等待几分钟然后通过访问`http://seafile.example.com`查看 Seafile 的Web界面.

这条命令会将宿主机上的`/opt/seafile-data`目录挂载到 Seafile 容器中,你可以在这里找到日志或其他的数据文件.

### 更多配置选项

#### 自定义管理员用户名和密码

默认的管理员账号以及密码分别为`me@example.com`，`asecret`.你可以通过设置容器的环境变量来改变初始化时生成的管理员的账号和密码.

例如:

```sh
docker run -d --name seafile \
  -e SEAFILE_SERVER_HOSTNAME=seafile.example.com \
  -e SEAFILE_ADMIN_EMAIL=me@example.com \
  -e SEAFILE_ADMIN_PASSWORD=a_very_secret_password \
  -v /opt/seafile-data:/shared \
  -p 80:80 \
  seafileltd/seafile:latest
```

如果您忘记了管理员密码，你可以添加一个新的管理员账号，然后通过这个新的管理员账号重置之前的管理员密码.

#### 向Let's encrypt申请SSL证书

如果您设置`SEAFILE_SERVER_LETSENCRYPT`为`true`, 那么容器会自动根据设置的主机名向Let's encrypt申请SSL证书

例如:

```sh
docker run -d --name seafile \
  -e SEAFILE_SERVER_LETSENCRYPT=true \
  -e SEAFILE_SERVER_HOSTNAME=seafile.example.com \
  -e SEAFILE_ADMIN_EMAIL=me@example.com \
  -e SEAFILE_ADMIN_PASSWORD=a_very_secret_password \
  -v /opt/seafile-data:/shared \
  -p 80:80 \
  -p 443:443 \
  seafileltd/seafile:latest
```

如果你想使用已经拥有的SSL证书:

- 创建`/opt/seafile-data/ssl`目录, 并将你的证书以及私钥放入这个目录中.
- 假设您的网站名称为`seafile.example.com`, 那么您的证书名称必须为`seafile.example.com.crt`，而且您的私钥名称必须为`seafile.example.com.key`

#### 修改 Seafile 服务的配置

Seafile 服务的配置会存放在`/shared/seafile/conf`目录下，你可以根据 [Seafile 手册](https://manual.seafile.com/)修改配置

修改之后需要重启容器.

```sh
docker restart seafile
```

#### 查询日志

Seafile 服务的日志会存放在`/shared/logs/seafile`目录下, 由于是将`/opt/seafile-data`挂载到`/shared`，所以同样可以在宿主机上的`/opt/seafile-data/logs/seafile`目录下找到.

系统日志会存放在`/shared/logs/var-log`目录下.

#### 添加新的管理员

确保您的容器正在运行，然后输入以下命令:

```sh
docker exec -it seafile /opt/seafile/seafile-server-latest/reset-admin.sh
```

然后根据提示输入用户名以及密码即可

### 目录结构

#### `/shared`

共享卷的挂载点,您可以选择在容器外部存储某些持久性信息.在这个项目中，我们会在外部保存各种日志文件和上传目录。 这使您可以轻松重建容器而不会丢失重要信息。

- /shared/db: mysql服务的数据目录
- /shared/seafile: Seafile 服务的配置文件以及数据文件
- /shared/logs: 日志目录
  - /shared/logs/var-log: 我们将容器内的`/var/log`挂载到本目录.您可以在`shared/logs/var-log/nginx/`中找到nginx的日志文件
  - /shared/logs/seafile: Seafile 服务运行产生的日志文件目录.比如您可以在 `shared/logs/seafile/seafile.log`文件中看到seaf-server的日志
- /shared/ssl: 存放证书的目录，默认不存在

### 升级 Seafile 服务

升级到 Seafile 服务的最新版本:

```sh
docker pull seafileltd/seafile:latest
docker rm -f seafile
docker run -d --name seafile \
  -e SEAFILE_SERVER_LETSENCRYPT=true \
  -e SEAFILE_SERVER_HOSTNAME=seafile.example.com \
  -e SEAFILE_ADMIN_EMAIL=me@example.com \
  -e SEAFILE_ADMIN_PASSWORD=a_very_secret_password \
  -v /opt/seafile-data:/shared \
  -p 80:80 \
  -p 443:443 \
  seafileltd/seafile:latest
```

如果您是使用`launcher`脚本的最先一批用户，您应该参考[从旧的结构升级](https://github.com/haiwen/seafile-docker/blob/master/upgrade_from_old_format.md).

### 问题排查方法

如果你运行的过程中碰到问题，可以运行"docker logs"、"docker exec"等docker命令来寻找更多的错误信息.

```sh
docker logs -f seafile
# or
docker exec -it seafile bash
```