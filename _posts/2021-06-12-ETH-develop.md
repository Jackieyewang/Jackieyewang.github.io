---
layout: post
title: "在本地搭建一个ethereum的私有节点(1)"
date: 2021-06-12
description: "在本地搭建一个ethereum的私有节点(1)"
tag: none
---   

在本地搭建一个ethereum的私有节点(1)


以太坊可以开发的环境可以是linux、windows、macOS，本系列文章只在linux系统(ubuntu)下开发，分为四篇。在进行以太坊开发之前需要搭建以太坊开发环境。

       以太坊环境搭建



go语言环境搭建

首先从https://golang.org/dl/下载一个go的压缩包，必须下载linux系统的压缩包，可以下载最新的压缩包，一般都能兼容目前所有需要的开发环境,本系列教程使用的go语言的版本go1.10.3 linux/amd64，下载开发包需要注意是否符合你的电脑，是32位还是64位的，现在的电脑一般都是64位，位数在安装ubuntu系统之前就应该在windows下看好，右键我的电脑，点击属性，在基本信息中的系统类型中可以查看到系统是x86还是x64。

压缩包可以转存到home文件夹下 tar -xvzf  文件名，命令解压到local下，这样会把go文件夹放到/ 主目录下，会更规范适合配置系统环境

tar -C /usr/local -xzf go1.10.3.linux-amd64.tar.gz

后面的文件名需要改为对应下载的文件名。

export PATH=$PATH:/usr/local/go/bin
运行后就配置好了go的环境，使用

go version
测试是否完成配置，终端会输出

go version go1.10.3 linux/amd64

--go1.10.3 linux/amd64为版本号

作者还将文件解压到放在home目录下操作，这样会比较方便操作。解压到home文件夹下后需要运行

export GOPATH=/home/wyw/home/go

export GOROOT=/usr/local/go

export PATH=$GOROOT/bin:$PATH

这样设置好了GOPATH和GOROOT会方便之后的操作

     建议最好将上述命令写入/etc/profile 文件，这样开机就会自动设置好go的环境，

sudo vim /etc/proile
没有vim 请sudo apt-get install 自行下载

在修改profile前，建议首先备份profile

sudo cp /etc/proile profile_store
       在vim界面下，先输入i表示开始input输入，在profile文件的最后一行，粘贴上述三条命令，粘贴快捷键是shift+ctrl+v，输入完成后按esc退出输入模式，开始输入编辑器命令:wq!  保存并退出vim

source /etc/profile
     完成系统go环境的配置。





geth客户端安装



     Geth是Go Ethereum开源项目的简称，它是使用Go语言编写且实现了Ethereum协议的客户端软件，也是目前用户最多，使用最广泛的客户端。通过Geth客户端与以太坊网络进行连接和交互可以实现账户管理、合约部署、挖矿等众多实用的功能。ganache也是客户端，两个都可以用来协助开发DAPP，本教程使用geth。Geth 支持Linux，Mac Os和Windows，支持两种类型的安装：用户的二进制可执行文件安装或脚本安装。

      二进制可执行文件在https://geth.ethereum.org/downloads/ 

或者 https://ethfans.org/wikis/Ethereum-Geth-Mirror下载，压缩包解压缩后直接可以运行。另一种方式直接安装可以输入命令

sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
更加方便的安装已经编译完成的geth客户端，这样安装的go-ethereum应该在/usr/local下，客官可以自己去文件管理器根据文件名搜索找到文件夹的位置。



       我更喜欢的安装方式是脚本安装即从源头安装，能够直观的看到geth客户端的编译过程，首先从github上下载源码

git clone https://github.com/ethereum/go-ethereum
        没有git请自行下载git。



构建geth需要安装Go和C编译器：

sudo apt-get install -y build-essential

最后，geth使用以下命令构建客户端。

cd go-ethereum
make geth




     最终 geth文件内容如下

图片

      完成geth客户端安装后，仍然需要配置环境变量，否则你需要在下载geth的go-ethereum/build/bin文件夹内才能执行二进制文件运行geth。

命令为

export PATH=$HOME/blockchain/eth/go-ethereum/build/bin
PATH=后面是你下载go-ethereum的目录，意思是指向你的geth二进制可执行文件。使用第二种直接命令安装可能不需要注意环境问题，但是会不方便之后的操作。这里不要写入到/etc/profile中，会影响终端命令的使用。



    开启geth客户端，首先需要进入到管理员界面

su root  

export PATH=$HOME/blockchain/eth/go-ethereum/build/bin

这样就进入到了geth开发环境

图片



       使用geth创建新账户

geth account new

需要输入password 密码，这个是账户的私钥，输出如下

图片



