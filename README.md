## Node App 部署

环境 (System)：
---
CentOS 7

服务器可视化管理工具（GUI）
---
MacOS系统推荐使用Cyberduck（小黄鸭）

![Cyberduck](https://cdn.cyberduck.io/img/cyberduck-icon-128.png)

Windows系统推荐使用 FileZilla

![FileZilla](https://ss2.baidu.com/6ONYsjip0QIZ8tyhnq/it/u=1002748247,1106168081&fm=58)

准备 (Requirement)
---

+ **git**: 分布式代码版本管理工具
+ **node**: 基于 Chrome V8 引擎的 JavaScript 服务端运行环境
+ **npm**: node 的倚赖包管理器
+ **pm2**: node 应用进程管理器

安装 (Install)
---

一般在 Linux 系统中安装程序有三种方式：

1. 下载源码，手动编译
2. 直接下载二进制文件
3. 用 yum 安装

这里推荐用第二种。

### Install git

> 如果系统有自带的 git，可执行 `yum remove -y git` 删除

1. 安装倚赖：

    ```
    yum install -y curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
    ```
  
1. 前往 [github](https://github.com/git/git/releases) 下载安装包，拷贝至 `/usr/src`(拷贝可通过可视化工具直接将相应文件拖入目标文件夹)，或者直接用 wget：

    ```
    cd /usr/src
    yum install -y wget
    wget https://www.kernel.org/pub/software/scm/git/git-2.10.2.tar.gz
    tar xvzf git-2.10.2.tar.gz
    ```
  
1. make 安装：

    ```
    make prefix=/usr/local all
    make prefix=/usr/local install
    ```

1. 检查 git 是否安装成功：

    ```
    git --version
    ```

### 安装 node 和 npm：

1. 前往 [node official site](https://nodejs.org/en/download/) 下载对应的编译好的 Linux 源码，或者直接用 wget 下载：

    ```
    cd /usr/src
    wget https://nodejs.org/dist/v6.10.3/node-v6.10.3-linux-x64.tar.gz
    tar xvf node-v6.10.3-linux-x64.tar.gz
    ```
  
1. 创建软链接至 `/usr/local/bin`

    ```
    ln -s /usr/src/node-v6.10.3-linux-x64/bin/node /usr/local/bin/node
    ln -s /usr/src/node-v6.10.3-linux-x64/bin/npm /usr/local/bin/npm
    ```
  
1. 检查 node 与 npm 是否生效

    ```
    node -v
    npm -v
    ``` 
  
1. 接下来要配置一下 npm，将 npm 的默认全局安装路径自定义到 `/$HOME/.node`:

    ```
    npm config set prefix $HOME/.node
    ```

1. 添加国内npm淘宝镜像:

    ```
    npm config set registry https://registry.npm.taobao.org/
    ```
  
1. 配置系统路径，使 npm 全局安装的模块命令行 bin 生效：

    ```
    echo "export PATH=$PATH:$HOME/.node/bin" >> ~/.bashrc
    source ~/.bashrc
    ``` 

### 安装 pm2：

```
npm install -g pm2
```
  
布署 (Deploy)
---

布署分为不用 git 和基于 git 两种方式。前者主要是用于 git 代码网络属于内网的环境下。

不用 git 的话，下载 git 项目的 tag 包或者将项目文件直接拷贝至目标主机。

**不基于 git 部署**

1. 安装依赖

    ``` 
    // 安装 node 倚赖包
    npm install --production
    ```
1. 运行项目

    ```
    // 如果不习惯用pm2来管理可以直接通过一些命令，pm2下面有详解
    node app.js
    ```

**推荐使用 git**

为了更好的版本管理与更轻松的解决版本倚赖，推荐使用 git。

git 的传输协议有 https 和 ssh 两种，我们采用更加安全快速的后者。服务器只需 git 的只读权限，因此不必配置 git user，只需生成 ssh key 做为项目的 deploy key（gitlab 项目右上角的菜单中选择 "Deploy Keys"，在此添加服务器的公钥 "id_rsa.pub" 中的内容）。

(ps. 关于如何生成 ssh key，[可参见此](http://git.cairenhui.com/gitlab/how-to-use/wikis/Generating-SSH-keys)，这里要输入的邮箱改为服务器名即可)。
  
1. 接下来 clone 项目代码至服务器：

    ```
    git clone git@git.cairenhui.com:<GROUP>/<PROJECT>.git <NODE_APP_DIR>
    ```
*注：尖括号内的为变量*
  
1. 检出版本 

    ```
    // 查看当前所有版本
    git tag
    
    // 切换到指定版本，这里的 <TAG> 是演示，比如可以是 1.0.0
    git checkout <TAG>
    ```
  
1. 安装倚赖：

    ``` 
    // 如果没用到 bower，跳过此步骤
    bower install --allow-root

    // 安装 node 倚赖包
    npm install --production
    ```
*注：不采用 git 布署的话，只需执行最后一步*
  
至此，无论是否使用 git，布署都已完成。

运行 (Run)
---

在项目根目录下，运行 `pm2 start pm2.json` 即可。

`pm2.json` 中的 `name` 字段为 App name，pm2 可以全局地操作它：`pm2 stop <APP_NAME>`

另外有几个比较有用的命令：

+ 查看所有的 node 应用进程：`pm2 list`
+ 查看某个应用的具体信息：`pm2 show <APP_NAME>`
+ 监控 CPU / Memory: `pm2 monit`
+ 查看应用消息日志：`pm2 logs <APP_NAME>`
+ 重启应用程序：`pm2 restart <APP_NAME>`

更多的命令用法请查看：https://github.com/Unitech/pm2#main-features

更新或回滚 (Update/Rollback)
---

1. 不使用 git：之前已提到过，直接在 gitlab 上下载 tag 包，解压缩覆盖至服务器
1. 使用 git：先通过 git 获取更新

    ```
    git fetch origin --tags
    ```
  
    再切换：
  
    ```
    git checkout <NEW_TAG>
    ```
  
    回滚也非常简单：
  
    ```
    git checkout <PREV_TAG>
    ```
