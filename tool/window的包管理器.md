<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Window的包管理器](#window%E7%9A%84%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8)
    - [安装scoop](#%E5%AE%89%E8%A3%85scoop)
    - [使用ari2多线程下载](#%E4%BD%BF%E7%94%A8ari2%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B8%8B%E8%BD%BD)
    - [添加储存库](#%E6%B7%BB%E5%8A%A0%E5%82%A8%E5%AD%98%E5%BA%93)
    - [安装多jdk](#%E5%AE%89%E8%A3%85%E5%A4%9Ajdk)
    - [其他命令](#%E5%85%B6%E4%BB%96%E5%91%BD%E4%BB%A4)
    - [按章MySQL](#%E6%8C%89%E7%AB%A0mysql)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Window的包管理器

> Scoop是Windows的命令行安装程序。 都知道window安装非常麻烦，每次安装软件要从官网下载可执行程序包，手动安装，安装开发环境更是麻烦的要死，像jdk、mysql、redis等等

而有了**scoop**包管理器之后，就只需一条命令即可安装完成，自动配置环境变量，非常的方便，话不多说，我们开始吧

**环境要求**
Windows 7 SP1 + / Windows Server 2008+ PowerShell 5（或更高版本，包括PowerShell Core）和.NET Framework
4.5（或更高版本） 必须为您的用户帐户启用PowerShell

## 安装scoop

先确保当前用户可以执行powershell脚本，执行以下命令

~~~powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
~~~

运行以下脚本安装scoop

~~~powershell
Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')

# or shorter
iwr -useb get.scoop.sh | iex
~~~

默认安装位置在C:\Users\<user>\scoop,全局的安装位置在C:\ProgramData\scoop

将scoop安装到自定义目录

~~~powershell
$env:SCOOP='D:\Applications\Scoop'
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
~~~

配置scoop全局安装的自定义目录

~~~powershell
$env:SCOOP_GLOBAL='F:\GlobalScoopApps'
[Environment]::SetEnvironmentVariable('SCOOP_GLOBAL', $env:SCOOP_GLOBAL, 'Machine')
~~~

## 使用ari2多线程下载

~~~powershell
scoop install ari2
~~~

## 添加储存库

- main 主库
- extras 扩展库
- games 游戏库
- nerd-fonts 字体库
- nirsoft Nirsoft程序库
- java java库
- jetbrains 安装jetbrains的IDE库
- nonportable 不可移植程序库
- php
- versions 替代版本库

使用以下命令添加储存库

~~~powershell
scoop bucket add bucketname
~~~

## 安装多jdk

使用scoop可以安装多个jdk，比如java8、java11、java15

使用命令切换不同的jdk

~~~powershell
scoop reset 对应的jdk版本
~~~

demo演示
![demo](../images/scoop.gif)

## 其他命令

-查看 scoop list

-搜索 scoop search

-安装 scoop install

-卸载 scoop uninstall

-查看缓存 scoop cache show

-更新 scoop update

-清除老版本 scoop cleanup

## 按章MySQL

```shell
scoop install mysql
```

默认安装最新的MySQL版本，我这里的是8.0.29

安装成功后，使用管理员身份打开cmd

执行mysql初始化命令

```shell
mysqld --initialize-insecure
```

会发现程序在mysql的根目录下自动创建了data文件夹以及相关的文件

注册mysql服务,路径改成自己的配置文件路径

```shell
mysqld --install MySQL --defaults-file="C:\Users\admin\scoop\apps\mysql\current\my.ini"
```

启动mysql服务

```shell
net start mysql
```

修改mysql密码，进行mysql控制台,第一次不需要输入直接回车进入

```shell
mysql -u root -p
```

mysql8.0以上使用以下修改密码，先执行`use mysql;`再执行

```shell
alter user 'root'@'localhost' identified by '设置的新密码';
```

执行成功，刷新权限

```shell
flush priviliges;
```

![](../images/mysql.png)

查看mysql所有命令
![](../images/mysql1.png)

## 整理

- scoop bucket常用

| Name        | Source                                        | Updated            | Manifests |
|-------------|-----------------------------------------------|--------------------|-----------|
| dev-tools   | https://github.com/anderlli0053/DEV-tools     | 2022/11/3 4:42:16  | 10694     |
| scoopMain   | https://github.com/wang-song/scoopMain        | 2022/9/28 15:23:02 | 3901      |
| lemon       | https://jihulab.com/hoilc/scoop-lemon         | 2022/11/3 5:22:03  | 636       |
| main        | https://github.com/ScoopInstaller/Main        | 2022/11/3 8:35:29  | 1108      |
| dorado      | https://github.com/h404bi/dorado              | 2022/11/3 8:29:07  | 222       |
| extras      | https://github.com/ScoopInstaller/Extras      | 2022/11/3 8:35:33  | 1722      |
| games       | https://github.com/Calinou/scoop-games        | 2022/11/3 8:35:10  | 244       |
| java        | https://github.com/ScoopInstaller/Java        | 2022/11/2 20:42:49 | 245       |
| JetBrains   | https://github.com/Ash258/Scoop-JetBrains     | 2022/11/3 3:07:38  | 100       |
| nirsoft     | https://github.com/kodybrown/scoop-nirsoft    | 2022/6/22 0:00:55  | 271       |
| nonportable | https://github.com/ScoopInstaller/Nonportable | 2022/11/3 1:02:48  | 104       |
| raresoft    | https://github.com/L-Trump/scoop-raresoft     | 2020/9/1 17:57:23  | 87        |
| scoop       | https://github.com/dodorz/scoop               | 2022/11/3 3:42:27  | 0         |
| scoopet     | https://github.com/ivaquero/scoopet           | 2022/11/2 21:22:36 | 49        |
| versions    | https://github.com/ScoopInstaller/Versions    | 2022/11/3 4:36:39  | 387       |

**更新仓库查看官方**<https://rasa.github.io/scoop-directory/by-stars.html>

- scoop 常用软件整理

```shell
scoop install clash-for-windows scoop-completion 7zip git mysql redis mongodb another-redis-desktop-manager aria2 arthas fiddler 
graalvm-jdk11 graalvm20-jdk8 
gradle insomnia marktext maven mvndaemon nodejs python tabby sudo tcping telegram 
googlechrome windows-terminal utools wechat screentogif fluent-terminal-np docker lx-music 
IntelliJ-IDEA-Ultimate DataGrip vscode 
```
