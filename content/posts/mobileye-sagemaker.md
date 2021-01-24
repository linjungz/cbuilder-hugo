---
title: "Mobileye如何在云上进行深度学习模型训练"
date: 2020-05-19T15:48:29+08:00
draft: False    
---

最近开始关注ADAS这个行业，静下心来找各种资料学习，自然而然从行业的领头羊Mobileye入手。

Mobileye是家以色列公司，1999年由色列希伯来大学的Amnon Shashua教授和Ziv Aviram创立，以低成本摄像头为基础，通过计算机视觉技术研发高级驾驶辅助系统（ADAS)。产品从研发到正式商用花了8年时间，而在2017年被Intel以153亿美元收购。

在油管上搜到了Mobileye联合创始人Amnon Shashua在今年CES2020上的一个主题演讲视频：
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgb04n5zj31d40ognpd.jpg)

在这一小时里面Shashua教授讲解了Mobileye基于纯摄像头技术来做感知：
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgb3noisj311u0la1kx.jpg)

通过多套独立算法来做系统内部冗余：
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgb55ionj311m0kotku.jpg)

基于2D传感器帮助系统获得3D的理解力：
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgb7a1h9j311k0ku4g2.jpg)

实现像素级场景分割：
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgb9jc0wj31140l4qke.jpg)

道路拥塞，行人横穿马路等复杂的路况下，很好的处理了的无保护左转场景: 
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgbfnz8nj312g0m6b29.jpg)

而这一切只在一块小小的芯片上完成计算：
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgbn7rkhj311a0l24cl.jpg)

不由得感叹犹太人真聪明，英特尔买这家公司可太有眼光了...

另外也看到Mobileye不仅仅局限在ADAS，从L2+到L4，布局出行服务RobotTaxi。同时做高精度地图(REM)，基于此还延伸至智慧城市：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgbsd3ibj31260kqk95.jpg)

这一小时的演讲信息量挺大的，需要一点点慢慢消化。就演讲前半部分讲到的感知，是基于深度学习来做的。Mobileye具体是如何来进行深度学习的模型训练的，这点挺让人好奇的。刚好在油管还有另一个视频，是Mobileye的算法工程师小哥 Chaim Rand 在去年AWS re:Invent上做的一个公开分享：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgajiqcej30uk0h4h05.jpg)

re:Invent是AWS每年一次技术大会，上面有很多AWS的用户会讲述他们使用AWS的经验（也包括踩过的坑）。比如在这个分享里面小哥介绍了他们如何利用SageMaker来帮助他们加速算法开发和这个过程中如何填坑。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexg9uza1rj30ue0h6n40.jpg)

小哥前面循例先简要介绍了Mobileye是做什么的，然后开始讲到他们做深度学习模型训练时遇到的挑战：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgbzri9zj30kx0bimzw.jpg)

可以看到Mobileye之前在自己的数据中心做模型训练，整个开发过程是从数据标注开始，然后用GPU集群进行模型训练，最后再把模型部署到他们自己专有的芯片EyeQ上。注意到比较大的挑战是：一个典型的模型会需要多达200TB的数据来进行训练。

接着讲到在数据中心做模型训练存在的问题：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexk0nxjl3j30kr0akjtz.jpg)

可以看到在本地数据中心进行训练有很多明显的劣势：包括GPU资源受到限制（嗯，算法工程师肯定是越多越好啦） ，另外存储资源也会有限制（从前面提到的训练数据量来看，可以想象Mobileye的数据存储会有多大），同时基础架构软硬件的升级也非常挑战（原话是说让他们直掉头发，确实从视频看小哥也头秃了。。。）

在这个背景下，他们开始尝试在AWS云上使用SageMaker来做训练。这里他总结了几个他喜欢SageMaker的原因：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc29v25j30jm0bddj7.jpg)


1.  无限制的GPU资源，按小哥的原话说，对算法工程师这简直就像是小孩到了糖果屋... 他们可以同时跑更多的训练任务而不用排队，这也是Mobileye能够在AWS云上把开发速度提高10倍的主要原因。 
2. SageMaker也可以使用支持不同实例类型，单GPU实例，或是多GPU实例，不同的Tensorflow版本等，满足他们不同模型训练任务的需求。
3. 向云上迁移的成本也比较低，SageMaker有比较友好的API，文档，示例等，
4. 安全是极其关键的一点，Mobileye的数据安全团队也评估过在云上进行训练是满足他们的安全要求，毕竟IP是Mobileye关键的资产。
5. 最后还特别提到了SageMaker的Pipe Mode

留意到前面讲到Mobileye的典型模型训练需要多达200TB的数据，接下来他还展开详细讲解了Pipe Mode如何帮助来解决大数据集进行训练的问题

首先是简单介绍了SageMaker Pipe Mode. 这个功能可以把训练所需要的数据从S3直接流式传输到训练实例，而不需要把数据集完整地从S3全量复制到每一个训练实例的本地存储（EBS）上：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc2uc1hj30kq0byn0a.jpg)

通过Pipe Mode，训练的数据集大小就没有限制了，训练实例本地也不需要存储数据集，成本可以大大降低。同时训练任务可以立即启动，不需要等待数据从S3复制到训练实例本地存储：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc3hnngj30kp0am420.jpg)

而且很重要一点是，多个训练实例可以同时从S3同一份训练数据集去拉取数据。看到这里我想起在一些模型训练平台上，经常会看到在S3前面部署一套高性能的分布式存储（如AWS自己的FSx for Lustre, 或是开源的BeeGFS等），供训练实例并发去高速读取训练数据。使用SageMaker Pipe Mode应该就不需要部署这套高性能的分布式存储。

SageMaker将Pipe的实现封装到了一个叫 tf.data.Datasets API里，因此可以比较方便的进行调用。小哥接下来还演示了如何设置 Pipe Mode的代码（毕竟400系列的session没有代码说不过去...)

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc4qxo4j30kq0a4q5c.jpg)

但没有一种技术是万能的，Pipe Mode也是一样。 接下来这部分我觉得特别有价值，就是介绍Mobileye在使用Pipe Mode时遇到的挑战以及他们是如何克服的，这些挑战包括如何将数据转换为支持的格式，Pipe Mode顺序读取的特性以及 Pipe 数量的限制（目前限制是20个）。

首先是数据格式的挑战：Pipe Mode支持 CSV, TFRecord 和 Protobuf recordIO . Mobileye选择了TFRecord(主要是相关的示例代码较多），因此他们需要将已有的海量的数据转换成TFRecord格式。这个转换过程他们使用了AWS Batch服务来进行任务调度，并使用了数十万个VCPU来并行处理。 在转换过程中，每个TFRecord文件为100MB，这个也是S3读写带宽比较优化的一个对象大小。由于在云上来进行并行的数据转换任务，整个数据准备时间得到极大的提升，从以天计到以小时来计。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc5k87xj30kw09a76h.jpg)

在讲到数据格式转换时，小哥也顺带介绍了Mobileye整个在云上的端到端的开发流程：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc62tjdj30kv0ahq64.jpg)

可以看到原始数据和标注文件存放在S3上面，并通过DynamoDB来进行元数据的管理，接着通过AWS Batch来进行并发的数据格式转换，生成TFRecord文件并写回S3。然后在SageMaker上进行模型的训练和评估，生成的模型会进行剪裁并运行部署脚本，以便运行在Mobileye自已的芯片EyeQ上面，这个部署脚本也是在SageMaker上运行的。

回到Pipe Mode的挑战上，除了数据格式，第二个问题是顺序读的特性。使用Pipe Mode的话，只能读到Pipe里的下一个数据，而无法对整个数据集任意进行访问（随机访问）：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc6y7k9j30kn06vq4c.jpg)

这个顺序读的特性带来了两个挑战。第一个挑战是Shuffling，这有点像打牌的时候进行洗牌一样。传统方式下对数据集可以进行随机访问，因此可以很方便进行shuffling。使用Pipe Mode时，可以利用ShuffleConfig Class来对pipe进行shuffle，这样确保在每个epoch，训练数据的顺序都是不一样的，同时他们还利用了TensorFlow Dataset的shuffle功能，在training batch level也进行shuffle。尽管这个方案并没有达到原来随机访问时的shuffle程度，但也已经足够使用了。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc7hmy9j30kq0adadk.jpg)

第二个挑战是boosting. 这里举了个例子，比如说训练一个进行车辆识别的模型，其他颜色的车识别都很成功，但针对粉色的车就一直无法识别。分析原因发现是数据集中粉色车的数据非常少（可能百万张照片中只有5张）。其中一种解决方案是在每个epoch里增加粉色车的数据量(boosting)。如果可以随机访问数据集的话，这个问题很好解决。但在Pipe Mode里面要如何来做呢？这里提到解决方案是创建两个Pipe, 一个Pipe只有粉色车，另一个Pipe是其他颜色的车，然后使用TF Dataset Interleaving API，从两个Pipe里取数据。 这两个Pipe可以设置不同的权重(Weight)，这样便可以方便地增加粉色车的数据量了。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc81uavj30k80bi42g.jpg)

这种方法可以有效解决一种数据类型数据量不够的问题，但如果有多种数据类型呢？这里引出了Mobileye遇到的第三个挑战，就是每个Training Session里Pipe的数量限制。为什么他们会觉得20个Pipe还不够用？其中一个原因还是boosting. 这里还是接着前面的例子识别粉色车，进一步展开，比如说可以识别粉色的小车，但无法识别粉色的卡车，无法识别雨中粉色的卡车等等，随着数据类型的增加，很快就到了Pipe的数量限制。 第二个原因是Horovod做分布式训练时，每个GPU要求独占Pipe。比如说一个GPU使用3个Pipe，那在一个8GPU的实例上，就需要配置8*3=24个Pipe，这就超过了Pipe的数量限制了。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc8xbgij30k406h0ua.jpg)

关于boosting，他们使用了SageMaker Manifest文件。在Manifest文件里面定义数据集的分组。如果需要加一倍粉色车的数据量，则在manifest文件里多写一行就可以，这样通过Manifest文件就可以更精细化的控制训练数据了。对于分布式训练遇到Pipe数量限制的问题，他们对比了TensorFlow内建的多GPU支持和Horovod分布式训练框架，看到性能是基本一样的，因此没有选用Horovod，而是使用TensforFlow内建的方案Estimator，从则避开这个Pipe数量限制。

除了上面详细分析的Pipe Mode，Mobileye在从数据中心转到SageMaker时，还进行了其他的考虑，包括分布式训练、Spot实例的应用和远程Debugging等等 ，这些细节在另外一个Blog里面有详细的阐述（可参见附录）

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgc9qh38j30kr0b0td3.jpg)

最后小结了一下Mobileye在使用SageMaker过程中的一些经验：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gexgcat5hlj30ky0abwid.jpg)

可以看到SageMaker对Mobileye DNN的开发起到了加速作用，其中的Pipe Mode可以比较好的解决模型训练时使用到大数据集的问题。由于利用到云极高的扩展性和其他AWS托管服务的集成，Mobileye整个开发时间缩短了10倍。虽然在使用SageMaker的过程中也遇到了一些问题，但都能通过一些方案比较好的解决掉。

另外在油管上还看到了Mobileye在AWS上进行大规模仿真和高精度地图生产的分享，后续值得好好研究一下。

注：本文材料来自于公开渠道，并基于个人经验与知识进行分析，仅代表个人观点

参考资料：

- Mobileye在CES2020的演讲视频：
https://www.youtube.com/watch?v=HPWGFzqd7pI
- Mobileye在AWS re:Invent的分享视频：
https://www.youtube.com/watch?v=iW0RASdjnOk
- Chaim Rand 关于 Mobileye使用SageMaker的博客： 
https://medium.com/@julsimon/making-amazon-sagemaker-and-tensorflow-work-for-you-893365184233