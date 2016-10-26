---
layout: post
title:  Windows下通过Bosh-lite安装单机版CloudFoundry
description: 
keywords: CloudFoundry,PaaS
category: CloudFoundry
tags: [CloudFoundry,PaaS,Architecture]
---

CloudFoundry是目前最为流行的PaaS平台之一。为了方便开发测试，我们需要在本地搭建一个迷你型的CloudFoundry环境。CloudFoundry官方提供了Bosh-lite工具用于支持。绝大部分情况下，我们会在Linux系统下安装，今天针对Windows系统，详细阐述一下安装步骤。

需要注意的是，由于CloudFoundry所有组件都部署在单台VM上，所以 __内存>8G，同时要连接上互联网。__

### 准备工作

下载并安装以下软件：

* Vagrant：用于创建和部署虚拟化环境，它使用VirtualBox虚拟化系统
* VirtualBox：Oracle的开源虚拟化系统，支持Windows、Linux、MacOS
* Git：版本管理工具
* Putty：小巧免费的SSH、Telnet工具

<!-- more -->

### 基本原理

* 通过Vagrant创建一个VirtualBox虚拟机，该虚拟机包含了Bosh Server
* 通过Bosh Client，连接Bosh Server，将CloudFoundry代码上传至Bosh Server虚拟机
* 通过一系列配置，在该虚拟机中部署用warden隔离的CloudFoundry各个组件

### 安装步骤

#### 1.获取bosh-lite

* 创建一个目录，例如：`D:\github`
* 打开命令行，进入到该目录
* 输入`git clone https://github.com/cloudfoundry/bosh-lite`下载，该目录将会包含一个`VagrantFile`的文件，该文件将会被vagrant用于创建虚拟机

#### 2.创建虚拟机

* 命令行进入到`D:\github\bosh-lite`
* 输入`vagrant up --provider=virtualbox`用于创建并启动虚拟机，该过程较为耗时，务必保证网络畅通。如果报`VT-x is disabled in the BIOS.(VERR_VMX_MSR_VMXON_DISABLED)`这样的错误，请重启电脑进入BIOS，将Virtualization的值改为Enable即可
* 完成之后，可以在VirtualBox中看到一个名字包含`bosh-lite_default`这样的虚拟机

#### 3.添加路由

因为CF组件是部署在这个bosh server中，我们需要从windows中访问CF组件，需通过这台bosh server作用路由。windows里执行：
`route add 10.244.0.0/19 192.168.50.4`

__192.168.50.4为bosh server虚拟机的地址，10.244.0.0/19为CF各组件所在的IP段__

#### 4.连接虚拟机

* 通过putty，ssh连接127.0.0.1:2222，用户名/密码：vagrant/vagrant
* 更新源：`sudo apt-get update`
__建议修改source.list文件，改为国内源，速度会快很多__
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vi /etc/apt/sources.list
:%s/us.arch/cn.arch/g
:wq
```

* 安装bundler：`sudo env PATH=$PATH gem install bundler`
* 下载bosh-lite：`git clone https://github.com/cloudfoundry/bosh-lite`
* bosh命令连接到Bosh Director：`bosh target 192.168.50.4`，用户名/密码：admin/admin
* 安装go，`sudo apt-get install golang`
* 配置环境变量
```
export GOPATH=~/go
export PATH=$PATH:$GOPATH/bin
```
* 安装spiff：`go get github.com/vito/spiff`。如果速度很慢，可[点击下载](http://pan.baidu.com/s/1GlEgQ)，解压后GOPATH/src下应该有`github.com/vito/spiff`这个文件夹。
* 下载cf-release代码：`git clone https://github.com/cloudfoundry/cf-release`
* 更新：`./scripts/update`，该过程会比较耗时
* 下载stemcell：`wget https://s3.amazonaws.com/bosh-warden-stemcells/bosh-stemcell-3262.2-warden-boshlite-ubuntu-trusty-go_agent.tgz`
* 上传stemcell：`bosh upload stemcell bosh-stemcell-3262.2-warden-boshlite-ubuntu-trusty-go_agent.tgz`
* 切换cf-release分支，本文使用的是v240版本：`git checkout tags/v240`
* 创建cf release包：`bosh create release releases/cf-240.yml`。这一步将会创建cf-240.tgz，大约3个G，将会非常漫长。建议提前将cf-blobstore文件下载下来。然后修改`config/final.yml`文件，blobstore节点修改一下，指向本地目录：
```
blobstore:
	provider: local
	options:
		blobstore_path: /home/vagrant/cf-release-blobs
```
* 上传release包：`bosh upload release cf-240.tgz`
* 生成部署文件cf.yml：`./scripts/generate-bosh-lite-dev-manifest`
* 开始部署：`bosh deploy`，耐心等待即可。
* 验证是否成功：`bosh vms`将会显示所有组件的列表，表明成功。或者`cf login -a api.bosh-lite.com --skip-ssl-validation -u admin -p admin`看是否可以登录。





