---
layout: post
date: 2019-06-05 10:21
comments: true
toc: true
title: nvm安装多版本node
category: Linux & 运维
tags:
- Shell
---


## 一、卸载已安装到全局的 node/npm
如果之前是在官网下载的 node 安装包，运行后会自动安装在全局目录，其中
**node** 命令在 `/usr/local/bin/node` ，**npm** 命令在全局 node_modules 目录中，具体路径为 `/usr/local/lib/node_modules/npm`

<!-- more -->

安装 **nvm** 之后最好先删除下已安装的 node 和全局 node 模块：

1.查看已经安装在全局的模块，以便删除这些全局模块后再按照不同的 node 版本重新进行全局安装
`npm ls -g —depth=0` 

2.删除全局 node_modules 目录
`sudo rm -rf /usr/local/lib/node_modules` 

3.删除 node
`sudo rm /usr/local/bin/node` 

4.删除全局 node 模块注册的软链
`cd /usr/local/bin && ls -l | grep "../lib/node_modules/" | awk '{print $9}'| xargs rm` 

或者如下:
```shell
#apt-get 卸载
sudo apt-get remove --purge npm
sudo apt-get remove --purge nodejs
sudo apt-get remove --purge nodejs-legacy
sudo apt-get autoremove

#手动删除 npm 相关目录
rm -r /usr/local/bin/npm
rm -r /usr/local/lib/node-moudels
find / -name npm
rm -r /tmp/npm* 
```


## 二、安装 nvm
`curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash`
安装完成后请重新打开终端环境，Mac 下推荐使用 oh-my-zsh 代替默认的 bash shell。

## 三、安装切换各版本 node/npm
1.安装最新稳定版 node，现在是 5.0.0
`nvm install stable `

2.安装 4.2.2 版本
`nvm install 4.2.2 `


## 四、使用 .nvmrc 文件配置项目所使用的 node 版本
如果你的默认 node 版本（通过 nvm alias 命令设置的）与项目所需的版本不同，则可在项目根目录或其任意父级目录中创建 .nvmrc 文件，在文件中指定使用的 node 版本号，例如：
1.进入项目根目录
`cd <项目根目录> `

2.添加 .nvmrc 文件, 当前项目使用node版本4.2.2
`echo 4 > .nvmrc`

3.无需指定版本号，会自动使用 .nvmrc 文件中配置的版本
`nvm use`

4.查看 node 是否切换为对应版本 
`node -v `

## 五 解决每次npm需要sudo权限问题
为当前账户添加node_modules目录读写权限即可。

`sudo chown -R $(whoami) ~/.npm`

