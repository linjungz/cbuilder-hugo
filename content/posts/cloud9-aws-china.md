---
title: "在 AWS 国内区域使用 Cloud9 Web IDE"
date: 2020-08-04T19:17:48+08:00
draft: false
---

Cloud9 是一个基于云的集成开发环境，被 AWS 收购后做为一个服务对外发布，即 [AWS Cloud9](https://aws.amazon.com/cn/cloud9/ "AWS Cloud9") 。使用Web IDE来做开发在某些场合下效率还是挺高的. AWS 很多面向客户的Workshop都基本上会使用到 Cloud9 来做为终端。对于不熟悉 Linux 的用户来说，在 Windows 下面连接 EC2 Linux 虚拟机还是挺费劲的。

记得在 2019 年北京 Summit 一个 Workshop 现场看到有很多用户就卡在连虚拟机上面，花了很长时间。可惜 Cloud9 服务暂时还没落地到国内区域，因此萌生了一个想法，希望能够把 Cloud9 Web IDE 部署在国内区域，以便自己组织市场活动时，新手用户可以很方便的使用 Cloud9 来在国内区域做实验。

Google 了一下发现 Cloud9 虽然被收购了，但是相关的软件还是继续开源的。 [Cloud9 3.0 SDK for Plugin Development](https://github.com/c9/core "Cloud9 3.0 SDK for Plugin Development") 可以被用来开发 Cloud9 相关的插件，也可以基于此构建一个 Web IDE。因此我把这个软件部署在一台 EC2 Linux 虚拟机上，最开始通过一个 CloudFormation 模板把虚拟机部署、安全组配置和初始密码设置等都进行自动化，通过模板实现了一键部署。去年 AWS 国内区域上线了[Marketplace](https://awsmarketplace.amazonaws.cn/?locale=zh "Marketplace") ，我尝试着联系了负责 Marketplace 的同事，经过一系列的流程终于把个 AMI 上线到了中国区的 Marketplace。Marketplace 上线前会对 AMI 进行安全扫描，这样也让用户可以更加放心的使用这个AMI。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghf0oyy6gbj31ji0u0qls.jpg)

通过这个 AMI，用户可以快速部署出基于 Cloud9 的 Web IDE

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghf0qlzhbyj31od0u0aj8.jpg)

在去年 11 月深圳一个 IoT 的市场活动上，我们就利用这个 AMI 来帮助客户在现场快速部署一个 Web IDE，方便他们连接到国内区域的 EC2 Linux 虚拟机进行动手实验

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghf0yq1uz1j313y0u0b2c.jpg)

Cloud9 确实是一个非常好用的 Web IDE，但目前这个 AMI 还欠缺对 Lambda 开发的一些支持，这个在 AWS Cloud9 里是支持。研究了一下对 Lambda 的支持应该是 AWS 收购了 Cloud9 后增加的功能，这部分代码并没有开源，因此看起来也没法加到之前的这个 AMI 里面。还是期待 AWS Cloud9 能够在国内区域尽快上线，毕竟托管服务还是会比这个 AMI 要强得多。

感兴趣的朋友可以在EC2启动向导上搜索Marketplace的AMI (Cloud9 Web IDE)并测试一下。

对了，这个AMI是免费的。:)